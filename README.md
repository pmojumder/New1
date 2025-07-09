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
