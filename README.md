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
