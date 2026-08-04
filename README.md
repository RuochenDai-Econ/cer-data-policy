# CER Data and Code Availability Policy

Public repository for the **China Economic Review** Data and Code Availability Policy, associated templates, and Claude Code skills for replication package verification.

## Contents

### Policy Documents (`/policy`)

- **[CER Data and Code Availability Policy v2](policy/China%20Economic%20Review-Data%20and%20Code%20Availability%20Policy-v2.md)** — The complete policy document. Aligned with the [Data and Code Availability Standard (DCAS)](https://zenodo.org/communities/ssde/records) and the [SSDE Template README](https://social-science-data-editors.github.io/template_README/).
- **[Frequently Asked Questions](policy/faq.md)** — Detailed Q&A covering tier selection, CSMAR/WIND licenses, raw data definitions, DOI publishing timeline, and more.
- **[CER Data and Code Availability Statement & Checklist](policy/4CER%20Data%20and%20Code%20Availability%20Statement%20%26%20Checklist.pdf)** — Mandatory compliance checklist for all CER submissions (PDF).

### Claude Code Skills (`/skills`)

These skills can be installed and used by authors and the Data Editor to verify replication packages.

#### `/cer-readme-check`
Check a README file against CER policy requirements. Produces a structured compliance report with 40+ checks across 9 categories.

**Install:** Copy `skills/cer-readme-check/` to `~/.claude/skills/cer-readme-check/`

#### `/cer-reproduce-check`
Automated replication package verification: (1) inspect folder structure and file organization, (2) detect and fix absolute paths, (3) trace data lineage for each intermediate data file, (4) run the full replication pipeline, (5) compare generated results against provided outputs.

**Install:** Copy `skills/cer-reproduce-check/` to `~/.claude/skills/cer-reproduce-check/`

## Quick Start

```bash
# Install skills
cp -r skills/cer-readme-check ~/.claude/skills/
cp -r skills/cer-reproduce-check ~/.claude/skills/

# In Claude Code, run:
/cer-readme-check path/to/README.pdf
/cer-reproduce-check path/to/replication_package/
```

## CER Dataverse

Submit replication packages to: <https://dataverse.harvard.edu/dataverse/cer>

## Contact

Ruochen Dai, 
dairuochenpku@gmail.com,
Associate Professor, Central University of Finance and Economics,
Data Editor, China Economic Review
