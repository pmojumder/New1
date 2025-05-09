	CCR_ACCOUNT_PERIODIC	CI08_Provider Contract No	WH_ACC_NO                     	INTEGER	N	Y				DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		WH_ACC_NO				[useful comments for analysts]
	CCR_ACCOUNT_PERIODIC	CI08_Provider Contract No	ACC_NO                        	CHAR(16)	N					DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		ACC_No				
	CCR_ACCOUNT_PERIODIC	CI01 Record Type	CCR_CNTRACT_TYP	CHAR(2)	N		"CASE 
WHEN  EAPF.SRCE_INST=1 AND EAPF.SRCE_SYS=1359 
AND EAPF.SRCE_PROD_CDE_FK LIKE ANY('3%','4%')  
AND (EAPF.CR_PORTF_FK NOT IN ('PDH', 'BTL')  OR PROD_SUMM_DESCR NOT IN ( 'PROPERTY FINANCE' , 'HOME LOAN'))
THEN 'CN'
WHEN EAPF.SRCE_INST=9 AND EAPF.SRCE_SYS=60   THEN 'CN'
WHEN EAPF.SRCE_INST=9 AND EAPF.CR_PORTF_FK  NOT IN ('PDH', 'BTL') THEN 'CN'
WHEN 
 EA5.SRCE_INST= CNTRCT_TYP.SRCE_INST
        AND EA5.SRCE_SYS= CNTRCT_TYP.SRCE_SYS
        AND EA5.SRCE_PROD_CDE_FK=CNTRCT_TYP.SRCE_PROD_CDE
        THEN CNTRCT_TYP.CCR_CNTRACT_TYP
ELSE 'CI'
END AS CCR_CNTRACT_TYP"			"DDEWV50P
DDEWV50P"	"ENT_ACCOUNT_PERIODIC_FACT
CCR_CNTRACT_TYP_REF"						
	CCR_ACCOUNT_PERIODIC		ACC_SETUP_DTE	DATE FORMAT 'YYYY-MM-DD'			"For CI use ACC_SETUP_DTE

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

If the account is new, then we need to repeat the Process for 'Take On' again to determine ACC_SETUP_DTE. "	"SELECT 
 A. WH_ACC_NO
,TEMP_FAC
,TEMP_FAC*-1 AS TEMP_NET_LIMIT
,A.ACC_OPEN_DTE
,B.ACC_SETUP_DTE
,CASE WHEN TEMP_NET_LIMIT <=-500 THEN 1 ELSE 0 END
FROM DDBWV01P.ACCOUNT_PERIODIC_SAVINGS A
INNER JOIN 
DDEWV50P.ENT_ACCOUNT_PERIODIC_FACT B
ON 
A.WH_ACC_NO=B.WH_ACC_NO
AND 
A.PERIOD_DTE=B.PERIOD_DTE
WHERE A.PERIOD_DTE = 1171229
AND B.CLOSE_DTE IS NULL  -- Checking if the acount is open in EAPF
AND TEMP_FAC >=500  -- This is to determine the Money Manager accounts with Limit

For the Money manager accounts, use TEMP_NET_LIMIT as NET_LIMIT_FOR_PRNCPL
and update the CCR_FLG='P' and ACC_SETUP_DTE =ACC_OPEN_DTE so that money manager accounts come into the scope of accounts for CCR."		DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		ACC_SETUP_DTE				Different Logic for CN .. Get previous value for CCR_Account_Periodic .. Ie last time it was on the table .. If never on the table .. Check the first  time that it when through a 140 .. 
	CCR_ACCOUNT_PERIODIC		SRCE_PROD_CDE_FK	VARCHAR(20)			Direct Mapping			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		SRCE_PROD_CDE_FK				
	CCR_ACCOUNT_PERIODIC		SECTOR_CDE_FK	SMALLINT			Direct Mapping			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		SECTOR_CDE_FK				
	CCR_ACCOUNT_PERIODIC		BSL_DAYS_PAST_DUE	SMALLINT			"CASE                                                
 WHEN CCR_CNTRACT_TYP ='CI' AND                      
 (AMT_PSTD < 0.5 OR AMT_PSTD IS NULL OR AMT_PSTD = '' )
 THEN NULL             
ELSE BSL_DAYS_PAST_DUE
updated in L51"			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		BSL_DAYS_PAST_DUE				
	CCR_ACCOUNT_PERIODIC		MATURITY_DTE	DATE FORMAT 'YYYY-MM-DD'			"FOR SRCE_SYS<>7:
CI: 
Logic: WHEN CCR_CNTRACT_TYP = 'CI' THEN _CNTRACT_MATURITY_DTE
To derive _CNTRACT_MATURITY_DTE, please follow below.
For Branch: 
CURRENT_CNTRACT_MATURITY_DTE is not null then CURRENT_CNTRACT_MATURITY_DTE from ACCOUNT_PERIODIC_BRANCH_ACC table for the reporting period.
ELSE
CNTRACT_MATURITY_DTE is not null then CNTRACT_MATURITY_DTE from ENT_ACCOUNT_PERIODIC_FACT table for the reporting period.
ELSE
CNTRACT_MATURITY_DTE is not null then CNTRACT_MATURITY_DTE from ACCOUNT_PERIODIC_BRANCH_ACC table for the reporting period.

For Loan: 
Latest available CNTRACT_MATURITY_DTE from ENT_ACCOUNT_PERIODIC_FACT table
ELSE
Latest available CNTRACT_MATURITY_DTE from ACCOUNT_PERIODIC_LOAN_ACC
For EEBS: 
Latest available CNTRACT_MATURITY_DTE from ENT_ACCOUNT_PERIODIC_FACT table
ELSE
Latest available CNTRACT_MATURITY_DTE from ACCOUNT_PERIODIC_MORTGAGE


CN:
WHEN CCR_CNTRACT_TYP = 'CN' AND EAFP.SRCE_PROD_CDE_FK =45001 THEN _CNTRACT_MATURITY_DTE
WHEN CCR_CNTRACT_TYP = 'CN' AND NAPS.BOOKED_CDE = ‘B’ and NAPS.TEMP_LIMIT_EXPIRY_DTE IS NOT NULL AND NAPS.TEMP_LIMIT_EXPIRY_DTE  > Reporting Period Date THEN NAPS.TEMP_LIMIT_EXPIRY_DTE
-- Note: If value is not populated for CNTRACT_MATURITY_DTE for current period then track back through previous period until a populated value is found and get that value.
FOR SRCE_SYS=7:
CASE WHEN EA5.SRCE_SYS=7 AND CCR_CNTRACT_TYP = 'CI'
 THEN COALESCE(APBM.CNTRACT_MATURITY_DTE,
  _MATURITY_DTE_T)
 ELSE _MATURITY_DTE_T END AS MATURITY_DTE
Note: _MATURITY_DTE_T is the value derived from the above section 'FOR SRCE_SYS<>7'"			DDEWV01P	"ACCOUNT_PERIODIC_BRANCH_ACC
ACCOUNT_PERIODIC_LOAN_ACC
ACCOUNT_PERIODIC_MORTGAGE
ACCOUNT_PERIODIC_BANKMASTER
SUB_NAPS_PRODUCT
APPLICATION_ACCOUNT_REL"		CNTRACT_MATURITY_DTE				
	CCR_ACCOUNT_PERIODIC		DR_INT_CAT	VARCHAR(50)			"CI:
For the driving set of accounts calculate the term based on the below:

_TERM= _END_DATE - _START_DATE
_START_DATE= For an account, pick the First day of the Earliest Period where COALESCE(FXD_INT_END_DTE,NXT_REPRICING_DTE,ACC_SETUP_DTE) is the same as the current value for the reporting period.
_END_DATE:
,CASE WHEN EAPF.FXD_INT_END_DTE < 1170630 THEN NULL ELSE EAPF.FXD_INT_END_DTE END AS FIXED_INT_END_IRT
,CASE WHEN EAPF.NXT_REPRICING_DTE < 1170630 THEN NULL ELSE EAPF.NXT_REPRICING_DTE END AS NXT_REPRICING_DTE_IRT
,CASE WHEN PREF.MATURITY_DTE < 1170630 THEN NULL ELSE PREF.MATURITY_DTE END AS CNTRACT_MATURITY_DTE_IRT
,COALESCE(FIXED_INT_END_IRT,NXT_REPRICING_DTE_IRT,CNTRACT_MATURITY_DTE_IRT) AS FIXED_END_DATE

-- Please refer 'DR_INT_CAT Updated' for the code for start and end date
  
Then use the below logic to find the DR INT CAT

CASE WHEN PREF.CCR_CNTRACT_TYP = 'CN' THEN 'VARIABLE'
                                                                WHEN EAPF.SRCE_SYS=1359 THEN  --BRANCH ACCOUNTING
                                                                        CASE WHEN EAPF.DR_INT_CAT IN (22,55) THEN 'TRACKER'
                                                                                    WHEN EAPF.SRCE_PROD_CDE_FK <> 56001 AND EAPF.DR_INT_CAT IN (21,47,48,52,51) THEN 'FIXED'
                                                                                    ELSE 'VARIABLE' END
                                                                WHEN EAPF.SRCE_SYS=6 THEN             -- LOAN ACCOUNTING
                                                                        CASE WHEN EAPF.DR_INT_CAT IN (6,14,17,19,90) THEN 'TRACKER'
                                                                                     WHEN EAPF.SRCE_PROD_CDE_FK <> 56001 AND (EAPF.DR_INT_CAT IN (21,47,48,52) OR (UPPER(PREF.PROD_SUMM_DESCR) = 'ASSET FINANCE' AND UPPER(PREF.PROD_NAME) NOT LIKE '%VARIABLE%')) THEN 'FIXED'
                                                                                     ELSE 'VARIABLE' END
                                                                WHEN EAPF.SRCE_SYS=85 THEN           -- EBS
                                                                        CASE WHEN EAPF.FUNDING_CATEGORY='TRACKER RATE LENDING' THEN 'TRACKER'
                                                                                    WHEN EAPF.FUNDING_CATEGORY='FIXED RATE LENDING' THEN 'FIXED'
                                                                                    ELSE 'VARIABLE' END
                                                                ELSE 'VARIABLE' END AS INT_TYP


--Next calc the duration based on first part
CASE WHEN INT_TYP = 'FIXED' THEN
CASE WHEN FIXED_END_DATE - FIXED_START <= 365 THEN 'FIXED RATE UP TO 1 YEAR'
WHEN FIXED_END_DATE - FIXED_START BETWEEN 366 AND 1825 THEN 'FIXED RATE BETWEEN 1 AND 5 YEARS'
WHEN FIXED_END_DATE - FIXED_START >1825 THEN 'FIXED RATE GREATER THAN 5 YEARS'
ELSE  'FIXED RATE UP TO 1 YEAR' END
ELSE INT_TYP END AS INTEREST_RATE_TYPE

CN: Variable"			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT	ENT_ACCOUNT_PERIODIC_FACT	DR_INT_CAT	FUNDING_CATEGORY			
	CCR_ACCOUNT_PERIODIC		DR_INT_RTE	DECIMAL(12,5)			"CASE 
WHEN SRCE_INST=9 AND SRCE_SYS=60 THEN '6.7' 
ELSE DR_INT_RTE 
END"			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		DR_INT_RTE				
1.7	CCR_ACCOUNT_PERIODIC	CI 32 First Payment Date	FIRST_TRANS_DTE	DATE FORMAT 'YYYY-MM-DD'			"CI:
Branch Accounting:
Step 1: Go to SUB_NAPS_PRODUCT_TABLE and take FIRST_REPAYMT_DTE for latest booked application on the account (BOOKED_CDE = ‘B’) prior to the Reporting Period
Step 2: For remaining accounts, go to DDEWV01P.TRANSACTION_DETAIL and find the POSTED_DTE for the first positive POSTED_AMT
Loan Accounting :
Take FIRST_LN_PMT_DTE from DDEWV01P.ACCOUNT_PERIODIC_LOAN_ACC
EBS:
Set to the 1st of the month after account setup(ACC_SETUP_DTE) ( First day of the next month after the account was setup)
CN:
Leave Null

V1.7: Additional logic added to ensure FIRST_TRANS_DTE cannot be before ACC_SETUP_DTE (applied to script SQCP0L50)"			"DDEWV50P
DDEWV01P"	"SUB_NAPS_PRODUCT
TRANSACTION_DETAIL
ACCOUNT_PERIODIC_LOAN_ACC
SUB_NAPS_PRODUCT
APPLICATION_ACCOUNT_REL"		"FIRST_REPAYMT_DTE
POSTED_DTE
FIRST_LN_PMT_DTE
~ACC_SETUP_DTE"				
	CCR_ACCOUNT_PERIODIC		ISO_CRNCY_CDE	CHAR(3)			direct transfer			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		ISO_CRNCY_CDE				
	CCR_ACCOUNT_PERIODIC		CCR_Restructure_EVENT	CHAR(15)	Y		"if (EADF.reported_grd in (""3B"",""4"",""7"",""8"" ) and EADF.bsl_days_past_due > 30) or (EADF.marp_ind is not null or blank)
then use Restructure Evt Logic tab
otherwise leave this field blank
Note: This columns is temporarily updated to 'NOT APPLICABLE'. An updated statement to be added in the final step to update to 'NOT APPLICABLE'

V1.7:
Use Restructure Evt Logic tab otherwise default to 'NOT APPLICABLE'"			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		"reported_grd
bsl_days_past_du
marp_ind"				
	CCR_ACCOUNT_PERIODIC		RPMT_DUE_AMT_E	DECIMAL(18,2)	Y		"CI:
Step 1: Check if the EAPF.CLOSE_DTE is Null
Step 2:
If Step 1 is true then process the below logic.
_RPMT_DUE_AMT_E

To calculate _RPMT_DUE_AMT_E: 
Pick the latest availble RPMT_DUE_AMT_E 
from ENT_ACCOUNT_BAL_CONV_PERIODIC EABC
INNER JOIN 
ENT_ACCOUNT_PERIODIC_FACT EAPF
ON 
EABC.WH_ACC_NO=EAPF.WH_ACC_NO
AND 
EABC.PERIOD_DTE=EAPF.PERIOD_DTE
WHERE 
EAPC.RPMT_DUE_AMT_E > 0
AND 
NEXT_RPMT_DTE IS NOT NULL
QUALIFY ROW_NUMBER() OVER(PARTITION BY WH_ACC_NO ORDER BY PERIOD_DTE DESC)=1

Step 2: If no value is derived from step 1 for _RPMT_DUE_AMT_E then follow the below
to calculate _RPMT_DUE_AMT_E: 
Pick the latest availble RPMT_DUE_AMT_E 
from ENT_ACCOUNT_BAL_CONV_PERIODIC EABC
INNER JOIN 
ENT_ACCOUNT_PERIODIC_FACT EAPF
ON 
EABC.WH_ACC_NO=EAPF.WH_ACC_NO
AND 
EABC.PERIOD_DTE=EAPF.PERIOD_DTE
WHERE 
EAPC.RPMT_DUE_AMT_E > 0
QUALIFY ROW_NUMBER() OVER(PARTITION BY WH_ACC_NO ORDER BY PERIOD_DTE DESC)=1
CN: Leave Null
Missing Accounts: Set value as 0"			DDEWV50P	ENT_ACCOUNT_BAL_CONV_PERIODIC		RPMT_DUE_AMT_E                				"Populate with RPMT_DUE_AMT_E for CI Only 
This is for open account/active contracts
Leave NULL for CN"
	CCR_ACCOUNT_PERIODIC		TOT_ARREARS_AMT_E	DECIMAL(18,2)			"This logic is applicable for SRCE_INST=1
CI:
TOT_ARREARS_AMT_E
CN: Leave Null
-- Note: Use the derived PMT_FREQ to populate Arrears
Missing Accounts: Set value as 0"	"This logic is applicable for SRCE_INST=9
CI:
CASE 
WHEN REPAYMT_FREQ_CDE='M' AND GROSS_ARREARS  <= 0 THEN 0
WHEN REPAYMT_FREQ_CDE='M' AND GROSS_ARREARS  > 0 THEN GROSS_ARREARS* -1
ELSE NULL
CN: Leave Null

-- Note: Use the derived PMT_FREQ to populate Arrears
Missing Accounts: Set value as 0"		"DDEWV50P
DDBWV01P"	ENT_ACCOUNT_BAL_CONV_PERIODIC	ACCOUNT_PERIODIC_MORTGAGE	TOT_ARREARS_AMT_E	GROSS_ARREARS			
	CCR_ACCOUNT_PERIODIC		CR_PORTF_FK	VARCHAR(50)			direct mapping			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		CR_PORTF_FK				
1.7	CCR_ACCOUNT_PERIODIC		NEXT_RPMT_DTE	DATE FORMAT 'YYYY-MM-DD'	Y		"CI:
Step 1: Check if the EAPF.CLOSE_DTE is Null
Step 2:
If Step 1 is true then process the below logic.
if EAPF.REPORTED_GRD = '8' or EAPF.CNTRACT_MATURITY_DTE < START OF CURRENT PERIOD ( 01-MM-YYYY) then leave Null else 
ELSE 
_NEXT_RPMT_DTE

To calculate _NEXT_RPMT_DTE: 
Step 1
Pick the latest availble NEXT_RPMT_DTE
from ENT_ACCOUNT_BAL_CONV_PERIODIC EABC
INNER JOIN 
ENT_ACCOUNT_PERIODIC_FACT EAPF
ON 
EABC.WH_ACC_NO=EAPF.WH_ACC_NO
AND 
EABC.PERIOD_DTE=EAPF.PERIOD_DTE
WHERE 
EAPC.RPMT_DUE_AMT_E > 0
AND 
NEXT_RPMT_DTE IS NOT NULL
QUALIFY ROW_NUMBER() OVER(PARTITION BY WH_ACC_NO ORDER BY PERIOD_DTE DESC)=1
Step 2: 
If the derived _NEXT_RPMT_DTE is > MATURITY_DTE    then set the MATURITY_DTE as _NEXT_RPMT_DTE

For the accounts in DDBWV01P.ACCOUNT_PERIODIC_MORTGAGE      
WHERE                          
DUE_DTE_FLAG='1' then set the value as the 7th calendar day of the next month
For example is reporting August end data then the value will be 07-09-20201

CN: LEAVE Null
Missing Accounts: Set value as Null

v1.7: Add line AND CLOSE_DTE IS NULL, i.e.
 ,CASE
 WHEN EA5.SRCE_INST=9 AND A6P.DUE_DTE_FLAG='1'
 AND CLOSE_DTE IS NULL       
 THEN DTE.TO_DTE+7
 WHEN EA5.CLOSE_DTE IS NULL        AND   CCR_CNTRACT_TYP='CI'
 THEN CASE
 WHEN REPORTED_GRD = '8'           OR
 EA5.CNTRACT_MATURITY_DTE    <  DTE.FROM_DTE
 THEN NULL
 ELSE
 CASE WHEN RD1.NEXT_RPMT_DTE >  MATURITY_DTE
 THEN MATURITY_DTE
 ELSE RD1.NEXT_RPMT_DTE

Updated in L51, 
WHEN CCR_CNTRACT_TYP = 'CI' AND CNTRCT_PHASE = 'ACTIVE' 
 AND NEXT_RPMT_DTE IS NULL                          
THEN MATURITY_DTE                                       
ELSE NEXT_RPMT_DTE                                      
 END
 END
 WHEN CCR_CNTRACT_TYP = 'CN' THEN  NULL
 ELSE NULL
 END NEXT_RPMT_DTE_DE
Check WriteOffs Sheet for additional logic"			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		NEXT_RPMT_DTE				
	CCR_ACCOUNT_PERIODIC		CCR_FIN_AMT_AFTER_CONV	DECIMAL(18,2)			"CI :
Take On and On going Logic: 
If CURR_MTH.FIN_AMT_BF_CONV <> PREV_MTH.FIN_AMT_BF_CONV THEN CURR_MTH.FIN_AMT*SAP_FX_DAILY_RTE 
ELSE PREV_MTH.FIN_AMT_AF_CONV
-- In the above SAP_FX_DAILY_RTE should be for that reporting month.
-- Refer 'SAP FX RATE' sheet for sample code.
To calculate the FIN_AMT for the driving set of accounts based on the Extract logic, 
we have to go to the Account Periodic Branch Acc Table to get the Limit. And the Term should be picked at that Period when we pick the Limit. Detailed logic is explained below.
Step 1: For the driving set of accounts, check between the current and the previous month for all the peiord data that is available in  Account Periodic Branch Acc and set a qualifying flag if it satisfies the below condition.
Branch: 
CASE WHEN (A.CNTRACT_MATURITY_DTE - B.CNTRACT_MATURITY_DTE > 30 OR (A.CNTRACT_MATURITY_DTE IS NOT NULL AND B.CNTRACT_MATURITY_DTE IS NULL)  --Contract Maturity increases or becomes populated
OR (A.NET_LIMIT_FOR_PRNCPL < B.NET_LIMIT_FOR_PRNCPL *1.05 AND A.NET_LIMIT_FOR_PRNCPL < B.BAL_AMT)  --Limit increases
OR (A.NET_LIMIT_FOR_PRNCPL IS NOT NULL AND B.NET_LIMIT_FOR_PRNCPL IS NULL AND A.NET_LIMIT_FOR_PRNCPL < B.BAL_AMT)  --Limit becomes populated
AND NET_LIMIT_FOR_PRNCPL < 0 AND NET_LIMIT_FOR_PRNCPL IS NOT NULL THEN 1 ELSE 0 END AS NEW_CONTRACT_FLAG  -- All identified records will have LIMIT populated
Loan: 
CASE WHEN (A.CNTRACT_MATURITY_DTE - B.CNTRACT_MATURITY_DTE > 30 OR (A.CNTRACT_MATURITY_DTE IS NOT NULL AND B.CNTRACT_MATURITY_DTE IS NULL)  --Contract Maturity increases or becomes populated
OR (A.FINANCE_TO_DTE_AMT < B.FINANCE_TO_DTE_AMT)  --Limit increases
OR (A.FINANCE_TO_DTE_AMT IS NOT NULL AND B.FINANCE_TO_DTE_AMT IS NULL))  --Limit becomes populated
AND A.FINANCE_TO_DTE_AMT < 0 AND A.FINANCE_TO_DTE_AMT IS NOT NULL THEN 1 ELSE 0 END AS NEW_CONTRACT_FLAG

EBS: 
 CASE WHEN (A.CNTRACT_MATURITY_DTE - B.CNTRACT_MATURITY_DTE > 30 OR (A.CNTRACT_MATURITY_DTE IS NOT NULL AND B.CNTRACT_MATURITY_DTE IS NULL)  --Contract Maturity increases or becomes populated
OR (A.ORIG_LOAN_AMT < B.ORIG_LOAN_AMT)  --Limit increases
OR (A.ORIG_LOAN_AMT IS NOT NULL AND B.ORIG_LOAN_AMT IS NULL))  --Limit becomes populated
AND A.ORIG_LOAN_AMT < 0 AND A.ORIG_LOAN_AMT IS NOT NULL THEN 1 ELSE 0 END AS NEW_CONTRACT_FLAG  -- All identified records will have LIMIT populated

Step 2: 
For any accounts which have no new credit agreements, go back to the first populated record for that account where Limit < 0 and not null
Step 3: 
Financed Amount doesn’t depend on Term being populated anymore – but Term does depend on Financed Amount. Take the value for Term at the same point in time where you take the value for Financed Amount. If Term is null at this time, then Number of Planned Payments will be reported to CCR as null
CN: 
Step 1: 
For Accounts where NET_LIMIT_FOR_PRNCPL <= -500, pick it from EADF.NET_LIMIT_FOR_PRNCPL
Step 2: 
CN Overdrafts:
Refer to CCR_CREDIT_STATUS field for this (On a side Note, NET_LIMIT_FOR_PRNCPL column for Branch has to be picked as at the reporting period, FINANCE_TO_DTE_AMT column for Loan has to be picked as at he reporting period), ORIG_LOAN_AMT column for EBS has to be picked as at he reporting period)
Note: The amount value should be reported as a positive value
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"			"DDEWV01P
DDBWV01P
DDEWV50P"	"ACCOUNT_PERIODIC_BRANCH_ACC
ACCOUNT_PERIODIC_LOAN_ACC"	ACCOUNT_PERIODIC_MORTGAGE	"Branch:
CNTRACT_MATURITY_DTE
NET_LIMIT_FOR_PRNCPL
TERM_NO_UNITS
PERIOD_DTE
Loan:
CNTRACT_MATURITY_DTE
FINANCE_TO_DTE_AMT
TERM_NO_UNITS
PERIOD_DTE"	"CNTRACT_MATURITY_DTE
ORIG_LOAN_AMT
ORIGINAL_TERM
PERIOD_DTE"			See FINANCE AMOUNT tab for More details
	CCR_ACCOUNT_PERIODIC		NET_LIMIT_FOR_PRNCPL_E	DECIMAL(18,2)			direct mapping				ENT_ACCOUNT_BAL_CONV_PERIODIC		NET_LIMIT_FOR_PRNCPL_E				
	CCR_ACCOUNT_PERIODIC	CI36 Outstanding Balance	BAL_AMOUNT_E	DECIMAL(18,2)			"CASE 
 WHEN  SRCE_INST=9 AND SECUR_POOL_ID=203 AND APM.BAL_AMT > 0 THEN (APM.BAL_AMT*-1)
 WHEN  SRCE_INST=9 AND SECUR_POOL_ID=203 AND APM.BAL_AMT < = 0 THEN 0
WHEN EA5.SRCE_SYS = 6                              
AND EA5.SRCE_PROD_CDE_FK LIKE ANY ('111%' , '121%')
THEN BC5.BAL_FOR_INT_E
 ELSE EABC.NET_BAL_FOR_PRNCPL_E
 END 
AS BAL_AMOUNT_E
Missing Accounts: Set value as 0
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"			"DDEWV50P
DDBWV01P"	"ENT_ACCOUNT_BAL_CONV_PERIODIC
ACCOUNT_PERIODIC_MORTGAGE"		"NET_BAL_FOR_PRNCPL_E
BAL_AMT"				
1.7	CCR_ACCOUNT_PERIODIC		OUTSTDG_BAL_AMT	DECIMAL(18,2)	Y		"CASE
WHEN CLOSE_DTE IS NOT NULL THEN NULL 
 WHEN _BAL_AMOUNT_E >=0 THEN 0 /*updated for v1.7*/
 WHEN _BAL_AMOUNT_E <0 THEN ABS(_BAL_AMOUNT_E)
 END
_BAL_AMOUNT_E = BAL_AMOUNT_E (Derived in Line 32)

-- Please refer to OS BAL & Payment NO sheet for code
Missing Accounts: Set value as 0
Check WriteOffs Sheet for additional logic as part of DEA-10203
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"										
1.7	CCR_ACCOUNT_PERIODIC		OUTSTDG_PMT_NO	INTEGER	Y		"CASE                                                                  
 WHEN PMT_FREQ NOT IN                                                 
 ('WEEKLY', 'DAILY', 'FORTNIGHTLY', 'FOUR WEEKLY', 'MONTHLY','HALF YEA
   RLY','BULLET','YEARLY','QUARTERLY')                                
 AND OUTSTDG_BAL_AMT>=0 THEN 0                                        
 WHEN PMT_FREQ NOT IN                                                 
 ('WEEKLY', 'DAILY', 'FORTNIGHTLY', 'FOUR WEEKLY', 'MONTHLY','HALF YEA
   RLY','BULLET','YEARLY','QUARTERLY')                                
 AND OUTSTDG_BAL_AMT IS NULL THEN NULL                                
 WHEN PMT_FREQ IN                                                     
 ('WEEKLY', 'DAILY', 'FORTNIGHTLY', 'FOUR WEEKLY', 'MONTHLY','HALF YEA
   RLY','BULLET','YEARLY','QUARTERLY')                                
 THEN                                                                 
   CASE                                                               
    WHEN OUTSTDG_BAL_AMT >=0                                          
    AND (RPMT_DUE_AMT_CRIF IS NULL                                   
      OR FLOOR(RPMT_DUE_AMT_CRIF)=0) THEN 0                          
      WHEN OUTSTDG_BAL_AMT=0 THEN 0                                  
      WHEN OUTSTDG_BAL_AMT >0 AND RPMT_DUE_AMT_CRIF >0               
      THEN                                                           
       CASE WHEN FLOOR(OUTSTDG_BAL_AMT/RPMT_DUE_AMT_CRIF) <=9999     
       THEN CAST(FLOOR(OUTSTDG_BAL_AMT/RPMT_DUE_AMT_CRIF) AS INTEGER)
        ELSE 9999                                                    
       END                                                           
     ELSE NULL                                                       
    END                                                              
  ELSE 0                                                             
 END  AS OUTSTDG_PMT_NO  

CASE
   WHEN PMT_FREQ NOT IN    ('WEEKLY', 'DAILY', 'FORTNIGHTLY', 'FOUR WEEKLY', 'MONTHLY',      'HALF YEARLY','BULLET','YEARLY','QUARTERLY')
   AND OUTSTDG_BAL_AMT>0 THEN 1     --- v1.7 update
   WHEN PMT_FREQ NOT IN    ('WEEKLY', 'DAILY', 'FORTNIGHTLY', 'FOUR WEEKLY', 'MONTHLY',
     'HALF YEARLY','BULLET','YEARLY','QUARTERLY')
   AND (OUTSTDG_BAL_AMT IS NULL OR OUTSTDG_BAL_AMT = 0) THEN 0   --- v1.7 update
   WHEN PMT_FREQ IN    ('WEEKLY', 'DAILY', 'FORTNIGHTLY', 'FOUR WEEKLY', 'MONTHLY',   'HALF YEARLY','BULLET','YEARLY','QUARTERLY')
   THEN
     CASE
      WHEN OUTSTDG_BAL_AMT >=0       AND (RPMT_DUE_AMT_CRIF IS NULL       OR Floor(RPMT_DUE_AMT_CRIF)=0) THEN 0
      WHEN OUTSTDG_BAL_AMT=0 THEN 0
      WHEN OUTSTDG_BAL_AMT >0 AND RPMT_DUE_AMT_CRIF >0
      THEN
       CASE WHEN Floor(OUTSTDG_BAL_AMT/RPMT_DUE_AMT_CRIF) <=9999
       THEN Cast(Floor(OUTSTDG_BAL_AMT/RPMT_DUE_AMT_CRIF) AS INTEGER)
        ELSE 9999
       END
     ELSE 0    ------ v1.7 update (active accounts with null fail with crif)
    END
  ELSE 0
 END  AS OUTSTDG_PMT_NO

-- Please refer to OS BAL & Payment NO sheet for code
Missing Accounts: Set value as 0
Check WriteOffs Sheet for additional logic as part of DEA-10203"										
	CCR_ACCOUNT_PERIODIC		AMT_PSTD	DECIMAL(18,2)	Y		"CASE WHEN _TOT_ARREARS_AMT_E> =0 
OR PMT_FREQ IN ('OTHERS', 'IRREGULAR INSTALMENTS')
THEN NULL  ELSE ABS(_TOT_ARREARS_AMT_E)
END
-- Please refer to OS BAL & Payment NO sheet for code
Check WriteOffs Sheet for additional logic as part of DEA-10203"										may not need update
	CCR_ACCOUNT_PERIODIC		NO_OF_PMT_PSTD	INTEGER	Y		"""CASE 
WHEN PMT_FREQ IN ('MONTHLY')THEN  
 CASE  WHEN  AMT_PSTD  >0 AND (RPMT_DUE_AMT_CRIF IS NULL OR CAST(RPMT_DUE_AMT_CRIF AS INTEGER)=0) THEN 0
    WHEN  AMT_PSTD >0 AND RPMT_DUE_AMT_CRIF >0 THEN 
   CASE WHEN CAST((AMT_PSTD /RPMT_DUE_AMT_CRIF) - 1  AS INTEGER) <=999 THEN CASE WHEN ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER)))  > 1 THEN CAST(FLOOR(ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER))) - 1)   AS INTEGER) ELSE 0 END
            ELSE 999 END ELSE NULL END

WHEN PMT_FREQ IN ('WEEKLY')THEN                                                                                                                         
 CASE  WHEN  AMT_PSTD  >0 AND (RPMT_DUE_AMT_CRIF IS NULL OR CAST(RPMT_DUE_AMT_CRIF AS INTEGER)=0) THEN 0
    WHEN  AMT_PSTD >0 AND RPMT_DUE_AMT_CRIF >0 THEN 
   CASE WHEN CAST((AMT_PSTD /(RPMT_DUE_AMT_CRIF*4.30)) - 1 AS INTEGER) <=999 THEN CASE WHEN ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER)* 4.30))  > 1 THEN CAST(FLOOR(ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER)* 4.30)) - 1)   AS INTEGER) ELSE 0 END 
            ELSE 999 END ELSE NULL END

WHEN PMT_FREQ IN ('FORTNIGHTLY') THEN                                                                                                                             
 CASE WHEN  AMT_PSTD  >0 AND (RPMT_DUE_AMT_CRIF IS NULL OR CAST(RPMT_DUE_AMT_CRIF AS INTEGER)=0) THEN 0
   WHEN  AMT_PSTD >0 AND RPMT_DUE_AMT_CRIF >0 THEN                                           
   CASE WHEN CAST((AMT_PSTD /(RPMT_DUE_AMT_CRIF*2.15)) - 1 AS INTEGER) <=999 THEN CASE WHEN ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER)* 2.15))  > 1 THEN CAST(FLOOR(ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER)* 2.15)) - 1)   AS INTEGER) ELSE 0 END
            ELSE 999 END ELSE NULL END
                                                                                                                                
WHEN PMT_FREQ IN ('HALF YEARLY', 'BULLET', 'YEARLY', 'QUARTERLY')THEN  
 CASE  WHEN  AMT_PSTD  >0 AND (RPMT_DUE_AMT_CRIF IS NULL OR CAST(RPMT_DUE_AMT_CRIF AS INTEGER)=0) THEN 0
    WHEN  AMT_PSTD >0 AND RPMT_DUE_AMT_CRIF >0 THEN 
   CASE WHEN CAST((AMT_PSTD /RPMT_DUE_AMT_CRIF)  AS INTEGER) <=999 THEN CASE WHEN ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER)))  >= 1 THEN CAST(FLOOR(ABS(CAST(AMT_PSTD AS INTEGER)/(CAST(RPMT_DUE_AMT_CRIF AS INTEGER))))   AS INTEGER) ELSE 0 END
            ELSE 999 END ELSE NULL END
WHEN  AMT_PSTD  >0 AND PMT_FREQ IS NULL THEN 0
ELSE NULL END                                                                                                                
AS NO_OF_PMT_PSTD_TMP
,CASE  WHEN CCR_CNTRACT_TYP ='CI' AND AMT_PSTD >= 0.5 AND                  
  (Trim(NO_OF_PMT_PSTD_TMP) IS NULL OR Trim(NO_OF_PMT_PSTD_TMP) = '') AND MATURITY_DTE <= CCR_REPORTED_DTE 
  THEN 1                                                                
WHEN CCR_CNTRACT_TYP ='CI' AND (AMT_PSTD < 0.5 OR  AMT_PSTD IS NULL) AND  
  NO_OF_PMT_PSTD_TMP = 999 AND MATURITY_DTE > CCR_REPORTED_DTE          
  THEN NULL                                                             
ELSE NO_OF_PMT_PSTD_TMP                                                
END                                                                    
AS NO_OF_PMT_PSTD
Check WriteOffs Sheet for additional logic as part of DEA-10203""	"										
	CCR_ACCOUNT_PERIODIC		RPMT_DUE_AMT_CRIF	DECIMAL(18,2)	Y		"CASE
WHEN OUTSTDG_BAL_AMT IS NULL AND AMT_PSTD IS NULL THEN NULL 
WHEN CLOSE_DTE IS NOT NULL THEN NULL 
WHEN EAPF.REPORTED_GRD = '8'  AND _RPMT_DUE_AMT_E IS NOT NULL THEN 
                         CASE WHEN OUTSTDG_BAL_AMT <_RPMT_DUE_AMT_E THEN OUTSTDG_BAL_AMT AS INTEGER) ELSE CAST(_RPMT_DUE_AMT_E AS INTEGER) 
WHEN EAPF.REPORTED_GRD = '8' AND  A._RPMT_DUE_AMT_E IS NULL 
THEN CAST(OUTSTDG_BAL_AMT AS INTEGER)  
ELSE CAST(_RPMT_DUE_AMT_E AS INTEGER) END
-- Please refer to OS BAL & Payment NO sheet for code
Missing Accounts: Set value as 0
Check WriteOffs Sheet for additional logic as part of DEA-10203

Last updates in L51
WHEN CCR_CNTRACT_TYP = 'CI'                              
AND  CNTRCT_PHASE IN ('CLOSED','CLOSED IN ADVANCE') 
THEN 0                                                   
WHEN CCR_CNTRACT_TYP = 'CI' AND CNTRCT_PHASE = 'ACTIVE'  
AND RPMT_DUE_AMT_CRIF IS NULL                        
THEN OUTSTDG_BAL_AMT                                     
ELSE RPMT_DUE_AMT_CRIF                                   "										
	CCR_ACCOUNT_PERIODIC		CCR_IND	CHAR(1)			'C'						NET_LIMIT_FOR_PRNCPL				Based on finance amount and scope. Check Kierans logic
	CCR_ACCOUNT_PERIODIC		CLOSE_DTE	DATE FORMAT 'YYYY-MM-DD'			"direct mapping
Missing Accounts: Last Calender date of the reporting month
Check WriteOffs Sheet for additional logic as part of DEA-10203
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"				ENT_ACCOUNT_PERIODIC_FACT		CLOSE_DTE				
	CCR_ACCOUNT_PERIODIC		CNTRCT_PHASE	CHAR(2)			"CI & CN:
STEP 1:
CASE 
WHEN EAPF.CLOSE_DTE  IS NULL THEN 'ACTIVE'
WHEN EAPF.CLOSE_DTE  IS NOT NULL  AND EAPF.CLOSE_DTE  < CNTRACT_MATURITY_DTE THEN 'CLOSED IN ADVANCE'
WHEN EAPF.CLOSE_DTE  IS NOT NULL THEN 'CLOSED'
END AS CNTRCT_PHASE
Missing Accounts: Set the value as 'CLOSED'
Check WriteOffs Sheet for additional logic as part of DEA-10203
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"				ENT_ACCOUNT_PERIODIC_FACT		CLOSE_DTE	CLOSE_DTE			
	CCR_ACCOUNT_PERIODIC		CCR_PROVIDER_CDE	CHAR(50)			See CCR_PROVIDER_CDE tab										"was legal entity
Link co_cde_fk from EAPF to Org Pc Dim .. Find the Parent
Then look at   Org Pc Dim for that Grand Parent 
Then look at   Org Pc Dim for that Great Grand Parent
Then look at   Org Pc Dim for that Great Great Grand Parent
Then look at   Org Pc Dim for that Great Great Great Grand Parent
Then look at   Org Pc Dim for that Great Great Great Great Grand Parent
Etc
Then case statement at the top decides when does the legal entity equates to AIB plc, AIB mortgae bank etc ..
"
	CCR_ACCOUNT_PERIODIC		PROD_NAME	VARCHAR(50)			direct transfer			"DDEWV01P
DDBWV01P"	PRODUCT_REFERENCE		PROD_NAME				
	CCR_ACCOUNT_PERIODIC		PROD_SUMM_DESCR	VARCHAR(50)			direct transfer			"DDEWV01P
DDBWV01P"	ENT_ACCOUNT_DAILY_FACT		PROD_SUMM_DESCR				
	CCR_ACCOUNT_PERIODIC		PROD_GRP_DESCR	VARCHAR(50)			"direct transfer

"			"DDEWV01P
DDBWV01P"	PRODUCT_REFERENCE		PROD_GRP_DESCR				INTEREST CATEGORY
	CCR_ACCOUNT_PERIODIC		FIX_FLOATING_IND	CHAR(1)			direct transfer				ENT_ACCOUNT_PERIODIC_FACT		FIX_FLOATING_IND				
	CCR_ACCOUNT_PERIODIC	CI11 Credit Status	CCR_CREDIT_STATUS	VARCHAR(50)			"CI: 
For Open accounts we could have three Status. 
1. Legal
2. Repossession
3. Voluntary Surrender
For Closed accounts we could have two status. 
Repossession & Voluntary surrender will be set only when it was set for the first time in the reporting month for that account. But for Legal it could have been set in the reporting month or in the previous months and it continued to be Legal untill the reporting month.
1. Settlement
2. Write-off
3. If it is not Settlement or Write-off, then we need to populate the most recent status when it was open.

As part of REGR-228
After the above checks are done additional checks is done for Loan accounts from CACS
Based on the data from SUB_PERIODIC_LOAN_ACC and column CLOSURE_CODE as below to be derived as credit status and the value from CACS takes higher precedence than anything calculated earlier
CLOSURE_CODE ='W' THEN 'WRITE OFF' 
CLOSURE_CODE ='L' THEN 'SETTLEMENT'
CLOSURE_CODE ='C' THEN 'PAID-OFF'

Please refer to the Credit Status Sheet for pseudo code
CN: 
Overdraft Cancelled is the only status to be set for CN. This will increase the number of Accounts in scope
Step 1: For the accounts based ON the initial EXTRACT criteria, check if the Limit is -1, 0 or null and Balance is in Debit
Step 2: For those accounts in Step 1, take the latest period when they had a Limit and the period when they had a Credit Balance. Of these, now select the ones who never had a credit balance after the Latest valid Limit was set
Step 3: For those accounts in Step 2, go to the transaction details and pick the Curr Ledger Bal based on the Max sequence number for each day
Step 4: For the accounts in Step 3, Find those accounts that were always in debit and then report those accounts with the Limit and Balance as at the reporting period ( June month for the Initial build) 
and the Credit Status as 'Overdraft Cancelled'. 
Please CN Credit status Sheet for the Pseudo Code

In Addition to the above logic below needs to be applied:
If PMT_FREQ = 'IRREGULAR INSTALMENTS' and FLOOR(ABS(TOT_ARREARS_AMT_E))>0 then set value as 'Irregular Instalment Payment Missed'
Note: Overdraft Cancelled is not to be reported for now unitl the Program Teams confirms the new logic
Missing Accounts: Set to 'N/A'
Check WriteOffs Sheet for additional logic as part of DEA-10203
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"	"CI: 
If ‘LOAN_PURPOSE’ in (93, 94, 95, 96) then ‘Voluntary Surrender’. Only report Voluntary Surrender in the month that it first appears on DDBWV01P.ACCOUNT_PERIODIC_MORTGAGE. 
Else 'N/A'
CN: 
For Branch Accounting we referred to the Branch Accounting Periodic to check the when a limit was last ‘on’ and then tracked the transaction history from that time to see if the account ever went in to credit. The same logic applies for Money Manager, but the tables are:

• Periodic: DDBWV01P.ACCOUNT_PERIODIC_SAVINGS
* Limit Field = TEMP_FAC
* Balance Field = BAL_AMT
* Period Field = PERIOD_DTE
• Transactions: DDBWV01P.TRANSACTION_DETAIL_SAVINGS (WHERE CR_DR_CDE = C)
* Value: POSTED_AMT
* Ledger Balance: AFTER_TRANS_BAL_AMT
Missing Accounts: Set to 'N/A'
Check WriteOffs Sheet for additional logic as part of DEA-10203
Check for Overdraft Cancellation Amendment Sheet as part of DEA-12247"		"DDEWV50P
DDEWV01P
DDBWV01P"	"FINANCE_ACCOUNT_EVENT_FACT
Branch:
ASU_CACS_LEGAL_PERIODIC
ACC_MARP_NONMARP_CODE_PERIODIC
TRANSACTION_DETAIL
BA_AMEND_LOG_PARENT_DETAIL
Account_Detail_Daily
Loan:
ACCOUNT_PERIODIC_LOAN_ACC
ACCOUNT_EVENT_LOAN_ACC_TRANS"	ACCOUNT_PERIODIC_MORTGAGE					"With Paul (check with 
Cancelled Overdraft CCR_CONTRACT_TYPE = 'CN' 
AND WH_ACC_NO IN
(SELECT wh_ACC_NO  FROM DDEWV50P.ENT_ACCOUNT_TIMESERIES_FACT
WHERE NET_LIMIT_FOR_PRNCPL = -1
AND bal_amt < 0 )

Look at Last Month and if set to last month then carry forward unless new limit "
	CCR_ACCOUNT_PERIODIC		TERM_NO_UNITS	SMALLINT			"This logic is applicable for SRCE_INST=1 and SRCE_SYS<>7
Refer Step 1 for TOT_NO_OF_PLANNED_PYMTS
For SRCE_SYS=7:
CASE WHEN EA5.SRCE_SYS=7 AND CCR_CNTRACT_TYP = 'CI'
 THEN CAST(((MATURITY_DTE - ACC_SETUP_DTE_DE
            )MONTH(4)) AS INTEGER)
 ELSE TERM_NO_UNITS_T END AS TERM_NO_UNITS_F
Note: ACC_SETUP_DTE_DE is the derived value as part of ACC_SETUP_DTE column in this mapping. MATURITY_DTE is also derived as part of this mapping"	"Note: This logic is applicable for SRCE_INST=9
ORIGINAL_TERM"				DDBWV01P.ACCOUNT_PERIODIC_MORTGAGE					
	CCR_ACCOUNT_PERIODIC		TOT_INSTALMT_PMT_FREQ_CDE	VARCHAR(50)			"This logic is applicable for SRCE_INST=1
Refer Step 1 for PMT_FREQ"	"Note: This logic is applicable for SRCE_INST=9
REPAYMT_FREQ_CDE"				DDBWV01P.ACCOUNT_PERIODIC_MORTGAGE					
	CCR_ACCOUNT_PERIODIC		PMT_FREQ	VARCHAR(15)	Y		"For SRCE_SYS=7:

CASE WHEN EA5.SRCE_SYS=7 THEN
       CASE WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE IN
       ( 'EVERY 10 MONTHS' , 'EVERY 11 MONTHS' , 'EVERY 20 DAYS' ,
         'EVERY 2 MONTHS' , 'EVERY 4 MONTHS' , 'EVERY 4 WEEKS' ,
         'EVERY 7 MONTHS' , 'EVERY 8 MONTHS' , 'EVERY 9 MONTHS',
         'EVERY 5 MONTHS')
       THEN 'IRREGULAR INSTALMENTS'
       WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE ='QUARTERLY'
       THEN 'QUARTERLY'
       WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE ='SEMI_ANNUALLY'
       THEN 'HALF YEARLY'
       WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE ='WEEKLY' THEN  'WEEKLY'
       WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE = 'BULLET' THEN 'BULLET'
       WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE = 'MONTHLY' THEN 'MONTHLY'
       WHEN EA5.TOT_INSTALMT_PMT_FREQ_CDE = 'ANNUALLY' THEN 'YEARLY'
       END
  END AS PMT_FREQ_FLEX
This logic is applicable for SRCE_INST=1 and SRCE_SYS=7
CI ONLY:
STEP 1:
For Branch Accounts only, From Account Periodic branch Acc find the below.
CASE 
 WHEN REPAYMT_UNIT = 2 AND REPAYMT_INTERVAL = 7 THEN 'WEEKLY' 
 WHEN REPAYMT_UNIT = 2 AND REPAYMT_INTERVAL = 14 THEN 'FORTNIGHTLY ' 
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL = 1 THEN 'MONTHLY' 
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL = 3 THEN 'QUARTERLY' 
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL = 6 THEN 'SEMI_ANNUALLY' 
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL = 12 THEN 'ANNUALLY' 
 WHEN REPAYMT_UNIT = 0 AND REPAYMT_INTERVAL =  0 THEN 'NO REPAYMENT DETAILS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  2 THEN 'EVERY 2 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  4 THEN 'EVERY 4 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  5 THEN 'EVERY 5 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  7 THEN 'EVERY 7 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  8 THEN 'EVERY 8 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  9 THEN 'EVERY 9 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  10 THEN 'EVERY 10 MONTHS'
 WHEN REPAYMT_UNIT = 1 AND REPAYMT_INTERVAL =  11 THEN 'EVERY 11 MONTHS'
 WHEN REPAYMT_UNIT = 2 AND REPAYMT_INTERVAL =  20 THEN 'EVERY 20 DAYS'
 WHEN REPAYMT_UNIT = 2 AND REPAYMT_INTERVAL =  28 THEN 'EVERY 4 WEEKS'
 WHEN REPAYMT_UNIT = 7 AND REPAYMT_INTERVAL =  1 THEN 'MONTHLY EXCEPT DECEMBER'
 WHEN REPAYMT_UNIT = 8 AND REPAYMT_INTERVAL =  1 THEN 'MONTHLY EXCEPT JANUARY'
 WHEN REPAYMT_UNIT = 9 AND REPAYMT_INTERVAL =  1 THEN 'MONTHLY EXCEPT DEC AND JAN' 
END AS _TOT_INSTALMT_PMT_FREQ_CDE
If incoming record is not populated with value for _TOT_INSTALMT_PMT_FREQ_CDE for current period then track back through periodic table until a populated value is found and keep as _TOT_INSTALMT_PMT_FREQ_CDE
For Loan Accounts Only:
If incoming record is not populated with value for EAPF.TOT_INSTALMT_PMT_FREQ_CDE for current period then track back through periodic table until a populated value is found and keep as _TOT_INSTALMT_PMT_FREQ_CDE
STEP 2:
Use the _TOT_INSTALMT_PMT_FREQ_CDE value derived in Step 1 in the logic below
For the NAPS related derivation, Use the driving set of accounts and join to the DDEWV01P.SUB_NAPS_PRODUCT table and pick the latest record for that account, based on MAX(BOOKED_TIME). Use DDEWV01P.APPLICATION_ACCOUNT_REL table to join from the driving set of accounts to DDEWV01P.SUB_NAPS_PRODUCT table. 

CASE 
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE = ANNUALLY ) THEN YEARLY
WHEN ((NAPS.REPAYMT_EXCPTN_CDE IN ('ROB','ONE','INT','ADB') AND NAPS.FINAL_BULLET_REPAYMT_AMT > 0) OR REPAYMT_EXCPTN_CDE IN ('ADB','ADO') ) AND NAPS.REPAYMT_FREQ_CDE = 'O'   THEN 'BULLET'
WHEN ((NAPS.REPAYMT_EXCPTN_CDE IN ('ROB','ONE','INT','ADB') AND NAPS.FINAL_BULLET_REPAYMT_AMT > 0) OR REPAYMT_EXCPTN_CDE IN ('ADB','ADO') ) AND NAPS.REPAYMT_FREQ_CDE <> 'O' THEN  'IRREGULAR INSTALMENTS'
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE IN ANY (EVERY 10 MONTHS , EVERY 11 MONTHS,EVERY 20 DAYS, EVERY 2 MONTHS, EVERY 4 MONTHS, EVERY 4 WEEKS , EVERY 7 MONTHS, EVERY 8 MONTHS, EVERY 9 MONTHS) THEN 'IRREGULAR INSTALMENTS'
WHEN SRCE_PROD_CDE_FK IN (55604,55431) OR (SRCE_PROD_CDE_FK IN (55001,55025) AND _cts_flag = 1) THEN 'IRREGULAR INSTALMENTS'
-- For the CTS_Flag column used in the above statement, please refer the code below for 'CTS FLAG Logic'
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE = FORTNIGHTLY ) THEN FORTNIGHTLY
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE LIKE MONTHLY%) THEN MONTHLY
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE IN ANY (NEVER,NO REPAYMENT DETAILS) THEN  OTHER
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE = QUARTERLY ) THEN QUARTERLY
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE = SEMI_ANNUALLY ) THEN HALF YEARLY
WHEN (_TOT_INSTALMT_PMT_FREQ_CDE = WEEKLY ) THEN WEEKLY
ELSE NULL;
CN: SET TO NULL

CTS FLAG LOGIC:
CREATE TABLE DDEWA01P.DW46571_CTS_ACC AS(
SEL A.WH_ACC_NO
  ,MAX(CASE WHEN c.CTS_IND = 'Y' THEN 1 ELSE 0 END) AS CTS_FLAG
FROM (SEL WH_ACC_NO FROM DDEWD01S.CCR_ACCOUNT_PERIODIC_FT2509 WHERE SRCE_PROD_CDE_FK IN (55604,55431,55001,55025)) A
LEFT JOIN DDEWV01P.CUSTOMER_ACCOUNT_REL_PERIODIC B
 ON A.WH_ACC_NO = B.WH_ACC_NO
LEFT JOIN (SEL WH_CUST_NO, CTS_IND FROM DDEWV01P.CUSTOMER_DETAIL_PERIODIC WHERE PERIOD_DTE = 1170630 AND CTS_IND = 'Y') C
 ON B.WH_CUST_NO = C.WH_CUST_NO
GROUP BY 1
)WITH DATA"	"Note: This logic is applicable for SRCE_INST=9

CASE WHEN REPAYMT_FREQ_CDE='M' THEN 'MONTHLY'"			"ACCOUNT_PERIODIC_BRANCH_ACC
ACCOUNT_PERIODIC_LOAN_ACC
SUB_NAPS_PRODUCT
APPLICATION_ACCOUNT_REL"	DDBWV01P.ACCOUNT_PERIODIC_MORTGAGE	"tot_instalmt_pmt_freq_cde
REPAYMT_EXCPTN_CDE
FINAL_BULLET_REPAYMT_AMT
REPAYMT_FREQ_CDE"	REPAYMT_FREQ_CDE			This logic was originally for total_number_of_planned_payments which now has different logic.
1.7	CCR_ACCOUNT_PERIODIC		TOT_NO_OF_PLANNED_PYMTS				"For SRCE_SYS=7

CASE WHEN EA5.SRCE_SYS=7 AND CCR_CNTRACT_TYP='CI'
      THEN CASE EA5.TOT_INSTALMT_PMT_FREQ_CDE
      WHEN 'ANNUALLY' THEN TERM_NO_UNITS_F/12
      WHEN 'EVERY 10 MONTHS' THEN TERM_NO_UNITS_F/10
      WHEN 'MONTHLY EXCEPT DEC AND JAN'
      THEN ((TERM_NO_UNITS_F*12)/10)
      WHEN 'EVERY 2 MONTHS' THEN TERM_NO_UNITS_F/2
      WHEN 'EVERY 4 MONTHS' THEN TERM_NO_UNITS_F/4
      WHEN 'EVERY 4 WEEKS' THEN TERM_NO_UNITS_F*4
      WHEN 'EVERY 7 MONTHS' THEN TERM_NO_UNITS_F/7
      WHEN 'EVERY 8 MONTHS' THEN TERM_NO_UNITS_F/8
      WHEN 'EVERY 9 MONTHS' THEN TERM_NO_UNITS_F/9
      WHEN 'FORTNIGHTLY' THEN TERM_NO_UNITS_F*2.166667
      WHEN 'MONTHLY' THEN TERM_NO_UNITS_F
      WHEN 'MONTHLY EXCEPT DECEMBER'
      THEN ((TERM_NO_UNITS_F*12)/11)
      WHEN 'MONTHLY EXCEPT JANUARY'
      THEN ((TERM_NO_UNITS_F*12)/11)
      WHEN 'QUARTERLY' THEN (TERM_NO_UNITS_F/3)
      WHEN 'SEMI_ANNUALLY' THEN TERM_NO_UNITS_F/6
      WHEN 'WEEKLY' THEN TERM_NO_UNITS_F*4.333333
      WHEN 'BULLET' THEN 1
      ELSE NULL
      END
END AS TOT_NO_OF_PLANNED_PYMTS_FLEX
For SRCE_SYS<>7
CI ONLY:
STEP 1:
If incoming record is not populated with value for _TOT_INSTALMT_PMT_FREQ_CDE for current period then track back through periodic table until a populated value is found.
Use TERM_NO_UNITS derived as part of FIN AMT calculation
STEP 2:
Use the value derived in Step 1 in the logic below
IF (PMT_FREQ IN('IRREGULAR INSTALMENTS','OTHER')) THEN 1
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'ANUALLY' ) THEN _TERM_NO_UNITS/12
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'MONTHLY EXCEPT DEC AND JAN' ) THEN  (_TERM_NO_UNITS*12)/10
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'FORTNIGHTLY' ) THEN _TERM_NO_UNITS*2.166667
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE ='MONTHLY' )THEN  _TERM_NO_UNITS
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'MONTHLY EXCEPT DECEMBER' )THEN   (_TERM_NO_UNITS*12)/11
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'MONTHLY EXCEPT JANUARY' ) THEN  (_TERM_NO_UNITS*12)/11
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'QUARTERLY' )THEN  _TERM_NO_UNITS/3
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'SEMI_ANNUALLY' )THEN  _TERM_NO_UNITS/6
ELSE IF ( _TOT_INSTALMT_PMT_FREQ_CDE = 'WEEKLY' )THEN  _TERM_NO_UNITS*4.333333 
ELSE IF (_TOT_INSTALMT_PMT_FREQ_CDE = 'BULLET ')  THEN  1
ELSE NULL;
;
CN: SET TO NULL

/*UPDATE v1.7 - add to end of case statement*/
WHEN CCR_FIN_AMT_E <> 0 AND RPMT_AMT_DUE_E <> 0
AND CCR_CNTRACT_TYP='CI'
THEN Floor(CCR_FIN_AMT_E / RPMT_DUE_AMT_E_DE)"	"Note: This logic is applicable for SRCE_INST=9
CASE WHEN REPAYMT_FREQ_CDE='M' THEN ORIGINAL_TERM"			"SUB_NAPS_PRODUCT
APPLICATION_ACCOUNT_REL"	ACCOUNT_PERIODIC_MORTGAGE		ORIGINAL_TERM			
1.7	CCR_ACCOUNT_PERIODIC	CI26 Payment Method	PMT_MTHD	VARCHAR(25)			"CI:
Branch Accounting (CI’s only):

CASE 
 WHEN CCR.ACC_NO LIKE ANY ('00930253%', '00930350%', '00930709%') THEN 'DIRECT DEBIT' -- Staff Business
 WHEN CCR.SRCE_SYS=1359 AND CCR_CNTRACT_TYP='CI' AND ADN.NAPS_REPAYMT_MTHD_CDE IN ('I', 'E') THEN 'DIRECT DEBIT' -- Branch
Loan Accounting:
 WHEN REPAYMT_MTHD_CDE IN (3,4) THEN 'DIRECT DEBIT'     -- Loan
 WHEN REPAYMT_MTHD_CDE IN (2) THEN 'STANDING ORDER'    -- Loan
 WHEN REPAYMT_MTHD_CDE IN (1) THEN 'CASH LODGEMENT'   -- Loan
 from DDEWV01P.ACCOUNT_PERIODIC_LOAN_ACC
EBS:
WHEN  D.PAYMENT_METHOD IN ('X', 'I') THEN 'DIRECT DEBIT'     -- EBS
WHEN  D.PAYMENT_METHOD IN ('S') THEN 'STANDING ORDER'   -- EBS
WHEN  D.PAYMENT_METHOD IN ('C') THEN 'CASH LODGEMENT'  -- EBS
from DDBWV01P.ACCOUNT_PERIODIC_MORTGAGE

• Leave outstanding NULL
v1.7 Leave outstanding 'OTHER'"			"DDEWV01P
DDBWV01P"	"ACCOUNT_PERIODIC_LOAN_ACC
SUB_NAPS_PRODUCT 
APPLICATION_DETAIL_NAPS
APPLICATION_ACCOUNT_REL"	ACCOUNT_PERIODIC_MORTGAGE					
	CCR_ACCOUNT_PERIODIC		CCR_EXCL_IND	CHAR(1)	Y		If the account on the CCR Account table is present in the Exclusion Reference table then 'Y' else 'N'			DDEWV01P	"CCR_ACCOUNT_PERIODIC
CCR_ACCOUNT_EXCL_PERIODIC_REF"		WH_ACC_NO				
	CCR_ACCOUNT_PERIODIC		FIN_AMT_LAYER_FLAG	CHAR(1)	Y		"For CI, Set value as 'Tier 1'
For CN, Set value as 'Tier 2'"										
	CCR_ACCOUNT_PERIODIC		REORG_IND	VARCHAR(20)	Y		Refer to REORG_IND Sheet	Refer to REORG_IND Sheet		DDEWU01P	"CCR_ACCOUNT_PERIODIC
ENT_CUST_ACCOUNT_REL_PERIODIC"						
1.7	CCR_ACCOUNT_PERIODIC		EXPOS_CLASS	VARCHAR(120)			",CASE
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  AND ECAP.CR_PORTF_FK <> 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Corporate of which; Other corporate,
  of which real estate related'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  AND ECAP.CR_PORTF_FK <> 'SME'
  AND ECAP.PCAR_SECTOR <> '11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Corporate of which; Other corporate,
  of which not secured by real estate'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  AND ECAP.CR_PORTF_FK = 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Corporate of which; SME, of which real estate related'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  AND ECAP.CR_PORTF_FK = 'SME'
  AND ECAP.PCAR_SECTOR <> '11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Corporate of which; SME, not secured by real estate'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE' AND ECAP.CR_PORTF_FK <> 'SME'
  THEN 'Corporates of which: Other corporate'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE' AND ECAP.CR_PORTF_FK = 'SME'
  THEN 'Corporates of which: SME'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  AND SEG.CUST_MGMT_HIER_LEVEL4 = 'Specialised Lending'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Corporate of which: specialised lending,
  of which real estate related'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  AND SEG.CUST_MGMT_HIER_LEVEL4 = 'Specialised Lending'
  THEN 'Corporate of which: specialised lending'
  WHEN ECAP.FINREP_CAT_CDE = 'CORPORATE'
  THEN 'Corporate'
  /*RETAIL*/
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL' AND ECAP.CR_PORTF_FK <> 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  AND ECAP.GCC_SECTOR_CDE = 'PDH'
  THEN 'Retail secured by real estate property non-SME,
  of which: owner occupier'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  AND ECAP.CR_PORTF_FK <> 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land
  AND Dvlpnt Acts' AND ECAP.GCC_SECTOR_CDE = 'BTL'
  THEN 'Retail secured by real estate
  property non-SME, of which: buy to let'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL' AND ECAP.CR_PORTF_FK <> 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land
  AND Dvlpnt Acts' AND ECAP.GCC_SECTOR_CDE NOT IN ('BTL','PDH')
  THEN 'Retail secured by real estate property non-SME,
  of which: other secured by real estate'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  AND ECAP.CR_PORTF_FK = 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Retail secured by real estate property SME'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  AND ECAP.CR_PORTF_FK <> 'SME'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Retail secured by real estate property non-SME'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  AND ECAP.PCAR_SECTOR ='11 Real Estate,Land and Dvlpnt Acts'
  THEN 'Retail secured by real estate property'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  AND ECAP.CR_PORTF_FK = 'SME'
  THEN 'Retail SME'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  AND ECAP.CR_PORTF_FK <> 'SME'
  THEN 'Retail non-SME'
  WHEN ECAP.FINREP_CAT_CDE = 'RETAIL'
  THEN 'Retail'
  ELSE 'Other non-credit obligation assets'
  END AS EXPOS_CLASS"			DDEWV50P	"ENT_COMBINED_ACC_PERIODIC
SEGMENTATION_GEN_PERIODIC"		"FINREP_CAT_CDE
CR_PORTF_FK
PCAR_SECTOR
CUST_MGMT_HIER_LEVEL4"		Exposure Class	Exposure Class	
1.7	CCR_ACCOUNT_PERIODIC		PURPOSE_CREDIT_TYP	VARCHAR(50)			"  ,CASE
  WHEN ECAP.PCAR_SECTOR = '05 Construction'
  THEN 'Construction Investment'
  WHEN ECAP.GCC_SECTOR_CDE LIKE '%COMMERCIAL%'
  THEN 'Commercial real estate purchase'
  WHEN ECAP.GCC_SECTOR_CDE IN ( 'PDH', 'BTL')
  THEN 'Residential real estate purchase'
  ELSE 'Other Purposes'
  END AS PURPOSE_CREDIT_TYP"			DDEWV50P	ENT_COMBINED_ACC_PERIODIC		"PCAR_SECTOR
GCC_SECTOR_CDE"		Purpose Credit Type	Purpose Credit Type	
1.7	CCR_ACCOUNT_PERIODIC		PMT_MADE_AMT	DECIMAL(18,2)			Direct Move			DDEWV50P	ENT_ACCOUNT_BAL_CONV_PERIODIC		CURR_MORT_CHRG_AMT_E		Payment Made Amount	Payment Made Amount	
1.7	CCR_ACCOUNT_PERIODIC		PMT_MADE_DTE	DATE FORMAT 'YYYY-MM-DD'			Direct Move			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		LAST_CR_DTE		Payment Made Date	Payment Made Date	
1.7	CCR_ACCOUNT_PERIODIC		MOF_LINK_CDE	VARCHAR(50)			Join the WH_ACC_NO from MOF_ACCOUNT_DAILY to the WH_ACC_NO in scope for CCR, and take the MOF_LINK_CODE value from MOF_ACCOUNT_DAILY and populate to this field			DDEWU01P	"MOF_ACCOUNT_DAILY
ENT_ACCOUNT_PERIODIC_FACT"		N/A				Not being populated currently
1.8	CCR_ACCOUNT_PERIODIC		ACC_NO_DERV	VARCHAR(36)			"For the overdraft accounts (CN) that are open in the reporting month, check how many times in the past they have been closed. Depending on the number of times it is closed previously, we need to associate that number along with the account number like the example values listed below
Example:
Actual Accounts:
0093327951002406
0093357062101185
0093412710958930
ACC_NO_DERV:
0093327951002406-3
0093357062101185-1
0093412710958930-2
"			DDEWM53P	CCR_ACCOUNT_PERIODIC		ACC_NO				
	CCR_ACCOUNT_PERIODIC		CCR_REPORTED_DTE	DATE FORMAT 'YYYY-MM-DD'	N		"Last Day of the Calendar Month
Missing Accounts: Last Calender date of the reporting month"				Derived						
	CCR_ACCOUNT_PERIODIC		PERIOD_DTE	DATE FORMAT 'YYYY-MM-DD'	N		"EDW standard field
Missing Accounts: Last Working date of the reporting month"			DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		PERIOD_DTE				
	CCR_ACCOUNT_PERIODIC		SRCE_SYS                      	SMALLINT	N					DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		SRCE_SYS                      				
	CCR_ACCOUNT_PERIODIC		SRCE_INST                     	SMALLINT	N					DDEWV50P	ENT_ACCOUNT_PERIODIC_FACT		SRCE_INST                     				
	CCR_ACCOUNT_PERIODIC		LOAD_LAST_ACTION              	CHAR(1)	N				I		Default						
	CCR_ACCOUNT_PERIODIC		LOAD_DTE                      	DATE FORMAT 'YYYY-MM-DD'	N		Date()				Default						
	CCR_ACCOUNT_PERIODIC		LOAD_TIME                     	INTEGER	N		Time()				Default						
