Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved.

Try the new cross-platform PowerShell https://aka.ms/pscore6

(.venv) PS C:\Products\prj> pip install -r requirements.txt                                                                                               
Requirement already satisfied: behave in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 1)) (1.2.6)
Requirement already satisfied: python-dotenv in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 2)) (1.0.1)
Requirement already satisfied: teradatasql in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 3)) (20.0.0.24)
Requirement already satisfied: behave-html-formatter in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 4)) (0.9.10)
Requirement already satisfied: requests in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 5)) (2.32.3)
Requirement already satisfied: openpyxl in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 6)) (3.1.5)
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
ERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
Could not fetch URL https://pypi.org/simple/allure-behave/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/allure-behave/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
ERROR: Could not find a version that satisfies the requirement allure-behave (from versions: none)
ERROR: No matching distribution found for allure-behave
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
(.venv) PS C:\Products\prj> ls


    Directory: C:\Products\prj


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        09/07/2025     16:12                .idea
d-----        24/02/2025     18:32                .venv
d-----        22/02/2025     20:14                .venv1
d-----        20/02/2025     18:44                allure-reports
d-----        09/07/2025     16:05                features
d-----        20/02/2025     21:14                reports
-a----        07/05/2025     15:33           1956 .env                                                                                                          
-a----        20/02/2025     21:12           3720 01_poc.feature
-a----        09/05/2025     19:47           1573 02_poc.feature
-a----        20/02/2025     18:44            147 behave.ini
-a----        20/02/2025     18:44          12976 gherkin.py
-a----        26/02/2025     22:06          35367 gherkin1.py
-a----        20/02/2025     18:44           4399 readme.md
-a----        09/07/2025     16:15            202 requirements.txt


(.venv) PS C:\Products\prj> pip install -r requirements.txt
Requirement already satisfied: behave in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 1)) (1.2.6)
Requirement already satisfied: python-dotenv in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 2)) (1.0.1)
Requirement already satisfied: teradatasql in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 3)) (20.0.0.24)
Requirement already satisfied: behave-html-formatter in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 4)) (0.9.10)
Requirement already satisfied: requests in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 5)) (2.32.3)
Requirement already satisfied: openpyxl in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 6)) (3.1.5)
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
ERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/allure-behave/
Could not fetch URL https://pypi.org/simple/allure-behave/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/allure-behave/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
ERROR: Could not find a version that satisfies the requirement allure-behave (from versions: none)
ERROR: No matching distribution found for allure-behave
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
(.venv) PS C:\Products\prj> pip install -r requirements.txt
Requirement already satisfied: behave in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 1)) (1.2.6)
Requirement already satisfied: python-dotenv in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 2)) (1.0.1)
Requirement already satisfied: teradatasql in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 3)) (20.0.0.24)
Requirement already satisfied: behave-html-formatter in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 4)) (0.9.10)
Requirement already satisfied: requests in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 5)) (2.32.3)
Requirement already satisfied: openpyxl in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 6)) (3.1.5)
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
ERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
Could not fetch URL https://pypi.org/simple/pandas/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pandas/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
ERROR: Could not find a version that satisfies the requirement pandas (from versions: none)
ERROR: No matching distribution found for pandas
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
(.venv) PS C:\Products\prj> pip install -r requirements.txt
Requirement already satisfied: behave in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 1)) (1.2.6)
Requirement already satisfied: python-dotenv in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 2)) (1.0.1)
Requirement already satisfied: teradatasql in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 3)) (20.0.0.24)
Requirement already satisfied: behave-html-formatter in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 4)) (0.9.10)
Requirement already satisfied: requests in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 5)) (2.32.3)
Requirement already satisfied: openpyxl in c:\products\prj\.venv\lib\site-packages (from -r requirements.txt (line 6)) (3.1.5)
WARNING: Retrying (Retry(total=4, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=3, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=2, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
WARNING: Retrying (Retry(total=1, connect=None, read=None, redirect=None, status=None)) after connection broken by 'SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))': /simple/pandas/
ERROR: Could not find a version that satisfies the requirement pandas==2.2.2 (from versions: none)
ERROR: No matching distribution found for pandas==2.2.2
Could not fetch URL https://pypi.org/simple/pip/: There was a problem confirming the ssl certificate: HTTPSConnectionPool(host='pypi.org', port=443): Max retries exceeded with url: /simple/pip/ (Caused by SSLError(SSLCertVerificationError(1, '[SSL: CERTIFICATE_VERIFY_FAILED] certificate verify failed: self-signed certificate in certificate chain (_ssl.c:1028)'))) - skipping
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj> pip install pandas --trusted-host pypi.org --trusted-host files.pythonhosted.org --trusted-host=pypi.python.org
Collecting pandas
  Obtaining dependency information for pandas from https://files.pythonhosted.org/packages/b2/c0/54415af59db5cdd86a3d3bf79863e8cc3fa9ed265f0745254061ac09d5f2/pandas-2.3.1-cp313-cp313-win_amd64.whl.metadata
  Downloading pandas-2.3.1-cp313-cp313-win_amd64.whl.metadata (19 kB)
Collecting numpy>=1.26.0 (from pandas)
  Obtaining dependency information for numpy>=1.26.0 from https://files.pythonhosted.org/packages/dd/c8/beaba449925988d415efccb45bf977ff8327a02f655090627318f6398c7b/numpy-2.3.1-cp313-cp313-win_amd64.whl.metadata
  Downloading numpy-2.3.1-cp313-cp313-win_amd64.whl.metadata (60 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 60.9/60.9 kB 4.5 MB/s eta 0:00:00
Collecting python-dateutil>=2.8.2 (from pandas)
  Obtaining dependency information for python-dateutil>=2.8.2 from https://files.pythonhosted.org/packages/ec/57/56b9bcc3c9c6a792fcbaf139543cee77261f3651ca9da0c93f5c1221264b/python_dateutil-2.9.0.post0-py2.py3-none-any.whl.metadata
  Downloading python_dateutil-2.9.0.post0-py2.py3-none-any.whl.metadata (8.4 kB)
Collecting pytz>=2020.1 (from pandas)
  Obtaining dependency information for pytz>=2020.1 from https://files.pythonhosted.org/packages/81/c4/34e93fe5f5429d7570ec1fa436f1986fb1f00c3e0f43a589fe2bbcd22c3f/pytz-2025.2-py2.py3-none-any.whl.metadata
  Downloading pytz-2025.2-py2.py3-none-any.whl.metadata (22 kB)
Collecting tzdata>=2022.7 (from pandas)
  Obtaining dependency information for tzdata>=2022.7 from https://files.pythonhosted.org/packages/5c/23/c7abc0ca0a1526a0774eca151daeb8de62ec457e77262b66b359c3c7679e/tzdata-2025.2-py2.py3-none-any.whl.metadata
  Downloading tzdata-2025.2-py2.py3-none-any.whl.metadata (1.4 kB)
Requirement already satisfied: six>=1.5 in c:\products\prj\.venv\lib\site-packages (from python-dateutil>=2.8.2->pandas) (1.17.0)
Downloading pandas-2.3.1-cp313-cp313-win_amd64.whl (11.0 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 11.0/11.0 MB 27.9 MB/s eta 0:00:00
Downloading numpy-2.3.1-cp313-cp313-win_amd64.whl (12.7 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 12.7/12.7 MB 25.1 MB/s eta 0:00:00
Downloading python_dateutil-2.9.0.post0-py2.py3-none-any.whl (229 kB)
Downloading tzdata-2025.2-py2.py3-none-any.whl (347 kB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 347.8/347.8 kB 22.0 MB/s eta 0:00:00
Installing collected packages: pytz, tzdata, python-dateutil, numpy, pandas
Successfully installed numpy-2.3.1 pandas-2.3.1 python-dateutil-2.9.0.post0 pytz-2025.2 tzdata-2025.2

[notice] A new release of pip is available: 23.2.1 -> 25.1.1
[notice] To update, run: python.exe -m pip install --upgrade pip
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj> behave features/06_poc.feature
Feature: Test the new Tier 2 CPVT_AIB_NPE_VAL_ALERT # features/06_poc.feature:1
  As an EDW engineer,
  I want to validate that CPVT_AIB_NPE_VAL_ALERT table is created and loaded correctly.
  @create
  Scenario: Test table CPVT_AIB_NPE_VAL_ALERT is created correctly                                                                  # features/06_poc.feature:6  
    Given the table CPVT_AIB_NPE_VAL_ALERT does not exists                                                                          # features/steps/gherkin.py:78
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\gherkin.py", line 80, in check_table_exists
          context.tables_to_drop.append(table_name)
          ^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'tables_to_drop'

    When the mainframe job EWT00001 runs                                                                                            # None
    Then the results of job execution are successful                                                                                # None
    Then the table CPVT_AIB_NPE_VAL_ALERT should match its definition in CPVT_AIB_NPE_VAL_ALERT_BRD_ROI.xlsx,CPVT_AIB_NPE_VAL_ALERT # None

  @load
  Scenario: Test the daily load of CPVT_AIB_NPE_VAL_ALERT                                                # features/06_poc.feature:14
    Given Load all EW tables from CPVT_AIB_NPE_VAL_ALERT.xlsx,SourceData                                 # features/steps/sheet.py:14
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\sheet.py", line 20, in step_load_all_data_from_excel
          excel_file = os.path.join(context.insert_table_excel_base_path, excel_file)
                                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'insert_table_excel_base_path'

    And the table CPVT_AIB_NPE_VAL_ALERT is empty                                                        # None
    When the mainframe job EWT01AIV runs                                                                 # None
    Then the results of job execution are successful                                                     # None
    And the table CPVT_AIB_NPE_VAL_ALERT should contain data from CPVT_AIB_NPE_VAL_ALERT.xlsx,TargetData # None
    And the table CPVT_AIB_NPE_VAL_ALERT has current statistics                                          # None

Failing scenarios:
  features/06_poc.feature:6  Test table CPVT_AIB_NPE_VAL_ALERT is created correctly
  features/06_poc.feature:14  Test the daily load of CPVT_AIB_NPE_VAL_ALERT

0 features passed, 1 failed, 0 skipped
0 scenarios passed, 2 failed, 0 skipped
0 steps passed, 2 failed, 8 skipped, 0 undefined
Took 0m0.011s
(.venv) PS C:\Products\prj> behave features/06_poc.feature
Feature: Test the new Tier 2 CPVT_AIB_NPE_VAL_ALERT # features/06_poc.feature:1
  As an EDW engineer,
  I want to validate that CPVT_AIB_NPE_VAL_ALERT table is created and loaded correctly.
  @create
  Scenario: Test table CPVT_AIB_NPE_VAL_ALERT is created correctly                                                                  # features/06_poc.feature:6  
    Given the table CPVT_AIB_NPE_VAL_ALERT does not exists                                                                          # features/steps/gherkin.py:78
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\gherkin.py", line 80, in check_table_exists
          context.tables_to_drop.append(table_name)
          ^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'tables_to_drop'

    When the mainframe job EWT00001 runs                                                                                            # None
    Then the results of job execution are successful                                                                                # None
    Then the table CPVT_AIB_NPE_VAL_ALERT should match its definition in CPVT_AIB_NPE_VAL_ALERT_BRD_ROI.xlsx,CPVT_AIB_NPE_VAL_ALERT # None

  @load
  Scenario: Test the daily load of CPVT_AIB_NPE_VAL_ALERT                                                # features/06_poc.feature:14
    Given Load all EW tables from CPVT_AIB_NPE_VAL_ALERT.xlsx,SourceData                                 # features/steps/sheet.py:14
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\sheet.py", line 20, in step_load_all_data_from_excel
          excel_file = os.path.join(context.insert_table_excel_base_path, excel_file)
                                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'insert_table_excel_base_path'

    And the table CPVT_AIB_NPE_VAL_ALERT is empty                                                        # None
    When the mainframe job EWT01AIV runs                                                                 # None
    Then the results of job execution are successful                                                     # None
    And the table CPVT_AIB_NPE_VAL_ALERT should contain data from CPVT_AIB_NPE_VAL_ALERT.xlsx,TargetData # None
    And the table CPVT_AIB_NPE_VAL_ALERT has current statistics                                          # None

Failing scenarios:
  features/06_poc.feature:6  Test table CPVT_AIB_NPE_VAL_ALERT is created correctly
  features/06_poc.feature:14  Test the daily load of CPVT_AIB_NPE_VAL_ALERT

0 features passed, 1 failed, 0 skipped
0 scenarios passed, 2 failed, 0 skipped
0 steps passed, 2 failed, 8 skipped, 0 undefined
Took 0m0.010s
(.venv) PS C:\Products\prj> behave features/06_poc.feature
Feature: Test the new Tier 2 CPVT_AIB_NPE_VAL_ALERT # features/06_poc.feature:1
  As an EDW engineer,
  I want to validate that CPVT_AIB_NPE_VAL_ALERT table is created and loaded correctly.
  @create
  Scenario: Test table CPVT_AIB_NPE_VAL_ALERT is created correctly                                                                  # features/06_poc.feature:6
    Given the table CPVT_AIB_NPE_VAL_ALERT does not exists                                                                          # features/steps/gherkin.py:78
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\gherkin.py", line 80, in check_table_exists
          context.tables_to_drop.append(table_name)
          ^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'tables_to_drop'

    When the mainframe job EWT00001 runs                                                                                            # None
    Then the results of job execution are successful                                                                                # None
    Then the table CPVT_AIB_NPE_VAL_ALERT should match its definition in CPVT_AIB_NPE_VAL_ALERT_BRD_ROI.xlsx,CPVT_AIB_NPE_VAL_ALERT # None

  @load
  Scenario: Test the daily load of CPVT_AIB_NPE_VAL_ALERT                                                # features/06_poc.feature:14
    Given Load all EW tables from CPVT_AIB_NPE_VAL_ALERT.xlsx,SourceData                                 # features/steps/sheet.py:14
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\sheet.py", line 20, in step_load_all_data_from_excel
          excel_file = os.path.join(context.insert_table_excel_base_path, excel_file)
                                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'insert_table_excel_base_path'

    And the table CPVT_AIB_NPE_VAL_ALERT is empty                                                        # None
    When the mainframe job EWT01AIV runs                                                                 # None
    Then the results of job execution are successful                                                     # None
    And the table CPVT_AIB_NPE_VAL_ALERT should contain data from CPVT_AIB_NPE_VAL_ALERT.xlsx,TargetData # None
    And the table CPVT_AIB_NPE_VAL_ALERT has current statistics                                          # None
  features/06_poc.feature:6  Test table CPVT_AIB_NPE_VAL_ALERT is created correctly
  features/06_poc.feature:14  Test the daily load of CPVT_AIB_NPE_VAL_ALERT

0 features passed, 1 failed, 0 skipped
0 scenarios passed, 2 failed, 0 skipped
0 steps passed, 2 failed, 8 skipped, 0 undefined
Took 0m0.008s
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj> behave features/06_poc.feature
Feature: Test the new Tier 2 CPVT_AIB_NPE_VAL_ALERT # features/06_poc.feature:1
  As an EDW engineer,
  I want to validate that CPVT_AIB_NPE_VAL_ALERT table is created and loaded correctly.
  @create
  Scenario: Test table CPVT_AIB_NPE_VAL_ALERT is created correctly  # features/06_poc.feature:6
    Given the table CPVT_AIB_NPE_VAL_ALERT does not exists          # features/steps/gherkin.py:78
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\gherkin.py", line 80, in check_table_exists
          context.tables_to_drop.append(table_name)
          ^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'tables_to_drop'


  @load
  Scenario: Test the daily load of CPVT_AIB_NPE_VAL_ALERT                                                # features/06_poc.feature:15
    Given Load all EW tables from CPVT_AIB_NPE_VAL_ALERT.xlsx,SourceData                                 # features/steps/sheet.py:14
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\sheet.py", line 20, in step_load_all_data_from_excel
          excel_file = os.path.join(context.insert_table_excel_base_path, excel_file)
                                    ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'insert_table_excel_base_path'

    And the table CPVT_AIB_NPE_VAL_ALERT is empty                                                        # None
    When the mainframe job EWT01AIV runs                                                                 # None
    Then the results of job execution are successful                                                     # None
    And the table CPVT_AIB_NPE_VAL_ALERT should contain data from CPVT_AIB_NPE_VAL_ALERT.xlsx,TargetData # None
    And the table CPVT_AIB_NPE_VAL_ALERT has current statistics                                          # None

0 features passed, 1 failed, 0 skipped
0 scenarios passed, 2 failed, 0 skipped
0 steps passed, 2 failed, 5 skipped, 0 undefined
Took 0m0.006s
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj>
(.venv) PS C:\Products\prj> behave features/06_poc.feature
Feature: Test the new Tier 2 CPVT_AIB_NPE_VAL_ALERT # features/06_poc.feature:1
  As an EDW engineer,
  I want to validate that CPVT_AIB_NPE_VAL_ALERT table is created and loaded correctly.
  @create
  Scenario: Test table CPVT_AIB_NPE_VAL_ALERT is created correctly  # features/06_poc.feature:6
    Given the table CPVT_AIB_NPE_VAL_ALERT does not exists          # features/steps/gherkin.py:78
      Traceback (most recent call last):
        File "C:\Products\prj\.venv\Lib\site-packages\behave\model.py", line 1329, in run
          match.run(runner.context)
          ~~~~~~~~~^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\matchers.py", line 98, in run
          self.func(context, *args, **kwargs)
          ~~~~~~~~~^^^^^^^^^^^^^^^^^^^^^^^^^^
        File "features\steps\gherkin.py", line 80, in check_table_exists
          context.tables_to_drop.append(table_name)
          ^^^^^^^^^^^^^^^^^^^^^^
        File "C:\Products\prj\.venv\Lib\site-packages\behave\runner.py", line 321, in __getattr__
          raise AttributeError(msg)
      AttributeError: 'Context' object has no attribute 'tables_to_drop'



Failing scenarios:
  features/06_poc.feature:6  Test table CPVT_AIB_NPE_VAL_ALERT is created correctly

0 features passed, 1 failed, 0 skipped
0 scenarios passed, 1 failed, 0 skipped
0 steps passed, 1 failed, 0 skipped, 0 undefined
Took 0m0.008s
(.venv) PS C:\Products\prj> ls                                                                                                             

