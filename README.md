# LEGO Dataset Analysis

Licensed IP grew from zero to 27.6% of LEGO's annual releases between 1999 and 2017 — but the deeper transformation was complexity. Average parts per set grew 8× over seven decades, from 32 in the 1950s to 259 by the 2020s, while total annual volume scaled from 5 sets to over 6,800. This project maps that transformation across five analytical dimensions using three relational datasets covering every LEGO product ever produced.

The analysis uses 15,710 sets, 596 themes, and 135 colours from Rebrickable, extended with inventory-level part data to trace colour introductions year by year. The pipeline answers: when did licensed IP enter the portfolio, how has set complexity stratified into distinct tiers, how did the colour vocabulary expand across decades, what does the full theme hierarchy look like when resolved to root parents, and how do all these metrics shift decade by decade.

---

## Quick Start

```bash
git clone https://github.com/xavier-oc-programming/lego-dataset-analysis.git
cd lego-dataset-analysis
pip install -r requirements.txt
jupyter notebook notebooks/analysis/lego_analysis.ipynb
```

The notebook decompresses `data/inventory_parts.csv.gz` automatically on first run — no manual setup required.

---

## Analysis Flow

```
pipeline
    │
    │  ── [Ingestion] ────────────────────────────────────────────────────
    ├── pd.read_csv()  →  colors.csv        →  colors      (135 colours)
    ├── pd.read_csv()  →  sets.csv          →  sets        (15,710 sets)
    ├── pd.read_csv()  →  themes.csv        →  themes      (596 themes)
    ├── pd.read_csv()  →  inventories.csv   →  inventories
    ├── gunzip + read  →  inventory_parts.csv.gz  →  inv_parts
    │
    │  ── [Original analysis] ────────────────────────────────────────────
    ├── colors.nunique / groupby('is_trans')           →  colour counts
    ├── sets.sort_values('year').head()                →  first sets / debut year
    ├── sets.sort_values('num_parts').tail()           →  top 5 largest sets
    ├── sets.groupby('year').count()                   →  sets_by_year
    ├── sets.groupby('year').agg(nunique)              →  themes_by_year
    ├── twinx dual-axis line chart                     →  sets & themes over time
    ├── groupby('year').agg(mean)                      →  avg_parts scatter plot
    ├── value_counts + pd.merge(themes)                →  top themes bar chart
    │
    │  ── [Analysis 1 — Licensed IP] ─────────────────────────────────────
    ├── keyword match on theme names                   →  ip_type column
    ├── merge(sets, themes[ip_type])                   →  sets_ip
    ├── groupby(['year','ip_type']).size().unstack()   →  stacked bar chart
    ├── ip_pct = Licensed / total per year             →  licensed share trend
    └── top-5 licensed themes by set count
    │
    │  ── [Analysis 2 — Complexity Clustering] ───────────────────────────
    ├── num_parts / annual_mean  →  relative_complexity feature
    ├── StandardScaler + KMeans(k=4)                   →  4 complexity tiers
    ├── label by mean parts: Starter→Standard→Advanced→Expert
    ├── scatter(year, num_parts, colour=cluster)       →  cluster scatter
    └── cluster distribution bar chart per top-10 theme
    │
    │  ── [Analysis 3 — Colour Evolution] ────────────────────────────────
    ├── inv_parts → inventories → sets  →  color_year (year per colour use)
    ├── groupby('color_id').year.min()  →  first_appearance per colour
    ├── cumsum of new colours by year   →  palette growth line chart
    ├── first_year where is_trans=='t'  →  transparent colour debut
    ├── new colours introduced per decade  →  decade bar chart
    └── hue-sorted swatch grid (all 135 colours)
    │
    │  ── [Analysis 4 — Theme Hierarchy] ─────────────────────────────────
    ├── recursive resolve_root(theme_id)  →  root_id + depth per theme
    ├── max depth = 2 (broad, not deep taxonomy)
    ├── sets.merge(root_id) → groupby(root_id).count()  →  root set totals
    └── bar chart: top 15 parent themes by cumulative set count
    │
    │  ── [Analysis 5 — Decade Summary] ──────────────────────────────────
    ├── sets['decade'] = (year // 10) * 10
    ├── groupby(decade): total sets, unique themes, avg parts, % licensed
    ├── most popular theme per decade
    └── pandas Styler with Blues gradient on numeric columns
```

---

## Key Findings

**Licensed IP grew steadily but never dominated.** Star Wars, introduced in 1999, remains the single largest licensed franchise at 776 sets. Licensed themes peaked at 27.6% of annual releases in 2017, up from 0% before 1999, 8.2% in 2000, 15.6% in 2010, and 22.0% in 2020. The portfolio remained majority original-IP throughout — but licensed sets disproportionately drove complexity and premium positioning.

**Set complexity grew 8× over seven decades.** Average parts per set rose from 32 in the 1950s to 259 in the 2020s. K-Means clustering identifies four tiers: Starter, Standard, Advanced, and Expert. Technic and architectural lines dominate the Expert cluster; City and seasonal sets anchor the Starter band. The gap between tiers has widened over time — Expert sets today have roughly 4× the parts of Expert sets from the 1980s.

**The colour palette tripled in the 2000s alone.** Of 132 colours used in sets, 53 were introduced in the 2000s — more than in the preceding five decades combined. Transparent colours debuted in 1954. The 1990s added 38 colours; the 2010s added only 14, suggesting the palette neared saturation. The full hue-sorted swatch grid is in `plots/colour_palette_swatches.png`.

**LEGO's theme hierarchy is broad, not deep.** The self-referential parent/child structure reaches a maximum depth of 2 levels. Town is the largest parent theme by cumulative set count (1,304 sets), followed by Duplo (1,268) and Gear (1,049). Star Wars ranks fourth with 791 sets despite spanning only 22 years. Technic leads on sub-theme count, reflecting decades of product line expansion.

**Volume scaled 1,000× while complexity grew 8×.** The decade table shows 5 sets in the 1940s, 1,212 in the 1980s, and 6,813 in the 2010s. The 2000s were the inflection decade: set count doubled, licensed share jumped from 0.7% to 8.5%, and 53 new colours entered the palette — all driven by the franchise deals signed at the turn of the millennium.

---

## Dataset Schema

### colors.csv

| Column | Type | Description |
|--------|------|-------------|
| id | int | Unique colour ID |
| name | str | Colour name |
| rgb | str | Hex RGB value (no `#` prefix) |
| is_trans | str | `t` = transparent, `f` = opaque |

### sets.csv

| Column | Type | Description |
|--------|------|-------------|
| set_num | str | Unique set identifier |
| name | str | Set name |
| year | int | Release year |
| theme_id | int | Foreign key → themes.id |
| num_parts | int | Part count |

**Computed columns**

| Column | Derived from | Description |
|--------|-------------|-------------|
| ip_type | theme name keyword match | `Licensed` or `Original` |
| relative_complexity | num_parts / annual mean | Part count relative to era average |
| complexity | KMeans cluster label | Starter / Standard / Advanced / Expert |
| root_id | recursive parent_id walk | Top-level parent theme ID |
| decade | year // 10 * 10 | Decade bin |

### themes.csv

| Column | Type | Description |
|--------|------|-------------|
| id | int | Unique theme ID |
| name | str | Theme name |
| parent_id | float | Parent theme ID; NaN if top-level |

### inventories.csv

| Column | Type | Description |
|--------|------|-------------|
| id | int | Inventory ID |
| version | int | Inventory version |
| set_num | str | Foreign key → sets.set_num |

### inventory_parts.csv.gz

| Column | Type | Description |
|--------|------|-------------|
| inventory_id | int | Foreign key → inventories.id |
| part_num | str | Part identifier |
| color_id | int | Foreign key → colors.id |
| quantity | int | Count in this inventory |
| is_spare | bool | Whether this is a spare part |

---

## Architecture

```
lego-dataset-analysis/
│
├── notebooks/
│   ├── analysis/
│   │   └── lego_analysis.ipynb        # main analysis — original + 5 improvements
│   └── concepts/                      # annotated concept notebooks
│       ├── 00__Overview.ipynb
│       ├── 01__HTML_Markdown_Notebooks.ipynb
│       ├── 02__Exploring_LEGO_Colours.ipynb
│       ├── 03__Oldest_and_Largest_Sets.ipynb
│       ├── 04__Sets_Published_over_Time.ipynb
│       ├── 05__Pandas_agg_Function.ipynb
│       ├── 06__Superimposed_Line_Charts.ipynb
│       ├── 07__Scatter_Plots_Parts_per_Set.ipynb
│       ├── 08__Relational_Schemas_Keys.ipynb
│       ├── 09__Merge_DataFrames_Bar_Charts.ipynb
│       └── 10__Learning_Points_Summary.ipynb
│
├── data/
│   ├── colors.csv                     # 135 LEGO colours with RGB and transparency
│   ├── sets.csv                       # 15,710 sets with year, theme, part count
│   ├── themes.csv                     # 596 themes in parent/child hierarchy
│   ├── inventories.csv                # set → inventory mapping
│   └── inventory_parts.csv.gz         # colour usage per inventory (decompressed at runtime)
│
├── plots/                             # generated charts (not committed)
│   ├── licensed_vs_original.png
│   ├── complexity_clusters_scatter.png
│   ├── complexity_clusters_by_theme.png
│   ├── colour_palette_growth.png
│   ├── colour_palette_swatches.png
│   ├── colour_introductions_by_decade.png
│   └── theme_hierarchy_top15.png
│
├── assets/
│   ├── bricks.jpg
│   ├── lego_sets.png
│   ├── lego_themes.png
│   └── rebrickable_schema.png
│
├── docs/
│   └── COURSE_NOTES.md
│
├── requirements.txt
├── .gitignore
└── README.md
```

---

## Visualisations

| File | Description |
|------|-------------|
| `licensed_vs_original.png` | Stacked bar: licensed vs original set count per year from 1980 |
| `complexity_clusters_scatter.png` | Scatter: year vs part count coloured by K-Means tier |
| `complexity_clusters_by_theme.png` | Bar chart: cluster distribution for the top 10 themes |
| `colour_palette_growth.png` | Line chart: cumulative colour count over time with transparent-colour debut marked |
| `colour_palette_swatches.png` | Grid of all 135 colours sorted by hue; white border = transparent |
| `colour_introductions_by_decade.png` | Bar chart: new colours introduced per decade |
| `theme_hierarchy_top15.png` | Bar chart: top 15 parent themes by total set count across all sub-themes |

Charts are generated at 150 dpi and saved to `plots/` when the notebook is executed. They are not committed — run the notebook to reproduce them.

---

## Operations Reference

| Value | Location | Description |
|-------|----------|-------------|
| `../../data/colors.csv` | lego_analysis.ipynb | Path to colours dataset |
| `../../data/sets.csv` | lego_analysis.ipynb | Path to sets dataset |
| `../../data/themes.csv` | lego_analysis.ipynb | Path to themes dataset |
| `../../data/inventories.csv` | lego_analysis.ipynb | Path to inventories dataset |
| `../../data/inventory_parts.csv.gz` | lego_analysis.ipynb | Compressed inventory parts (auto-decompressed on first run) |
| `../../plots/` | lego_analysis.ipynb | Output directory for all charts |
| `figsize=(16, 10)` | lego_analysis.ipynb | Default figure size |
| `dpi=150` | lego_analysis.ipynb | Chart export resolution |
| `k=4` | Analysis 2 | Number of K-Means clusters |
| `random_state=42` | Analysis 2 | KMeans seed for reproducibility |

---

## Background

This project was built as part of 100 Days of Code — The Complete Python Pro Bootcamp, Day 74: Aggregate and Merge Data with Pandas. See [docs/COURSE_NOTES.md](docs/COURSE_NOTES.md) for the original exercise brief and concept notes.

---

## Dependencies

| Package | Purpose |
|---------|---------|
| pandas | DataFrame loading, groupby, merge, aggregation, Styler |
| matplotlib | Line charts, scatter plots, bar charts, swatch grids |
| numpy | Numerical operations, NaN handling |
| scikit-learn | KMeans clustering, StandardScaler |
| notebook | Jupyter notebook runtime |
