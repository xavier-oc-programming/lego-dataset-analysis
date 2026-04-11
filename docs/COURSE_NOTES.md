# Course Notes — Day 74: Aggregate and Merge Data with Pandas

**Course:** 100 Days of Code — The Complete Python Pro Bootcamp  
**Day:** 74  
**Topics:** Data aggregation, merging DataFrames, relational schemas, visualisation

---

## Exercise Brief

Analyse the LEGO dataset sourced from [Rebrickable](https://rebrickable.com/downloads/) to answer the following questions:

- **Colours** — How many different colours does the LEGO company produce? How many are transparent vs. opaque?
- **Oldest sets** — In which year were the first LEGO sets released? How many sets did the company sell when it first launched?
- **Largest set** — What is the most enormous LEGO set ever created and how many parts did it have?
- **Sets over time** — When did the LEGO company really expand its product offering? Can we spot a change in company strategy based on year-on-year theme and set counts?
- **Complexity trend** — Did LEGO sets grow in size and complexity over time? Do older LEGO sets tend to have more or fewer parts than newer sets?
- **Theme leaders** — Which LEGO theme has the most sets? Is it an original LEGO theme or a licensed one (e.g. Harry Potter, Marvel)?

---

## Key Concepts Covered

### HTML & Markdown in Notebooks
- Using HTML tags inside Jupyter Markdown cells for richer formatting
- Unordered lists with custom bullet types (`<ul type="square">`)
- Embedding images with `<img>` tags

### Data Exploration
- `pd.read_csv()` for loading tabular data
- `.head()`, `.tail()`, `.shape`, `.dtypes`, `.describe()`
- `.nunique()` — count of distinct values in a column
- `.isna().values.any()` — check for missing values
- Value counts and boolean filtering

### Aggregation
- `.groupby()` — group rows by a column value
- `.agg()` — apply multiple aggregation functions at once
  ```python
  sets_by_year.agg({'set_num': pd.Series.count, 'num_parts': pd.Series.mean})
  ```
- `.sort_values()`, `.nlargest()`, `.nsmallest()`

### Merging DataFrames
- `pd.merge(left, right, on='key')` — inner join on a shared column
- Relational database concepts: primary keys, foreign keys
- The `themes` table uses a self-referential `parent_id` to represent sub-themes

### Visualisation (matplotlib)
- Line charts — `ax.plot()`
- Superimposing two lines with separate Y axes using `ax.twinx()`
- Setting axis colours, labels, and tick parameters to match each dataset
- Scatter plots — `ax.scatter()` with `alpha` for overplotting
- Bar charts — `.plot(kind='bar')` on a grouped DataFrame

---

## Data Files

| File | Description |
|------|-------------|
| `colors.csv` | All LEGO colours with RGB hex and transparency flag |
| `sets.csv` | Every LEGO set: set number, name, year, theme ID, part count |
| `themes.csv` | LEGO themes with optional parent theme (self-referential) |
