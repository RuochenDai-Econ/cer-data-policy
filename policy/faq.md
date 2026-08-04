# CER Data and Code Availability Policy — Frequently Asked Questions

---

**Q1: My data comes from CSMAR / WIND / CNRDS with an institutional license. Is it "restricted data"?**

Check your institution's data use agreement with the database provider. Depending on the license terms, choose either **Tier-0** (no restricted data — you can share the raw data files) or **Tier-1** (raw data cannot be redistributed, but you provide the complete cleaning code and anonymized/intermediate data for public replication). In either case, your README must clearly specify: the data source (official platform name), the exact version or release date, the variables used, and the access steps so that other researchers with the same institutional access can obtain and reproduce your work from scratch. If you are unsure about your specific license terms, consult the Data Editor.

---

**Q2: I used data from a public statistical yearbook / government open data portal / web-scraping. Do I need to mark it as restricted?**

No. Publicly available data is not restricted, regardless of the effort involved in collection. The "effort" of data collection does not constitute a restriction on sharing. Such data should be fully disclosed and included in the replication package.

---

**Q3: How do I choose the correct data tier?**

Apply these principles when choosing a tier:

- **"No restricted data"** applies when all data can be freely shared, including data collected from public sources (e.g., web-scraped data, statistical yearbooks, government open data portals). In these cases, authors should upload the complete raw dataset and full code. Commercial databases like CSMAR/WIND are included here — see Q1.
- **Tier-1** applies when raw data is contractually restricted but authors can provide anonymized, censored, or aggregated intermediate datasets sufficient for public replication. *Rule of thumb: if the data can be downloaded directly from a website to your local computer, you should generally select Tier-1 — unless the data use agreement explicitly restricts the openness of even processed and anonymized data.*
- **Tier-2** applies when data cannot be publicly shared but the Data Editor can be granted private access — including on-site verification at a secure data lab, remote desktop access, or secure file transfer. Survey data processed within university lab environments (e.g., CFPS, CHFS) typically falls under Tier-2. *Rule of thumb: Tier-2 is only applicable if your entire data processing must be conducted within the secure environment of a survey lab.*
- **Tier-3** is reserved for exceptional cases where no form of verification is possible. Authors must justify this in the Cover Letter at initial submission.

When uncertain, select the **least restrictive tier** that applies and consult the Data Editor.

---

**Q4: My survey data must be processed within a secure lab environment and cannot leave the lab. Which tier applies?**

Tier-2. The Data Editor will coordinate private verification — either through secure file transfer, remote desktop access, or an on-site visit to the lab. Your README must document the lab's application procedures, access restrictions, and data version details.

---

**Q5: Can I upload the Compliance Checklist to Dataverse?**

No. The Checklist must be uploaded to the **Elsevier Editorial System** alongside your formal response. Dataverse is for the replication package only.

---

**Q6: The Data Editor found minor discrepancies between my code output and the manuscript tables. What should I do?**

- **Minor variations** (e.g., due to software version differences, fixed random seeds): Update the manuscript tables directly. **Changes in the revised manuscript must be marked in red** and explained in your response letter to the Data Editor. Do not include the discrepancy explanation in the README itself.
- **Substantial differences** that affect the study's core findings: These may require further editorial review or re-review by referees.

In all cases, document the discrepancy and the resolution in your response to the Data Editor.

---

**Q7: I use external Stata commands (e.g., `boottest`, `reghdfe`). How should I handle them?**

Either:

- Include all external `.ado` files in a dedicated subfolder of the replication package, **or**
- List them explicitly in the README with installation instructions (e.g., `ssc install boottest`).

Also note in the README if any command has a long expected runtime.

---

**Q8: What counts as "raw data"? Can I put my cleaned analytical dataset in the raw_data folder?**

CER recognizes two acceptable categories of raw data (see §3.1 for details):

1. **Original source data** — data directly from the primary producer or official distributor in its original, unmodified form (e.g., CSMAR downloaded from the official GTA platform, statistical yearbook files from NBS, CFPS microdata from ISSS).
2. **Peer-reviewed processed data** — a dataset constructed and documented by a peer-reviewed published study may serve as your starting raw data, provided you formally cite the source study and its publicly documented methodology.

What is **not** acceptable as raw data: processed datasets from **non-peer-reviewed** sources (such as informal online forums, non-academic data-sharing platforms). If your current starting data falls into this category, you must either:

- **(a)** Re-acquire the data from the original official source and provide the full cleaning pipeline yourself; or
- **(b)** Treat the processed file as an **intermediate dataset**, and publicly provide the complete cleaning code and log files that document every step from the true raw data to this intermediate file. The non-peer-reviewed processed file alone is not sufficient; the transformation process must be fully transparent and reproducible.

Think: *could another researcher with access to the same original sources start from scratch using only your code and reproduce your exact analytical dataset?* If the answer is no, your replication package is incomplete.

---

**Q9: My raw data came from multiple sources. How should I organize the replication package?**

Each source dataset should be clearly documented with a formal data citation in the README. The folder structure should make it obvious which raw file came from which source. If some sources are restricted, place them in a clearly marked subfolder and document the access protocol for each restricted source individually.

---

**Q10: I have self-collected survey/experimental data. Can I select Tier-2?**

CER strongly recommends publicly releasing a de-identified version of self-collected data (stripped of personal identifiers, containing only the variables needed for analysis). This is standard practice. If your IRB prohibits even anonymized sharing, your README must clearly state: (a) the exact IRB situation and approving institution, and (b) that future researchers should contact the **official institutional body** that oversaw the data collection (not you personally) for access.

---

**Q11: Do I need to include log files even if my data is not restricted?**

Yes. Log files are always required. They are the primary evidence that your code runs from start to finish without errors and produces the reported results. For Tier-2/Tier-3 data, log files are especially critical since the raw data cannot be inspected directly.

---

**Q12: My manuscript requires a DOI link for the Data Availability statement, but the link shows an error page. Is this a problem?**

No — this is completely normal. The DOI link will not function until the Data Editor officially publishes your replication package on Dataverse. Once verification is complete, the Data Editor will publish the package and the DOI will become active. **Typically, you can wait until the Data Editor has published your package before submitting through the Elsevier Editorial System. This ensures your package passes verification without issues and avoids unnecessary delays in the system review cycle.** If there are any issues during the replication check, the Data Editor will contact you directly via email.

---

**Q13: What if the restricted data I used comes from a university or college lab rather than a national survey project?**

The same Tier-2 documentation requirements apply. Your README must provide the institution's full name, the specific lab or research center, the official contact information, and the application procedures. If the lab only provides access to its own members, or if it offers paid remote desktop services, document this clearly. The goal is for any future researcher reading your README to know exactly who to contact and what to expect.

---

