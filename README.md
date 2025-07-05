 Breast Cancer Oncology Clinical Trial – SDTM Mapping (Phase I Study)

1.Study Overview
Title: Phase I, Open-Label, Multi-Center Oncology Trial
Objective: Evaluate safety and tumor response to an investigational drug in patients with solid tumors.
Therapeutic Area: Oncology – Focus on Breast Cancer
Design: Non-randomized, early-phase study

2. Project Folder Structure
Folder	          Description
raw_data/	Raw Excel files (e.g. DM.xlsx, TU.xlsx)
specs/	        Mapping specifications for SDTM
sas_code/	SAS code for transforming CDM to SDTM
output/	        Final SDTM datasets (DM, TU, TR, RS)
README.md	Project explanation


3. Breast cancer is one of the most common and impactful cancers affecting women worldwide. In oncology clinical trials, especially early-phase studies, it is crucial to accurately monitor and evaluate tumor progression. This includes identifying tumor sites, measuring tumor size over time, and assessing response to treatment.

To ensure consistent reporting across studies, CDISC SDTM standards provide dedicated tumor-related domains:
TU (Tumor Identification) – Tracks the anatomical locations of tumors.
TR (Tumor Measurements) – Captures tumor size (e.g., diameter) at specific time points.
RS (Tumor Response) – Evaluates how tumors respond to therapy using categories such as Complete Response (CR), Partial Response (PR), Stable Disease (SD), or Progressive Disease (PD), often based on RECIST criteria.

These domains ensure data traceability from initial tumor identification through to response assessment, enabling regulatory-compliant and submission-ready datasets for oncology studies.



step 1: Import the Data:-

PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM\DEATH.xlsx" 
            DBMS=XLSX 
            OUT=death
            REPLACE;
            GETNAMES=YES;
            RUN;

PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM\DM.xlsx" 
            DBMS=XLSX 
            OUT=DM
            REPLACE;
            GETNAMES=YES;
            RUN;

PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM\DS.xlsx"
            DBMS=XLSX 
            OUT=DS
            REPLACE;
            GETNAMES=YES;
            RUN;

PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM\EX.xlsx"
            DBMS=XLSX 
            OUT=EX
            REPLACE;
            GETNAMES=YES;
            RUN;
PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM\IE.xlsx"
            DBMS=XLSX
			OUT=IE
			REPLACE;
			GETNAMES=YES;
			RUN;
PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM\SPCPKB1.xlsx"
            DBMS=XLSX
			OUT=SPCPKB1
			REPLACE;
			GETNAMES=YES;
			RUN;


SETUP CODE:;

options mprint mlogic;
%let Root = /folders/myfolders/Clinical Project - Part 1;

Step 2: Creating libraries:-

 libname CDM  "&root./data/data_sas/CDM";
 libname SDTM "&root./data/data_sas/SDTM";
 libname ADAM "&root./data/data_sas/ADAM";



Step 3: Creating sas data sets
 /*CDM datasets*/

%macro CDM (Domain= ) ;

PROC IMPORT DATAFILE= "&root./data/data_excel/CDM/&domain..xlsx"
            DBMS=XLSX OUT= CDM.&domain  ;
     GETNAMES=YES;
RUN;

%mend CDM ;

%CDM (Domain = DEATH ) ;
%CDM (Domain = DM ) ;
%CDM (Domain = DS ) ;
%CDM (Domain = EX ) ;
%CDM (Domain = IE ) ;
%CDM (Domain = SPCPKB1 ) ;


/*SDTM datasets*/

%macro SDTM (Domain= ) ;

PROC IMPORT DATAFILE= "&root./data/data_excel/SDTM/&domain..xlsx"
            DBMS=XLSX OUT= SDTM.&domain  ;
     GETNAMES=YES;
RUN;

%mend SDTM ;

%SDTM (Domain = TA ) ;.;

libname cdm "C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\CDM";
libname sdtm "C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\SDTM";
libname adam "C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\ADAM";


proc copy in=work out=Cdm;
select death dm ds ex ie spcpkb1;
run;



PROC IMPORT DATAFILE="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\clinical_project_-_part_1\Clinical Project - Part 1\Data\Data_excel\SDTM\TA.xlsx"
            DBMS=XLSX
			OUT=TA
			REPLACE;
			GETNAMES=YES;
			RUN;


proc copy in=work out=sdtm;
select Ta;
run;

5. SDTM Domain Derivation Logic
1.DM Domain
Derived USUBJID, RFSTDTC, RFENDTC, ARMCD, etc.
Mapped death data from DEATH domain
Merged treatment exposure from EX domain

2.TU Domain – Tumor Identification
Created TULINKID, TUTESTCD, TUSEQ

3.TR Domain – Tumor Measurements
Derived TRTESTCD, TRORRES, TRSTRESC
Sequenced with TRSEQ

4.RS Domain – Tumor Response
Based on RESPONSE in RS data
Created RSSEQ, RSLINKID



Data DM1 ;
Set CDM.DM ;

STUDYID = "XYZ" ;
DOMAIN  = "DM" ;
USUBJID = Strip(STUDYID) || "/" || Strip(Put(SUBJECT,best.)) ;

SUBJID = SUBJECT  ;
Run ;

Data SPCPKB1 ;
Set CDM.SPCPKB1 ;
Where IPFD1DAT ne " " and PSCHDAY = 1 and PART = "A";
RFSTDTC = IPFD1DAT || "T" || IPFD1TIM ;
run ;

Data EX ;
Set CDM.EX ;
if EXENDAT ne " " or EXSTDAT ne " " ;
proc sort ; By SUBJECT EXSTDAT EXENDAT ;  
run ;

Data EX1 ;
Set EX ;
By SUBJECT EXSTDAT EXENDAT ;
If last.SUBJECT ;
Run ; 

Data DM2 ;
Merge DM1 (in=a) SPCPKB1 EX1 CDM.DS CDM.DEATH (where=(DTHDESIG = "1" )) CDM.IE (where=(IEYN = "0")) ;
By SUBJECT ;
If = a ;
Run ; 

Data DM3 (rename=(ETHNIC1 =ETHNIC));
Length ETHNIC1 $60 ;
Set DM2 ;
If EXENDAT ne "" then RFENDTC = EXENDAT ;
Else If EXENDAT = "" then RFENDTC = EXSTDAT ; 
else RFENDTC = IPFD1DAT || "T" || IPFD1TIM ;

RFXSTDTC = RFSTDTC ;  
RFXENDTC = RFENDTC ; 
RFPENDTC = DSSTDAT ;
DTHDTC   = DTH_DAT ;
SITEID   = CENTRE  ;
BRTHDTC  = BRTHDAT ;
AGE = AGE ;
AGEU     = "YEARS" ; 

If DTHDTC ne " " then DTHFL = "Y" ;
If SEX='C20197' then SEX = "M" ;
Else if SEX ='C16576' then SEX = "F" ;
Else SEX = "U" ;

If RACE = 'C41260' then RACE = 'ASIAN';
If RACE = 'C41261' then RACE = 'WHITE';

If ETHNIC = 'C41222' then ETHNIC1 = 'NOT HISPANIC OR LATINO' ;

iF RFSTDTC NE " " then ARMCD = "A01-A02-A03" ;
Else if IEYN = "0" and RFSTDTC = " "  then ARMCD = "SCRNFAIL"  ;
Else ARMCD = "NOTASSGN" ;

Drop ETHNIC ;

Proc sort ; By ARMCD ;
Run ;

Data TA ;
Set SDTM.TA (keep= ARMCD ARM) ;
Proc sort nodupkey ; By ARMCD ;
Run ; 

Data DM4 ;
Merge DM3 (in=a) TA ;
By ARMCD ;
If a ;

If ARMCD = "SCRNFAIL" then ARM = "Screen Failure" ; 
If ARMCD = "NOTASSGN" then ARM = "Not Assigned" ;
Run ;

Data DM5 ;
Set DM4 ;
ACTARMCD = ARMCD ;  
ACTARM   = ARM ;

CO = put(CENTRE,6.) ;

If substr(Strip(CO),1,2) = "23" or substr(Strip(CO),1,2) = "23"  then COUNTRY = "FRA" ; 
If substr(Strip(CO),1,2) = "70" or substr(Strip(CO),1,2) = "70"  then COUNTRY = "ESP" ; 
If substr(Strip(CO),1,2) = "60" then COUNTRY = "KOR" ; 

DMDTC = VIS_DAT ;
CENTRE = CENTRE ; 
PART = PART ; 
RACEOTH = Upcase(RACEOTH) ; 
VISITDTC    = VIS_DAT  ;

run ;

Data DM6 ;
Length STUDYID $21 DOMAIN $8 USUBJID $30 SUBJID 8 RFSTDTC $19 RFENDTC $19 RFXSTDTC $19 RFXENDTC $19
   RFPENDTC $19 DTHDTC $19 DTHFL $1 SITEID 5 BRTHDTC $19 AGE 8 AGEU $10 SEX $1 RACE $60 ETHNIC $60
   ARMCD $20 ARM $200 ACTARMCD $20 ACTARM $200 COUNTRY $3 DMDTC $19 CENTRE 8 PART $1 RACEOTH $200
   VISITDTC$19 ;

Set DM5 (rename=(SEX=SEX1 SITEID = SITEID1));

Label  STUDYID ="Study Identifier"
DOMAIN ="Domain Abbreviation"
USUBJID ="Unique Subject Identifier"
SUBJID ="Subject Identifier for the Study"
RFSTDTC ="Subject Reference Start Date/Time"
RFENDTC ="Subject Reference End Date/Time"
RFXSTDTC ="Date/Time of First Study Treatment"
RFXENDTC ="Date/Time of Last Study Treatment"
RFPENDTC ="Date/Time of End of Participation"
DTHDTC ="Date/Time of Death"
DTHFL ="Subject Death Flag"
SITEID ="Study Site Identifier"
BRTHDTC ="Date/Time of Birth"
AGE ="Age"
AGEU ="Age Units"
SEX ="Sex"
RACE ="Race"
ETHNIC ="Ethnicity"
ARMCD ="Planned Arm Code"
ARM ="Description of Planned Arm"
ACTARMCD ="Actual Arm Code"
ACTARM ="Description of Actual Arm"
COUNTRY ="Country"
DMDTC ="Date/Time of Collection"
CENTRE ="Centre Number"
PART ="Study Part Code"
RACEOTH ="Other Race Specification"
VISITDTC="Date of Visit" ;

SEX = SEX1 ;
SITEID = SITEID1 ;

Keep STUDYID DOMAIN USUBJID SUBJID RFSTDTC RFENDTC RFXSTDTC RFXENDTC RFPENDTC DTHDTC DTHFL
SITEID BRTHDTC AGE AGEU SEX RACE ETHNIC ARMCD ARM ACTARMCD ACTARM COUNTRY DMDTC
CENTRE PART RACEOTH VISITDTC ;
Run ;


Data SDTM.DM ;
Set DM6 ;
Proc sort ; By USUBJID ;
Run ;


/*PROC SQL;*/
/*    SELECT STUDYID, USUBJID, COUNT(*) AS DUP_COUNT*/
/*    FROM WORK.DM1*/
/*    GROUP BY STUDYID, USUBJID*/
/*    HAVING COUNT(*) > 1;*/
/*QUIT; PROC SQL;*/
/*    SELECT STUDYID, USUBJID, ECDOSE, ECSTDTC*/
/*    FROM WORK.EC*/
/*    WHERE ECDOSE IS MISSING OR ECSTDTC IS MISSING;*/
/*QUIT;*/
/* PROC COMPARE BASE=WORK.EC FINAL=WORK.EC_FINAL LISTALL;*/
/*    ID USUBJID;*/
/*RUN;*/


proc sort data=dm6;by usubjid;run;
proc compare base=Dm6 compare=ds;
   by usubjid;
 
run;


proc import datafile="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\TU.xlsx"
out=raw_tu 
dbms=xlsx
replace;
run;

data TU;
    set raw_tu;
    length DOMAIN $2 TUTESTCD $6 TUTEST $25 TULINKID $12 TULAT $6;
    DOMAIN = "TU";
	USUBJID = catx('',STUDYID,SUBJECTID);

    TUTESTCD = "TUMID";
    TUTEST = "Tumor Identification";
    TULINKID = cats("TU", upcase(TULOC));
    TULAT = ""; 
run;
proc sort data=TU; by USUBJID; run;

data TU1;
    set TU;
    by USUBJID;
    retain TUSEQ;
    if first.USUBJID then TUSEQ = 1;
    else TUSEQ + 1;
run;


proc import datafile="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\TR.xlsx"
out=raw_TR
dbms=xlsx
replace;
run;

data TR;
    set raw_tr;
    length DOMAIN $2 USUBJID $25 TRTESTCD $10 TRTEST $25
           TRORRES $10 TRSTRESC $10 TRLOC $20 TRLAT $10
           TRDTC $10 TRLINKID $15 TRGRPID $20;

    DOMAIN     = "TR";
    USUBJID    = catx("-", STUDYID, SUBJID);
    TRTESTCD   = "TUMDIA";
    TRTEST     = "Tumor Diameter";
    TRORRES    = strip(put(DIAM, best.));
    TRSTRESC   = TRORRES;
    TRLAT      = "";  
    TRLINKID   = cats("TU", upcase(TRLOC));
run;
proc sort data=TR; by USUBJID TRDTC TRLINKID; run;

data TR_final;
    set TR;
    by USUBJID;
    retain TRSEQ;
    if first.USUBJID then TRSEQ = 1;
    else TRSEQ + 1;
run;

proc print data=TR_final; run;

proc import datafile="C:\Users\ashok pc\OneDrive\Desktop\SDTMIG Guidelines\RS.xlsx"
out=raw_RS
dbms=xlsx
replace;
run;
data raw_rs_clean;
    set raw_rs;
    DOMAIN     = "RS";
    USUBJID    = catx("-", STUDYID, SUBJID);
    RSTESTCD   = "TUMRESP";
    RSTEST     = "Tumor Response";
    RSORRES    = strip(RESPONSE);
    RSSTRESC   = RSORRES;
    RSLAT      = "";  
    RSLINKID   = cats("TU", upcase(RSLOC));
run;

proc sort data=raw_RS; by USUBJID RSDTC RSLINKID; run;

data RS_final;
    set RS;
    by USUBJID;
    retain RSSEQ;
    if first.USUBJID then RSSEQ = 1;
    else RSSEQ + 1;
run;

proc print data=RS_final; run;



📊 End-to-End CDISC SDTM Mapping for an Oncology Clinical Trial (Phase I)


🧪 Study Overview

- **Study Title**: Phase I, Open-Label, Multi-Center Oncology Trial
- **Therapeutic Area**: Oncology
- **Population**: Patients with solid tumors
- **Design**: Non-randomized, early-phase safety and tumor response study
  
This project demonstrates the development of SDTM-compliant datasets using **Base SAS** for a **Phase I open-label oncology clinical trial**. 
The project includes both standard SDTM domains and oncology-specific domains: **TU (Tumor Identification), TR (Tumor Results), and RS (Response Assessment)**.



Study Background:

Title: Open-Label, Non-Randomized, Multi-Centre, Phase 1 Study  
Therapeutic Area: Oncology  
Objective:Evaluate safety and tumor response to a new investigational drug in subjects with solid tumors.

📁 Project Structure

- `raw_data/` – Contains mock input files (`TU.xlsx`, `TR.xlsx`, `RS.xlsx`)
- `specs/` – Mapping specification sheet used to convert raw data to SDTM
- `sas_code/` – SAS programs to generate each domain (`TU`, `TR`, `RS`)
- `output/` – Final SDTM datasets with derived variables
- `README.md` – This project overview


Tools Used:

- **SAS 9.4**: Data transformation, derivation logic, PROC steps
- **Excel**: Raw mock data & mapping specifications
- **CDISC Standards**: SDTMIG v3.2 (Tumor Domains)



Domains Created
----------------------------------------------------------------------------
| Domain | Description                  | Key Variables                     |
|--------|------------------------------|-----------------------------------|
| TU     | Tumor Identification         | TUTESTCD, TULINKID, TUSEQ         |
| TR     | Tumor Results (Measurements) | TRTESTCD, TRDTC, TRORRES, TRSEQ   |
| RS     | Tumor Response Evaluation    | RSTESTCD, RSCAT, RSSTRESC, RSSEQ  |
| DM       Demographics
| DS       Disposition
| EX       Exposure
| DEATH    Death Details
| IE       Inclusion/Exclusion
-----------------------------------------------------------------------------



💡 Key Learnings

- Developed a complete SDTM mapping flow from raw to SDTM using SAS
- Understood oncology-specific SDTM implementation for TU/TR/RS domains
- Derived complex clinical variables and ensured CDISC compliance
- Validated and formatted data as per mock shells for submission-readiness


 Output Highlights

- **Sequencing Logic:** `TUSEQ`, `TRSEQ`, `RSSEQ` derived using `BY USUBJID`
- **Linking Tumor Locations:** `TULINKID`, `TRLINKID`, `RSLINKID` used for traceability
- **Compliance Validated:** Dataset format and structure aligned with SDTM rules

---

 Future Additions

- ADaM derivations from TR (e.g., BSL, PCHG)
- TLFs based on RS response categories
- Integration with define.xml and Pinnacle 21 validation

---
 🔗 Author

**Prasanthi Kata**  
Aspiring SAS Clinical Programmer  
GitHub: 🔗 GitHub](https://github.com/Prasanthi-sas)









