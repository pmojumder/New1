from behave import given, when, then
import traceback
from utils import DatabaseOperations, DataProcessor, SHA256Hasher, ExcelProcessor, MainframeApiService, PDSProcessor,
    ExcelToSQLInserter, DataUtility

from behave import given, when, then

from utils import MainframeApiService

import os


import time
import os
import time

# Create a global variable to store the DatabaseOperations instance
db_ops = None
import logging

logger = logging.getLogger(__name__)


###########################################
# STEP: Verify the table is empty         #
###########################################
@given('the table {table_name} is empty')
def is_table_empty(context, table_name):
    if getattr(context, 'skip_background', True):
        return

    global db_ops
    if db_ops is None:
        db_ops = DatabaseOperations(context)
    try:
        context.table_name = table_name
        check_query = db_ops.build_sql_query("CHECK_COUNT", context.table_name, context.db_name)
        result = db_ops.execute_cursor(check_query)
        if result[0] != 0:
            logger.error(f"{table_name} is not empty.")
            raise AssertionError
    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


###########################################
# STEP: Populate corporate entity table   #
###########################################
@given('the {corporate_entity} table {table_name} contains')
def step_impl_validations(context, table_name, corporate_entity):
    global db_ops
    sub_directory = corporate_entity
    db_ops = DatabaseOperations(context)  # Initialize the DatabaseOperations instance
    processor = PDSProcessor(base_directory=context.DDL_PATH)  # Initialize the PDSProcessor instance
    processor.check_for_multiple_ddl(table_name, sub_directory)
    context.tables_to_drop.append(table_name)
    utils = DataUtility(context, sub_directory)

    try:
        utils.check_and_drop_table(table_name)

        ddl_statement = processor.read_ddl_file(table_name, sub_directory)
        db_ops.execute_cursor(ddl_statement)
        ##
        if context.table:
            data = [dict(row.items()) for row in context.table]
            if data:
                data = utils.handle_not_null_columns(table_name, table_data=data)
                utils.insert_table_data(table_name, data)

    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


###########################################
# STEP: Ensure table does not exist       #
###########################################
@given('the table {table_name} does not exists')
def check_table_exists(context, table_name):
    context.tables_to_drop.append(table_name)
    if getattr(context, 'skip_background', True):
        return
    pass
    global db_ops
    if db_ops is None:
        db_ops = DatabaseOperations(context)
    try:
        context.table_name = table_name
        query = db_ops.build_sql_query("IF_EXISTS", context.table_name, context.db_name)
        result = db_ops.execute_cursor(query)
        if result:
            raise AssertionError(f"{table_name} does exist.")


    except Exception as ex:
        # Handle all other exceptions
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


###########################################
# STEP: Validate table schema matches     #
###########################################
@then('the table {table_name} should match its definition in {file_name},{sheet_name}')
def check_expected_data(context, table_name, file_name, sheet_name):
    global db_ops
    if db_ops is None:
        db_ops = DatabaseOperations(context)
    context.tables_to_drop.append(table_name)

    try:
        # Get Schema from DB and compute SHA
        db_schema_query = db_ops.build_sql_query("DB_SCHEMA", table_name, context.db_name)
        ddl_result = db_ops.execute_cursor(db_schema_query, query_type="many")
        if not ddl_result:
            raise ValueError(f"No schema information returned for table {table_name} in database {context.db_name}")

        processor = DataProcessor()
        list_of_tuples = processor.get_list_of_tuples(ddl_result)
        sha_db, schema_db = SHA256Hasher.compute_sha256_from_list(list_of_tuples)

        processor = ExcelProcessor(file_path=context.excel_base_path, excel_name=file_name, sheet_name=sheet_name,
                                   base_excel_url=context.excel_base_url, username=context.excel_username,
                                   password=context.excel_password)
        # Extract data
        extracted_data = processor.extract_columns_from_excel(
            expected_columns=["Target Field", "Data Type", "Primary Key"])

        processed_data = processor.excel_schema_sha_calc(extracted_data)
        sha256_excel, schema_excel = SHA256Hasher.compute_sha256_from_list(processed_data)

        diff = [item for item in schema_excel if item not in schema_db]

        if context.excel_base_url:
            processor.delete_temp_file()

        assert sha_db == sha256_excel, f"Table {context.db_name}.{table_name}'s schema defined within tab {sheet_name} of excel file {file_name} is different from the schema created by mainframe job. The difference is {diff}"



    except Exception as ex:
        # Handle all other exceptions
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


###########################################
# EXECUTE MAINFRAME STEPS                 #
###########################################
@when('the mainframe job {job_name} runs')
def execute_mainframe_job(context, job_name):
    try:
        submitter = MainframeApiService(context.api_username, context.api_password, context.api_base_url)
        context.token = submitter.login()
        job_location = f"{context.pds}({job_name})"
        response = submitter.submit_job(job_location, context.token)

        context.job_id = response.get('jobId')
        context.job_name = response.get('jobName')

        assert context.job_id and context.job_name, f"Job  failed: Job Name: {job_name}"
        logger.info(f"Job submitted successfully: Job ID: {context.job_id}, Job Name: {context.job_name}")

    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise AssertionError


###########################################
# CHECK MAINFRAME JOB STATUS              #
###########################################
@then('the results of job execution are successful')
def check_job_status(context):
    try:
        submitter = MainframeApiService(context.api_username, context.api_password, context.api_base_url)
        max_retries = int(context.retry)  # Maximum number of status checks (e.g., 60 retries)
        retry_interval = int(context.retry_interval)  # Time to wait between checks in seconds
        retries = 0  # Initialize

        while retries < max_retries:
            status = submitter.job_status(context.job_name, context.job_id, context.token)
            # Check if job is complete
            if status.get('returnCode'):
                return_code = status.get('returnCode')
                logger.info(f"Job {context.job_id} completed with status: {return_code}")

                # Assert that the return code is 'CC 0000'
                if return_code != 'CC 0000':
                    raise AssertionError(f"Job {context.job_id} failed with return code: {return_code}")
                break

            # Wait before polling again
            time.sleep(retry_interval)
            retries += 1

        if retries == max_retries:
            raise TimeoutError(f"Job {context.job_id} did not complete within the expected time frame.")

    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise AssertionError


###########################################
# VALIDATE TABLE DATA                     #
###########################################

@then('the table {table_name} should contain')
def check_expected_data(context, table_name):
    global db_ops
    if db_ops is None:
        db_ops = DatabaseOperations(context)
    try:
        expected_data = [dict(row.items()) for row in context.table]
        db_ops.validate_source_table_data(context.db_name, table_name, expected_data)
    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


###########################################
# VALIDATE TABLE STATISTICS               #
###########################################
@then('the table {table_name} has current statistics')
def stats_impl(context, table_name):
    global db_ops
    if db_ops is None:
        db_ops = DatabaseOperations(context)
    try:
        query = db_ops.build_sql_query("COLLECT SUMMARY STAT", table_name, context.db_name)
        db_ops.execute_cursor(query)
        query = db_ops.build_sql_query("COLLECT STAT", table_name, context.db_name)
        db_ops.execute_cursor(query)
        query = db_ops.build_sql_query("CHECK STAT", table_name, context.db_name)
        result = db_ops.execute_cursor(query)

        assert result[2], f"COLLECT STAT is not being performed on {context.db_name}.{table_name}"

    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


###########################################
# ENSURE VIEW DATA  POPULATION            #
###########################################
@given('the {corporate_entity} view {view_name} contains')
def step_impl_view(context, view_name, corporate_entity):
    global db_ops
    db_ops = DatabaseOperations(context)  # Initialize the DatabaseOperations instance

    context.tables_to_drop.append(view_name)
    utils = DataUtility(context, corporate_entity)
    try:
        utils.check_and_drop_view(table_name=view_name)
        utils.check_and_drop_table(table_name=view_name)

        utility_db = f'DD{corporate_entity}U01P'

        query = db_ops.build_sql_query("CREATE_TB", view_name, context.db_name, utility_db=utility_db)

        db_ops.execute_cursor(query)
        if context.table:
            data = [dict(row.items()) for row in context.table]
            utils.execute_insert_queries(table_name=view_name, table_data=data)


    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


########################################################
# ENSURE TABLE DATA  POPULATION VIA EXCEL FILE AND SHEET#
########################################################
@given('the {corporate_entity} table {table_name} contains data from {excel_file},{sheet_name}')
def step_impl_excel_table_insert(context, table_name, corporate_entity, excel_file, sheet_name):
    global db_ops
    sub_directory = corporate_entity
    db_ops = DatabaseOperations(context)  # Initialize the DatabaseOperations instance
    processor = PDSProcessor(base_directory=context.DDL_PATH)  # Initialize the PDSProcessor instance
    processor.check_for_multiple_ddl(table_name, sub_directory)
    context.tables_to_drop.append(table_name)
    excel_full_path = os.path.join(context.insert_table_excel_base_path, excel_file)
    excel_ops = DataUtility(context, sub_directory)

    try:
        excel_ops.check_and_drop_table(table_name)
        ddl_statement = processor.read_ddl_file(table_name, sub_directory)
        db_ops.execute_cursor(ddl_statement)

        reader = ExcelToSQLInserter(excel_full_path, sheet_name)
        data = reader.read_table_from_excel(table_name)

        if data:
            excel_ops.insert_table_data(table_name, data)

    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


#########################################################
# ENSURE VIEW DATA POPULATION VIA EXCEL FILE AND SHEET  #
#########################################################
@given('the {corporate_entity} view {view_name} contains data from {excel_file},{sheet_name}')
def step_impl_excel_table_insert(context, view_name, corporate_entity, excel_file, sheet_name):
    global db_ops
    sub_directory = corporate_entity
    excel_full_path = os.path.join(context.insert_table_excel_base_path, excel_file)
    db_ops = DatabaseOperations(context)  # Initialize the DatabaseOperations instance
    excel_ops = DataUtility(context, sub_directory)
    context.tables_to_drop.append(view_name)
    try:
        excel_ops.check_and_drop_view(view_name)
        excel_ops.check_and_drop_table(table_name=view_name)
        utility_db = f'DD{corporate_entity}U01P'
        query = db_ops.build_sql_query("CREATE_TB", view_name, context.db_name, utility_db=utility_db)
        db_ops.execute_cursor(query)
        reader = ExcelToSQLInserter(excel_full_path, sheet_name)
        data = reader.read_table_from_excel(view_name)
        if data:
            excel_ops.execute_insert_queries(table_name=view_name, table_data=data)


    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise


#########################################################################
# ENSURE TABLE DATA  VALIDATION WHEN  TARGET DATA RESIDE IN EXCEL SHEET #
#########################################################################
@then('the table {table_name} should contain data from {excel_file},{sheet_name}')
def check_expected_data(context, table_name, excel_file, sheet_name):
    global db_ops
    if db_ops is None:
        db_ops = DatabaseOperations(context)
    try:
        excel_full_path = os.path.join(context.insert_table_excel_base_path, excel_file)
        reader = ExcelToSQLInserter(excel_full_path, sheet_name)
        expected_data = reader.read_table_from_excel(table_name)
        db_ops.validate_source_table_data(context.db_name, table_name, expected_data)
    except Exception as ex:
        logger.error(f"An unexpected error occurred: {ex} {traceback.format_exc()}")
        raise
