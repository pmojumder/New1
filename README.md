For CI use ACC_SETUP_DTE

CN:
Below logic is for Take On:
Step 1: 
For the accounts that are in scope, find the min of Event date from ACCOUNT_EVENT_AS_LIMIT_AMEND table.
CREATE TABLE DDEWD06S.CCR_CN_ACC_SETUP_DTE
AS 
(
SELECT 
 A.WH_ACC_NO
,MIN(A.EVT_DTE ) AS EVT_DTE
,B.WH_ACC_NO AS CCR_WH_ACC_NO
FROM DDEWD01S.CCR_ACCOUNT_PERIODIC_FT1509 B
LEFT OUTER JOIN 
DDEWV01P.ACCOUNT_EVENT_AS_LIMIT_AMEND A
ON 
A.WH_ACC_NO=B.WH_ACC_NO
WHERE 
B.CCR_CNTRACT_TYP='CN'
GROUP BY 1,3
) WITH DATA

Step 2:
For those accounts for which we are not able to find the value from Step 1, Go directly and check from EAPF when the first time the Limit Existed and pick the corresponding date. 
SELECT WH_ACC_NO,ACC_SETUP_DTE,NET_LIMIT_FOR_PRNCPL,PERIOD_DTE FROM DDEWV50P.ENT_ACCOUNT_PERIODIC_FACT 
WHERE WH_ACC_NO IN 
(
SELECT CAST(CCR_WH_ACC_NO AS INTEGER) FROM DDEWD06S.CCR_CN_ACC_SETUP_DTE WHERE EVT_DTE IS NULL
)
AND 
NET_LIMIT_FOR_PRNCPL<0
ORDER BY 1,4
Step 3:
Use the values obtained in Step 1 & Step 2 for Acc Setup Date for CN.

Below logic is for On Going:
If the account exists in the Previous month, then Pick the value of ACC_SETUP_DTE that was derived last month.

If the account is new, then we need to repeat the Process for 'Take On' again to determine ACC_SETUP_DTE. 

Source Table: ENT_ACCOUNT_PERIODIC_FACT
Source Field: ACC_SETUP_DTE

Target Table: CCR_ACCOUNT_PERIODIC
Target Field: ACC_SETUP_DTE
----------------------------------------------


Hey Plabani - I am just looking at the logic there and there is an insert there to carry forward records from the previois month's CCR_ACCOUNT_PERIODIC if they aren't added by the logic for the current month and some other criteria there too. Thinking it's probably best to handle this kind of record on it's own in a Feature checking all columns rather than including it in individual column tests if that makes sense?
