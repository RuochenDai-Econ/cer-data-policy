---
description: |
  Automatically verify a CER replication package by: (1) inspecting folder structure and file organization, (2) fixing absolute paths in code to use relative paths, (3) running the full replication pipeline, (4) comparing generated results and logs against the provided ones. Use when: checking if a replication package actually reproduces, verifying code runs end-to-end, pre-submission reproducibility audit, Data Editor verification workflow. Trigger phrases: "reproduce this package", "verify replication", "run the replication", "check reproducibility", "reproduce check", "run this replication package".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
user-invocable: true
---

# CER Replication Package — Automated Reproduce Check

Verify that a replication package runs end-to-end and produces results consistent with the provided outputs. Designed for the CER Data Editor's verification workflow but usable by authors for pre-submission self-checks.

## Workflow

Five phases, executed in order. Do not skip phases — each phase informs the next.

---

### Phase 1: Inspect — Folder Structure and File Inventory

Map the entire package before touching anything.

**1.1 Locate the package root**

The user provides a path to a zipped package or a directory. If a zip file, unzip first:

```bash
unzip -o <package.zip> -d /tmp/cer_reproduce/
```

**1.2 Generate a file tree**

```bash
find <pkg_root> -type f | head -100
```

Also list directories only:

```bash
find <pkg_root> -type d | sort
```

**1.3 Identify the software**

Look for telltale files:
- `.do` files → Stata
- `.R` or `.Rmd` files → R  
- `.py` or `.ipynb` files → Python
- `Makefile` → mixed

Check the README for declared software and version.

**1.4 Identify the master script**

Search for master/entry-point scripts:
```bash
find <pkg_root> -iname "main.do" -o -iname "master.do" -o -iname "_main.do" -o -iname "run_all.R" -o -iname "main.R" -o -iname "run.py" -o -iname "Makefile" -o -iname "run.sh"
```

If not found, look at the README for instructions, or identify the top-level `.do`/`.R`/`.py` file.

**1.5 Inventory all files by category**

Create a structured inventory:

| Category | Pattern | Count |
|----------|---------|-------|
| Code (cleaning) | `.do` files in `raw_data_cleaning_program/` | |
| Code (analysis) | `.do` files in `analysis_program/` | |
| Public raw data | files in `public_raw_data/` | |
| Restricted raw data | files in `restricted_raw_data/` | |
| Intermediate data | files in `intermediate_data/` | |
| Results | `.xlsx`, `.csv`, `.tex`, `.png`, `.pdf` in `results/` | |
| Logs | `.log`, `.smcl` in `logs/` | |
| Documentation | `README.*` | |

**1.6 Data Lineage Trace (Critical for Tier-1/Tier-2)**

For every data file (`.dta`, `.csv`, `.xlsx`) in the package, determine whether it is raw data or intermediate data. Then trace its provenance:

1. **Classify each data file:**
   - **Raw data**: obtained directly from the original source, unmodified. For Tier-2, raw data is typically NOT in the package.
   - **Intermediate data**: derived/processed/merged from raw sources. These are IN the package and must have documented provenance.

2. **For each intermediate data file, find its generating code:**
   ```bash
   # Search all code files for references to each data file
   for f in <data_files>; do
     echo "=== $f ===" 
     grep -rn "$f" --include="*.do" --include="*.R" --include="*.py" <pkg_root>
   done
   ```
   
   Specifically check for:
   - `save` commands that create the file (Stata: `save "filename", replace`)
   - `export` / `write` commands that produce it
   - The file being read via `use` / `merge` / `read` but never written → this means the generation code is MISSING

3. **Identify files with missing generation code:**
   ```bash
   # For each data file, check: is it ever SAVED by any code?
   for f in <data_files>; do
     saved=$(grep -rl "save.*$f" --include="*.do" <pkg_root> | wc -l)
     if [ "$saved" -eq 0 ]; then
       echo "MISSING generation code for: $f"
     fi
   done
   ```

4. **Check README claims against code:**
   - Does the README state which raw source each intermediate file comes from?
   - Does the README name the specific `.do` file that generates each intermediate file?
   - If the README says "from China Statistical Yearbook" but no code extracts/constructs the `.dta` from the yearbook, flag it.

5. **Verify code location:**
   - Data cleaning/generation code should be in the `raw_data/` folder (or equivalent), separate from analysis code.
   - If cleaning code is mixed with analysis code, flag as a structural issue.

**1.7 Report Phase 1 findings**

Produce a summary before proceeding. Flag structural issues:
- No `main_clean.do` in `raw_data_cleaning_program/`
- No `main.do` in `analysis_program/`
- No README found  
- No log files found
- No `results/` folder found
- Missing `public_raw_data/` or `restricted_raw_data/` separation
- Raw and intermediate data mixed in the same folder
- Single monolithic code file (no modular structure — all analysis in one `.do` file)
- README mentions software not detected in package
- **Data lineage gaps**: Data files with no generation code; intermediate files not traceable to raw source; README claims not matching actual code

Include a **Data Lineage table** in the report:

| Data File | Type (raw/intermediate) | Raw Source | Generating Code | Status |
|-----------|------------------------|------------|-----------------|--------|
| prov_data.dta | intermediate | CRHPS + Yearbook | **MISSING** | ❌ |
| positive_CFPS.dta | intermediate | CFPS 2016 | positive_CFPS.do | ✅ |

**Ask the user to confirm before proceeding** if major structural issues are found, or if the software is not installed.

---

### Phase 2: Fix Paths — Make Code Portable

Scan ALL code files for absolute paths and fix them.

**2.1 Find absolute paths**

```bash
grep -rn 'C:/Users\|C:\\Users\|/Users/[a-zA-Z]\|/home/[a-zA-Z]\|D:/\|E:/\|F:/\|setwd(' <pkg_root> --include="*.do" --include="*.R" --include="*.py" --include="*.m"
```

Also check for `cd "C:` or `cd /Users` patterns in Stata code.

**2.2 Determine the working directory strategy**

By convention, the replication package root should be the working directory. Two approaches:

- **Stata**: Add `cd "REPO_ROOT"` at the top of the master script, and ensure all paths are relative to it. If the code uses globals like `global DATA "C:/.../data"`, replace with `global DATA "./data"`.
- **R**: Replace `setwd("C:/Users/...")` with `setwd("REPO_ROOT")`. Replace absolute file paths.
- **Python**: Replace absolute paths in `pd.read_csv("C:/...")` etc. with relative paths.

**2.3 Apply fixes**

For each file with absolute paths:
1. Read the file
2. Identify the absolute path pattern
3. Replace with relative equivalent based on the package structure
4. Add a `cd` or `setwd` at the top of the master script pointing to the package root

**If the code already uses relative paths**: Report "No path fixes needed."

**2.4 Verify the fix**

After fixing, re-scan to confirm no absolute paths remain:

```bash
grep -c 'C:/Users\|C:\\Users\|/Users/[a-zA-Z]\|/home/[a-zA-Z]' <fixed_files>
```

**2.5 Document what was changed**

List every file modified and the change made. This goes in the final report.

---

### Phase 3: Run — Execute the Replication

**3.1 Set up environment**

Based on detected software:

**Stata:**
```bash
# Check Stata is available
which stata || which stata-se || which stata-mp
# Install packages if needed (from README list)
stata -b do setup_packages.do   # if provided
```

**R:**
```bash
# Check R is available
which R
# Install packages from README list or renv
R -e 'install.packages(c(...))'
```

**Python:**
```bash
which python3
pip install -r requirements.txt  # if provided
```

**3.2 Run the master script**

Execute from the package root:

**Stata:**
```bash
cd <pkg_root> && stata -b do main.do
```
This produces `main.log`. If the master script is a different file, use that.

**R:**
```bash
cd <pkg_root> && Rscript main.R 2>&1 | tee reproduce.log
```

**Python:**
```bash
cd <pkg_root> && python3 main.py 2>&1 | tee reproduce.log
```

**3.3 Capture results**

After execution:
- Note the **exit code** (0 = success, non-zero = error)
- Capture the **console output** and save as `<pkg_root>/_cer_reproduce_output.log`
- Check if expected output files were generated
- Note any error messages, warnings, or missing dependencies

**3.4 Handle failures**

If the master script fails:
- Read the error message
- Determine if it's fixable (missing package, wrong software version) or fatal (missing data, broken code)
- If fixable (e.g., `ssc install` needed), apply the fix and retry once
- If fatal, document the error and skip Phase 4 comparison

---

### Phase 4: Compare — Results and Logs

Compare newly generated outputs against the provided ones.

**4.1 Compare log files**

If the package includes log files (`.log`, `.smcl`), compare against the newly generated log:

```bash
# Strip timestamps and runtime lines before comparing
grep -v '^.......running.*\.\.\.$' <old_log> | grep -v '^Stata.*Copyright' | grep -v '^.*running on' > /tmp/old_clean.log
grep -v '^.......running.*\.\.\.$' <new_log> | grep -v '^Stata.*Copyright' | grep -v '^.*running on' > /tmp/new_clean.log
diff /tmp/old_clean.log /tmp/new_clean.log
```

Key differences to flag:
- Different coefficient values
- Different sample sizes (N)
- Missing or extra tables in output
- Different standard errors
- Different significance levels

Differences to IGNORE (normal):
- Timestamps and dates
- Stata version/copyright banners
- Runtime/performance messages
- `set seed` differences (if seed is fixed, this shouldn't happen)
- File path differences (absolute vs relative)

**4.2 Compare output files**

For each output file type:

**Excel (.xlsx):**
```bash
python3 -c "
import pandas as pd
old = pd.read_excel('<old>')
new = pd.read_excel('<new>')
print('Columns match:', list(old.columns) == list(new.columns))
print('Shape match:', old.shape == new.shape)
print('Max difference:', (old.select_dtypes('number') - new.select_dtypes('number')).abs().max().max())
"
```

**CSV (.csv):**
```bash
diff <(python3 -c "print(open('<old>').read())") <(python3 -c "print(open('<new>').read())")
```

**LaTeX tables (.tex):**
```bash
# Compare numbers only, ignore formatting
grep -oE '[0-9]+\.[0-9]+' <old> > /tmp/old_nums.txt
grep -oE '[0-9]+\.[0-9]+' <new> > /tmp/new_nums.txt
diff /tmp/old_nums.txt /tmp/new_nums.txt
```

**Figures (.png, .pdf):**
- Check both files exist
- Compare file sizes (major difference may indicate issue)
- Visual comparison is manual — flag as "requires manual check"

**4.3 Classify discrepancies**

| Type | Severity | Definition |
|------|----------|------------|
| Exact match | ✅ PASS | Numerically identical |
| Minor float | ⚠️ WARN | Differences < 1e-6 (floating point) |
| Visible difference | ❌ FAIL | Differences > 1e-6 in coefficients/statistics |
| Missing output | ❌ FAIL | Output file not generated |
| Extra output | ⚠️ WARN | New output file not in original package |
| N/A | — | Data not available for comparison |

---

### Phase 5: Report

Produce a structured verification report:

```markdown
# CER Replication Verification Report

**Package:** `<path or DOI>`
**Software:** <Stata 17 / R 4.2 / Python 3.10>
**Master script:** <main.do>
**Date:** <today>

## Phase 1: Package Structure

### Directory Tree
<file tree>

### File Inventory
| Category | Count |
|----------|-------|
| Code files | X |
| Raw data files | X |
| Intermediate files | X |
| Result files | X |
| Log files | X |

### Data Lineage
| Data File | Type | Claimed Raw Source | Generating Code | Status |
|-----------|------|--------------------|-----------------|--------|
| file.dta | intermediate | Yearbook 2020 | **MISSING** | ❌ |
| file.dta | intermediate | CFPS 2016 | positive_CFPS.do | ✅ |

### Structure Issues
- <any issues found>

## Phase 2: Path Fixes

### Files Modified
| File | Original Path | Replaced With |
|------|--------------|---------------|
| main.do | C:/Users/author/data | ./data |

### No Changes Needed
<list of files that were already relative>

## Phase 3: Execution

### Command
```
cd <pkg_root> && stata -b do main.do
```

### Result
- **Exit code:** 0 / non-zero
- **Runtime:** X minutes Y seconds
- **Errors:** <none / details>
- **Warnings:** <none / details>

## Phase 4: Comparison

### Log File Comparison
- Provided log: `results/main.log` (XX KB)
- Generated log: `main.log` (XX KB)
- **Result:** ✅ Match / ⚠️ Minor differences / ❌ Significant differences

<If differences: list each difference with line numbers>

### Output File Comparison
| Output File | Provided | Generated | Status | Max Diff |
|-------------|----------|-----------|--------|----------|
| results/table1.xlsx | 15 KB | 15 KB | ✅ | 0 |
| results/figure1.png | 120 KB | 118 KB | ⚠️ Manual check | — |

## Summary

- **Reproducibility:** ✅ Full / ⚠️ Partial / ❌ Failed
- **Files matched:** X / Y
- **Files differed:** X
- **Files not generated:** X

## Recommended Actions
<if any issues: specific fixes needed>
<if all clear: Package is ready for acceptance>
```

---

## Software Detection and Commands

| Software | File extensions | Run command | Log output | Package install |
|----------|----------------|-------------|------------|-----------------|
| Stata | `.do` | `stata -b do <file>` | `.log` | `ssc install`, `net install` |
| Stata MP | `.do` | `stata-mp -b do <file>` | `.log` | `ssc install`, `net install` |
| R | `.R`, `.Rmd` | `Rscript <file>` | `.Rout` | `install.packages()` |
| Python | `.py` | `python3 <file>` | capture stdout | `pip install` |
| MATLAB | `.m` | `matlab -batch "run('<file>')"` | `.diary` | manual |

## Important Notes

- **Always check which software is installed** before running. If Stata is not available, report it and stop.
- **Never overwrite original files.** Work in a copy: `cp -r <pkg_root> /tmp/cer_reproduce_work/`.
- **Respect run time limits.** If README says 14 hours, warn the user before starting. For runs >30 minutes, ask user confirmation.
- **Log everything.** Save all terminal output for the final report.
- **For Tier-2 packages**: raw data is missing by design. The code should fail gracefully at the data-loading step — this is expected and should be noted, not flagged as an error.
- **For packages without raw data**: skip the data-cleaning steps (they can't run), but check that the analysis code runs on provided intermediate data.
