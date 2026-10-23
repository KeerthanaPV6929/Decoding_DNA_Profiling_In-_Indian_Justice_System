# Decoding DNA Profiling in India's Justice System

An interactive **Power BI dashboard** that analyses a survey of **612 criminal justice professionals** on their awareness, use and perception of DNA profiling in India.

> **Key finding:** Views are moderately positive (60% agree, average score 3.67 / 5) and similar across groups. The weakest area is the **practical use** of DNA profiling in criminal investigation.

![Dashboard preview](Screenshot/Dashboard.png)
<!-- Add a screenshot of your dashboard at images/dashboard.png -->

---

## Table of Contents

1. [Project Overview](#project-overview)
2. [Objectives](#objectives)
3. [Dataset](#dataset)
4. [Survey Constructs](#survey-constructs)
5. [Tools and Technologies](#tools-and-technologies)
6. [Data Preparation](#data-preparation)
7. [Data Model](#data-model)
8. [DAX Measures](#dax-measures)
9. [Dashboard Walkthrough](#dashboard-walkthrough)
10. [Key Findings](#key-findings)
11. [Limitations](#limitations)
12. [Future Work](#future-work)
13. [How to Use](#how-to-use)
14. [Author](#author)

---

## Project Overview

DNA profiling is a key forensic tool in modern criminal investigation. Its value, however, depends on the people who use it: whether they know it, trust it, and have the training and legal clarity to apply it.

This project turns survey responses into a single-page dashboard that answers one question:

**What do justice system professionals think about DNA profiling, and where is the gap?**

## Objectives

- Measure awareness, use and perceived reliability of DNA profiling.
- Assess legal and procedural knowledge, institutional capacity and ethical concerns.
- Compare views by gender, age, education, profession, experience and region.
- Identify the weakest areas to support recommendations.

## Dataset

| Item | Detail |
|---|---|
| Source | Primary survey (questionnaire) |
| Respondents | 612 criminal justice professionals |
| Background fields | Gender, Age Group, Education, Professional Background, Institutional Association, Professional Experience, Region, Work Association |
| Survey items | 44 statements rated on a 5-point Likert scale |
| Scale | 1 = Strongly Disagree, 2 = Disagree, 3 = Neutral, 4 = Agree, 5 = Strongly Agree |
| Data quality | No missing values, no duplicate rows, all scores between 1 and 5 |

The dataset is supported by a **questionnaire** and a **code book** that define each statement and demographic code.

## Survey Constructs

The 44 statements are grouped into 7 constructs.

| Code | Construct | Items |
|---|---|---|
| ADP | Awareness of DNA Profiling | 6 |
| UDP | Use of DNA Profiling in Criminal Investigation | 7 |
| PRD | Perceived Reliability of DNA Evidence | 7 |
| LPA | Legal and Procedural Awareness | 6 |
| ICT | Institutional Capacity and Training | 6 |
| EPC | Ethical and Privacy Concerns | 6 |
| OPD | Overall Perception of DNA Profiling | 6 |

## Tools and Technologies

- **Power BI Desktop** for modelling and visualisation
- **Power Query (M)** for data cleaning and reshaping
- **DAX** for calculated measures
- **Excel** for the source data and lookup tables
- **Custom JSON theme** (neon purple) for the dashboard design

## Data Preparation

Steps carried out in Power Query:

1. Loaded the survey data from Excel.
2. Removed a trailing space from a column name (`Work Association`).
3. Added a **RespondentID** index column.
4. Set data types (text for background fields, whole number for the 44 items).
5. Added **sort-order columns** for Age Group, Professional Experience and Education so categories display in logical order.
6. Created a **long table** by unpivoting the 44 item columns (612 × 44 = **26,928 rows**) with columns `RespondentID`, `Item`, `Score` and `Construct Code`.
7. Built lookup tables for items, constructs and Likert labels.

## Data Model

A star-schema style model:

| Table | Purpose |
|---|---|
| `Survey_Wide` | One row per respondent: background details (used for slicers and group charts) |
| `Survey_Long` | One row per answer: the scores used in calculations |
| `Dim_Items` | 44 statements with short labels and full text |
| `Dim_Construct` | 7 constructs with names and display order |
| `Dim_Likert` | Score-to-label mapping (1 to 5) |
| `_Measures` | Container for all DAX measures |

**Relationships (one-to-many, single direction):**

```
Survey_Wide[RespondentID]  ->  Survey_Long[RespondentID]
Dim_Items[Item]            ->  Survey_Long[Item]
Dim_Construct[Construct Code] -> Dim_Items[Construct Code]
Dim_Likert[Score]          ->  Survey_Long[Score]
```

## DAX Measures

```DAX
Respondents = DISTINCTCOUNT(Survey_Wide[RespondentID])

Avg Score = AVERAGE(Survey_Long[Score])

Agree % =
DIVIDE(
    CALCULATE(COUNTROWS(Survey_Long), Survey_Long[Score] >= 4),
    COUNTROWS(Survey_Long)
)

Neutral % =
DIVIDE(
    CALCULATE(COUNTROWS(Survey_Long), Survey_Long[Score] = 3),
    COUNTROWS(Survey_Long)
)

Disagree % =
DIVIDE(
    CALCULATE(COUNTROWS(Survey_Long), Survey_Long[Score] <= 2),
    COUNTROWS(Survey_Long)
)

Top Construct =
VAR t =
    ADDCOLUMNS(
        ALLSELECTED(Dim_Construct[Construct Name]),
        "@Score", CALCULATE(AVERAGE(Survey_Long[Score]))
    )
RETURN
    CONCATENATEX(TOPN(1, t, [@Score], DESC), Dim_Construct[Construct Name], ", ")

Lowest Construct =
VAR t =
    ADDCOLUMNS(
        ALLSELECTED(Dim_Construct[Construct Name]),
        "@Score", CALCULATE(AVERAGE(Survey_Long[Score]))
    )
RETURN
    CONCATENATEX(TOPN(1, t, [@Score], ASC), Dim_Construct[Construct Name], ", ")
```

All measures respond to slicers, so every number updates for the selected group.

## Dashboard Walkthrough

The dashboard is a single page designed to read from top to bottom.

| Section | Visuals | What it shows |
|---|---|---|
| Summary | Cards: Respondents, Avg Score, Agree %, Lowest Construct, Top Construct | Overall level and the weakest and strongest topics |
| Filters | Slicers: Region, Gender, Professional Experience | Filters the whole page |
| Who answered | Donut (Gender), column chart (Education) | Sample profile |
| Main result | 100% stacked bar of answers by construct | Distribution from Strongly Disagree to Strongly Agree |
| Statements | Bar chart of Agree % by statement (lowest-agreement) | Which individual statements get the least agreement |
| Group comparison | Agree % by Age Group, Region and Professional Background | How similar or different the groups are |

## Key Findings

- **612 respondents**, average score **3.67 / 5**, and **60.1%** of answers are Agree or Strongly Agree.
- The weakest construct is **Use of DNA Profiling in Criminal Investigation (3.61)**, and the strongest is **Overall Perception (3.70)**.
- Views are **similar across gender, age, region and profession**. Group differences are only a few percentage points.
- **South India** shows the clearest gap in use (3.43 when filtered).
- The lowest-agreement statements are mostly about **practical use**, such as how widely DNA profiling is used and whether it helps when eyewitness evidence is weak.

**Recommendation:** strengthen training, forensic laboratory capacity, and clear procedures for consent and data protection.

## Limitations

- Differences between groups are small, and no significance test has been run.
- Some groups have small samples (for example North-East India, 43 respondents).
- The data reflects opinions, not measured case outcomes.
- Likert averages summarise ordinal data, so the dashboard also shows the full distribution.

## Future Work

- Add significance tests (for example ANOVA or chi-square) for group differences.
- Test reliability of each construct (for example Cronbach's alpha).
- Include open-ended comments through text analysis.
- Compare survey views with real case data on DNA evidence use.

## How to Use

1. Download or clone this repository.
2. Open the `.pbix` file in **Power BI Desktop**.
3. Use the slicers (Region, Gender, Professional Experience) to filter the page.
4. Click any bar or slice to cross-filter the other visuals, and click again to clear.

To rebuild from the source data:

1. Load `Coded_Survey_Dataset.xlsx` in Power Query.
2. Load the lookup tables from `Dim_Tables.xlsx`.
3. Recreate the relationships and DAX measures listed above.
4. Import `Neon_Purple_Exact_Theme.json` from **View > Themes > Browse for themes**.

