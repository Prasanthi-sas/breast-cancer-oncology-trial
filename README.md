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









