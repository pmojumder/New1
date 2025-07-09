import os
import pandas as pd
import logging
import traceback
from utils import DataUtility
from behave import given

# Initialize logger
logger = logging.getLogger(__name__)

###############################################################################
# LOAD TABLE DATA  WHEN ENTIRE DATA RESIDE IN EXCEL SHEET AND ONE  MAINE SHEET#
###############################################################################
@given('Load all {corporate_entity} tables from {excel_file},{sheet_name}')
def step_load_all_data_from_excel(context, corporate_entity, excel_file, sheet_name):
    """
    Behave step for loading all tables from a specific sheet.
    """
    sub_directory = corporate_entity
    excel_file = os.path.join(context.insert_table_excel_base_path, excel_file)
    loader = DataUtility(context, sub_directory)
    loader.load_excel_file(excel_file, sheet_name)


###############################################################################################
# LOAD TABLE DATA  WHEN MAIN TAB HAS LIST OF ALL SOURCE SHEETS AND EACH SHEET HAS DATA        #
###############################################################################################
@given('Load the {corporate_entity} tables from excel file {file_name} and the main tab {main_tab}')
def step_given_process_excel(context, corporate_entity, file_name, main_tab):
    """
    Behave step for loading tables from multiple sheets listed in the main tab.
    """
    base_path = context.insert_table_excel_base_path.strip().strip('"')
    file_name = file_name.strip().strip('"')
    excel_file = os.path.join(base_path, file_name)
    sub_directory = corporate_entity

    try:
        # Load the Excel file
        excel_data = pd.ExcelFile(excel_file)

        # Parse the main tab to get the list of sheet names
        main_tab_data = excel_data.parse(main_tab)
        sheet_names = main_tab_data['Tabs'].tolist()  # Adjust column name as per your Excel file structure

        # Process each sheet
        loader = DataUtility(context, sub_directory)
        for sheet_name in sheet_names:
            loader.load_excel_file(excel_file, sheet_name)
    except Exception as ex:
        logger.error(f"An error occurred: {ex}")
        logger.error(traceback.format_exc())
