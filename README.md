# 🧱 Lego Sets Analysis — Python (Pandas)

> A Python-based data analysis project exploring the history of Lego's licensed sets — answering key business questions about Star Wars dominance, theme popularity over time, and Lego's product growth from 1950 to 2017.

---

## 📌 Table of Contents

- [Project Overview](#project-overview)
- [Problem Statement](#problem-statement)
- [Objectives](#objectives)
- [Dataset](#dataset)
- [Tools & Libraries](#tools--libraries)
- [Project Workflow](#project-workflow)
  - [Step 1: Data Loading & Exploration](#step-1-data-loading--exploration)
  - [Step 2: Merging the Datasets](#step-2-merging-the-datasets)
  - [Step 3: Data Cleaning](#step-3-data-cleaning)
  - [Step 4: Answering the Business Questions](#step-4-answering-the-business-questions)
- [Key Findings](#key-findings)
- [Code Highlights](#code-highlights)
- [Limitations](#limitations)
- [What I Learned](#what-i-learned)
- [Future Scope](#future-scope)
- [Project Structure](#project-structure)
- [Connect](#connect)

---

## Project Overview

Lego is one of the most recognized toy brands in the world — but it hasn't always been smooth sailing. In the late 1990s, Lego faced a serious financial crisis and was only able to turn things around through two key moves: an internally developed brand called Bionicle, and the introduction of its first licensed theme — Star Wars.

This project analyzes Lego's historical set data to understand the impact of licensed themes on Lego's product lineup, with a specific focus on Star Wars. Using Python and Pandas, the analysis answers three business questions that help explain Lego's product strategy and growth trajectory over nearly seven decades.

---

## Problem Statement

Lego's product catalog has grown significantly since the 1950s, and the introduction of licensed themes in the late 1990s marked a turning point in the company's history. The goal of this project is to analyze Lego's set release data to quantify the role of licensed themes — particularly Star Wars — in Lego's overall product mix, and to understand how theme dominance has shifted over time.

---

## Objectives

Three core questions were investigated:

1. **What percentage of all licensed sets ever released were Star Wars themed?**
2. **In which year was Star Wars not the most popular licensed theme (in terms of number of sets released that year)?**
3. **How many unique Lego sets were released each year from 1950 to 2017?**

---

## Dataset

Two CSV files were used for this analysis:

### lego_sets.csv

| Column | Description |
|---|---|
| `set_num` | Unique identifier for each Lego set (null = duplicate or invalid) |
| `name` | Name of the Lego set |
| `year` | Year the set was released |
| `num_parts` | Number of pieces in the set |
| `theme_name` | Sub-theme of the set |
| `parent_theme` | Parent theme the set belongs to (e.g., Star Wars, Harry Potter) |

### parent_themes.csv

| Column | Description |
|---|---|
| `id` | Unique theme ID |
| `name` | Name of the parent theme |
| `is_licensed` | Boolean — whether the theme is a licensed IP (True/False) |

> ⚠️ Note: A null value in `set_num` indicates a duplicate or invalid record and was excluded from percentage calculations as specified in the dataset instructions.

---

## Tools & Libraries

| Tool | Purpose |
|---|---|
| **Python 3** | Core programming language |
| **Pandas** | Data loading, merging, filtering, grouping, and aggregation |
| **Jupyter Notebook** | Interactive development and documentation |

---

## Project Workflow

### Step 1: Data Loading & Exploration

Both datasets were loaded using `pd.read_csv()` and inspected using `.head()` to understand their structure before any transformation.

```python
import pandas as pd

df = pd.read_csv('lego_sets.csv')
df.head()

theme = pd.read_csv('parent_themes.csv')
theme.head(50)
```

Key observations from initial exploration:
- `lego_sets.csv` contains set-level data but lacks a direct `is_licensed` flag
- `parent_themes.csv` contains the `is_licensed` column mapped to parent theme names
- The two datasets needed to be joined on `parent_theme` (from sets) and `name` (from themes)

---

### Step 2: Merging the Datasets

A merge was performed to bring the `is_licensed` flag from `parent_themes` into the main sets dataframe. Since the join columns had different names (`parent_theme` vs `name`), the `left_on` and `right_on` parameters were used explicitly.

```python
merged_df = df.merge(theme, left_on='parent_theme', right_on='name')
```

This produced two `name` columns in the result (`name_x` and `name_y`). The redundant `name_y` column was dropped using `inplace=True` to modify the dataframe directly:

```python
merged_df.drop(columns='name_y', inplace=True)
```

> 💡 **Learning moment:** Using `.drop()` without `inplace=True` returns a new dataframe but doesn't modify the original — a common beginner mistake that was caught and corrected during this step.

---

### Step 3: Data Cleaning

Before calculating percentages, null values in the `set_num` column were identified and handled.

**Checking for nulls:**
```python
print(merged_df['set_num'].isnull().sum())
# Output: 153
```

153 records had null `set_num` values, indicating duplicates or invalid entries. As per the dataset instructions, these were excluded from licensed set calculations using `dropna(subset=['set_num'])` — this targeted approach drops only rows where `set_num` is null, rather than removing any row with any null value.

```python
licensed = merged_df[merged_df['is_licensed']]
licensed = licensed.dropna(subset=['set_num'])
```

---

### Step 4: Answering the Business Questions

#### Question 1 — What percentage of all licensed sets were Star Wars themed?

```python
Star_wars = licensed[licensed['parent_theme'] == 'Star Wars']
the_force = int(Star_wars.shape[0] / licensed.shape[0] * 100)
print(the_force)
# Output: 51
```

**Answer: 51%** — More than half of all licensed Lego sets ever released were Star Wars themed.

---

#### Question 2 — In which year was Star Wars not the most popular licensed theme?

The approach here was to group licensed sets by year and theme, count the number of sets per group, then extract the top theme per year and find where Star Wars wasn't on top.

```python
licensed_sorted = licensed.sort_values('year')
licensed_sorted['Count'] = 1

summed_df = licensed_sorted.groupby(['year', 'parent_theme']).sum().reset_index()

max_df = summed_df.sort_values('Count', ascending=False).drop_duplicates(['year'])
max_df.sort_values('year', inplace=True)
```

**Answer: 2017** — Super Heroes surpassed Star Wars as the most-released licensed theme that year.

---

#### Question 3 — How many unique sets were released each year (1950–2017)?

A cleaned dataframe was created by removing null `set_num` rows, then grouped by year to count total unique sets released.

```python
clean_df = merged_df[~merged_df['set_num'].isnull()]
clean_df['count'] = 1

sets_per_year = clean_df.groupby(['year']).sum().reset_index()[['year', 'count']]
```

**Key trend:** Lego released just 7 sets in 1950. By 2014, that number had grown to 715 sets — a 100x increase over six decades.

---

## Key Findings

- **51% of all licensed Lego sets ever released were Star Wars themed** — confirming just how dominant the franchise has been in Lego's licensed product strategy since 1999.
- **2017 was the only year Star Wars was not the top licensed theme**, when Super Heroes (DC/Marvel) released more sets that year.
- **Lego's product catalog grew dramatically over time** — from single digits in the early 1950s to 500–700+ sets per year in the 2010s, reflecting both the company's recovery from its late-90s crisis and the expansion driven by licensed IP deals.
- **Star Wars sets began in 1999** — the same year Lego was navigating financial difficulties — directly tying the brand's recovery to the Star Wars licensing deal.
- The analysis confirms the historical narrative: licensed themes, led by Star Wars, played a measurable and significant role in Lego's product growth.

---

## Code Highlights

**Targeted null removal with `dropna(subset=...)`**
Rather than dropping all rows with any null value, only rows where the key identifier column (`set_num`) was null were removed — preserving as much valid data as possible.

```python
licensed = licensed.dropna(subset=['set_num'])
```

**Finding the top theme per year with `drop_duplicates`**
After sorting by count in descending order, `drop_duplicates(['year'])` was used to keep only the top-ranked theme for each year — a clean and efficient way to get the winner per group.

```python
max_df = summed_df.sort_values('Count', ascending=False).drop_duplicates(['year'])
```

**Two methods for the same calculation**
The Star Wars percentage was deliberately calculated two different ways to verify consistency — using `.shape[0]` and `len()` — both confirming 51%.

```python
the_force = int(Star_wars.shape[0] / licensed.shape[0] * 100)
force = int(len(Star_wars) / len(licensed) * 100)
```

---

## Limitations

- The dataset covers sets up to 2017 only — more recent licensed themes like Minecraft, Stranger Things, or Marvel Phase 4 sets are not included.
- `num_parts` had missing values for a number of sets, which was not imputed — any analysis involving part counts would need to address this separately.
- The percentage calculation rounds to the nearest integer, so the actual value sits somewhere between 51% and 52%.
- The analysis uses set count as the measure of "popularity" — a revenue or unit sales based measure might tell a different story.
- A `SettingWithCopyWarning` was observed when adding a column to a filtered dataframe slice — this is a common Pandas behavior and should ideally be addressed using `.loc[]` in a production context.

---

## What I Learned

- How to merge two dataframes with differently named join keys using `left_on` and `right_on` in Pandas.
- The difference between `inplace=True` and reassigning a dataframe — and why it matters when chaining operations.
- How `dropna(subset=[...])` allows targeted null removal on specific columns rather than blanket row deletion.
- How to use `groupby()` + `drop_duplicates()` as an efficient pattern for finding the top value per group without writing complex filtering logic.
- The importance of verifying results using two different methods — both approaches produced 51%, building confidence in the answer.
- How clean, well-commented notebooks make analysis reproducible and easier to share with others.

---

*This project was completed as part of a structured data analytics course using a real-world inspired Lego dataset.*
