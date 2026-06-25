# PL-300 Power BI Data Analyst: Study Guide
## Based on Capstone Project Experience

---

## How to Use This Guide
Each concept below was applied hands-on in the Diabetic Hospital Readmissions capstone project. The "What to Know" section covers what the exam tests. The "What We Did" section ties it back to the project so you can recall it from muscle memory.

---

## PHASE 1: Prepare the Data (25-30% of exam)

---

### 1. Connecting to Data Sources

**What to Know:**
- Power BI can connect to flat files (CSV, Excel), databases, cloud sources, and more
- Each source has a connector accessed via Get Data
- Power Query opens automatically after connecting

**What We Did:**
- Connected to `diabetic_data.csv` (101,766 rows, 50 columns)
- Connected to `IDS_mapping.csv` (67 rows, lookup table)

**Exam Tips:**
- Know the difference between Import, DirectQuery, and Live Connection modes
- Import loads data into the model (what we used); DirectQuery queries the source live

---

### 2. Data Profiling

**What to Know:**
- Column Quality: shows % valid, error, empty
- Column Distribution: shows distinct vs unique value counts
- Column Profile: shows min, max, average, value distribution
- By default profiling is based on top 1000 rows -- must change to entire dataset for accurate results

**What We Did:**
- Enabled all three profiling views in the View ribbon
- Changed profiling to entire dataset
- Identified: `weight` (97% null), `payer_code` (52% null), `medical_specialty` (53% null), `race` (contains `?`), `gender` (contains `"Unknown/Invalid"`)

**Exam Tips:**
- Know where to find each profiling option (View ribbon in Power Query)
- Know that `?` is not the same as null -- must be explicitly replaced

---

### 3. Resolving Data Quality Issues

**What to Know:**
- Replace Values: replaces a specific value with another
- Replace Errors: replaces error values
- Nulls can be replaced with a default value or left as null depending on business need
- High null % alone is not sufficient reason to drop a column -- analytical value matters

**What We Did:**
- Replaced `?` in `race` with null
- Replaced `"Unknown/Invalid"` in `gender` with null
- Replaced `?` in `medical_specialty` with `"Unknown"` (kept as a category)
- Replaced nulls in `payer_code` with `"Unknown"` (retained despite 52% nulls)
- Removed `weight` entirely (97% null, no analytical value)

**Key Decision:**
- `payer_code` was retained despite high nulls because nulls alone don't justify removal if the field has analytical value. The exam may test your judgment on this.

---

### 4. Staging Query Pattern

**What to Know:**
- A staging query is a raw source query that is never modified
- All downstream queries (dim tables) reference the staging query
- Prevents breaking changes when transformations are applied to the fact table
- Enable Load = OFF on staging queries so they don't load into the model

**What We Did:**
- Created `diabetic_data_staging` as a frozen raw source
- Set Enable Load = OFF
- All dim tables that pull from the main CSV reference `diabetic_data_staging`, not `Fact_Encounters`

**Why It Matters:**
- Without this pattern, removing a column from the fact table breaks all downstream dim queries
- This is a best practice pattern, not just a workaround

---

### 5. Reference Queries

**What to Know:**
- A Reference query points to another query as its source
- Changes to the source query propagate downstream
- Different from Duplicate -- a Duplicate copies the query independently

**What We Did:**
- Created all dim tables as Reference queries from `diabetic_data_staging`
- Each dim selects only the columns it needs using `Table.SelectColumns`

---

### 6. Creating Dimension Tables

**What to Know:**
- Dimension tables contain descriptive attributes
- Must have a unique primary key column (no duplicates)
- Remove Duplicates ensures uniqueness on the PK column

**What We Did:**

| Table | PK | Key Columns | Source |
|---|---|---|---|
| Dim_Patient | patient_nbr (Text) | race, gender, age | diabetic_data_staging |
| Dim_PrimaryDiagnosis | diag_1 (Text) | DiagnosisGroup | diabetic_data_staging |
| Dim_A1cResult | Index (surrogate) | A1Cresult, A1cTested | diabetic_data_staging |
| Dim_AdmissionType | admission_type_id | description | IDS_mapping rows 1-8 |
| Dim_DischargeDisposition | discharge_disposition_id | description | IDS_mapping |
| Dim_AdmissionSource | admission_source_id | description | IDS_mapping |

---

### 7. M Code -- Custom Columns

**What to Know:**
- Custom columns are added using `Table.AddColumn` in M
- Conditional logic uses `if / then / else` syntax
- Text functions: `Text.StartsWith`, `Text.Contains`, `Text.Upper`
- String comparisons in M are case-sensitive

**What We Did -- DiagnosisGroup column in Dim_PrimaryDiagnosis:**
```m
Table.AddColumn(#"Removed Duplicates", "DiagnosisGroup", each 
    if Text.StartsWith([diag_1], "250") then "Diabetes"
    else if [diag_1] >= "390" and [diag_1] <= "459" then "Circulatory"
    else if [diag_1] = "785" then "Circulatory"
    else if [diag_1] >= "460" and [diag_1] <= "519" then "Respiratory"
    else if [diag_1] = "786" then "Respiratory"
    else if [diag_1] >= "520" and [diag_1] <= "579" then "Digestive"
    else if [diag_1] = "787" then "Digestive"
    else if [diag_1] >= "800" and [diag_1] <= "999" then "Injury"
    else "Other")
```

**What We Did -- A1cTested column in Dim_A1cResult:**
```m
Table.AddColumn(#"Added Index", "A1cTested", each 
    if [A1Cresult] = "None" then "Not Tested" else "Tested")
```

---

### 8. Index / Surrogate Keys

**What to Know:**
- A surrogate key is an artificial PK with no business meaning
- Added using Add Index Column in Power Query
- Required when no natural unique key exists in the data

**What We Did:**
- Added an Index column to `Dim_A1cResult` as a surrogate key (A1Cresult values are not unique enough on their own for a clean PK)

---

### 9. Splitting a Stacked Lookup File

**What to Know:**
- Some lookup files contain multiple lookup tables stacked vertically
- Use `Table.SelectRows` with index range filtering to extract each section
- `Table.AddIndexColumn` first, then filter by row number range

**What We Did:**
- `IDS_mapping` contained admission types, discharge dispositions, and admission sources stacked together
- Filtered by row index ranges to create three separate dim tables

---

### 10. Data Type Management

**What to Know:**
- Data types must match on both sides of a relationship
- ID columns that are numeric identifiers (not measures) should be Text type
- Whole Number columns show a sigma (∑) symbol -- this means Power BI will try to aggregate them
- Change types in Power Query (preferred) or in Model/Data view

**What We Did:**
- Changed `patient_nbr`, `admission_type_id`, `discharge_disposition_id`, `admission_source_id` to Text in `Fact_Encounters`
- Changed corresponding dim table PK columns to Text in Model view to resolve sigma symbols

---

## PHASE 2: Model the Data (25-30% of exam)

---

### 11. Star Schema Design

**What to Know:**
- Star schema: one fact table surrounded by dimension tables
- Fact table contains measures and foreign keys
- Dim tables contain descriptive attributes and a unique PK
- Avoids many-to-many relationships

**What We Did:**
```
diabetic_data_staging (not loaded)
        |
        ├── Dim_Patient
        ├── Dim_PrimaryDiagnosis
        ├── Dim_A1cResult
        ├── Dim_AdmissionType
        ├── Dim_DischargeDisposition
        ├── Dim_AdmissionSource
        └── Fact_Encounters (center of star)
```

---

### 12. Relationships

**What to Know:**
- Cardinality options: One-to-many, Many-to-one, One-to-one, Many-to-many
- Star schema always uses Many-to-one (*:1) from fact to dim
- Cross-filter direction: Single (dim filters fact) or Both (bidirectional)
- Single is the default and correct for most star schema models
- Active vs Inactive relationships -- only one active relationship can exist between two tables

**What We Did:**
- Created 6 Many-to-one relationships, all Single cross-filter direction, all Active
- Fact_Encounters is always the "many" side

**Exam Tips:**
- Many-to-many should be avoided -- it indicates a data modeling problem
- Bidirectional filtering can cause ambiguity and performance issues

---

### 13. Model Hygiene

**What to Know:**
- Hide FK columns from report view so they don't clutter the Fields pane
- Disable load on source-only queries (like staging and raw lookup tables)
- Sigma symbols on ID columns indicate wrong data type -- fix to Text

**What We Did:**
- Disabled load on `IDs_mapping`
- Hidden 6 FK columns from Fact_Encounters report view
- Fixed sigma symbols on dim PK columns by changing to Text type

---

### 14. Date Table

**What to Know:**
- Power BI requires a proper Date table for time intelligence functions to work
- Must have a contiguous date range (no gaps)
- Must be marked as a Date table in Table tools
- The Date column must be unique (one row per date)

**What We Did:**
```dax
Dim_Date = 
ADDCOLUMNS(
    CALENDAR(DATE(1999,1,1), DATE(2008,12,31)),
    "Year", YEAR([Date]),
    "Month", MONTH([Date]),
    "Month Name", FORMAT([Date], "MMMM"),
    "Quarter", "Q" & QUARTER([Date]),
    "Year-Month", FORMAT([Date], "YYYY-MM"),
    "Weekday", FORMAT([Date], "DDDD"),
    "Is Weekend", IF(WEEKDAY([Date],2) >= 6, TRUE, FALSE)
)
```
- Marked as official Date table using Table tools ribbon

---

### 15. Measure Container Table

**What to Know:**
- Best practice to store all measures in a dedicated table
- Create an empty table (Enter Data) and name it `_Measures`
- Underscore prefix keeps it at the top of the Fields pane
- Does not need relationships to other tables

**What We Did:**
- Created `_Measures` via Enter Data
- Deleted the placeholder Column1 after creation
- All 19 measures live here

---

### 16. DAX Measures

**What to Know -- Key Functions:**

| Function | Purpose | Example |
|---|---|---|
| COUNTROWS | Count rows in a table | `COUNTROWS(Fact_Encounters)` |
| DISTINCTCOUNT | Count unique values | `DISTINCTCOUNT(Fact_Encounters[patient_nbr])` |
| AVERAGE | Average of a column | `AVERAGE(Fact_Encounters[time_in_hospital])` |
| SUM | Sum of a column | `SUM(Fact_Encounters[number_inpatient])` |
| DIVIDE | Safe division (no divide by zero error) | `DIVIDE([Numerator], [Denominator], 0)` |
| CALCULATE | Evaluate expression in modified filter context | `CALCULATE([Total Encounters], Fact_Encounters[readmitted] = "<30")` |
| FILTER | Returns a filtered table | Used inside CALCULATE |
| ALL | Removes filters from a table/column | `CALCULATE([Readmission Rate], ALL(Dim_PrimaryDiagnosis))` |
| ALLEXCEPT | Removes all filters except specified columns | `CALCULATE([Total Encounters], ALLEXCEPT(Fact_Encounters, Fact_Encounters[readmitted]))` |
| RANKX | Ranks values across a table | `RANKX(ALL(Dim_PrimaryDiagnosis[DiagnosisGroup]), [Readmission Rate], , DESC, Dense)` |

**What We Did -- All 19 Measures:**
```dax
Total Encounters = COUNTROWS(Fact_Encounters)

Total Patients = DISTINCTCOUNT(Fact_Encounters[patient_nbr])

Avg Time in Hospital = AVERAGE(Fact_Encounters[time_in_hospital])

Readmissions < 30 Days = 
COUNTROWS(FILTER(Fact_Encounters, Fact_Encounters[readmitted] = "<30"))

Readmission Rate = DIVIDE([Readmissions < 30 Days], [Total Encounters], 0)

Encounters w/ A1c Tested = 
CALCULATE([Total Encounters], Dim_A1cResult[A1cTested] = "Tested")

A1c Test Rate = DIVIDE([Encounters w/ A1c Tested], [Total Encounters], 0)

Encounters w/ Med Change = 
CALCULATE([Total Encounters], Fact_Encounters[change] = "Ch")

Avg Lab Procedures = AVERAGE(Fact_Encounters[num_lab_procedures])

Avg Medications = AVERAGE(Fact_Encounters[num_medications])

Readmission Rate All Diagnoses = 
CALCULATE([Readmission Rate], ALL(Dim_PrimaryDiagnosis))

Readmission Rate vs Average = 
[Readmission Rate] - [Readmission Rate All Diagnoses]

Readmission Rank by Diagnosis = 
RANKX(
    ALL(Dim_PrimaryDiagnosis[DiagnosisGroup]),
    [Readmission Rate],
    ,
    DESC,
    Dense
)

Encounters % of Total = 
DIVIDE(
    [Total Encounters],
    CALCULATE([Total Encounters], ALLEXCEPT(Fact_Encounters, Fact_Encounters[readmitted])),
    0
)

Total Inpatient Visits = SUM(Fact_Encounters[number_inpatient])

Total Emergency Visits = SUM(Fact_Encounters[number_emergency])

Total Outpatient Visits = SUM(Fact_Encounters[number_outpatient])

Encounters on Diabetes Med = 
CALCULATE([Total Encounters], Fact_Encounters[diabetesMed] = "Yes")

Diabetes Med Rate = DIVIDE([Encounters on Diabetes Med], [Total Encounters], 0)
```

---

## PHASE 3: Visualize and Analyze
*To be added as work progresses*

---

## PHASE 4: Manage and Secure
*To be added as work progresses*

---

## Quick Reference -- Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `Expression.Error: column 'X' not found` | Dim query references Fact table which had column removed | Change dim source to `diabetic_data_staging` |
| Sigma (∑) on ID column in dim table | Column is Whole Number type instead of Text | Change data type to Text in Data view |
| Many-to-many relationship | Duplicate values in dim PK column | Add Remove Duplicates step in Power Query |
| Query Errors folder in Power Query | Power BI auto-generated error detection query | Not a broken query -- check the actual query, not the error wrapper |

---

## PHASE 3: Visualize and Analyze (25-30% of exam)

---

### 17. Card Visuals

**What to Know:**
- Card visual displays a single measure value
- Different from the KPI visual (which requires a target and date axis)
- Format measures directly on the measure (not the visual) for consistency everywhere it's used
- Percentage format: select measure > Measure tools > % button
- Custom decimal format: set Format to Decimal Number, use the .00 button

**What We Did:**
- 4 Card visuals on Executive Summary: Total Encounters (102K), Readmission Rate (11%), A1c Test Rate (17%), Avg Time in Hospital (4.40 days)
- Formatted rate measures as Percentage via Measure tools ribbon
- Renamed measure to `Avg Time in Hospital (days)` since custom format string wasn't editable in the ribbon

**Exam Tips:**
- KPI visual requires a date axis -- if no date column exists in the fact table, use Card instead
- Format the measure, not the visual -- that way the format applies everywhere the measure is used

---

### 18. Bar Charts

**What to Know:**
- Clustered Bar Chart: horizontal bars, good for category comparisons
- Y-axis: category field; X-axis: measure
- Power BI auto-sorts -- can change sort order via the visual's "..." menu

**What We Did:**
- Bar chart: Readmission Rate by DiagnosisGroup
- Discovered (Blank) category -- fixed by filtering blank rows out of Dim_PrimaryDiagnosis in Power Query
- Result: Diabetes highest readmission rate (~13%), Respiratory lowest (~10%)

**Exam Tips:**
- (Blank) in a visual usually means unmatched values between fact and dim table, OR nulls in the dim key column
- Fix blanks in Power Query, not by visual-level filtering -- cleaner and more robust

---

### 19. Slicers

**What to Know:**
- Slicer filters all visuals on the page by default
- Field well accepts any column or measure
- Slicer header title is set in Format pane > Slicer header > Title text (not in the ribbon)
- Default style is a list -- can change to dropdown, tile, or between in Format pane

**What We Did:**
- Slicer on Admission Type using `description` from `Dim_AdmissionType`
- Renamed slicer header from "description" to "Admission Type"

---

### 20. Drill-through Pages

**What to Know:**
- Drill-through lets users right-click a data point and navigate to a detail page filtered to that context
- Set up on the destination page (not the source page)
- Drag a field into the **Drill through** well in the Visualizations pane on the detail page
- Power BI auto-generates a Back button on the drill-through page
- Cross-report drill-through is also possible (different .pbix files)

**What We Did:**
- Created Page 2 "Drill-through" with DiagnosisGroup as the drill-through field
- Table visual shows encounter_id, DiagnosisGroup, readmitted, time_in_hospital
- Tested: right-click bar on Executive Summary > Drill through > filters Page 2 to that DiagnosisGroup

**Key Gotcha:**
- Numeric columns default to Sum aggregation in table visuals -- set Summarization to "Don't summarize" on `encounter_id` and similar ID columns in Column tools ribbon
- Measures like Readmission Rate show 100% at the row level (each row is one readmitted encounter = 100%) -- remove from row-level tables, keep for aggregate views only

**Exam Tips:**
- Drill-through field must match the field used in the source visual for the right-click option to appear
- Back button is auto-generated -- do not delete it

---

### 21. AI Visuals -- Key Influencers

**What to Know:**
- Key Influencers automatically identifies which fields most influence a metric
- Analyze field: the outcome you want to explain (e.g. readmitted)
- Explain by fields: dimensions that might drive the outcome
- Shows multiplier (e.g. 1.27x) -- how much more likely the outcome is when that condition is true
- Two tabs: Key Influencers (individual factors) and Top Segments (combinations of factors)

**What We Did:**
- Analyze: `readmitted` (to be `<30`)
- Explain by: `DiagnosisGroup`, `time_in_hospital`, `num_medications`, `A1cTested`
- Top finding: time_in_hospital > 5 increases readmission likelihood 1.27x; A1c Not Tested increases it 1.19x

**Exam Tips:**
- Key Influencers is under Insert > AI visuals, not the Visualizations pane
- Numeric fields are auto-binned by Power BI

---

### 22. AI Visuals -- Decomposition Tree

**What to Know:**
- Decomposition Tree breaks down a measure by multiple dimensions interactively
- Analyze field: the measure to decompose (e.g. Readmission Rate)
- Explain by fields: dimensions to break down by
- Users click "+" on each node to drill down further
- AI Split option automatically finds the dimension that explains the most variance

**What We Did:**
- Analyze: `Readmission Rate`
- Explain by: `DiagnosisGroup`, `A1cTested`, `description` (Dim_AdmissionType)
- Found Emergency admission type has highest readmission rate (12%)

---

### 23. Q&A Visual

**What to Know:**
- Allows users to type natural language questions and get auto-generated visuals
- Power BI interprets the question and selects the best visual type
- Q&A is retiring in December 2026 -- may be replaced by Copilot features
- To restore Q&A icon if hidden: Visualizations pane > "..." > Restore default visuals

**What We Did:**
- Added Q&A visual on Page 4
- Tested query: "readmission rate by DiagnosisGroup" -- returned correct bar chart

---

### 24. Bookmarks and Button Navigation

**What to Know:**
- Bookmarks capture the state of a report page (filters, visibility, slicers)
- Enable Bookmarks pane via View tab
- Add a bookmark via the Bookmarks pane > Add
- Wire bookmarks to buttons via Format pane > Action > Type: Bookmark
- Ctrl+click a button in edit mode to test; users click directly in reading/published view

**What We Did:**
- Created two bookmarks on Executive Summary: "All Admissions" and "Emergency Only"
- "Emergency Only" bookmark captures the slicer filtered to Emergency
- Added two blank buttons, labeled them, wired each to its bookmark
- Tested toggle navigation between views

---

### 25. Conditional Formatting

**What to Know:**
- Applied via Format pane > Cell elements (for table/matrix visuals)
- Background color, font color, data bars, and icons are all options
- Color scale (Gradient) uses min/max values to assign colors
- Summarization must match intent -- use Average for numeric columns like `time_in_hospital`, not Count
- Access via the **fx** button next to the formatting option

**What We Did:**
- Applied background color scale to `time_in_hospital` on Page 2 drill-through table
- Format style: Gradient | Field: `time_in_hospital` | Summarization: Average
- Minimum color: white/green (low days = good) | Maximum color: red (long stays = high risk)

**Exam Tips:**
- Summarization defaults to Count -- always verify it matches your intent
- Color scale applies to the column values, not row counts

---

### 26. Filter Types

**What to Know:**
- Three filter scopes in the Filters pane:
  - **Filters on all pages (Report-level):** applies to every page in the report
  - **Filters on this page (Page-level):** applies only to the current page
  - **Filters on this visual (Visual-level):** applies only to the selected visual
- Filters pane must be visible (View tab > Filters)
- Filter types: Basic filtering, Advanced filtering, Top N

**What We Did:**
- Report-level: `gender` from Dim_Patient dragged into "Filters on all pages"
- Page-level: `readmitted` on Page 2 dragged into "Filters on this page"
- Visual-level: `DiagnosisGroup` on bar chart on Page 1 dragged into "Filters on this visual"

**Exam Tips:**
- Know the difference between a slicer (visible on canvas, user-interactive) and a filter (in the Filters pane, can be hidden from report readers)
- Filters can be locked or hidden from end users in published reports

---

### 27. Custom Tooltip Pages

**What to Know:**
- A tooltip page is a report page that appears on hover over a visual
- Must be enabled: Format pane > Page information > Allow use as tooltip: On
- Canvas size should be set to Tooltip type for correct dimensions
- Wire to a visual: select visual > Format pane > General > Tooltips > Type: Report page > select the page
- Tooltip is filtered to the data point being hovered

**What We Did:**
- Created Page 5 "Tooltip" with two Card visuals: Avg Time in Hospital (days) and Readmission Rate
- Set Allow use as tooltip: On and canvas size to Tooltip
- Wired to bar chart on Page 1
- Tested: hovering over a diagnosis bar shows filtered avg days and readmission rate for that group

**Exam Tips:**
- Tooltip page must have "Allow use as tooltip" enabled or it won't appear as an option
- The tooltip filters to context -- hovering over Diabetes bar shows Diabetes-specific values

---

### 28. Custom JSON Theme

**What to Know:**
- Themes control colors, fonts, and visual formatting across the entire report
- Import via View tab > Themes > Browse for themes
- Theme file is a `.json` file with defined color palette and visual style overrides
- dataColors array sets the default color sequence for visuals
- visualStyles can target specific visual types (card, barChart, slicer, etc.)

**What We Did:**
- Created `WellvanaHealthTheme.json` with:
  - Primary color: `#1F6B8E` (teal blue)
  - Secondary: `#2E9E6B` (green)
  - Alert: `#E84C3D` (red)
  - Font: Segoe UI, 11pt throughout
  - Card callout values: bold, 28pt, teal
- Imported via View > Themes > Browse for themes

**Exam Tips:**
- Theme applies to the whole report -- individual visual formatting overrides the theme
- Know the key theme JSON properties: name, dataColors, background, foreground, visualStyles
