from contextlib import contextmanager
import teradatasql
import traceback
from dotenv import load_dotenv
import os
from steps.utils import MainframeApiService, PDSProcessor


def download_ddl(submitter, processor):
    wildcard = '*W.REF.CONTROL.*P'
    token = submitter.login()
    # Get and filter PDS list
    pds_list_response = submitter.get_dataset_lists(wildcard, token=token)
    pds_list = pds_list_response['items']
    filtered_list = processor.filter_pds_list(pds_list)

    # Process each PDS
    for pds in filtered_list:
        pds_name = pds['name']
        members_response = submitter.get_pds_members(pds_name, token)
        members = members_response['items']
        
        for member in members:
            if member.startswith('CR'):
                dataset = f"{pds_name}({member})"
                content = submitter.get_content(dataset, token)
                processor.save_member_content(member, content)


@contextmanager
def teradata_connection(host, user, password, database=None):
    try:
        # Establish the connection
        connection = teradatasql.connect(
            host=host,
            user=user,
            password=password,
            logmech="LDAP"
        )
        cursor = connection.cursor()
        print("Database connection established.")
        if database:
            cursor.execute(f"DATABASE {database}")
        yield cursor  # Provide the cursor for executing queries
    except Exception as ex:
        print("Error during database operation:", ex)
        print(traceback.format_exc())
        raise
    finally:
        if cursor:
            cursor.close()
            print("Cursor closed.")
        if connection:
            connection.close()
            print("Connection closed.")


def before_all(context):
    load_dotenv()
    """
    Context manager for Teradata database connection and cursor.
    Automatically opens and closes the connection and cursor.
    """
    context.host = os.getenv("HOSTNAME")
    context.user = os.getenv('TERADATAUSERNAME')
    context.password = os.getenv('SECRET_PASSWORD')
    context.teradata_connection = teradata_connection
    context.api_base_url = os.getenv('API_BASE_URL')
    context.api_username = os.getenv('ZOWE_API_USERNAME')
    context.api_password = os.getenv('ZOWE_API_PASSWORD')
    context.db_name = os.getenv('DB_NAME')
    context.excel_base_path = os.getenv("BRD_BASE_PATH")
    context.pds = os.getenv("MAINFRAME_PDS")
    context.retry_interval = os.getenv("JOBS_RETRY_INTERVAL")
    context.retry   = os.getenv("JOB_RETRIES")
    context.excel_base_url = os.getenv("BRD_BASE_URL")
    if context.excel_base_url == "":
        context.excel_base_url = None
    context.excel_username = os.getenv("BRD_USERNAME")
    context.excel_password = os.getenv("BRD_PASSWORD")
    context.REFRESH_DDL = os.getenv("REFRESH_DDL")

    if context.REFRESH_DDL == 'True':
        submitter = MainframeApiService(context.api_username, context.api_password, context.api_base_url)
        pdsprocessor = PDSProcessor()
        download_ddl(submitter, pdsprocessor)


def before_scenario(context, scenario):
    if 'cleanup' in scenario.tags:
        context.skip_background = True
    else:
        context.skip_background = False
--------------------------
utils.py

import traceback
import requests
import json
import hashlib
from typing import Any, Dict, List, Optional, Tuple, Union
from openpyxl import load_workbook
import re
import os
from requests_ntlm import HttpNtlmAuth
import teradatasql
import logging
import datetime
import pandas as pd

# Configure a global logger instance for the utils module
logger = logging.getLogger(__name__)


class DatabaseOperations:
    def __init__(self, context: Any):
        self.context = context

    def execute_cursor(self, query: str, query_type: Optional[str] = None) -> Union[list, tuple]:
        result: Union[list, tuple] = []
        with self.context.teradata_connection(self.context.host, self.context.user, self.context.password,
                                              self.context.db_name) as cursor:
            try:
                cursor.execute(query)
                if query_type == "many":
                    result = cursor.fetchall()
                else:
                    result = cursor.fetchone()
                # logger.info(f"The SQL statement {query} runs successfully.")
            except teradatasql.DatabaseError as e:
                logger.error(f"Database connection error: {e} on query {query}")
                raise
            except Exception as e:
                logger.error(f"Error during sql execution: {e} on {query}")
                raise
        return result

    def build_sql_query(self, query_type: str, table_name: str, db_name: str,
                        data: Optional[List[Dict[str, Any]]] = None, schema: Optional[List[Dict[str, str]]] = None,
                        utility_db: Optional[str] = None) -> Union[str, List[str], None]:
        try:
            if query_type == "TRUNCATE":
                return f"DELETE {db_name}.{table_name} ALL"

            elif query_type == "DROP":
                return f"DROP TABLE {db_name}.{table_name};"

            elif query_type == "VIEW_DROP":
                return f"DROP VIEW {db_name}.{table_name};"

            elif query_type == "CREATE":
                if not schema:
                    raise ValueError("Schema is required for CREATE queries.")
                columns = ", ".join([f"{col['column_name']} {col['data_type']}" for col in schema])
                query = f"CREATE TABLE {db_name}.{table_name} ({columns});"
                return [query]

            elif query_type == "INSERT":
                if not data:
                    raise ValueError("Data is required for INSERT queries.")
                queries = []
                for row in data:
                    columns = ", ".join(row.keys())
                    values = ", ".join([f"NULL" if v is None else f"'{v}'" for v in row.values()])
                    query = f"INSERT INTO {db_name}.{table_name} ({columns}) VALUES ({values});"
                    queries.append(query)
                return queries

            elif query_type == "CREATE METADATA":
                return f"SELECT RequestText FROM DBC.Tables  WHERE TableName = '{table_name}' AND Tablekind ='T';"

            elif query_type == "CHECK_COUNT":
                return f"SELECT COUNT(*) AS RowCount FROM {db_name}.{table_name};"

            elif query_type == "IF_EXISTS":
                return f"SELECT TableName from DBC.TablesV where DatabaseName='{db_name}' and TableName='{table_name}' and TableKind ='T';"

            elif query_type == "IF_VIEW_EXISTS":
                return f"SELECT TableName from DBC.TablesV where DatabaseName='{db_name}' and TableName='{table_name}' and TableKind ='V';"

            elif query_type == "DB_SCHEMA":
                return f"""SELECT ColumnName, ColumnType, ColumnLength, DecimalTotalDigits, DecimalFractionalDigits FROM DBC.ColumnsV WHERE DatabaseName = '{db_name}' AND TableName = '{table_name}' ORDER BY ColumnId"""

            elif query_type == "CHECK STAT":
                return f"SELECT DatabaseName,TableName,LastCollectTimestamp FROM dbc.tablestatsv WHERE DatabaseName = '{db_name}' AND TableName = '{table_name}';"

            elif query_type == "COLLECT SUMMARY STAT":
                return f"COLLECT SUMMARY STATISTICS ON {db_name}.{table_name};"

            elif query_type == "COLLECT STAT":
                return f"COLLECT STATISTICS ON {db_name}.{table_name};"

            elif query_type == "NOT_NULL_COLUMNS":
                return f"""SELECT ColumnName, ColumnType FROM dbc.ColumnsV WHERE DatabaseName = '{db_name}' AND TableName = '{table_name}' AND Nullable = 'N'"""

            elif query_type == 'CREATE_TB':
                return f"CREATE TABLE {db_name}.{table_name} AS (SELECT * FROM  {utility_db}.{table_name}) WITH NO DATA;"

            elif query_type == 'CREATE_VIEW':
                return f"CREATE VIEW {db_name}.{table_name} AS (SELECT * FROM  {utility_db}.{table_name} WHERE 1 =0);"

        except Exception as ex:
            logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
            return None

    def generate_placeholder_value(self, column_type):
        placeholders = {
            'I': 0,  # Integer
            'F': 0.0,  # Float
            'CV': 'X',  # VarChar
            'CF': 'X',  # Char
            'DA': '1970-01-01',  # Date
            'TS': '1970-01-01 00:00:00',  # Timestamp
            'AT': '1970-01-01 00:00:00',  # Time
            'D': 0.0,  # Decimal
            'BF': b'',  # Byte
            'BV': b'',  # VarByte
            'BO': False,  # Boolean
            'T': '',  # Text
            'N': None,  # Null
            'S': '',  # String
            'B': False,  # Bit (Boolean)
            'L': 0,  # Long Integer
            'SB': 0,  # Small Integer
            'BI': 0,  # Big Integer
            'DT': '1970-01-01 00:00:00',  # Datetime
            'UUID': '00000000-0000-0000-0000-000000000000',  # UUID
            'JS': '{}',  # JSON
            'XML': '<root></root>',  # XML
            'ENUM': 'UNKNOWN',  # Enum
            'SET': set(),  # Set
            'ARRAY': [],  # Array
            'OBJ': {},  # Object (Dictionary)
        }
        return placeholders.get(column_type, None)

    def validate_source_table_data(self, db_name: str, table_name: str, expected_data: List[Dict[str, Any]]) -> None:
        try:
            # expected_data: List[Dict[str, Any]] = [dict(row.items()) for row in self.context.table]
            # Get current date in 'YYYY-MM-DD' format
            today_date = datetime.datetime.now().strftime('%Y-%m-%d')

            where_conditions = []

            for row in expected_data:
                # condition = " AND ".join([f"{col} = '{val}'" if val is not None else f"{col} IS NULL" for col, val in row.items()])
                condition = " AND ".join([
                                             f"{col} = '{today_date}'" if val == 'CURR_DTE' else f"{col} = '{val}'" if val is not None else f"{col} IS NULL"
                                             for col, val in row.items()])
                where_conditions.append(f"({condition})")

            where_clause = " OR ".join(where_conditions)

            validation_query = f"SELECT COUNT(*) FROM {db_name}.{table_name} WHERE {where_clause};"

            result = self.execute_cursor(validation_query)

            count = result[0] if result else 0

            assert count == len(expected_data), (
                f"Validation failed for table {table_name}: "
                f"Expected {len(expected_data)} rows, but found {count} matching rows."
            )

            logger.info(f"Validation passed for table {table_name}")
        except Exception as e:
            logger.error(f"Error validating data for table {table_name}: {e} {traceback.format_exc()}")
            raise


class MainframeApiService:
    """
    A class to handle API login, job submission, job status, retrieve PDS & PDS Members, and read PDS content from a mainframe system.
    """

    def __init__(self, username: str, password: str, url: str):
        """
        Initializes the class with credentials and API URL.

        Args:
            username (str): Username for authentication.
            password (str): Password for authentication.
            url (str): Base URL for the mainframe API.
        """
        self.username = username
        self.password = password
        self.base_url = url
        self.session = requests.Session()  # Create a session for persistent cookies

    def login(self) -> str:
        """
        Logs in to the mainframe API and retrieves the authentication token.

        Returns:
            str: The authentication token.
        """
        login_url = f"{self.base_url}/apicatalog/api/v1/auth/login"
        payload = json.dumps({
            "username": self.username,
            "password": self.password
        })
        headers = {'Content-Type': 'application/json'}

        response = self.session.post(login_url, headers=headers, data=payload, verify=True)
        response.raise_for_status()  # Raise an exception for bad status codes

        cookies = response.cookies
        if 'apimlAuthenticationToken' in cookies:
            return cookies['apimlAuthenticationToken']
        else:
            raise ValueError("Authentication failed. No token received.")

    def submit_job(self, job_location: str, token: str) -> dict:
        """
        Submits a job to the mainframe system.

        Args:
            job_location (str): Location of the JCL file.
            token (str): Authentication token.

        Returns:
            dict: Response from the API.
        """
        job_submission_url = f"{self.base_url}/jobs/api/v2/dataset"
        payload = json.dumps({"file": job_location})
        headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'Authorization': f'Bearer {token}'
        }

        response = self.session.post(job_submission_url, headers=headers, data=payload, verify=True)
        response.raise_for_status()

        return response.json()

    def job_status(self, jobname: str, jobid: str, token: str) -> dict:
        """
        This API returns the details of a job for a given job name and identifier.

        Args:
            jobname (str): JobName.
            jobid (str): JobId.
            token (str): Authentication token.

        Returns:
            dict: Response from the API.
        """
        job_status_url = f"{self.base_url}/jobs/api/v2/{jobname}/{jobid}"
        payload = {}
        headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'Authorization': f'Bearer {token}'
        }

        response = self.session.get(job_status_url, headers=headers, data=payload, verify=True)
        response.raise_for_status()

        return response.json()

    def get_content(self, dataset: str, token: str) -> dict:
        """
        This API returns the DDL from PDS Member.

        Args:
            dataset (str): The name of the dataset (XYZ.ZZZ.ZZZ(CRIXX)).
            token (str): Authentication token.

        Returns:
            dict: The DDL in JSON format.
        """
        ddl_url = f"{self.base_url}/datasets/api/v2/{dataset}/content"
        headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'Authorization': f'Bearer {token}'
        }

        response = self.session.get(ddl_url, headers=headers, verify=True)
        response.raise_for_status()

        return response.json()

    def get_dataset_lists(self, filter: str, token: str) -> dict:
        """
        This API returns the list of PDS datasets matching the provided filter.

        Args:
            filter (str): The wildcard filter for listing datasets.
            token (str): Authentication token.

        Returns:
            dict: The list of datasets in JSON format.
        """
        pds_list = f"{self.base_url}/datasets/api/v2/{filter}/list"
        headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'Authorization': f'Bearer {token}'
        }

        response = self.session.get(pds_list, headers=headers, verify=True)
        response.raise_for_status()

        return response.json()

    def get_pds_members(self, dataset: str, token: str) -> dict:
        """
        This API returns the list of members inside a specified PDS.

        Args:
            dataset (str): The name of the PDS dataset.
            token (str): Authentication token.

        Returns:
            dict: The list of members in JSON format.
        """
        members_list = f"{self.base_url}/datasets/api/v2/{dataset}/members"
        headers = {
            'Content-Type': 'application/json',
            'Accept': 'application/json',
            'Authorization': f'Bearer {token}'
        }

        response = self.session.get(members_list, headers=headers, verify=True)
        response.raise_for_status()

        return response.json()

    def clean_ddl(self, ddl: str) -> str:
        """
        Cleans the DDL by removing comments and mainframe syntax.

        Args:
            ddl (str): The DDL string to be cleaned.

        Returns:
            str: The cleaned DDL string.
        """
        # Remove comments from the DDL
        ddl_cleaned = re.sub(r'/\*.*?\*/', '', ddl, flags=re.DOTALL)

        # Remove mainframe syntax lines and any trailing spaces or newlines
        ddl_cleaned = re.sub(r'\.IF.*$', '', ddl_cleaned, flags=re.MULTILINE).strip()

        return ddl_cleaned


class DataProcessor:
    def __init__(self):
        # Initialize the type map as a class attribute
        self.type_map: Dict[str, str] = {
            'I': 'INTEGER',  # Integer
            'I1': 'BYTEINT',  # Byte Integer
            'I2': 'SMALLINT',  # Small Integer
            'I8': 'BIGINT',  # Big Integer
            'CF': 'CHAR',  # Fixed-length Character
            'CV': 'VARCHAR',  # Variable-length Character
            'N': 'NUMBER',  # Number
            'D': 'DECIMAL',  # Decimal
            'DA': 'DATE',  # Date
            'F': 'FLOAT',  # Floating Point Number
            'AT': 'TIME',  # Time
            'TS': 'TIMESTAMP',  # Timestamp
            'BO': 'BLOB',  # Binary Large Object
            'CO': 'CLOB',  # Character Large Object
            'BF': 'BYTE',  # Binary Byte
            'BV': 'VARBYTE',  # Variable-length Binary Byte
            'PD': 'PERIOD',  # Period
            'DY': 'INTERVAL YEAR TO MONTH',  # Interval Year to Month
            'DS': 'INTERVAL DAY TO SECOND',  # Interval Day to Second
            'HM': 'TIME WITH TIME ZONE',  # Time with Time Zone
            'SZ': 'TIMESTAMP WITH TIME ZONE',  # Timestamp with Time Zone
            'AN': 'ARRAY',  # Array
            'UD': 'UDT',  # User-Defined Type
            'XM': 'XML',  # XML
            'EV': 'EVENT',  # Event
            'RS': 'RESULT SET',  # Result Set
            'JN': 'JSON',  # JSON
            'ML': 'MLMODEL',  # Machine Learning Model
            'SD': 'SPATIAL',  # Spatial
            'A1': 'INTERVAL YEAR',  # Interval Year
            'A2': 'INTERVAL YEAR TO MONTH',  # Interval Year to Month
            'A3': 'INTERVAL MONTH',  # Interval Month
            'A4': 'INTERVAL DAY',  # Interval Day
            'A5': 'INTERVAL DAY TO HOUR',  # Interval Day to Hour
            'A6': 'INTERVAL DAY TO MINUTE',  # Interval Day to Minute
            'A7': 'INTERVAL DAY TO SECOND',  # Interval Day to Second
            'A8': 'INTERVAL HOUR',  # Interval Hour
            'A9': 'INTERVAL HOUR TO MINUTE',  # Interval Hour to Minute
            'AA': 'INTERVAL HOUR TO SECOND',  # Interval Hour to Second
            'AB': 'INTERVAL MINUTE',  # Interval Minute
            'AC': 'INTERVAL MINUTE TO SECOND',  # Interval Minute to Second
            'AD': 'INTERVAL SECOND',  # Interval Second
            'B1': 'VARGRAPHIC',  # Variable-length Graphic String
            'B2': 'LONG VARGRAPHIC',  # Long Variable-length Graphic String
            'D1': 'DECIMAL INTEGER',  # Decimal Integer
            'D2': 'DECIMAL BYTEINT',  # Decimal Byte Integer
            'D3': 'DECIMAL SMALLINT',  # Decimal Small Integer
            'D4': 'DECIMAL BIGINT',  # Decimal Big Integer
            'YM': 'INTERVAL YEAR TO MONTH',  # Interval Year to Month (Alias)
            'MS': 'INTERVAL MINUTE TO SECOND',  # Interval Minute to Second (Alias)
            'YR': 'INTERVAL YEAR',  # Interval Year
            'PM': 'PERIOD(MONTH)',  # Period Month
            'PZ': 'PERIOD(TIMESTAMP WITH TIME ZONE)',  # Period Timestamp with Time Zone
            'VA': 'VARCHAR',  # Variable-length Character (Alias)
            'PT': 'PERIOD(TIME)',  # Period Time
            'MO': 'INTERVAL MONTH',  # Interval Month
            'PS': 'PERIOD(SECOND)',  # Period Second
            'TZ': 'TIMEZONE',  # Timezone
            'HS': 'INTERVAL HOUR TO SECOND',  # Interval Hour to Second
            'DT': 'DATE',  # Date (Alias)
            'UT': 'USER DEFINED TYPE',  # User-Defined Type (Alias)
            '++': 'Unknown',  # Unknown or undefined type
        }

    def get_data_type_info(self, column_type: str, column_length: Optional[int], decimal_total_digits: Optional[int],
                           decimal_fractional_digits: Optional[int]) -> Tuple[str, Optional[Union[int, str]]]:
        """
        Retrieve the data type information based on the provided column type and attributes.

        Args:
            column_type (str): The shorthand column type.
            column_length (Optional[int]): The length of the column for CHAR or VARCHAR types.
            decimal_total_digits (Optional[int]): Total digits for DECIMAL types.
            decimal_fractional_digits (Optional[int]): Fractional digits for DECIMAL types.

        Returns:
            tuple: A tuple containing the base type and additional information (e.g., length or precision).
        """
        # Strip and get the base type from the type map
        base_type: Optional[str] = self.type_map.get(column_type.strip(), None)
        if base_type is None:
            logger.info(f"Unknown base type: {column_type}")
            return column_type, None

        # Determine additional attributes based on the base type
        if base_type in ['CHAR', 'VARCHAR']:
            return base_type, column_length
        elif base_type == 'DECIMAL':
            return base_type, f"{decimal_total_digits},{decimal_fractional_digits}"
        elif base_type in ['INTEGER', 'SMALLINT', 'DATE', 'TIME', 'TIMESTAMP', 'BIGINT']:
            return base_type, None
        else:
            return base_type, column_length

    def get_list_of_tuples(self, ddl_result: List[Tuple[str, str, Optional[int], Optional[int], Optional[int]]]) -> \
    List[Tuple[str, str, Optional[str]]]:
        """
        Process the DDL result to get a list of tuples with column information.

        Args:
            ddl_result (List[Tuple[str, str, Optional[int], Optional[int], Optional[int]]]): The result from the DDL query containing column details.

        Returns:
            List[Tuple[str, str, Optional[str]]]: A list of tuples with column name, data type, and length/precision information.
        """
        result: List[Tuple[str, str, Optional[str]]] = []
        for column in ddl_result:
            column_name, column_type, column_length, decimal_total_digits, decimal_fractional_digits = column
            data_type, length = self.get_data_type_info(column_type, column_length, decimal_total_digits,
                                                        decimal_fractional_digits)
            if length is not None:
                result.append((column_name, data_type, str(length)))
            else:
                result.append((column_name, data_type))
        return result


class SHA256Hasher:
    @staticmethod
    def normalize_data_type(data_type: str) -> str:
        """
        Normalize the data type to handle variations in formatting.

        Args:
            data_type (str): The data type string to normalize.

        Returns:
            str: The normalized data type string.
        """
        # Convert data type to uppercase for case insensitivity
        data_type = data_type.upper()

        # Normalize integer formats
        if 'INTEGER' in data_type:
            return 'INTEGER'

        # Normalize date formats
        if 'DATE' in data_type:
            return 'DATE'

        # Add more normalization rules as needed

        return data_type

    @staticmethod
    def compute_sha256_from_list(input_list: List[Tuple[str, str, Optional[str]]]) -> str:
        """
        Compute a consistent SHA-256 hash from a list of tuples with 2 or 3 elements.

        Args:
            input_list (List[Tuple[str, str, Optional[str]]]): A list of tuples where each tuple consists of:
                                                               - An index (key)
                                                               - A value
                                                               - An optional third element (e.g., extra data)

        Returns:
            str: A SHA-256 hash string.
        """
        normalized_list = [
            (key, SHA256Hasher.normalize_data_type(value), extra[0]) if len(t) == 3 else (
            key, SHA256Hasher.normalize_data_type(value))
            for t in input_list
            for key, value, *extra in [t]
        ]
        sorted_list = sorted(normalized_list, key=lambda x: x[0])
        # logger.info(f'Testing................................. {sorted_list}')
        concatenated_string = ''.join(
            f"{key}:{value}:{extra}" if len(t) == 3 else f"{key}:{value}"
            for t in sorted_list
            for key, value, *extra in [t]
        )
        # logger.info(f'Schema-{concatenated_string}')
        sha256_hash = hashlib.sha256(concatenated_string.encode('utf-8')).hexdigest()
        return sha256_hash, sorted_list


class ExcelProcessor:
    def __init__(self, file_path: str, excel_name: str, sheet_name: str, base_excel_url: Optional[str] = None,
                 username: Optional[str] = None, password: Optional[str] = None):
        self.file_path = file_path
        self.excel_name = excel_name
        self.sheet_name = sheet_name

        # Check if base_excl_url is provided
        if base_excel_url:
            # Check if credentials are provided
            if username and password:
                full_url = f"{base_excel_url}/{self.excel_name}"
                response = requests.get(full_url, auth=HttpNtlmAuth(username, password), verify=True)

            if response.status_code == 200:
                self.temp_file_path = excel_name
                with open(self.temp_file_path, 'wb') as f:
                    f.write(response.content)
                self.file_path = self.temp_file_path
            else:
                logger.error(f"Error downloading file from URL: {response.status_code}")
                raise Exception(f"Error downloading file from URL: {response.status_code}")
        else:
            self.file_path = os.path.join(self.file_path, self.excel_name)
            logger.info(f"Reading Excel {self.excel_name} from {self.file_path} ")

        try:
            self.workbook = load_workbook(self.file_path, data_only=True)
        except Exception as e:
            logger.error(f"Error loading workbook: {e}")
            raise

        try:
            self.sheet = self.workbook[sheet_name]
        except KeyError:
            logger.error(f"Sheet {sheet_name} not found in workbook.")
            raise

    def delete_temp_file(self):
        if self.temp_file_path and os.path.exists(self.temp_file_path):
            os.remove(self.temp_file_path)
            logger.info(f"Temporary file {self.temp_file_path} has been deleted.")

    def excel_schema_sha_calc(self, extracted_data: List[Dict[str, str]]) -> List[Tuple[str, str, Optional[str]]]:
        try:
            tuple_data = [(row['Target Field'].strip().upper(), row['Data Type'].strip().upper()) for row in
                          extracted_data]
            processed_data = []
            for column_name, data_type in tuple_data:
                # match = re.match(r"(\w+)\((\d+)\)", data_type)
                match = re.match(r"(\w+)\((\d+)(?:,(\d+))?\)", data_type)
                if match:
                    # base_type, size = match.groups()
                    base_type = match.group(1)
                    size = match.group(2)
                    scale = match.group(3)
                    if scale:
                        processed_data.append((column_name, base_type, f"{size},{scale}"))
                    else:
                        processed_data.append((column_name, base_type, str(size)))
                else:
                    processed_data.append((column_name, data_type))
            return processed_data
        except Exception as e:
            logger.error(f"Error processing schema data: {e}")
            return []

    def find_header_row(self, expected_columns: List[str]) -> int:
        try:
            for i, row in enumerate(self.sheet.iter_rows(values_only=True), start=1):
                if set(expected_columns).issubset(set(row)):
                    return i
            return -1
        except Exception as e:
            logger.error(f"Error finding header row: {e}")
            return -1

    def extract_columns_from_excel(self, expected_columns: List[str]) -> List[Dict[str, str]]:
        try:
            header_row_idx = self.find_header_row(expected_columns)
            if header_row_idx == -1:
                logger.error("Header row with expected columns not found.")
                return []

            header_row = list(self.sheet.iter_rows(min_row=header_row_idx, max_row=header_row_idx, values_only=True))[0]
            column_indices = {col: idx for idx, col in enumerate(header_row) if col in expected_columns}

            data = []
            for row in self.sheet.iter_rows(min_row=header_row_idx + 1, values_only=True):
                row_data = {col: row[idx] for col, idx in column_indices.items()}
                if row_data.get("Target Field") is None:
                    continue
                else:
                    data.append(row_data)

            return data
        except Exception as e:
            logger.error(f"Error extracting columns from Excel: {e}")
            return []


class PDSProcessor:
    def __init__(self, base_directory='DDL'):
        """
        Initializes the PDSProcessor with the base directory where the DDL files will be saved.

        Args:
            base_directory (str): The base directory where the DDL files will be saved.
        """
        self.base_directory = base_directory

    def filter_pds_list(self, pds_list):
        """
        Filters the PDS list to include only those names that start with 'B' or 'E'
        and do not end with 'BASELINE', 'BKP', 'OLAP', or 'OLD'.

        Args:
            pds_list (list): List of PDS dictionaries with 'name' keys.

        Returns:
            list: Filtered list of PDS dictionaries.
        """
        excluded_suffixes = ('BASELINE', 'BKP', 'OLAP', 'OLD')
        return [pds for pds in pds_list if
                (pds['name'].startswith(('B', 'E')) and not pds['name'].endswith(excluded_suffixes))]

    def save_member_content(self, member_name, content, sub_directory):
        """
        Saves the cleaned DDL content of a member to a file named after the table.

        Args:
            member_name (str): The name of the member.
            content (dict): The content dictionary containing 'records' key.
            sub_directory (str): The sub-directory based on PDS name prefix.

        Raises:
            Exception: If there is an error during the process.
        """
        try:
            ddl = content['records']

            # Remove comments from the DDL
            ddl_cleaned = re.sub(r'/\*.*?\*/', '', ddl, flags=re.DOTALL)

            ddl_cleaned = re.sub(r'\*\s.*?\*/', '', ddl_cleaned)  # Remove special comments
            # Remove mainframe syntax lines and any trailing spaces or newlines
            ddl_no_comment = re.sub(r'\.IF.*$', '', ddl_cleaned, flags=re.MULTILINE).strip()

            # Extract the table name using regex
            pattern_for_tb_name = r"CREATE\s+(?:\w+\s+)*TABLE\s+(\w+)"
            match = re.search(pattern_for_tb_name, ddl_no_comment, re.IGNORECASE)
            if match:
                table_name = match.group(1)
            else:
                table_name = member_name

            directory = os.path.join(self.base_directory, sub_directory)
            os.makedirs(directory, exist_ok=True)

            # Check if the base file exists and compare contents
            base_file_path = os.path.join(directory, f"{table_name}.sql")
            if os.path.exists(base_file_path):
                with open(base_file_path, 'r', encoding='utf-8') as base_file:
                    base_content = base_file.read()
                if base_content == ddl_no_comment:
                    # If contents are the same, skip saving
                    return

            # Ensure unique filename
            # base_file_path = os.path.join(directory, f"{table_name}.sql")
            file_path = base_file_path
            counter = 1
            while os.path.exists(file_path):
                file_path = os.path.join(directory, f"{table_name}_{'dup_'}{counter}.sql")
                counter += 1

            with open(file_path, 'w', encoding='utf-8') as file:
                file.write(ddl_no_comment)

        except Exception as ex:
            logger.error(f"An error occurred: {ex}, {traceback.format_exc} {member_name}")

    def read_ddl_file(self, table_name, sub_directory):
        """
        Reads the DDL file for the specified table.

        Args:
            table_name (str): The name of the table.
            sub_directory (str): The sub-directory based on PDS name prefix.

        Returns:
            str: The DDL statement.
        """
        file_path = os.path.join(self.base_directory, sub_directory, f"{table_name.strip()}.sql")
        if os.path.exists(file_path):
            with open(file_path, 'r', encoding='utf-8') as file:
                return file.read()
        raise FileNotFoundError(f"DDL file for table {table_name} not found in {file_path}")

    def check_for_duplicates(self):
        """
        Checks for duplicate SQL files in each subdirectory and logs warnings.
        """
        for subdir, _, files in os.walk(self.base_directory):
            base_files = {}
            for file in files:
                if file.endswith('.sql'):
                    match = re.match(r'^(.*?)(_dup_\d+)?\.sql$', file)
                    if match:
                        base_name = match.group(1)
                        if base_name in base_files:
                            logger.warning(
                                f"Multiple DDL found: {file} in {subdir}\nOriginal file: {base_files[base_name]} in {subdir}")
                            # print(f"Multiple DDL found: {file} in {subdir}\nOriginal file: {base_files[base_name]} in {subdir}")
                        else:
                            base_files[base_name] = file

    def check_for_multiple_ddl(self, table_name, sub_directory):
        """
        Checks for duplicate SQL files pattern in subdirectory and logs warnings.
        """
        duplicate_pattern = f"{table_name}_dup_1.sql"
        lookup_path = os.path.join(self.base_directory, sub_directory)
        for filename in os.listdir(lookup_path):
            if filename == duplicate_pattern:
                logger.error(
                    f"Multiple DDL {duplicate_pattern} found in {lookup_path}. Remove the duplicates while keeping original in non dup format.")
                raise FileExistsError(
                    f"Multiple DDL {duplicate_pattern} found in {lookup_path}. Remove the duplicates while keeping original in non dup format.")


class ExcelToSQLInserter:
    """
    A utility class to extract data from a specific table in an Excel sheet and prepare it for SQL insertion.
    """

    def __init__(self, excel_file: str, sheet_name: str) -> None:
        """
        Initializes the ExcelToSQLInserter object.

        Args:
            excel_file (str): Path to the Excel file.
            sheet_name (str): Name of the sheet to process.
        """
        self.excel_file = excel_file
        self.sheet_name = sheet_name
        self.data_list: List[Dict[str, Any]] = []

    @staticmethod
    def normalize_string(s: str) -> str:
        """
        Normalize a string by removing extra spaces, unwanted characters, and whitespace.
        Ensures the string is cleaned and standardized.

        Args:
            s (str): The string to normalize.

        Returns:
            str: The normalized string with unwanted characters removed.
        """
        # Remove unwanted characters like '?' or any extra special characters
        unwanted_chars_pattern = r'[^\w:#]'
        cleaned_string = re.sub(unwanted_chars_pattern, '', s)
        return re.sub(r'\s+', '', cleaned_string).strip()

    def read_table_from_excel(self, table_name: str) -> List[Dict[str, Any]]:
        """
        Reads a specific table from the Excel sheet using a marker.

        Args:
            table_name (str): The name of the table to extract data from.

        Returns:
            List[Dict[str, Any]]: A list of dictionaries representing the table data.

        Raises:
            ValueError: If the marker or table data is not found or if column names are not unique.
        """
        try:
            # Initialize the marker
            marker = 'NO TABLE YET'

            # Read the Excel file into a DataFrame
            df = pd.read_excel(self.excel_file, self.sheet_name, header=None)
            df = df.apply(lambda x: x.map(lambda y: y.strip() if isinstance(y, str) else y))

            # Find the marker for the table
            marker = self.normalize_string(f'#DATA:{table_name}')
            marker_row = df[df.iloc[:, 0].map(lambda x: isinstance(x, str) and self.normalize_string(x)) == marker]

            if marker_row.empty:
                logger.error(f"Marker '{marker}' not found in the Excel sheet for {table_name}.")
                raise ValueError(f"Marker '{marker}' not found in the Excel sheet for {table_name}.")

            table_start = marker_row.index[0] + 1

            # Determine the end of the table (next marker or end of file)
            table_end = table_start
            while table_end < len(df) and not str(df.iloc[table_end, 0]).startswith('#DATA:'):
                table_end += 1

            # Extract the table data
            table_data = df.iloc[table_start:table_end].dropna(axis=1, how='all').dropna(axis=0, how='all')

            # Check if table_data is empty
            if table_data.empty:
                logger.warning(f"Marker '{marker}' found, but no data is present after the marker.")
                self.data_list = []
                return self.data_list

            # Extract column names and data rows separately
            column_names = table_data.iloc[0].tolist()
            data_rows = table_data.iloc[1:]

            # Check for non-unique columns
            if len(column_names) != len(set(column_names)):
                raise ValueError(f"DataFrame columns are not unique for marker '{marker} {column_names} '.")

            data_rows.columns = column_names

            # Handle case where column names exist but no rows of data
            if data_rows.empty:
                logger.warning(f"Marker '{marker}' and columns are defined, but no row data is present.")
                self.data_list = []
            else:
                self.data_list = data_rows.to_dict(orient='records')

        except Exception as ex:
            logger.error(f"{ex} for {marker} - {traceback.format_exc()}")
            raise
        else:
            logger.info(f"data to be inserted for {marker}: {self.data_list}")
            return self.data_list


class DataUtility:
    """
    A class to load and process data from an Excel file into a database.
    """

    def __init__(self, context: Any, sub_directory: str) -> None:
        """
        Initializes the DataUtility.

        Args:
            context (Any): The application context containing configuration and database details.
            sub_directory (str): Sub-directory path for additional processing resources.
        """
        self.context = context
        self.sub_directory = sub_directory
        self.db_ops = DatabaseOperations(context)
        self.processor = PDSProcessor(base_directory=context.DDL_PATH)

    def load_excel_file(self, excel_file: str, sheet_name: str) -> None:
        """
        Load the Excel file and process the specified sheet.

        Args:
            excel_file (str): Path to the Excel file.
            sheet_name (str): Name of the sheet to process.
        """
        try:
            # Create an instance of ExcelToSQLInserter
            inserter = ExcelToSQLInserter(excel_file, sheet_name)

            # Read the entire sheet to find all tables
            df = pd.read_excel(excel_file, sheet_name=sheet_name, header=None)
            df = df.apply(lambda x: x.map(lambda y: y.strip() if isinstance(y, str) else y))

            # Iterate through the sheet to find all markers (table names)
            for i in range(len(df)):
                if isinstance(df.iloc[i, 0], str) and df.iloc[i, 0].startswith("#DATA:"):
                    table_name = inserter.normalize_string(df.iloc[i, 0].replace("#DATA:", ""))
                    logger.info(f"Found table marker: {table_name}")

                    # Extract and process table data
                    self.process_table_data(table_name, excel_file, sheet_name)
        except Exception as ex:
            logger.error(f"Failed to process sheet '{sheet_name}' in '{excel_file}': {ex}")
            logger.error(traceback.format_exc())
            raise

    def check_and_drop_view(self, table_name: str) -> None:
        """
        Check if a table exists and drop it if necessary.

        Args:
            table_name (str): Name of the table to check and drop.
        """
        try:
            # Check if view  exists
            exist_query = self.db_ops.build_sql_query("IF_VIEW_EXISTS", table_name, self.context.db_name)
            if self.db_ops.execute_cursor(exist_query):
                # Drop the view
                drop_sql = self.db_ops.build_sql_query("VIEW_DROP", table_name, self.context.db_name)
                self.db_ops.execute_cursor(drop_sql)
                logger.info(f"Table '{table_name}' was dropped successfully.")
        except Exception as ex:
            logger.error(f"Error checking and dropping table '{table_name}': {ex}")
            raise

    def check_and_drop_table(self, table_name: str) -> None:
        """
        Check if a table exists and drop it if necessary.

        Args:
            table_name (str): Name of the table to check and drop.
        """
        try:
            # Check if table exists
            exist_query = self.db_ops.build_sql_query("IF_EXISTS", table_name, self.context.db_name)
            if self.db_ops.execute_cursor(exist_query):
                # Drop the table
                drop_sql = self.db_ops.build_sql_query("DROP", table_name, self.context.db_name)
                self.db_ops.execute_cursor(drop_sql)
                logger.info(f"Table '{table_name}' was dropped successfully.")
        except Exception as ex:
            logger.error(f"Error checking and dropping table '{table_name}': {ex}")
            raise

    def create_table(self, table_name: str) -> None:
        """
        Create a table using the DDL file or a default query.

        Args:
            table_name (str): Name of the table to create.
        """
        try:
            # Attempt to read the DDL file
            query = self.processor.read_ddl_file(table_name, self.sub_directory)
        except FileNotFoundError:
            # Fallback: Build default CREATE TABLE query
            utility_db = f'DD{self.sub_directory}U01P'
            query = self.db_ops.build_sql_query("CREATE_TB", table_name, self.context.db_name, utility_db=utility_db)

        # Execute the CREATE TABLE query
        self.db_ops.execute_cursor(query)
        logger.info(f"Table '{table_name}' was created successfully.")

    def process_table_data(self, table_name: str, excel_file: str, sheet_name: str) -> None:
        """
        Process data for a specific table.

        Args:
            table_name (str): Name of the table.
            inserter (ExcelToSQLInserter): Instance of ExcelToSQLInserter used to extract data.
        """
        try:
            # Instantiate ExcelToSQLInserter locally
            inserter = ExcelToSQLInserter(excel_file, sheet_name)
            table_data = inserter.read_table_from_excel(table_name)
            self.context.tables_to_drop.append(table_name)
            logger.info(f"Processing table: {table_name}")

            # Step 1: Check if the table exists and drop it if necessary
            self.check_and_drop_table(table_name)

            # Step 2: Create the table
            self.create_table(table_name)

            # Step 3: Insert table data
            if table_data:
                self.insert_table_data(table_name, table_data)

        except Exception as ex:
            logger.error(f"Error processing table '{table_name}': {ex}")
            logger.error(traceback.format_exc())
            raise

    def handle_not_null_columns(self, table_name: str, table_data: List[Dict[str, Any]]) -> List[Dict[str, Any]]:
        """
        Handle NOT NULL columns by filling in placeholder values for missing data.

        Args:
            table_name (str): Name of the table.
            table_data (List[Dict[str, Any]]): List of data rows to process.

        Returns:
            List[Dict[str, Any]]: Updated table data with NOT NULL columns handled.
        """
        try:
            # Fetch NOT NULL columns and their types
            not_null_query = self.db_ops.build_sql_query("NOT_NULL_COLUMNS", table_name, self.context.db_name)
            not_null_columns = self.db_ops.execute_cursor(not_null_query, query_type="many")
            column_types = {col[0]: col[1] for col in not_null_columns}

            # Fill in placeholder values for NOT NULL columns
            for row in table_data:
                for column, col_type in column_types.items():
                    if column not in row:
                        null_data = self.db_ops.generate_placeholder_value(col_type.strip())
                        row[column] = null_data

            logger.info(f"Handled NOT NULL columns for table: {table_name}")
            return table_data

        except Exception as ex:
            logger.error(f"Error handling NOT NULL columns for table '{table_name}': {ex}")
            raise

    def execute_insert_queries(self, table_name: str, table_data: List[Dict[str, Any]]) -> None:
        """
        Build and execute insert queries for the specified table.

        Args:
            table_name (str): Name of the table.
            table_data (List[Dict[str, Any]]): List of data rows to insert.
        """
        try:
            # Build the insert queries
            sqls = self.db_ops.build_sql_query("INSERT", table_name, self.context.db_name, data=table_data)
            msr_query = " ".join(sqls)

            # Execute the insert queries
            self.db_ops.execute_cursor(msr_query)
            logger.info(f"Data was inserted into table '{table_name}' successfully.")
        except Exception as ex:
            logger.error(f"Error executing insert queries for table '{table_name}': {ex}")
            raise

    def insert_table_data(self, table_name: str, table_data: List[Dict[str, Any]]) -> None:
        """
        Insert data into the specified table.

        Args:
            table_name (str): Name of the table.
            table_data (List[Dict[str, Any]]): List of data rows to insert in dictionary format.
        """
        try:
            # Step 1: Handle NOT NULL columns if the flag is set
            if self.context.is_non_business_not_null == 'True':
                table_data = self.handle_not_null_columns(table_name, table_data)

            # Step 2: Build and execute the insert queries
            self.execute_insert_queries(table_name, table_data)

        except Exception as ex:
            logger.error(f"Error inserting data into table '{table_name}': {ex}")
            raise
