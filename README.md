# LEGO Dataset Analysis

Exploratory analysis of the LEGO dataset — sets, colours, themes, and complexity over time.

This analysis investigates the full history of LEGO product releases using three relational datasets from Rebrickable. It answers questions about when LEGO launched its first sets, which themes dominate the catalogue, how product complexity has evolved, and what strategic inflection points are visible in the year-on-year data. Charts produced include a dual-axis line chart of sets and themes over time, a scatter plot of average part count by year, and a bar chart of the top LEGO themes by set count.

The datasets come from [Rebrickable](https://rebrickable.com/downloads/), which compiles a complete inventory of every LEGO element ever produced. Three CSV files are used: `colors.csv` (135 colours with RGB values and transparency flags), `sets.csv` (over 11,000 sets with year, theme, and part count), and `themes.csv` (457 themes in a self-referential parent/child hierarchy). The pipeline loads and joins these files with pandas, aggregates by year and theme, and derives computed columns for analysis.

No external APIs or credentials are required. All data is committed to the repository as curated seed files sourced from Rebrickable.

---

## Table of Contents

1. [Quick start](#1-quick-start)
2. [Analysis flow](#2-analysis-flow)
3. [Features](#3-features)
4. [Dataset schema](#4-dataset-schema)
5. [Architecture](#5-architecture)
6. [Notebook reference](#6-notebook-reference)
7. [Configuration reference](#7-configuration-reference)
8. [Course context](#8-course-context)
9. [Dependencies](#9-dependencies)

---

## 1. Quick start

```bash
git clone https://github.com/xavier-oc-programming/lego-dataset-analysis.git
cd lego-dataset-analysis
pip install -r requirements.txt
jupyter notebook
```

Open `practice/A_01_LEGO_Analysis.ipynb` first to run the full analysis end-to-end.  
The `theory/` notebooks contain annotated lesson explanations for each concept.

---

## 2. Analysis flow

```
pipeline
    │
    │  ── [Ingestion] ────────────────────────────────────────────────────
    ├── pd.read_csv()  →  colors.csv   →  colors
    ├── pd.read_csv()  →  sets.csv     →  sets
    ├── pd.read_csv()  →  themes.csv   →  themes
    │
    │  ── [Colour exploration] ───────────────────────────────────────────
    ├── colors['name'].nunique()              →  total count of unique colours
    ├── colors.groupby('is_trans').count()    →  transparent vs. opaque split
    │
    │  ── [Set exploration] ──────────────────────────────────────────────
    ├── sets.sort_values('year').head()             →  first sets ever released and debut year
    ├── sets[sets['year'] == first_year].shape[0]   →  number of sets sold in debut year
    ├── sets.sort_values('num_parts', ascending=False).head()  →  top 5 largest sets
    │
    │  ── [Year-on-year aggregation] ─────────────────────────────────────
    ├── sets.groupby('year').count()                      →  sets_by_year
    ├── sets.groupby('year').agg({'theme_id': nunique})   →  themes_by_year
    ├── themes_by_year.rename({'theme_id': 'nr_themes'})  →  clean column name
    ├── [:-2] slice on both series                        →  exclude incomplete final two years
    │
    │  ── [Year trend visualisation] ─────────────────────────────────────
    ├── ax1 = plt.gca() / ax2 = ax1.twinx()   →  dual Y-axis figure
    ├── ax1.plot(sets_by_year)                 →  sets over time on left axis (green)
    ├── ax2.plot(themes_by_year)               →  themes over time on right axis (blue)
    │
    │  ── [Complexity trend] ─────────────────────────────────────────────
    ├── sets.groupby('year').agg({'num_parts': mean})  →  avg_parts — average parts per year
    ├── plt.scatter(avg_parts.index, avg_parts)        →  part count growth trend over time
    │
    │  ── [Theme ranking] ────────────────────────────────────────────────
    ├── sets['theme_id'].value_counts()               →  set count per theme_id
    ├── pd.DataFrame({'id': ..., 'set_count': ...})   →  convert Series to DataFrame
    ├── pd.merge(set_theme_count, themes, on='id')    →  merged_df — theme names joined in
    └── plt.bar(merged_df.name[:10], merged_df.set_count[:10])  →  top 10 themes by set count
```

---

## 3. Features

- Count of unique LEGO colours, split by transparent vs. opaque
- Year LEGO first launched and number of sets in the debut year
- Largest LEGO set ever created (by part count)
- Year-on-year count of new sets and new themes (dual-axis line chart)
- Average parts-per-set over time — does complexity increase? (scatter plot)
- Top LEGO themes by total set count, with parent theme resolved (bar chart)

---

## 4. Dataset schema

### colors.csv

| Column | Type | Description |
|--------|------|-------------|
| id | int | Unique colour ID |
| name | str | Colour name (e.g. "Black", "Bright Green") |
| rgb | str | Hex RGB value without `#` prefix |
| is_trans | str | `t` if transparent, `f` if opaque |

### sets.csv

| Column | Type | Description |
|--------|------|-------------|
| set_num | str | Unique set identifier (primary key) |
| name | str | Set name |
| year | int | Year the set was released |
| theme_id | int | Foreign key → themes.id |
| num_parts | int | Number of parts in the set |

**Computed columns (added at runtime)**

| Column | Derived from | Description |
|--------|-------------|-------------|
| set_count | groupby(year) | Number of sets released per year |
| avg_parts | groupby(year).mean() | Average part count per year |

### themes.csv

| Column | Type | Description |
|--------|------|-------------|
| id | int | Unique theme ID (primary key) |
| name | str | Theme name (e.g. "Technic", "Star Wars") |
| parent_id | float | ID of parent theme; NaN if top-level |

---

## 5. Architecture

```
lego-dataset-analysis/
│
├── theory/                              # Lesson notebooks — concepts and annotated solutions
│   ├── 00__Overview.ipynb               # Day 74 goals and final output preview
│   ├── 01__HTML_Markdown_Notebooks.ipynb# HTML in Markdown cells, image embedding
│   ├── 02__Exploring_LEGO_Colours.ipynb # nunique, value counts, boolean filters
│   ├── 03__Oldest_and_Largest_Sets.ipynb# sort_values, nlargest, nsmallest
│   ├── 04__Sets_Published_over_Time.ipynb# groupby + line chart
│   ├── 05__Pandas_agg_Function.ipynb    # .agg() with multiple aggregations
│   ├── 06__Superimposed_Line_Charts.ipynb# twinx, dual Y-axes, axis colouring
│   ├── 07__Scatter_Plots_Parts_per_Set.ipynb# scatter plot, alpha, trend reading
│   ├── 08__Relational_Schemas_Keys.ipynb# primary/foreign keys, schema diagrams
│   ├── 09__Merge_DataFrames_Bar_Charts.ipynb# pd.merge, bar charts
│   └── 10__Learning_Points_Summary.ipynb# Day summary and key takeaways
│
├── practice/
│   └── A_01_LEGO_Analysis.ipynb         # Student exercise notebook — full analysis
│
├── data/
│   ├── colors.csv                       # 135 LEGO colours with RGB and transparency
│   ├── sets.csv                         # 11,000+ LEGO sets with year and part count
│   └── themes.csv                       # 457 themes in parent/child hierarchy
│
├── assets/
│   ├── bricks.jpg                       # Header image used in notebooks
│   ├── lego_sets.png                    # Reference image — example sets
│   ├── lego_themes.png                  # Reference image — themes breakdown
│   └── rebrickable_schema.png           # Entity-relationship diagram for datasets
│
├── docs/
│   └── COURSE_NOTES.md                  # Exercise brief, key concepts, data description
│
├── requirements.txt                     # Pinned package requirements
├── .gitignore
└── README.md
```

---

## 6. Notebook reference

### theory/

| Notebook | Key methods covered | Question answered |
|----------|--------------------|--------------------|
| 00__Overview | — | What will we build today? |
| 01__HTML_Markdown_Notebooks | HTML in Markdown, `<img>` | How to style notebooks? |
| 02__Exploring_LEGO_Colours | `.nunique()`, boolean filter | How many colours exist? How many transparent? |
| 03__Oldest_and_Largest_Sets | `.sort_values()`, `.nlargest()` | When did LEGO launch? What is the largest set? |
| 04__Sets_Published_over_Time | `.groupby()`, `.plot()` | How has the number of sets grown each year? |
| 05__Pandas_agg_Function | `.agg({col: fn})` | How to apply multiple aggregations at once? |
| 06__Superimposed_Line_Charts | `ax.twinx()`, tick params | How to compare two metrics on different scales? |
| 07__Scatter_Plots_Parts_per_Set | `ax.scatter()`, `alpha` | Has set complexity grown over time? |
| 08__Relational_Schemas_Keys | primary/foreign key concepts | How are the three tables related? |
| 09__Merge_DataFrames_Bar_Charts | `pd.merge()`, `.plot(kind='bar')` | Which themes have the most sets? |
| 10__Learning_Points_Summary | — | What did we learn today? |

### practice/

| Notebook | Key methods covered | Question answered |
|----------|--------------------|--------------------|
| A_01_LEGO_Analysis | `.read_csv()`, `.nunique()`, `.groupby()`, `.agg()`, `pd.merge()`, `.plot()`, `ax.twinx()`, `ax.scatter()` | Full analysis: colours, oldest sets, largest set, year trends, top themes |

---

## 7. Configuration reference

| Value | Location | Description |
|-------|----------|-------------|
| `../data/colors.csv` | A_01_LEGO_Analysis.ipynb | Relative path to colours dataset |
| `../data/sets.csv` | A_01_LEGO_Analysis.ipynb | Relative path to sets dataset |
| `../data/themes.csv` | A_01_LEGO_Analysis.ipynb | Relative path to themes dataset |
| `figsize=(16, 10)` | A_01_LEGO_Analysis.ipynb | Default figure size for charts |
| `alpha=0.4` | A_01_LEGO_Analysis.ipynb | Scatter plot point transparency |

---

## 8. Course context

100 Days of Code — The Complete Python Pro Bootcamp, Day 74: Aggregate and Merge Data with Pandas.  
See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for the full exercise brief and concept notes.

---

## 9. Dependencies

| Module | Used in | Purpose |
|--------|---------|---------|
| pandas | practice/, theory/ | DataFrame loading, groupby, merge, aggregation |
| matplotlib | practice/, theory/ | Line charts, scatter plots, bar charts |
| numpy | practice/ | Numerical operations (implicitly via pandas) |
| notebook | all | Jupyter notebook runtime |
