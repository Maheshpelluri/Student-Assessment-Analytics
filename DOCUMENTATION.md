# Student Assessment Performance & Attendance Analytics Dashboard — Documentation

Technical documentation for the Power BI dashboard in `dashboard/Student_Assessment_Performance_Dashboard.pbix`. This file is the detailed technical companion to `README.md`.

## Table of Contents

1. [Project Overview](#1-project-overview)
2. [Problem Statement](#2-problem-statement)
3. [Objectives](#3-objectives)
4. [Data Source](#4-data-source)
5. [Original LMS Dataset](#5-original-lms-dataset)
6. [Data Cleaning & Transformation](#6-data-cleaning--transformation)
7. [Final Dataset Structure](#7-final-dataset-structure)
8. [Data Quality Issues & Solutions](#8-data-quality-issues--solutions)
9. [Power BI Data Model / Structure](#9-power-bi-data-model--structure)
10. [Dashboard Architecture](#10-dashboard-architecture)
11. [Page 1 — Assessment Overview Dashboard](#11-page-1--assessment-overview-dashboard)
12. [Page 2 — Department & Section Performance Dashboard](#12-page-2--department--section-performance-dashboard)
13. [Page 3 — Student Performance Explorer](#13-page-3--student-performance-explorer)
14. [Page 4 — Assessment Follow-Up Tracker (Never Attempted Students)](#14-page-4--assessment-follow-up-tracker-never-attempted-students)
15. [DAX Measures & Calculated Columns](#15-dax-measures--calculated-columns)
16. [KPI Definitions](#16-kpi-definitions)
17. [Visualization Logic](#17-visualization-logic)
18. [Filtering & Slicers](#18-filtering--slicers)
19. [Data Refresh Workflow](#19-data-refresh-workflow)
20. [Privacy / Data Security](#20-privacy--data-security)
21. [Project Outcomes](#21-project-outcomes)
22. [Future Enhancements](#22-future-enhancements)

---

## 1. Project Overview

This project is an interactive Power BI solution built to analyze student assessment participation and academic performance for Audisankara College of Engineering & Technology. It converts recurring raw LMS assessment exports into a cleaned, standardized reporting dataset, then models it with DAX to power a 4-page dashboard used by trainers, mentors, academic coordinators, HODs, and management.

## 2. Problem Statement

Assessment data is exported from the LMS portal after each assessment. In raw form, this export was not analysis-ready:

- It contained unnecessary columns not needed for reporting.
- Department names were inconsistent across exports (e.g. `ECE` vs. `Electronics and Communication Engineering`).
- Roll Numbers sometimes had leading/trailing spaces, risking incorrect unique-student counts.
- New assessment data had to be appended continuously as further assessments were conducted.
- Trainers had no simple, repeatable way to see total/attempted/non-attempted students, compare performance by course/department/section, or identify students who had never attempted any assessment — these were previously assembled manually.

## 3. Objectives

- Build a single, centrally maintained analytical dataset from recurring LMS assessment exports.
- Standardize inconsistent department naming and clean Roll Number identifiers.
- Provide accurate, refreshable KPIs for total, attempted, and non-attempted students, and assessment participation rate.
- Enable performance comparison across courses, departments, and sections.
- Provide a percentage-based grade classification.
- Provide a student-level lookup view for trainers and mentors.
- Provide an operational follow-up list of students who have never attempted an assessment across the available assessment history.

## 4. Data Source

**Primary source:** LMS Portal assessment exports.

Assessment data was exported from the LMS after each assessment. The LMS API was **not** used, and data extraction/ingestion was **not** automated — each export was prepared and appended manually using tabular (Excel-based) data preparation.

## 5. Original LMS Dataset

The raw LMS export contained additional fields not required for analytical reporting. During data preparation, unnecessary columns were removed and only the fields relevant to the final analytical structure (see Section 7) were retained.

## 6. Data Cleaning & Transformation

Workflow followed for every assessment export:

```
LMS Export
  → Remove unwanted columns
  → Standardize Department names
  → Clean Roll Numbers using TRIM
  → Verify/prepare Percentage and marks_obtained
  → Create Status (Attempted / Non-Attempted)
  → Add assessment Date manually
  → Append new assessment-day data to the master dataset
  → Refresh Power BI
  → Validate dashboard output
```

- **Department standardization:** Inconsistent values (e.g. `ECE` vs. `Electronics and Communication Engineering`) were standardized to a single label (`ECE`), since Power BI treats each distinct text value as a separate category for grouping and filtering.
- **Roll Number cleaning:** Leading/trailing spaces were removed using TRIM, since inconsistent whitespace in a text identifier affects `DISTINCTCOUNT`, filtering, and duplicate detection.
- **Date:** Added manually during preparation, since it was not reliably present in a usable form in the raw export and is required for date-based filtering and trend analysis.
- **Incremental appends:** Each new assessment batch is cleaned into the same structure and appended to the master dataset; Power BI is refreshed afterward so the model reflects the update.

This is a manual tabular preparation process, not an automated ETL pipeline.

## 7. Final Dataset Structure

| # | Column | Type | Description |
|---|---|---|---|
| 1 | Full Name | Text | Student name |
| 2 | Roll Number | Text | Unique student identifier |
| 3 | Course / Batch | Text | Training course or batch |
| 4 | College Name | Text | Student's college |
| 5 | Department | Text | Academic department |
| 6 | Section | Text | Class section within the department |
| 7 | Max Marks | Numeric | Maximum marks for the assessment |
| 8 | Total Time Taken | Numeric/Duration | Time taken to complete the assessment |
| 9 | Percentage | Numeric | Assessment score expressed as a percentage |
| 10 | `marks_obtained` (displayed as "Obtained Marks") | Numeric | Marks scored; blank if the assessment was not attempted |
| 11 | Status | Text | Attempted / Non-Attempted (assessment-level) |
| 12 | Date | Date | Assessment date (added manually) |

**Note on field naming:** `marks_obtained` is the original technical column name in the model and is used exactly as-is in every DAX formula (`Table1[marks_obtained]`). It is only written as "Obtained Marks (marks_obtained)" in human-readable descriptions.

## 8. Data Quality Issues & Solutions

| Issue | Example | Solution |
|---|---|---|
| Unwanted LMS columns | Extra non-analytical fields in raw export | Removed during preparation |
| Inconsistent department naming | `ECE` vs. `Electronics and Communication Engineering` | Standardized to one label per department |
| Leading/trailing Roll Number spaces | `"20A01"` vs. `"20A01 "` | Cleaned with TRIM |
| Multiple assessment records per student | Same student across several assessment rows | `DISTINCTCOUNT(Table1[Roll Number])` used for unique-student KPIs instead of row count |
| New assessment-day data | Additional export after each assessment | Standardized append into the master dataset, then refresh |
| Missing/unreliable assessment date | Date not present in usable form in raw export | Added manually during preparation |
| Distinguishing assessment-level non-attempted from never-attempted | A student attempts Assessment 1 but misses Assessment 2 | Implemented as separate DAX objects — `Status`/Not Attempted measure (assessment-level) vs. `Student Attempt Status` calculated column (full history) |

Validation was performed after every cleaning and refresh cycle (see Section 19).

## 9. Power BI Data Model / Structure

The cleaned dataset (`Table1`) is a single flat analytical table — a formal star schema with separate fact/dimension tables was not implemented.

**Data grain:** `ONE STUDENT × ONE ASSESSMENT`. The same student can appear in multiple rows because a student may participate in multiple assessments. This is why `DISTINCTCOUNT(Table1[Roll Number])` is used instead of `COUNTROWS(Table1)` for every unique-student KPI — a plain row count would overstate the student population.

Logical dimensions derived from `Table1`:

- **Student:** Full Name, Roll Number
- **Academic:** Course, College, Department, Section
- **Assessment:** Date, Max Marks, `marks_obtained`, Percentage, Total Time Taken
- **Analytical:** Status, Grade

## 10. Dashboard Architecture

| Page | Title | Purpose |
|---|---|---|
| 1 | Assessment Overview Dashboard | Executive/high-level assessment monitoring |
| 2 | Department & Section Performance Dashboard | Department- and section-level performance analysis |
| 3 | Student Performance Explorer | Detailed student-level analysis and lookup |
| 4 | Assessment Follow-Up Tracker | Identify students who have never attempted any assessment |

Design: a consistent beige/gold visual theme is applied across all four pages — white KPI cards with rounded corners and light borders, gold as the primary analytical chart color, slate gray as the secondary comparison color, and a header/slicer/chart layout maintained consistently page to page. No separate Power BI theme JSON file was created; styling was applied directly in Power BI Desktop.

## 11. Page 1 — Assessment Overview Dashboard

**Purpose:** High-level overview of student participation, attendance, and assessment performance.

**KPI cards:** Total Students · Attempted Students · Non-Attempted Students · Attendance Percentage

**Slicers:** Date, Course

**Visuals:**

| Visual | Type | Answers |
|---|---|---|
| Average Percentage by Course | Clustered column chart | Which courses have the strongest/weakest average performance? |
| Average Percentage Trend | Line chart | How is average performance changing across assessment dates? |
| Attendance Trend by Course | Line chart | How is assessment participation changing over time, by course? |

## 12. Page 2 — Department & Section Performance Dashboard

**Purpose:** Department- and section-level performance analysis.

**Slicers:** Date, Course, Department, Section

**KPI:** Average Percentage

**Visuals:**

| Visual | Type | Details |
|---|---|---|
| Average Percentage by Department | Horizontal bar chart | Compares average performance across departments |
| Grade Distribution by Department | Bar chart | Axis: Department · Legend: Grade · Value: student count — shows each department's Grade A–D composition |
| Assessment Participation by Department | Clustered column chart | Compares Attempted Students vs. Non-Attempted Students by department |

Grade is placed in the **Legend** (not just used as a Count of Grade value) so each department's bar is broken into its Grade A–D composition, rather than collapsing to one aggregate figure per department.

## 13. Page 3 — Student Performance Explorer

**Purpose:** Individual student-level lookup and analysis.

**Table fields:** Full Name, Roll Number, Course, Department, Section, Percentage, Grade

**Slicers/filters:** Roll Number, Course, Department, Section, Grade, Status, Date

**Additional KPI cards:** Total Students, and a Grading Policy reference card (see Section 15).

This page lets trainers and mentors move from aggregate, chart-level analysis to individual student records.

## 14. Page 4 — Assessment Follow-Up Tracker (Never Attempted Students)

**Purpose:** Identify students who have never attempted any assessment and provide a ready-to-use follow-up list, eliminating the need to manually prepare a separate list.

**Slicers:** Course, Department, Section, and **Student Attempt Status** (set to `Never Attempted` to isolate at-risk students)

**Table fields:** Full Name, Roll Number, Course, Department, Section, Student Attempt Status

Operational workflow supported by this page:

```
Course → Department → Section → Never Attempted Students → Follow-up
```

## 15. DAX Measures & Calculated Columns

Power BI distinguishes between **measures** (calculated dynamically based on the current filter context) and **calculated columns** (evaluated row-by-row and stored on the table). This project uses both:

| Type | DAX Objects |
|---|---|
| Measures | Total Students, Attempted Students, Not Attempted Students, Attendance Percentage, Average Percentage, Grading Policy (display text) |
| Calculated Columns | Grade, Student Attempt Status |

### Total Students *(Measure)*

```dax
Total Students =
DISTINCTCOUNT(Table1[Roll Number])
```
Unique student count in the current filter context. DISTINCTCOUNT is required because the data grain is one row per student per assessment — a row count would overstate the population.

### Attempted Students *(Measure)*

```dax
Attempted Students =
CALCULATE(
    DISTINCTCOUNT(Table1[Roll Number]),
    FILTER(
        Table1,
        NOT(ISBLANK(Table1[marks_obtained]))
    )
)
```
Distinct students with a non-blank `marks_obtained` value for the assessment(s) in the current filter context.

### Not Attempted Students *(Measure)*

```dax
Not Attempted Students =
CALCULATE(
    DISTINCTCOUNT(Table1[Roll Number]),
    FILTER(
        Table1,
        ISBLANK(Table1[marks_obtained])
    )
)
```
Distinct students with a blank `marks_obtained` value for the assessment(s) in the current filter context. This is an **assessment-level** classification — distinct from the history-level "Never Attempted" calculated column (Section 14).

> **Note:** Because a student can attempt one assessment and miss another, `Total Students`, `Attempted Students`, and `Not Attempted Students` are not expected to sum together across a multi-assessment selection. They represent assessment-context participation states, not a permanent label per student.

### Attendance Percentage *(Measure)*

```dax
Attendance Percentage =
DIVIDE(
    [Attempted Students],
    [Total Students],
    0
)
```
Represents **assessment participation** — the proportion of students who attempted the assessment within the current filter context. This is not classroom attendance. `DIVIDE` safely returns 0 rather than an error when the denominator is zero.

### Average Percentage *(Measure)*

```dax
Average Percentage =
AVERAGEX(
    FILTER(
        Table1,
        NOT(ISBLANK(Table1[percentage]))
    ),
    Table1[percentage]
)
```
Percentage — rather than raw `marks_obtained` — is used as the normalized performance metric because different assessments may have different maximum marks. Percentage normalizes performance to a common 0–100 scale, keeping comparisons consistent across assessments, courses, departments, sections, and dates. Blank percentage records (non-attempts) are excluded before averaging.

### Grading Policy *(Measure — display text)*

```
Grade A : 80% - 100%
Grade B : 60% - 79%
Grade C : 30% - 59%
Grade D : Below 30%
Blank / Not Attempted : No assessment marks
```
A text measure displayed as a reference card on Page 3, using `UNICHAR(10)` for line breaks in the text visual.

### Grade *(Calculated Column)*

```dax
Grade =
SWITCH(
    TRUE(),
    ISBLANK(Table1[marks_obtained]), BLANK(),
    Table1[percentage] >= 80, "Grade A",
    Table1[percentage] >= 60, "Grade B",
    Table1[percentage] >= 30, "Grade C",
    "Grade D"
)
```
Evaluated row-by-row on `Table1`. A student scoring below 30% receives Grade D; a student with no `marks_obtained` does **not** receive Grade D — the column stays blank, so non-participation is never represented as poor academic performance.

### Student Attempt Status *(Calculated Column)*

```dax
Student Attempt Status =
VAR CurrentRoll = Table1[Roll Number]
VAR AttemptCount =
    COUNTROWS(
        FILTER(
            ALL(Table1),
            Table1[Roll Number] = CurrentRoll
                && NOT(ISBLANK(Table1[marks_obtained]))
        )
    )
RETURN
    IF(
        AttemptCount = 0,
        "Never Attempted",
        "Attempted"
    )
```
Identifies students who have **never** attempted any assessment across their entire available history (used on Page 4). `ALL(Table1)` lets the calculation inspect the complete `Table1` history for that student's Roll Number — not just the current row — so it can correctly check whether any of their records has a non-blank `marks_obtained` value anywhere in their history. If `AttemptCount` is zero across every record for that Roll Number, the student is classified `"Never Attempted"`.

**Not Attempted vs. Never Attempted**

| Concept | DAX Object | Scope | Used For |
|---|---|---|---|
| Not Attempted | Measure | A single specific assessment | Assessment-level participation KPIs/charts (Pages 1–2) |
| Never Attempted | Calculated column (`Student Attempt Status`) | Entire assessment history for a student | Follow-up list of at-risk students (Page 4) |

## 16. KPI Definitions

| KPI | Definition |
|---|---|
| Total Students | Unique students in the current filter context |
| Attempted Students | Unique students who attempted the assessment(s) in context |
| Non-Attempted Students | Unique students who did not attempt the assessment(s) in context |
| Attendance Percentage | Assessment participation rate: Attempted Students ÷ Total Students |
| Average Percentage | Average assessment percentage, excluding non-attempted records |
| Grade | Percentage-based classification (A–D) per assessment record, blank if not attempted |

## 17. Visualization Logic

- **KPI cards** surface the core measures for at-a-glance monitoring on Pages 1–3.
- **Clustered column / bar charts** compare Average Percentage or participation across categorical dimensions (Course, Department).
- **Line charts** track Average Percentage and Attendance Percentage over the Date field for trend analysis.
- **Grade Distribution** uses Grade as the Legend (not just a count value) so each category's bar shows its full A–D composition.
- **Tables** (Pages 3–4) surface record-level detail for lookup and follow-up, filtered by the page's slicers.

## 18. Filtering & Slicers

| Page | Slicers |
|---|---|
| 1 | Date, Course |
| 2 | Date, Course, Department, Section |
| 3 | Roll Number, Course, Department, Section, Grade, Status, Date |
| 4 | Course, Department, Section, Student Attempt Status |

All slicers cross-filter the KPI cards, charts, and tables on their page dynamically.

## 19. Data Refresh Workflow

After each new assessment export is cleaned and appended to the master dataset, Power BI must be manually refreshed. Power BI caches the data model at load time; without a refresh, KPIs, visuals, and the `Student Attempt Status` calculated column continue to reflect the data as of the last refresh and will not include newly appended records.

Validation performed after each cleaning/refresh cycle:

- Roll Number consistency
- Department naming consistency
- Duplicate identification
- Student count, attempted count, and non-attempted count validation
- Percentage and Grade classification validation
- Date validation
- Power BI refresh validation

## 20. Privacy / Data Security

This repository does not contain real or sample student records. No student names, Roll Numbers, email addresses, marks, or other personally identifiable information are included. Only the dataset **structure** is documented (Section 7). The `.pbix` file included in `dashboard/` should not be re-shared with live student data connected; if the file currently contains cached student-level data, that data should be removed or replaced with anonymized/sample data before any public distribution.

## 21. Project Outcomes

- Replaced manually prepared assessment summaries with a centralized, refreshable dashboard.
- Improved visibility into participation gaps across courses, departments, and sections.
- Faster identification of non-attempted students at the assessment level.
- A dedicated, filterable list of students who have never attempted any assessment (Page 4), removing the need to build that list manually.
- Faster student-level lookup for trainers and mentors via Page 3.

No numerical business-impact figures (e.g., time saved, percentage improvements) are claimed, as none were measured as part of this project.



