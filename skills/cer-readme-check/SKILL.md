---
description: |
  Check a README file from a CER replication package against China Economic Review's Data and Code Availability Policy requirements. Use when: (1) Authors want to self-check their README before CER submission, (2) Checking compliance with CER README standards, (3) Verifying replication package documentation meets journal requirements. Trigger phrases: "check my README", "verify README compliance", "CER README check", "pre-submission README audit", "does my README meet CER requirements".
allowed-tools: Read, Bash, Glob
user-invocable: true
---

# CER README Compliance Check

Check a README file against [China Economic Review's Data and Code Availability Policy](https://dataverse.harvard.edu/dataverse/cer) requirements. Produces a structured report of issues found, ordered by severity.

## Workflow

### 1. Locate the README

The user provides a path to a README file or replication package directory. If given a directory, search for the README:

```bash
find <dir> -maxdepth 2 -iname "readme*" -o -iname "*.md" -o -iname "*.txt" -o -iname "*.pdf" | head -5
```

If multiple candidates exist, ask the user which to check. Support `.md`, `.txt`, `.pdf`, `.doc`, `.docx` files.

### 2. Read the README — PDF Handling

Read the full README file. **For PDF files**, extract text using the best available method:

```bash
# Preferred: pdftotext (preserves layout best)
pdftotext -layout "<file>" /tmp/readme_check.txt

# Fallback: python
python3 -c "
import sys
try:
    import pdfplumber
    with pdfplumber.open(sys.argv[1]) as pdf:
        for page in pdf.pages:
            print(page.extract_text() or '')
except ImportError:
    import subprocess
    subprocess.run(['pdftotext', '-layout', sys.argv[1], '/tmp/readme_check.txt'])
" "<file>"
```

**PDF Table Extraction Caveats:** PDF tables are frequently garbled during extraction, especially:
- Chinese characters may be split across lines or combined with adjacent columns
- Column alignment is often lost (values from different columns merge into one line)
- Multi-line cell content may appear as separate rows
- Unicode characters (e.g., `_含合作高`, `�`) indicate extraction corruption

**Critical rule for PDF-based checks:** Before marking any check as **FAIL** for "missing" content, first verify whether the content might be present in the PDF but garbled by extraction. Specifically:
- If a table *appears* to exist in the extracted text (fragments of column headers, data values) but is unreadable due to extraction artifacts → mark as **WARN** with the note "PDF table extraction may have garbled content — verify manually"
- If there is *no trace at all* of the expected content → mark as **FAIL**
- When uncertain, flag both possibilities: "Content may be present in PDF but unreadable due to extraction"

**Check for extraction quality before evaluating tables:**
```bash
# Quick check: do extracted lines contain garbled Chinese or merged columns?
grep -c '[a-zA-Z].*[一-鿿].*[a-zA-Z]' /tmp/readme_check.txt  # mixed CN/EN on same line
grep -c '_' /tmp/readme_check.txt  # potential filename fragments from corrupted cells
```

### 3. Read and Check

Run each item in the checklist against the extracted content. For each check, determine: **PASS**, **FAIL**, or **WARN**. For PDF sources, apply the PDF caveat rule above when evaluating table-dependent checks (especially items 2.9, 3.1, 3.3, 7.1, 8.1).

### 3. Report

Produce a structured report with these sections:

- **PDF Extraction Note** (if applicable): Brief note on extraction quality — any garbled tables, Chinese text corruption, or column misalignment observed. Flag which checks may be affected.
- **Summary**: Total checks, pass/fail/warn counts, and a count of checks that carry PDF extraction caveats
- **Critical Issues** (must fix before submission): Items that violate non-negotiable policy requirements
- **Warnings** (should fix): Items that are incomplete, unclear, or may be affected by PDF extraction quality
- **Passed Checks**: Items that look good
- **Not Applicable**: Checks skipped due to data tier or paper type

Each finding must include:
- The check item description
- Why it matters (one sentence)
- What to fix (actionable guidance)

## Checklist

The checklist mirrors the 8 required README sections in CER Policy §4.2, which follows the [SSDE Template README](https://social-science-data-editors.github.io/template_README/). Additional red-flag scans are in Section H.

### 1. Overview

| # | Check | How to verify |
|---|-------|---------------|
| 1.1 | **Brief summary present** | README opens with a short description of the package contents and how to proceed |
| 1.2 | **Key info in overview** | Software, master script name, and estimated total runtime are mentioned early |

### 2. Data Availability and Provenance Statements

| # | Check | How to verify |
|---|-------|---------------|
| 2.1 | **Availability statement** | States whether all, some, or no data are publicly available |
| 2.2 | **Data citations for every dataset** | Each dataset has a formal citation: data producer, year, title, publisher, access date. See policy §2 for format. |
| 2.3 | **Details on each data source** | For each source: file name, format, whether provided in package, how authors obtained access |
| 2.4 | **Raw data source check** | Acceptable raw data: (a) official primary source, OR (b) peer-reviewed published study with citation. Flag non-peer-reviewed informal sources (数据皮皮侠, 草莓科研服务网, etc.) as CRITICAL. |
| 2.5 | **CSMAR/WIND/CNRDS license check** | If commercial databases used, verify README states the license tier (Tier-0 if raw data can be shared, Tier-1 if only cleaning code can be shared). Check that the author has confirmed their institution's license terms — raw data files from commercial databases typically cannot be uploaded to public repositories. |
| 2.6 | **Public resources not wrongly restricted** | Flag statistical yearbooks, government open data, web-scraped public data marked as restricted |
| 2.7 | **Descriptive stats for raw data** | Each raw dataset includes basic descriptive statistics (observations by year, key variable summaries) |
| 2.8 | **Data version/wave** | Exact version or wave of each dataset stated (especially for survey data: CFPS, CHFS, etc.) |
| 2.9 | **Summary of Data Availability table** | Recommended: table with Data.Name, Data.Files, Location, Provided, Citation |

### 3. Data Dictionary and Dataset List

| # | Check | How to verify |
|---|-------|---------------|
| 3.1 | **Variable definitions** | Each variable's exact meaning and type (numerical, categorical, string) defined |
| 3.2 | **Codebook for unlabeled formats** | For `.csv` or other formats without embedded labels, a separate codebook file is referenced |
| 3.3 | **Dataset List — three tables by folder** | README must contain three separate tables: (a) **public_raw_data/** table — each file with name and source; (b) **restricted_raw_data/** table — each file with name, source, and access instructions; (c) **intermediate_data/** table — each file with name, the do-file that generates it, and which raw data files it uses. Every `.dta` file in the package must appear in one of these tables. |
| 3.4 | **Raw vs. intermediate distinction** | Clear separation between raw data files and intermediate/derived data files. Public raw data and restricted raw data must be in separate folders. |
| 3.5 | **Intermediate data provenance** | For every intermediate/derived data file, the dataset list must specify which `.do` file (or other code) generates it from which raw data. For Tier-1/Tier-2 especially: check that each intermediate file's generating code is named and exists in the package. |

### 4. Computational Requirements

| # | Check | How to verify |
|---|-------|---------------|
| 4.1 | **Software and version** | All required software with exact version numbers (e.g., "Stata 17.0", not just "Stata") |
| 4.2 | **External packages listed** | All packages/libraries with versions and installation instructions. For Stata: `ssc install` commands or `.ado` folder included. |
| 4.3 | **Runtime estimate** | Approximate total time to reproduce. Commands >5 minutes individually noted. |
| 4.4 | **Hardware/storage requirements** | Any special memory, storage, or hardware requirements stated |

### 5. Description of Programs

| # | Check | How to verify |
|---|-------|---------------|
| 5.1 | **Master cleaning script** | A `main_clean.do` (or equivalent) in `raw_data_cleaning_program/` orchestrates all raw→intermediate cleaning steps |
| 5.2 | **Master analysis script** | A `main.do` (or equivalent) in `analysis_program/` orchestrates all analysis steps from intermediate data to results |
| 5.3 | **Code structure overview** | High-level description of each program or subfolder and its purpose. Code is separated: cleaning code in `raw_data_cleaning_program/`, analysis code in `analysis_program/`. |
| 5.4 | **Modular structure** | Separate scripts for data cleaning, analysis, and each table/figure (not one monolithic file) |
| 5.5 | **Complete code + log for ALL tiers** | Every stage from raw through results has both code and log files. No gaps. No tier exempt. |

### 6. Instructions to Replicators

| # | Check | How to verify |
|---|-------|---------------|
| 6.1 | **Step-by-step instructions** | Clear, linear sequence to reproduce results. No gaps. |
| 6.2 | **Setup steps documented** | Any manual setup (edit config file, download data, install packages) explicitly listed |
| 6.3 | **No absolute paths** | Instructions and code use relative paths only. Flag `C:/Users/...`, `/home/...`, etc. |
| 6.4 | **Log files mentioned** | States that execution logs (`.log`, `.smcl`) are included and where they are located |

### 7. List of Tables, Figures, and Programs

| # | Check | How to verify |
|---|-------|---------------|
| 7.1 | **Script-to-output mapping** | Each table/figure mapped to the program (and line number) that generates it, plus the output file name |

### 8. References

| # | Check | How to verify |
|---|-------|---------------|
| 8.1 | **Data references in README** | Full bibliographic references for all data sources, in consistent format (journal's reference style) |
| 8.2 | **References match citations** | Every data citation in §2 has a corresponding full reference here |

### 9. Access Protocol for Restricted Data (if applicable)

Run only if README mentions restricted data or Tier-2/Tier-3:

| # | Check | How to verify |
|---|-------|---------------|
| 9.1 | **Provider identity** | Full name of data provider (commercial vendor, university lab, government agency) |
| 9.2 | **Access method** | On-site visit, virtual machine/remote desktop, or secure download link |
| 9.3 | **Application process** | Who to contact, materials needed, expected timeline |
| 9.4 | **Exact official dataset name** | As recognized by the provider, so third parties know what to request |
| 9.5 | **IRB details** (self-collected data) | IRB approving institution and protocol number |
| 9.6 | **Institutional contact body** | Researchers directed to the official institution, not just the author personally |

### H. Common Red Flags (Critical)

Scan the README for these patterns. Any match is a critical failure:

| # | Pattern | Why it's a problem |
|---|---------|-------------------|
| H1 | "Data available upon request" without details | Must specify who, what process, what conditions |
| H2 | "Data cannot be shared" without alternative | Even Tier-3 requires documenting why and how others might eventually access |
| H3 | Absolute file paths | `C:/Users/...`, `/home/...` — code won't run on another machine |
| H4 | Non-peer-reviewed processed data as "raw data" | Must be from official source or peer-reviewed study; otherwise label as intermediate with upstream code |
| H5 | Public data marked as restricted | Statistical yearbooks, government data, web-scraped data are not restricted |
| H6 | CSMAR/WIND marked Tier-2 | Downloadable databases → "No restricted data" |
| H7 | No variable definitions | README must have a data dictionary |
| H8 | Data from informal platforms | 数据皮皮侠, 草莓科研服务网, etc. → must re-acquire from official source or treat as intermediate with full upstream code |
| H9 | Self-collected data Tier-2 without IRB | Default: share de-identified version. If not possible, document IRB + institutional contact. |
| H10 | "Contact author for data" | Must direct to official institutional body, not author personally |
| H11 | Pipeline gap | Any missing code or log between raw → intermediate → analysis → results |
| H12 | Intermediate data with no generation code | Any `.dta`/`.csv` file in the data folder that is used by the analysis but has no corresponding `.do` file (or other code) in the package that generates it. For every intermediate file, the README must state which code creates it from which raw source. This check is CRITICAL for Tier-1/Tier-2. |

## Tier-Appropriate Expectations

The data tier is detected from the package contents and the tracking system — the README is NOT required to state it explicitly. Adjust checks based on the detected tier:

- **No restricted data / Tier-0**: All raw data files should be in the package. Sections 1–8 all apply fully. Section 9 (access protocol) is N/A.
- **Tier-1**: Raw data restricted but anonymized/intermediate data provided. Section 9 partially applies — document what is restricted and why. All other sections apply.
- **Tier-2**: No public data. Section 9 is CRITICAL — the README is the roadmap for future replicators. Log files (check 6.4) and pipeline completeness (check 5.4) are especially important.
- **Tier-3**: Exceptional. Section 9 mandatory and must be extremely detailed. Full legal/technical barrier explanation required.

## Report Format

```markdown
# CER README Compliance Report

**File:** `<path>`
**Data Tier:** <detected or stated tier>
**Date:** <today>

## Summary
- Total checks: X | Pass: Y | Fail: Z | Warn: W | N/A: N

## ❌ Critical Issues (Must Fix)
> These issues must be resolved before the replication package can be accepted.

1. **[2.4] Unacceptable raw data source**
   - **Found:** Data from 数据皮皮侠 described as raw data
   - **Why it matters:** Non-peer-reviewed processed data cannot serve as raw data. See policy §3.1 and FAQ Q8.
   - **Fix:** Re-acquire from the official source with full cleaning pipeline, OR treat as intermediate data with upstream code disclosed

... (repeat for each critical issue)

## ⚠️ Warnings (Should Fix)
> These items need attention but are not blocking.

...

## ✅ Passed Checks
> These items look good.

- 2.2: Data citations present for all 4 datasets
- 4.1: Software and version stated (Stata 17.0)

## N/A — Not Applicable
> These checks were skipped.

- §9 (Access Protocol): No restricted data detected / Tier-0
```

## Post-Check Guidance

After producing the report:

1. If critical issues found: "Please fix all critical issues above and re-run the check. The most common reasons READMEs are returned are: (a) missing Data Provenance and Rights section, (b) data incorrectly marked as restricted (e.g., statistical yearbooks, CSMAR), and (c) insufficient access protocol details for restricted data. See the [FAQ](https://dataverse.harvard.edu/dataverse/cer) for guidance on these topics."

2. If only warnings: "Your README is in good shape. Address the warnings above for a stronger submission. For tier-specific questions, consult the FAQ."

3. If all clear: "Your README appears compliant with CER policy requirements. You are ready to upload to the CER Dataverse for pre-submission review. Remember: the Data Editor can pre-check your package before formal resubmission."

## References

- CER Data and Code Availability Policy v2: see the journal website
- CER FAQ: see the journal website (separate document linked from the policy page)
- SSDE Template README: <https://social-science-data-editors.github.io/template_README/>
- CER Dataverse: <https://dataverse.harvard.edu/dataverse/cer>
- CER Checklist: downloadable from the policy page on the journal website
