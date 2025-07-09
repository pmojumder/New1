 behave features/06_poc.feature
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
