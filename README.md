Updated in this version	Target Table	Mapping with Req Doc	Target Field	Data Type	NULLABLE	Primary Key	Transformation Logic ROI	EBS - Transformation Logic	Default Value	Source Database	Source Table	EBS-Source Table	Source Field	EBS-Source Field	Business Name	Metadata	Comments
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
----
Feature: Test the transformation of CCR_CNTRACT_TYP in ENT_ACCOUNT_PERIODIC_FACT using CCR_CNTRACT_TYP_REF

  Scenario: Validate CCR_CNTRACT_TYP mapping with reference table fallback
    Given the table ENT_ACCOUNT_PERIODIC_FACT exists
      | SRCE_INST | SRCE_SYS | SRCE_PROD_CDE_FK | CR_PORTF_FK | PROD_SUMM_DESCR |
      | 1         | 1359     | 301              | RETL        | PERSONAL LOAN   |
      | 9         | 60       | 401              | BTL         | HOME LOAN       |
      | 9         | 50       | 402              | RETL        | PERSONAL LOAN   |
      | 5         | 22       | 701              | RETL        | AUTO LOAN       |

    And the table CCR_CNTRACT_TYP_REF exists
      | SRCE_INST | SRCE_SYS | SRCE_PROD_CDE | CCR_CNTRACT_TYP |
      | 5         | 22       | 701           | CN              |

    When the job EWM01CP0 runs
    Then the results of job execution are successful

    And the table ENT_ACCOUNT_PERIODIC_FACT contains the data
      | SRCE_INST | SRCE_SYS | SRCE_PROD_CDE_FK | CCR_CNTRACT_TYP |
      | 1         | 1359     | 301              | CN              |
      | 9         | 60       | 401              | CN              |
      | 9         | 50       | 402              | CN              |
      | 5         | 22       | 701              | CN              |

    And the table ENT_ACCOUNT_PERIODIC_FACT should match its definition in ENT_ACCOUNT_PERIODIC_FACT_BRD.xlsx,ENT_ACCOUNT_PERIODIC_FACT
