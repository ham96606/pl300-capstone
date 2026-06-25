# PL-300 Power BI Data Analyst: Capstone Project

A hands-on capstone project covering all four PL-300 exam domains, built using real clinical data. Every concept in the study guide was applied step-by-step in Power BI Desktop and Power BI Service.

---

## What's in This Repo

| File | Description |
|---|---|
| `study-guide/PL300_Study_Guide.md` | Full exam study guide -- 36 numbered concepts with What to Know, What We Did, and Exam Tips |
| `study-guide/PL300_Project_Progress.md` | Phase-by-phase checkbox tracker covering every PL-300 skill |
| `theme/ExampleTheme.json` | Sample custom JSON theme demonstrating the theme file structure |
| `.gitignore` | Excludes .pbix and raw data files |

---

## Dataset

This project uses the **Diabetes 130 US Hospitals (1999-2008)** dataset.

**You must download it yourself** -- the raw data is not included in this repo.

- Kaggle: https://www.kaggle.com/datasets/brandao/diabetes
- UCI ML Repository: https://archive.ics.uci.edu/dataset/296/diabetes+130-us+hospitals+for+years+1999-2008

**Attribution (required by CC BY 4.0 license):**
> Strack, B., DeShazo, J.P., Gennings, C., Olmo, J.L., Ventura, S., Cios, K.J., and Clore, J.N. (2014). Impact of HbA1c Measurement on Hospital Readmission Rates: Analysis of 70,000 Clinical Database Patient Records. BioMed Research International, Article ID 781670. http://dx.doi.org/10.1155/2014/781670

You will need two files:
- `diabetic_data.csv` -- 101,766 rows, 50 columns (main fact/patient data)
- `IDS_mapping.csv` -- 67 rows, 2 columns (ID lookup mappings)

---

## Project Overview

### Dataset
Real clinical data from 130 US hospitals covering diabetic patient encounters from 1999-2008. The project analyzes hospital readmission rates and the impact of A1c testing on patient outcomes.

### Four Phases (mirroring the PL-300 exam domains)

| Phase | Domain | Coverage |
|---|---|---|
| Phase 1 | Prepare the Data (25-30%) | Power Query, data profiling, cleaning, dimension tables, M code |
| Phase 2 | Model the Data (25-30%) | Star schema, relationships, DAX measures, Date table |
| Phase 3 | Visualize and Analyze (25-30%) | Cards, bar charts, slicers, drill-through, AI visuals, bookmarks, conditional formatting, filters, tooltips, themes |
| Phase 4 | Manage and Secure (15-20%) | RLS, publishing, workspaces, scheduled refresh, sensitivity labels, dashboards, subscriptions, dataflows |

### Star Schema Design
```
diabetic_data_staging (raw source -- not loaded to model)
        |
        ├── Dim_Patient
        ├── Dim_PrimaryDiagnosis
        ├── Dim_A1cResult
        ├── Dim_AdmissionType
        ├── Dim_DischargeDisposition
        ├── Dim_AdmissionSource
        └── Fact_Encounters

_Measures (19 DAX measures)
Dim_Date (CALENDAR 1999-2008)
```

---

## How to Use This Repo

1. Download the dataset from Kaggle or UCI (links above)
2. Read through `PL300_Project_Progress.md` to see every step in order
3. Follow `PL300_Study_Guide.md` for the exact M code, DAX, and Power BI instructions for each concept
4. Build the report yourself in Power BI Desktop -- the learning is in the doing

---

## PL-300 Skills Covered

Every skill tested on the PL-300 exam is covered, including:

- Data profiling and quality resolution
- Staging query pattern and reference queries
- Star schema design and relationships
- 19 DAX measures (CALCULATE, DIVIDE, ALL, ALLEXCEPT, RANKX, and more)
- All three filter types (report, page, visual)
- AI visuals (Key Influencers, Decomposition Tree)
- Bookmarks and button navigation
- Row-Level Security (define, test, publish, assign)
- Scheduled refresh and gateway requirements
- Dashboards, subscriptions, and dataflows

See `PL300_Project_Progress.md` for the full checklist.

---

## License

Study guide and project documentation: [MIT License](LICENSE)

Dataset: CC BY 4.0 -- see attribution above.
