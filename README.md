# 📊 Student Assessment Performance & Attendance Analytics Dashboard

An interactive **Power BI analytics dashboard** developed to analyze student assessment performance, participation, course-wise trends, department/section performance, and students who have never attempted an assessment.

---

## 📌 Project Overview

The project transforms raw **LMS assessment exports** into a structured analytical dataset and presents the results through an interactive Power BI dashboard.

The solution helps academic coordinators, HODs, trainers, and mentors quickly understand:

- Overall student assessment participation
- Attempted vs. non-attempted students
- Average assessment percentage
- Course-wise performance
- Department-wise performance
- Section-level performance
- Assessment trends
- Student-level performance
- Students who have never attempted an assessment

The dashboard is designed to reduce manual analysis and provide a centralized view of assessment-related insights.

---

## 🎯 Problem Statement

Assessment data exported from the LMS contained several unnecessary fields and recurring data-quality issues.

Common problems included:

- Unwanted LMS-generated columns
- Inconsistent department names
- Leading/trailing spaces in Roll Numbers
- Multiple assessment records for the same student
- Manual identification of non-attempted students
- Difficulty comparing performance across assessments with different maximum marks

These issues made manual analysis time-consuming and increased the possibility of inconsistent reporting.

The objective was to develop a centralized Power BI dashboard that converts the LMS assessment data into meaningful and interactive analytical insights.

---

## 💡 Solution

The project follows a structured data analytics workflow:

```text
LMS Assessment Export
	↓
Data Cleaning
	↓
Remove Unwanted Columns
	↓
Standardize Department Names
	↓
Clean Roll Numbers
	↓
Prepare Assessment Fields
	↓
Create Attempt Status
	↓
Add Assessment Date
	↓
Consolidate Assessment Data
	↓
Power BI Refresh
	↓
DAX Calculations
	↓
Interactive Dashboard
	↓
Student Follow-up Analysis
```

---

## 🔗 Live dashboard preview

Open the interactive preview in Power BI Service:

https://app.powerbi.com/view?r=eyJrIjoiN2I5YTI4ODEtZDE4Zi00NGM1LWI1ODQtZjE1NzNjMDgzZDU2IiwidCI6IjM3MjIxNDEwLWQzMzUtNDQ0OS05YjcwLWJmZTk3ZmM4Yzc4MiJ9

---

## 🖼️ Screenshots

The `screenshots/` folder contains visual previews of the dashboard pages:

- `assessment-overview.png` — Assessment Overview Dashboard
- `department-performance.png` — Department & Section Performance
- `student-performance.png` — Student Performance Explorer
- `assessment-followup.png` — Assessment Follow-Up Tracker

If screenshots are missing, add the PNGs to the `screenshots/` folder. Do not publish screenshots containing real student names or Roll Numbers.

---

## 📁 Repository Structure

```
student-assessment-powerbi-dashboard/
│
├── README.md
├── DOCUMENTATION.md
├── .gitignore
├── screenshots/
│   ├── assessment-overview.png
│   ├── department-performance.png
│   ├── student-performance.png
│   └── assessment-followup.png
└── dashboard/  (Power BI file removed from repository; use the Power BI Service link above)
```

---

## ⚠️ Privacy Notice

This repository does not include raw student datasets. The Power BI `.pbix` file has been removed from the public repository to avoid exposing student-level data. Use the provided Power BI Service link to explore the dashboard interactively. If you need to publish a `.pbix`, ensure it contains only anonymized/sample data.

---

If you'd like, I can import the attached screenshots into the `screenshots/` folder and finalize the README with embedded images — confirm and I'll add them and push the changes.