# ppmi-mdsupdrs-mcid
Tools to build longitudinal MDS-UPDRS phenotypes and MCID-based progression labels from PPMI data.

# PPMI MDS-UPDRS Longitudinal & MCID Helper

Tools to transform raw PPMI MDS-UPDRS CSVs into longitudinal, analysis-ready phenotypes with visit-to-visit deltas and MCID-based progression labels.

This repo contains small, reusable helpers and a notebook that:

- Read PPMI MDS-UPDRS Part I–IV CSVs
- Compute per-visit total scores for each part and the global UPDRST
- Derive visit-to-visit deltas, time intervals, and MCID-based worsening flags

---

## Motivation

PPMI provides rich, multi-visit MDS-UPDRS data, but it is not immediately ready for longitudinal modeling. In practice you need to:

- Harmonize total scores across parts and visits  
- Pick a consistent time axis (here: `INFODT`, not just `EVENT_ID`)  
- Compute clinically meaningful changes (deltas) and thresholds (MCID)  

This repo aims to make that preprocessing **transparent, reproducible, and easy to reuse** across notebooks and projects, especially for progression modeling and patient-level trajectory analysis.

---

## Features

- **Parsing of PPMI MDS-UPDRS raw files**
  - Part I (rater, patient questionnaire)
  - Part II (patient questionnaire)
  - Part III (motor exam, including `PDSTATE` ON/OFF when available)
  - Part IV (motor complications)

- **Unified per-visit table**
  - Keys: `PATNO`, `EVENT_ID`, `INFODT`
  - Totals:  
    - `UPDRS1_rater_total`, `UPDRS1_patient_total`  
    - `UPDRS2_total`, `UPDRS3_total`, `UPDRS4_total`  
    - Combined Part I: `UPDRS1_total`  
    - Global: `UPDRST_total`
  - `PDSTATE` preserved from Part III when available (ON / OFF exam)

- **Visit-to-visit longitudinal intervals**
  - Visits sorted by `INFODT` within each `PATNO`
  - Consecutive intervals: current visit → next visit
  - Time between visits: `delta_days`, `delta_years`
  - Deltas for each part and total: `Δ_UPDRS1`, `Δ_UPDRS2`, `Δ_UPDRS3`, `Δ_UPDRS4`, `Δ_UPDRST`

- **MCID-based progression flags**
  - MCID worsening per part:  
    - `MCID_updrs1`, `MCID_updrs2`, `MCID_updrs3`, `MCID_updrs4`, `MCID_updrst`
  - Composite:
    - `MCID_composite_parts` (any part crosses its MCID)
    - `MCID_composite_parts_plus_total` (parts and/or total cross MCID)
  - NA-aware logic:
    - If a delta cannot be computed (missing scores), the MCID flag is `<NA>` rather than silently 0.

---

## Repository structure (suggested)

```text
ppmi-updrs-longitudinal/
├─ notebooks/
│  └─ 01_ppmi_updrs_longitudinal_mcid.ipynb
├─ src/
│  └─ ppmi_updrs/
│     ├─ __init__.py
│     ├─ io.py               # file reading, column selection
│     ├─ scoring.py          # totals per part, combined totals
│     └─ longitudinal.py     # visit-to-visit deltas, MCID flags
├─ data_raw/                 # (gitignored) PPMI CSVs
├─ data_derived/             # (gitignored) derived tables
├─ README.md
└─ LICENSE
```
You can start with just the notebook + a single longitudinal.py and grow from there.

---

## Installation

This project is intentionally light and uses pandas and numpy (and optionally seaborn/matplotlib for exploration).

Example minimal install:

```bash
pip install pandas numpy
```

Or via a requirements file:

```bash
pip install -r requirements.txt
```

---

## Usage (high level)

   1. Export the relevant MDS-UPDRS CSVs from PPMI, e.g.:

      *   MDS-UPDRS_Part_I_*.csv

      *   MDS-UPDRS_Part_I_Patient_Questionnaire_*.csv

      *   MDS_UPDRS_Part_II__Patient_Questionnaire_*.csv

      *   MDS-UPDRS_Part_III_*.csv

      *   MDS-UPDRS_Part_IV__Motor_Complications_*.csv

   2. Run the notebook 01_ppmi_updrs_longitudinal_mcid.ipynb, which:

      *   Reads each file into a DataFrame

      *   Cleans and keeps only the total scores per part

      *   Merges to a single per-visit table (df_totals)

      *   Parses INFODT into INFODT_dt (datetime)

      *   Builds visit-to-visit intervals with MCID labels (intervals)

   3. (If using as a module) Example code:

```python
from ppmi_updrs.io import load_updrs_parts
from ppmi_updrs.longitudinal import compute_updrs_deltas_and_mcid_visit_to_visit

df_totals = load_updrs_parts(path_to_csv_folder="data_raw")
intervals = compute_updrs_deltas_and_mcid_visit_to_visit(df_totals)

intervals.to_csv("data_derived/ppmi_updrs_intervals.csv", index=False)
```

---

## MCID thresholds

By default, MCID worsening thresholds are:

   * Part I: ΔUPDRS1 ≥ 2.45 (optional / can be toggled)

   * Part II: ΔUPDRS2 ≥ 2.51

   * Part III: ΔUPDRS3 ≥ 4.63

   * Part IV: ΔUPDRS4 ≥ 1.00

   * Total: ΔUPDRST ≥ 10.59

These values live in a single helper function (mcid_from_delta) and can be easily updated to match:

  * A specific publication

  * Your analysis plan / protocol

  * Sensitivity analyses with stricter or more liberal thresholds

---

## Caveats & notes

  * This code assumes PPMI variable names and file layouts as of 2025. If PPMI changes their exports, you may need to adjust column names in the I/O section.

  * Always confirm that your use of PPMI data complies with:

     * PPMI Data Use Agreements

     * Local IRB / ethics approvals

     * Institutional policies for de-identification and data handling

  * This repo focuses on phenotype construction, not modeling; use the resulting intervals table as input for your regression, survival, or ML pipelines.

---

## Author

> This notebook and helpers were developed by: Ana Jimena Hernández Medrano, MD, MSc  
> Clinical researcher & data scientist working on PD genetics and progression modeling in underrepresented populations.

---

## MIT License

```java
Copyright (c) [2025] [Ana Jimena Hernández Medrano]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the “Software”), to deal
in the Software without restriction, including without limitation the rights 
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell 
copies of the Software, and to permit persons to whom the Software is 
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in 
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED “AS IS”, WITHOUT WARRANTY OF ANY KIND, EXPRESS OR 
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, 
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE 
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER 
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING 
FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS 
IN THE SOFTWARE.
```
