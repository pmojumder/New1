import os
from dotenv import load_dotenv, find_dotenv
from steps.utils import MainframeApiService, PDSProcessor
from environment import DDLDownloader
import traceback
import shutil

# Load environment variables from the .env file
load_dotenv(find_dotenv())

try:
    api_base_url = os.getenv('API_BASE_URL')
    api_username = os.getenv('ZOWE_API_USERNAME')
    api_password = os.getenv('ZOWE_API_PASSWORD')
    ddl_path = os.getenv('DDL_PATH')

    if not all([api_base_url, api_username, api_password, ddl_path]):
        raise ValueError("Missing one or more required environment variables.")

    if os.path.exists(ddl_path):
        shutil.rmtree(ddl_path)
        print(f"Old DDL path {ddl_path} deleted succesfully.")

    submitter = MainframeApiService(api_username, api_password, api_base_url)
    pdsprocessor = PDSProcessor(base_directory=ddl_path)

    download_ddl = DDLDownloader(submitter, processor=pdsprocessor)
    download_ddl.download_ddl_files()

    print("DDL download completed successfully.")
except Exception as e:
    print(f"An error occurred: {e} {traceback.format_exc()}")


