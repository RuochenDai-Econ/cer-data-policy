---
description: |
  Automatically verify a CER replication package by: (1) inspecting folder structure and file organization, (2) scanning for absolute paths in code, (3) verifying code coherence and runnability WITHOUT re-running the pipeline (default), (4) checking that the provided logs and results are consistent with the code. A full pipeline re-run happens only when the editor explicitly asks. Use when: checking if a replication package actually reproduces, verifying code runs end-to-end, pre-submission reproducibility audit, Data Editor verification workflow. Trigger phrases: "reproduce this package", "verify replication", "run the replication", "check reproducibility", "reproduce check", "run this replication package".
allowed-tools: Bash, Read, Write, Edit, Glob, Grep
user-invocable: true
---

# CER Replication Package — Automated Reproduce Check

Verify that a replication package's code is coherent and would run with minimal manual intervention, and that the provided logs and results are consistent with the code. **By default the pipeline is NOT re-run**: the editor relies on the author-provided logs plus code inspection; a full re-run happens only when the editor explicitly asks for one (runtimes are long, restricted data/API keys are often missing, and the author's logs already demonstrate execution). Designed for the CER Data Editor's verification workflow but usable by authors for pre-submission self-checks.

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

5. **The single most important finding: is there ANY raw→intermediate construction code?**
   - If the package ships derived data files (e.g., tablewise/figurewise `.dta`) with **no upstream construction code at all**, that is a BLOCKING failure (H11/H12) — for Tier-1/Tier-2 especially. A blanket "intellectual property" or "confidentiality" justification for withholding the construction pipeline is NEVER an accepted waiver: **all code and all construction methods must be disclosed — no exceptions.** Published science is a contribution to knowledge, not a patent; "proprietary" scoring algorithms, network measures, and identifier crosswalks are all covered. A polished, self-verifying analysis layer does not compensate for a missing construction layer.
   - Also verify the folder layer exists: no `public_raw_data/` / `restricted_raw_data/` / `raw_data_cleaning_program/` separation is itself a structural failure to report.

6. **Verify code location:**
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

### Phase 3: Verify Code Coherence and Runnability (default — do NOT re-run)

**Default policy: no full pipeline re-run.** The editor verifies by inspection: the author's own logs already demonstrate that the code ran, and re-running costs 60–90+ minutes and often needs restricted data or API keys that are unavailable. A full re-run happens only when the editor explicitly asks.

**3.1 Code coherence — the pipeline hangs together**

- **Master scripts:** does `main.do` exist, and does every do-file it calls actually exist on disk (exact name, incl. case)? List all files referenced by the master scripts and diff against the files present:
  ```bash
  # e.g. extract `do "..."` targets from main.do / main_clean.do and check each exists
  grep -oE 'do "\$[a-zA-Z]+/[^"]+"' <master>.do | sed 's/do "//;s/"//' | while read -r p; do
    [ -e "<pkg_root>/$p" ] || echo "MISSING: $p"
  done
  ```
- **Dependency order:** does `main_clean.do` run construction steps in valid order (each step's inputs produced by earlier steps)? Trace the chain raw → intermediate → analysis → results.
- **Data references resolve:** every `use` / `merge` / `import` path in the code points to a file present in the package, or to a documented restricted/downloadable source. Grep the do-files and check each path on disk.
- **Output targets exist:** every `save` / `export` / `graph export` writes into a folder that exists (or is `capture mkdir`'d).

**3.2 Easy runnability — minimal manual intervention**

- **Single-edit root:** all paths go through one global macro (`$ROOT`) or are relative; zero absolute paths (Phase 2 covers the scan). Individual do-files should run standalone or define the fallback (`if "$ROOT" == "" global ROOT "."`).
- **Packages:** README and/or master script list all required packages with install commands (`ssc install ...`); no unlisted dependencies.
- **No hidden setup:** setup steps (edit config, download data, API keys) explicitly documented in README instructions.
- **Version statement:** the code's `version` / software requirement matches the README.

**3.3 Running code (ONLY with the editor's explicit request)**

Never run anything on your own initiative. When the editor asks:

- **Smoke test (fast):** run a tiny subset in a copy under `/tmp` — e.g. one exhibit do-file, or only the package-install block of `main.do` — to catch immediate failures.
- **Full re-run:** work in a copy (`cp -r <pkg_root> /tmp/cer_reproduce_work/`), never in the original. Check the software is available and install the packages the README lists:

```bash
which stata || which stata-se || which stata-mp   # Stata
which R                                            # R
which python3                                      # Python
```

  Then run the master script from the package root, capturing a log:

```bash
cd <pkg_root> && stata -b do main.do               # Stata → main.log
cd <pkg_root> && Rscript main.R 2>&1 | tee reproduce.log
cd <pkg_root> && python3 main.py 2>&1 | tee reproduce.log
```

  Then go to Phase 4.4 and compare the generated outputs against the provided ones. For anything >30 minutes, always confirm with the editor first.

---

### Phase 4: Verify Provided Logs and Results Are Consistent with the Code

Compare the author-provided logs and results against the **code** (not against a new run). The generated-vs-provided diff in §4.4 applies only when a full re-run was requested and completed.

**4.1 Log completeness**

- Every do-file must have a corresponding log (per-exhibit or per-step, plus the master log).
- Each log ends cleanly: Stata `end of do-file` / `log close`, R no traceback, Python no `Traceback`.
- Grep for errors — with care: Stata `r(###)` has false positives like `legend(r(1)...)` in graph options; always check the context line before flagging.

**4.2 Log vs code consistency**

- The sequence of executed do-files in the master log matches the call order in `main.do` (spot-check).
- Key numbers in the logs are plausible against README claims: sample sizes (`Number of obs`), dataset dimensions, number of clusters.

**4.3 Results vs code consistency**

- For every exhibit in the README's script-to-output mapping, the output file exists in `results/` with the documented name and format.
- Timestamps agree: result files and log `closed on` dates come from the same run.
- Spot-check one or two numbers: a coefficient in `results/TableX` vs the same estimate printed in the corresponding log.

**4.4 Generated-vs-provided comparison (only after a full re-run, if requested)**

Compare newly generated outputs against the provided ones:

**Compare log files**

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

**Compare output files**

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

**Classify discrepancies**

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

## Phase 3: Code Coherence & Runnability

### Coherence
- Master scripts call only do-files that exist: <yes/no + missing list>
- Dependency order valid: <yes/no>
- All `use`/`save` paths resolve: <yes/no + unresolved list>

### Runnability
- Single-edit root / relative paths: <yes/no>
- Packages listed with install commands: <yes/no>
- Setup steps documented: <yes/no>
- Smoke test (if requested): <result>

## Phase 4: Log & Results Consistency (vs code)

### Log completeness
- Logs: X/X do-files covered; all end cleanly: <yes/no>; errors: <none / details>

### Log vs code
- Executed sequence matches main.do: <yes/no>
- Key Ns plausible vs README: <details>

### Results vs code
- All exhibits in README mapping present in results/: <yes/no + missing>
- Timestamps consistent: <yes/no>
- Spot-checked numbers match log: <details>

### Generated-vs-provided (only if a full re-run was requested)
| Output File | Provided | Generated | Status | Max Diff |
|-------------|----------|-----------|--------|----------|
| results/table1.xlsx | 15 KB | 15 KB | ✅ | 0 |
| results/figure1.png | 120 KB | 118 KB | ⚠️ Manual check | — |

## Summary

- **Structure/lineage:** ✅ / ⚠️ / ❌
- **Portability:** ✅ / ⚠️ / ❌ (absolute paths? single-edit root?)
- **Author's own run (logs):** ✅ complete / ❌ gaps — X/Y logs clean
- **Reproducibility (inspected, not re-run):** ✅ expected to run / ⚠️ concerns
- **Blocking issues:** <none / list>

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

- **Default: do NOT re-run the pipeline.** Verify by code inspection plus the author-provided logs (Phases 3–4). A full re-run happens only when the editor explicitly asks; for anything >30 minutes always confirm first.
- **If a re-run IS requested:** work in a copy (`cp -r <pkg_root> /tmp/cer_reproduce_work/`), never overwrite originals; check the software is installed first (if not, report and stop); respect run-time limits; log everything.
- **Restricted data in the package is normal for the editor's copy.** Authors send the editor complete packages (including restricted raw data) for verification; the README may state those files are not included because it describes the public package. Do not flag this as an issue — but do note in the report that the author must strip restricted files before Dataverse upload.
- **For Tier-2 packages**: raw data is missing by design. The code should fail gracefully at the data-loading step — this is expected and should be noted, not flagged as an error.
- **For packages without raw data**: skip the data-cleaning steps (they can't run), but check that the analysis code runs on provided intermediate data.
