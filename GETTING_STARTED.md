# Getting Started

This guide is for anyone picking up this project from scratch. Read this before opening Power BI or touching any files.

---

## What This Project Is

A full hands-on capstone project that covers every concept tested on the Microsoft PL-300 Power BI Data Analyst exam. You will build a complete Power BI solution from raw data -- including data prep, a star schema data model, a multi-page report, and security/publishing in Power BI Service.

The study guide and progress tracker are **reference documents**, not a tutorial. They tell you what to do and give you the exact code, but they assume you are following along in Power BI. If you have never used Power BI before, pair this project with a beginner video course or Microsoft Learn first.

---

## Prerequisites

### Software
- **Power BI Desktop** -- free download at https://powerbi.microsoft.com/desktop
- Use the latest version; some UI elements (especially Phase 4) may look different on older versions

### Account
- **Free Power BI account** -- covers everything in Phases 1-3 (Desktop only)
- **Power BI Pro or Premium Per User (PPU) license** -- required for Phase 4 (publishing, Service features, RLS in Service, subscriptions)
- Pro is included with Microsoft 365 E3/E5, or available as a standalone ~$10/month
- You can start a 60-day Pro trial at app.powerbi.com if you do not have a license

### Skills assumed
- Basic familiarity with data concepts (rows, columns, tables, relationships)
- No DAX or M code experience required -- all code is provided
- No prior Power BI experience required, but expect a steeper learning curve on Phases 1-2

---

## Step 1: Download the Dataset

The raw data files are not included in this repo. Download them before you start.

1. Go to https://www.kaggle.com/datasets/brandao/diabetes
2. Download the dataset (you may need a free Kaggle account)
3. You need two files:
   - `diabetic_data.csv` -- 101,766 rows, 50 columns
   - `IDS_mapping.csv` -- 67 rows, 2 columns
4. Save both files to the same folder on your computer -- you will point Power BI to this folder

---

## Step 2: Understand the Project Structure

The project is split into four phases that match the four PL-300 exam domains:

| Phase | What you build | Where you work |
|---|---|---|
| Phase 1 | Power Query transformations, dimension tables, staging query | Power BI Desktop -- Power Query Editor |
| Phase 2 | Star schema relationships, DAX measures, Date table | Power BI Desktop -- Model and Data views |
| Phase 3 | Report pages, visuals, filters, bookmarks, theme | Power BI Desktop -- Report view |
| Phase 4 | RLS, publish, dashboard, subscriptions | Power BI Desktop + Power BI Service (web) |

**Do the phases in order.** Each phase depends on the previous one. Do not skip ahead.

---

## Step 3: Understand the Two Reference Documents

### `PL300_Project_Progress.md`
Your build checklist. Work through it top to bottom. Each checkbox is one discrete step. When you complete a step, check it off. If something is not working, re-read the step -- the exact configuration is documented.

### `PL300_Study_Guide.md`
Your exam reference. Organized by concept number (1-36). Each concept has:
- **What to Know** -- what the PL-300 exam tests on this topic
- **What We Did** -- the exact steps, M code, or DAX used in this project
- **Exam Tips** -- common exam traps and gotchas

Use the study guide when:
- You need the exact M code or DAX for a step
- You want to understand *why* a decision was made
- You are reviewing for the exam after completing the build

---

## Step 4: Key Concepts to Understand Before You Start

These are the most important ideas in the project. If any of these are unfamiliar, look them up before starting Phase 1.

**Star schema** -- A data model pattern with one fact table (transactions) surrounded by dimension tables (descriptive attributes). This project builds a star schema with one fact table and six dimension tables. Think of it like a hub and spoke -- the fact table is the hub.

**Staging query** -- A frozen copy of the raw source data in Power Query. All dimension tables are built from this copy, not from the fact table. This prevents a common breaking change where modifying the fact table breaks all the dimension tables downstream. This is the single most important pattern in Phase 1.

**DAX** -- Data Analysis Expressions. The formula language used to create calculated measures in Power BI. All 19 measures are provided in the study guide -- you do not need to write them from scratch, but you should understand what each one does.

**M code** -- The language behind Power Query transformations. You will write two custom columns using M. The exact code is in the study guide.

**Row-Level Security (RLS)** -- A feature that restricts which rows of data a user can see based on their role. Defined in Desktop, tested in Desktop, then assigned to users in Power BI Service. Covered in Phase 4.

---

## Step 5: Recommended Build Order Within Each Phase

Follow the progress tracker in order, but here are the critical sequencing rules:

**Phase 1:**
- Create `diabetic_data_staging` (the frozen raw source) **before** creating any dimension tables
- Create all dimension tables as **Reference queries** from `diabetic_data_staging`, not from `Fact_Encounters`
- Set Enable Load = OFF on `diabetic_data_staging` and `IDS_mapping` before closing Power Query

**Phase 2:**
- Fix all data types **before** creating relationships -- mismatched types cause relationship errors
- Create the `_Measures` table **before** writing any DAX measures
- Write simpler measures first (COUNTROWS, AVERAGE) before dependent measures (DIVIDE, CALCULATE)

**Phase 3:**
- Import the custom theme **before** building visuals -- it sets the default colors
- Build Page 1 (Executive Summary) first -- drill-through and tooltip pages depend on it
- Create the tooltip page last -- it needs the measures to already exist

**Phase 4:**
- Define and test RLS in Desktop **before** publishing -- you cannot create RLS roles in the Service
- Publish **after** RLS is confirmed working

---

## Common Mistakes to Avoid

| Mistake | What happens | How to avoid |
|---|---|---|
| Building dim tables from `Fact_Encounters` instead of `diabetic_data_staging` | Removing a column from the fact table breaks all dim tables | Always reference `diabetic_data_staging` |
| Renaming the source query before creating reference queries | Breaks all downstream queries | Create reference queries first, rename after |
| Leaving ID columns as Whole Number type | Sigma symbols appear, relationships may fail | Change to Text type before creating relationships |
| Setting Summarization to Count on conditional formatting | Color scale shows row counts instead of values | Always check Summarization -- set to Average for numeric columns |
| Deleting the auto-generated Back button on drill-through pages | Users cannot navigate back | Leave the Back button -- Power BI generates it automatically |
| Testing RLS by publishing without testing in Desktop first | Broken RLS is harder to debug in Service | Always use Modeling > View as > [role] in Desktop first |

---

## What This Project Does Not Cover

The capstone covers the majority of PL-300 topics but not everything. Before booking your exam, also study:

- **Deployment pipelines** (Dev/Test/Prod promotion in Power BI Service)
- **Paginated reports** (different from regular Power BI reports -- built in Power BI Report Builder)
- **Performance optimization** (aggregations, query reduction, Import vs DirectQuery tradeoffs)
- **Endorsement** (certifying and promoting semantic models in the Service)
- **Composite models** (combining Import and DirectQuery in one model)

Recommended resources for these topics:
- Microsoft Learn: https://learn.microsoft.com/en-us/training/paths/prepare-data-power-bi/
- Guy in a Cube (YouTube) -- excellent free PL-300 content
- Practice exams on MeasureUp or Whizlabs

---

## You Are Ready When...

Before booking the PL-300 exam, make sure you can:

- [ ] Explain the staging query pattern and why it matters
- [ ] Write a basic DAX measure using CALCULATE and DIVIDE without looking it up
- [ ] Describe the difference between Import, DirectQuery, and Live Connection
- [ ] Explain when a gateway is required for scheduled refresh
- [ ] Define RLS from scratch in a new file
- [ ] Explain the difference between a dashboard and a report in Power BI Service
- [ ] Score 80%+ consistently on practice exams

Good luck.
