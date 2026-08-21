# China Economic Review: Data and Code Availability Policy

> **Version 2.0 — August 2026**
>
> This policy aligns with the [Data and Code Availability Standard (DCAS)](https://zenodo.org/communities/ssde/records) established by the Social Science Data Editors and the American Economic Association (AEA). CER's requirements are a practical subset of DCAS tailored to the journal's scope and author community.

---

## 1. Introduction and Objectives

The *China Economic Review* (CER) is committed to the highest standards of transparency and reproducibility. Empirical research must be documented and accessible to allow for verification and to facilitate cumulative scientific progress. This policy aligns with the Data and Code Availability Standard (DCAS) established by the Social Science Data Editors and the American Economic Association (AEA).

---

## 2. General Requirements

Authors of accepted papers containing empirical work, simulations, or experimental work must provide a complete **Replication Package** prior to final publication.

**Repository:** Authors must upload the package to a trusted data repository. The [CHINA ECONOMIC REVIEW Dataverse](https://dataverse.harvard.edu/dataverse/cer) is strongly preferred. Please consult the Dataverse Guides for upload instructions.

> **Pre-Submission Preview:** Uploading the package to the CER Dataverse allows the Data Editor to preview and pre-check the files before formal resubmission, significantly reducing review turnaround. Packages uploaded to other repositories cannot be previewed and will enter the full system review cycle.

> **Checklist Upload:** The *CER Data and Code Availability Statement & Checklist* must be uploaded directly to the **Elsevier Editorial System** alongside the formal response — **not** to Dataverse. The checklist template can be downloaded from this page.

**Data Citation:** All data used in the study must be properly cited in the reference list, following the AEA Data Citation Guidelines. Each dataset citation must include: the data producer, publication year, dataset title, publisher/distributor, and the date on which the data were obtained or accessed. For example:

> National Bureau of Statistics. 2007. "2007 National Input-Output Table for 135 Industries." National Bureau of Statistics. Data obtained on 2016-09-01.

**Manuscript Disclosure:** The manuscript must include a "Data Availability" section specifying the DOI of the replication package and a declaration stating whether the package is sufficient to fully reproduce all results presented in the paper.

**Verification:** Final acceptance is contingent upon the Data Editor's successful verification of the replication package.

---

## 3. Proprietary, Restricted, and Sensitive Data

CER acknowledges that datasets may be subject to third-party restrictions (e.g., proprietary data vendors, confidential administrative records). To balance research transparency with data security, CER adopts a **Three-Tiered Accountability Framework** for restricted data.

### 3.1 Compliance, Data Rights, and Sensitivity Standards

Before applying the tiered framework, authors must perform a comprehensive audit of their data access and sharing permissions:

**Author's Responsibility for Rights Verification:** By submitting any materials to the Journal, including but not limited to documentation, program code, log files, screenshots, datasets, and related files, the submitting author(s) represent and warrant that such materials do not contain state secrets, confidential commercial information, sensitive personally identifiable information, restricted micro-data, or content that infringes upon personal privacy, third-party rights, or any applicable laws and regulations, unless lawful authorization has been duly obtained or appropriate protective measures (including anonymization or aggregation) have been implemented.

The author(s) further represent and warrant that they possess the legal right, copyright, and all necessary permissions to disclose and share the submitted materials, whether publicly or privately with the Journal. Such responsibility includes full compliance with Institutional Review Board (IRB) requirements, Data Use Agreements (DUA), and data provider restrictions. The Journal shall not request materials that authors are not legally entitled to share, and bears no responsibility for verifying authors' legal rights or permissions.

**Definition of Raw Data:** CER recognizes that in many research contexts — particularly research using Chinese data — data may come from diverse sources, including administrative records, commercial databases, survey microdata, and public statistical publications. The term **"raw data"** in a CER replication package refers to the starting point of the data pipeline that feeds into the analysis. CER recognizes **two acceptable categories** of raw data:

1. **Original source data:** Data directly obtained from the primary data producer or official distributor in its original, unmodified form. Examples include:

   - Administrative records (e.g., tax records, patent filings, land transaction records) downloaded from the issuing government agency or a formally established commercial data vendor
   - Commercial databases (e.g., CSMAR, WIND, CNRDS) downloaded from the **official platform** (not via third-party redistributors)
   - Survey microdata (e.g., CFPS, CHFS, CGSS) obtained from the official data provider under the provider's data use agreement
   - Public statistical data (e.g., statistical yearbooks, census data) from the original publisher
2. **Peer-reviewed processed data:** A processed or cleaned dataset that was constructed and documented by a **peer-reviewed published study** may serve as the starting "raw data" for your replication package, provided that **both** of the following conditions are met:

   - The source study is formally cited; **and**
   - Its data-cleaning methodology has been publicly documented (e.g., via its own replication package).

   You must also explicitly disclose in your README that you are using this peer-reviewed dataset as your starting point.

> **Note on unacceptable raw data:** For what does NOT qualify as raw data and how to handle such cases, see FAQ Q7.

**Privacy Protection:** Sensitive personally identifiable information (PII) or restricted micro-data must not be included in any public replication package. Authors must ensure all public data is de-identified in accordance with applicable privacy laws and provider protocols.

**Publicly Sourced Data:** Data manually collected or web-scraped from public sources by the authors is **not considered restricted**. CER requires full disclosure of such datasets to facilitate cumulative research and maximize the paper's academic impact. The "effort" involved in data collection does not constitute a restriction on sharing. Similarly, **self-collected experimental data** must be made public after de-identification — stripping personal identifiers and retaining only the variables necessary for analysis. This is standard practice for experimental research.

### 3.2 Tiered Hierarchy for Replication and Verification

If your raw data is subject to restrictions, you must proceed through the following tiers in order:

**Tier 1: Public Disclosure — Analytical Data**

*Objective:* Provide the research community with the maximum level of transparency permissible.

- **Minimum Viable Variables:** Authors must negotiate with data providers to release the minimum subset of variables necessary to replicate the core results.
- **Intermediate Data Standard:** If raw data cannot be shared, authors must provide anonymized, censored, or aggregated analytical datasets in the public domain to allow for the replication of all reported analytical results (including all tables, figures, and coefficients).
- **Cleaning Process Disclosure:** Even when raw data is restricted, the complete code and log files that transform the raw data into the anonymized analytical dataset must still be provided. The transformation chain — from raw source data through all cleaning steps to the shared intermediate files — must be fully documented and reproducible.

**Tier 2: Internal Verification (The Data Editor Pathway)**

*Trigger:* This tier applies only when even anonymized analytical datasets (Tier 1) are legally or contractually prohibited from public release.

- **Verification Channels:** Authors must provide the Data Editor with private means to verify reproducibility. This includes private data sharing, access to a secure remote server, or other encrypted transfer methods.
- **Institutional Data Labs:** If data is hosted in a restricted university or institutional lab, authors are required to assist the Data Editor in applying for access or facilitating the replication check within that secure environment.
- **Confidentiality:** CER guarantees that data shared for internal verification will be treated as strictly confidential. Upon completion of the reproducibility check, the Data Editor will permanently delete all datasets obtained through internal verification channels.

**Tier 3: Non-Verifiable Disclosure (The Exceptional Case)**

*Trigger:* This applies only if the data is so restricted that no form of verification (neither Tier 1 nor Tier 2) is possible.

- **Mandatory Disclosure:** Authors must explicitly disclose this lack of verification opportunity in their Cover Letter at the time of initial submission, explaining the specific legal or technical barriers. The journal editor will determine whether a manuscript under this exceptional status may proceed to peer review, based on the specific constraints disclosed by the authors.
- **Submission Questionnaire:** For such manuscripts, authors must select "No" for Question 5: "Can all results be checked for replication by the Data Editor according to the Data and Code Availability Policy?"

> **Cross-cutting requirement:** Regardless of the tier selected, the replication package must always include the complete code and log files covering the full data pipeline — from raw data through intermediate processing to final analytical results. No tier exempts authors from providing the transformation chain. See §4 for details.

> **How to choose a tier?** See FAQ Q3–Q6 for tier selection guidance and common scenarios.

### 3.3 Data Persistence and Enforcement

**Three-Year Private Archive:** Regardless of the tier applied, authors must maintain a complete private archive of the original data on their institutional systems for at least three (3) years following publication to answer future queries.

**Revocation of Acceptance:** If a manuscript is found to be non-verifiable and this was not disclosed in the Cover Letter and Questionnaire during initial submission, CER reserves the right to revoke the Conditional Acceptance upon the Data Editor's review.

---

## 4. Components of the Replication Package

**This section applies to ALL tiers.** Regardless of data restrictions, every replication package must document and provide the complete pipeline from raw data to final results. There must be no gap between the raw data and the analytical dataset — code and log files covering every step of the transformation must be included.

### 4.1 The Four Components

| Component                                | Description                                                                                                                                                                                                       |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **1. Raw Data & Cleaning Process** | The raw data (as defined in §3.1), the complete data-cleaning code, and the corresponding log files used to transform raw data into the analytical dataset. Raw data files should be included unless restricted. |
| **2. Analysis Data & Programs**    | The cleaned/intermediate data, full analysis code, and log files for all estimations, simulations, and figure/table generation.                                                                                   |
| **3. Result & Log Files**          | All output files (tables, figures) and comprehensive log files showing the complete execution of all programs.                                                                                                    |
| **4. README File**                 | A document (using the standard template) providing a file manifest, data provenance documentation, and clear replication instructions.                                                                            |

> **Mandatory for all tiers:** The replication package must contain the code and log files for **the entire pipeline** — from raw data through intermediate processing to final analytical results:
>
> **Raw Data → Cleaning Code + Log → Intermediate Data → Analysis Code + Log → Results**
>
> This requirement applies equally to Tier-0 through Tier-3. The Data Editor must be able to verify every transformation step, whether through public files (Tier-0/1), private access (Tier-2), or in-situ verification (Tier-3).

Authors must provide the complete **bridging code** that transforms raw data into analytical datasets, structured as a traceable pipeline.

**Replication Package for Restricted Access:** Even when raw data cannot be publicly shared, authors are still required to submit the cleaning codes and log files for Component 1 and the analytical programs and log files for Component 2. The replication files must be **"ready-to-run,"** meaning the code should be structured to function immediately once the restricted raw data is placed in the designated folders. The gap between raw data access and final results must be fully bridged by documented, executable code.

**Raw Data Organization:** Raw data must be separated into two categories within the replication package:
- **`public_raw_data/`** — raw data files that can be legally shared with the public (e.g., government statistical yearbooks, public administrative records, web-scraped data).
- **`restricted_raw_data/`** — raw data files that are subject to license agreements, data use agreements, or other legal restrictions that prevent public redistribution (e.g., CSMAR, CNRDS, survey microdata). If the files cannot be included in the package, the folder must contain documentation describing each file, its source, and the access procedure. Example files for a single year or subset are encouraged.

### 4.2 README Documentation Requirements

The README file is the central document of the replication package. CER requires authors to follow the [Social Science Data Editors&#39; Template README](https://social-science-data-editors.github.io/template_README/) as the standard model. The template provides detailed guidance and examples for each section below. At a minimum, a CER-compliant README must contain all of the following sections:

**1. Overview**

- A brief summary of the replication package contents and how to proceed from start to finish.
- Mention the software used, the master script name, and the expected total runtime.

**2. Data Availability and Provenance Statements**

- A statement about whether all, some, or no data are publicly available.
- For **each data source**: formal data citation (see §2 for format), file name, format, whether provided in the package, and a description of how the authors obtained access.
- For **restricted data**: step-by-step instructions for other researchers to acquire the same data — including provider identity, application procedures, access method (on-site / remote desktop / secure download), fees, eligibility, and the exact official dataset name.
  - It should include the exact version or wave of each dataset, and **basic descriptive statistics for each raw dataset** (e.g., number of observations by year, key variable summaries).
- A **Summary of Data Availability** table is strongly recommended (see the SSDE template for format).

**3. Data Dictionary and Dataset List**

- **Data Dictionary:** A definition of each variable's exact meaning and type (numerical, categorical, string, etc.). For datasets in formats that do not embed labels (e.g., `.csv`), a separate codebook file should be provided and referenced.
- **Dataset List:** Organized by folder, with three separate tables:
  - **`public_raw_data/` table:** Each file listed with its name and original source.
  - **`restricted_raw_data/` table:** Each file listed with its name, source, and access instructions. Example files are encouraged where full files cannot be provided.
  - **`intermediate_data/` table:** Each file listed with its name, the do-file (or other code) that generates it, and which raw data files it uses as input.
- Every `.dta` file in the package must appear in one of these three tables. Every `.do` file must appear either as generating code in the intermediate data table or in the program description section.

**4. Computational Requirements**

- **Software:** All required software with exact version numbers.
- **Packages/Libraries:** All external packages or libraries, with versions and installation instructions. For Stata, either list all required `ssc install` commands or include the `.ado` files in a dedicated subfolder.
- **Memory, Runtime, and Storage:** Approximate time to reproduce all results, and any special hardware or storage requirements. Commands with significant runtime (e.g., exceeding 5 minutes) should be noted.

**5. Description of Programs**

- A high-level overview of the code structure: the master script, the sequence of programs, and the purpose of each program or subfolder.
- The master script must orchestrate the full pipeline from raw data to final results.

**6. Instructions to Replicators**

- Step-by-step instructions to reproduce all results, in strict linear sequence.
- Include any manual setup steps (e.g., editing a configuration file, downloading external data, installing packages).
- Instructions must use **relative paths** only. Log files (`.log`, `.smcl`) must be included and their location documented.

**7. List of Tables, Figures, and Programs**

- A mapping of each table and figure in the manuscript (and appendix) to the program and line number that generates it, and the output file produced.

**8. References**

- Full bibliographic references for all data sources cited in the README, in the journal's reference style.

---

## 5. Research Category Specifications

### 5.1 Econometric and Simulation Papers

In addition to the general package requirements in §4, authors of econometric and simulation papers must provide:

- All data construction and cleaning programs that produce the analytical dataset from raw data sources
- All estimation programs used to generate the reported results (tables, figures, and in-text numbers)
- For simulation or calibration exercises: the full simulation code with fixed random seeds, and clear documentation of all parameter choices and calibration targets

### 5.2 Experimental Papers

Authors must provide full instructions, subject selection criteria, all computer programs/scripts used to run the experiment, and the raw experimental data with code necessary to replicate the analysis. **The de-identified (anonymized) experimental record data must be published** as part of the replication package — stripped of personal identifiers and retaining only the variables necessary for analysis. If IRB restrictions prohibit even anonymized sharing, the README must clearly document the IRB institution, protocol number, and the institutional contact body for data access requests.

### 5.3 Theoretical Papers

For papers that do not contain any data analysis and therefore are exempt from replication requirements, you also need to add a "Data Availability" section to your final manuscript stating: *"This is a purely theoretical paper that does not contain empirical work, computational simulations, or experimental data."*

---

## 6. Timeline and Final Acceptance

**Submission and Pre-Check Workflow:** Upon Conditional Acceptance, authors should first upload the replication package to the **CER Dataverse** for pre-check by the Data Editor. This pre-check allows the Data Editor to identify issues before the formal resubmission, avoiding multiple rounds of system-level review. Once the Data Editor confirms the package is ready, authors submit **one final official resubmission** through the Elsevier Editorial System. Authors have four (4) weeks from Conditional Acceptance to initiate this process.

**Tier-2 Processing:** Replication packages involving Tier-2 restricted data require specialized verification coordination (including private access arrangements or on-site lab visits). These submissions are processed sequentially based on submission timestamp, and the verification timeline will be longer than for unrestricted packages.

**Data Editor Authority:** Final acceptance is granted only after the Data Editor confirms reproducibility. If a functional package is not provided or fails verification without prior disclosure, the Conditional Acceptance may be revoked.

**Post-Publication Oversight:** Following the public release of the replication package, the Data Editor will receive and review reports or inquiries from the academic community. If a valid issue is identified, the Data Editor will verify the concern with the authors and facilitate the necessary data corrections or formal responses. Should these corrections reveal significant discrepancies that undermine the study's primary findings or conclusions, the journal reserves the right to take further action, including issuing an Erratum or a Retraction.

---

## 7. Code and Data Preparation Notes

To ensure the replication package meets the "ready-to-run" standard, authors should adhere to the following technical requirements:

### 7.1 Code Requirements

1. **Relative Paths:** All code must use relative paths, not absolute paths tied to specific local drives (e.g., do not use `C:/Users/Author/Documents/...`). Code with absolute paths will fail when run on another machine.
2. **Random Seeds:** Any code involving randomization, simulation, or placebo tests must `set seed` to ensure exact reproducibility across runs.
3. **Modular Structure:** A master script (e.g., `main.do` or `run_all.R`) should orchestrate the full pipeline, with each table or figure generated by an individual script file. Do not place all code in a single monolithic file.
4. **External Dependencies:** For Stata, all external commands (e.g., ado-files) must either be included in a dedicated subfolder of the replication package, or be explicitly listed in the README with installation instructions. Commands with significant runtime (e.g., exceeding 5 minutes) should be noted in the README with expected duration.
5. **Table Export:** All estimation results must be exported as output files (e.g., `.xlsx`, `.csv`, `.tex`), not merely displayed in the console or Results window.
6. **Variable Consistency:** Variable names and labels in output files must be consistent with those referenced in the manuscript. Use clear labels for all variables in output tables.
7. **Log Files:** Original log files (`.log` or `.smcl` for Stata) showing the successful execution of all programs must be included. For Tier-2/Tier-3 data, log files are especially critical as they are the primary means for the Data Editor to verify that scripts run successfully and produce the paper's results.

### 7.2 Data Pipeline Transparency

The replication package should maintain a clear separation between raw data, intermediate data, and final analytical datasets. Authors should use a **folder structure** that reflects this pipeline:

```
/replication_package/
├── README.pdf
├── public_raw_data/                  # Raw data that can be shared publicly
├── restricted_raw_data/              # Raw data under license/DUA (or documentation + examples)
├── intermediate_data/                # All derived/aggregated/merged datasets
├── raw_data_cleaning_program/        # do-files: raw → intermediate
│   └── main_clean.do                 # Master script that runs all cleaning steps
├── analysis_program/                 # One do-file per table/figure
│   └── main.do                       # Master script that runs all analysis steps
├── results/                          # All generated outputs (tables, figures)
└── logs/                             # All execution log files
```

A replicator should be able to trace each variable from its raw source through to the final reported result:

> **Raw Data → Intermediate Data → Final Tables**

---

## 8. Frequently Asked Questions

Please see the separate [Frequently Asked Questions](faq.md) document for detailed answers to common questions, including:

- CSMAR/WIND and commercial database policies (Q1)
- How to choose the correct data tier (Q3)
- What counts as raw data (Q8)
- Self-collected data and IRB requirements (Q10)
- DOI links and the Dataverse publication timeline (Q12)
- University lab restricted data (Q13)

---

## 9. Resources

The latest version of this policy, the FAQ, and the CER self-check tools are maintained on GitHub:

**Repository:** [github.com/RuochenDai-Econ/cer-data-policy](https://github.com/RuochenDai-Econ/cer-data-policy)

---
