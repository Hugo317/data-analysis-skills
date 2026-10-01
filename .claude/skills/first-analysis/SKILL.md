---
name: first-analysis
description: Perform a standard first-pass analysis of a new dataset before deeper analysis. Use this when starting work on an unfamiliar CSV, Excel file, dataframe, database extract, or similar business dataset. Inspect structure, data quality, column roles, distributions, dates, business metrics, products, regions, concentration, and potential analytical questions. This should be saved and showed in an ipynb.
---

# First Analysis

When starting analysis on a new dataset, perform a consistent first-pass inspection before making assumptions or doing deeper analysis.

The goal is not merely to print pandas outputs. The goal is to understand what the dataset contains, identify data-quality issues, recognize its business context, surface useful findings, and suggest sensible next analytical questions.

## 1. Load all the data

Load the complete dataset into the appropriate analysis environment.

- Do not inspect only a sample unless the dataset is too large to load.
- Preserve the original data.
- Report dataset shape: rows × columns.
- Identify data types for every column.
- If obvious date columns exist, identify their date range.
- If loading fails or data is malformed, explain the issue before continuing.

## 2. Basic dataframe inspection

Run the standard inspection:

```python
df.info()
```

Then:

```python
df.head()
```

Also report:

- Number of rows
- Number of columns
- Column names
- Data types
- Memory usage when available

## 3. Missing values

Calculate missing values for every column.

Prefer a table:

| Column | Missing Count | Missing % |
|---|---:|---:|

Example:

```python
missing = pd.DataFrame({
    "missing_count": df.isna().sum(),
    "missing_pct": df.isna().mean() * 100
}).sort_values("missing_count", ascending=False)
```

Highlight columns with meaningful missingness rather than merely dumping the output.

Do not automatically recommend deleting missing values. First determine whether the missingness appears important.

## 4. Descriptive statistics

Run:

```python
df.describe(include="all")
```

For numeric columns, pay attention to:

- count
- mean
- standard deviation
- minimum
- quartiles
- maximum
- median

For categorical columns, pay attention to:

- count
- number of unique values
- most common value
- frequency of the most common value

Do not blindly interpret categorical statistics as numeric statistics.

## 5. Number of unique values and cardinality

Calculate:

```python
df.nunique(dropna=False).sort_values()
```

Use this to identify:

- IDs
- categorical columns
- high-cardinality columns
- columns with only one value
- possible duplicate identifiers

Classify columns where possible:

```text
ID
Date
Categorical
Numerical
Currency
Percentage
Quantity
Text
Geographical
Boolean
```

Use column names AND actual values to make this classification. Do not rely only on names.

Flag likely:

- Identifier columns
- Low-cardinality categorical columns
- High-cardinality categorical columns
- Constant columns
- Columns that may contain mostly unique values

## 6. Data quality checks

Check for:

### Duplicate rows

```python
df.duplicated().sum()
```

### Duplicate IDs

If an ID-like column exists, check whether it is actually unique.

### Constant columns

Find columns with only one unique value.

### Impossible or suspicious values

Look for values that may be logically invalid based on the column meaning, such as:

- Negative age
- Negative quantity where negative quantity is not expected
- Negative revenue where not expected
- Discount below 0% or above 100%
- Profit larger than revenue when the business definition would make that suspicious
- Invalid dates
- Infinite values
- Unexpected null-like strings such as `"N/A"`, `"unknown"`, `"-"`, or empty strings

Do not automatically label these as errors. Flag them for investigation and explain why they are suspicious.

### Category consistency

For important categorical columns, check for inconsistencies such as:

```text
Sweden
sweden
SWEDEN
 Sweden
```

Also check obvious spelling or formatting inconsistencies.

## 7. Numerical distributions and outliers

For important numerical columns, inspect:

- Mean
- Median
- Standard deviation
- Min/max
- Quartiles
- Skewness
- Potential outliers

Look for large differences between mean and median.

Flag potentially important patterns such as:

```text
Revenue is highly right-skewed.
Quantity contains unusually large values.
Average order value is substantially above the median.
```

Do not automatically remove outliers.

Explain that an outlier may be:

- An error
- A legitimate large transaction
- A special customer/product
- A data-entry issue

## 8. Date analysis

If date columns exist, inspect:

- Minimum date
- Maximum date
- Total time span
- Missing dates
- Invalid dates
- Duplicate dates where relevant
- Records per month/year
- Gaps in the time series

Example output:

```text
Dataset period:
January 2022 → December 2025

Records by year:
2022: 24,321
2023: 31,552
2024: 42,103
2025: 38,221
```

Identify whether the data is:

- Daily
- Weekly
- Monthly
- Yearly
- Transaction-level

## 9. Detect the business context

After the generic inspection, identify what kind of dataset this is.

Look for columns suggesting:

- Company/business: company, seller, manufacturer, brand, business name
- Profit: profit, net profit, gross profit, margin
- Revenue: revenue, sales, turnover
- Cost: cost, purchase cost, COGS
- Product: product, product name, item, SKU, model
- Region: region, country, state, city, territory, market, area
- Customer: customer, client, account
- Quantity: quantity, units, volume
- Price: price, selling price, purchase price, unit price
- Dates: date, order date, sale date, transaction date

Do not assume column names exactly match these examples. Use semantic equivalents and inspect values.

## 10. Company and profitability analysis

If the dataset represents a company or contains company-level financial information, investigate profitability.

If profit already exists:

- Show total profit.
- Show average profit where meaningful.
- Show profit by relevant dimensions.
- Identify whether the data represents one company or multiple companies.

If profit does not exist but revenue/sales and cost are available, calculate:

```python
profit = revenue - cost
```

Only do this after checking that the columns actually represent compatible revenue and cost concepts.

If margin can be calculated:

```python
margin_pct = profit / revenue * 100
```

Be precise about the definition:

```text
Profit = Revenue - Cost
Margin % = Profit / Revenue × 100
```

Do not confuse margin with markup.

If multiple companies exist, compare:

- Revenue
- Cost
- Profit
- Margin
- Growth where dates permit

## 11. Product analysis

If the company has products, identify the best and worst-performing products.

Preferred hierarchy for defining "best":

1. Profit
2. Revenue / sales
3. Quantity / units sold

Example:

```python
df.groupby("product")["profit"].sum().sort_values(ascending=False).head(10)
```

Normally show:

- Top 10 products by profit
- Bottom 10 products by profit

When possible, compare:

| Product | Revenue | Cost | Profit | Margin % | Quantity |
|---|---:|---:|---:|---:|---:|

Do not call a product "best" based on revenue if profit is available without explaining the metric.

Look for situations where rankings differ, e.g.:

```text
Product A has the highest revenue but Product B has the highest profit.
```

These differences are often analytically important.

## 12. Region analysis

If the dataset contains regions, identify the best and worst-performing regions.

Preferred hierarchy:

1. Profit
2. Revenue / sales
3. Quantity / volume

Show where appropriate:

- Top regions by profit
- Bottom regions by profit
- Top regions by revenue
- Top regions by quantity
- Margin by region

Be explicit about which metric determines "best."

## 13. Contribution and concentration analysis

For important dimensions such as products, regions, customers, or companies, calculate how much of the total business comes from the top groups.

Examples:

```text
Top 10 products generate 64% of total profit.
Top 3 regions generate 71% of total revenue.
Top 20 customers generate 48% of sales.
```

This is often more useful than a simple ranking.

When appropriate, perform a Pareto-style analysis:

```text
20% of products generate 78% of profit.
```

Describe this as concentration or a possible Pareto pattern rather than claiming a universal 80/20 law.

## 14. Segment and combination analysis

When multiple useful dimensions exist, look for meaningful combinations.

Examples:

```text
Region × Product
Customer × Region
Year × Product
Year × Region
```

Look for high-value combinations such as:

```text
Germany + Product X → highest profit combination
Sweden + Product Y → highest margin
```

Do not create every possible combination. Only analyze combinations that make business sense.

## 15. Relationships between important metrics

If the dataset contains multiple important numerical business metrics, investigate obvious relationships.

Examples:

```text
Revenue ↔ Quantity
Revenue ↔ Price
Profit ↔ Revenue
Profit ↔ Discount
Profit ↔ Quantity
```

At this stage, keep the analysis exploratory.

Useful methods can include:

- Correlation for appropriate numeric variables
- Simple grouped comparisons
- Scatterplots when useful

Do not claim causation from correlation.

## 16. Recommended visualizations

Do not automatically create large numbers of charts.

Only recommend or create visualizations that answer useful analytical questions.

For example:

```text
Recommended next visualizations:
1. Monthly revenue and profit trend
2. Top 10 products by profit
3. Profit by region
4. Profit margin distribution
5. Revenue vs quantity
```

Choose charts based on the structure of the dataset.

Avoid generating charts merely because a column exists.

## 17. Initial observations

Finish the first analysis with:

### Initial observations

Mention only observations supported by the data.

Possible observations:

- Major missing-value problems
- Suspicious data types
- Duplicate-looking identifiers
- Duplicate records
- Highly concentrated categories
- Strongly dominant products
- Strongly dominant regions
- Negative or unusual financial values
- Obvious outliers
- Large differences in profitability or margins
- Important date gaps
- Differences between revenue and profit rankings

Do not make causal claims at this stage.

## 18. Assumptions log

Record important assumptions made during the analysis.

Example:

```text
Assumptions:
- `sales` appears to represent gross revenue.
- `cost` appears to represent product acquisition cost.
- Profit was therefore calculated as sales - cost.
- `region` appears to represent sales region.
```

If an assumption materially affects a result, make it explicit.

Never silently invent business definitions.

## 19. Questions this dataset can answer

Based on the discovered columns and business context, generate 5–10 useful analytical questions for the next stage.

Examples:

```text
1. Which products generate the highest profit?
2. Which regions are growing fastest?
3. Are high-revenue products also the most profitable?
4. How has profit changed over time?
5. Which products have poor margins?
6. Is revenue concentrated among a small number of customers?
7. Which regions have strong revenue but weak margins?
```

Questions should be specific to the actual dataset rather than generic filler.

## 20. Final analyst-style summary

End with a concise summary rather than leaving the user with raw pandas output.

Example:

```text
FIRST ANALYSIS SUMMARY

Dataset:
1,240,231 rows × 14 columns

Data quality:
⚠️ 2 columns contain significant missing values
⚠️ 1.4% duplicate rows
⚠️ 3 potential outlier areas

Business:
Revenue: $42.1M
Profit: $6.8M
Margin: 16.2%

Products:
Top product: Product X
Top 10 products: 61% of profit

Regions:
Top region: Europe
Top 3 regions: 74% of profit

Most interesting findings:
• Profit is highly concentrated.
• Revenue and profit rankings differ considerably.
• Product X has the highest revenue, but Product Y has the highest profit.
• The dataset has a noticeable gap in the 2023 time series.

Recommended next analysis:
1. Investigate product profitability.
2. Analyze monthly profit trends.
3. Investigate regional margin differences.
```

The summary should be based on actual findings from the dataset.

## Output structure

For a normal business dataset, structure the first analysis roughly as:

1. **Dataset overview**
2. **Data quality**
3. **Missing values**
4. **Descriptive statistics**
5. **Unique values and cardinality**
6. **Column-role classification**
7. **Date analysis** — if applicable
8. **Business context**
9. **Profitability** — if applicable
10. **Products** — if applicable
11. **Regions** — if applicable
12. **Concentration / Pareto analysis** — if applicable
13. **Segment analysis** — if useful
14. **Relationships between metrics** — if useful
15. **Recommended visualizations**
16. **Initial observations**
17. **Assumptions**
18. **Questions this dataset can answer**
19. **Final analyst-style summary**

## Important rules

- Start broad before drilling down.
- Never assume that a column means something without checking its values and type.
- Do not calculate profit or margin using ambiguous columns without explaining the assumption.
- Prefer aggregated business metrics over arbitrary rankings.
- Use profit rather than revenue when the user asks which products/regions are "best" and profit is available.
- Do not automatically remove missing values, duplicates, or outliers.
- Flag suspicious values before deciding how to treat them.
- Do not make causal claims from simple correlations.
- Do not generate charts just for the sake of generating charts.
- Keep the first analysis exploratory.
- The purpose is to understand the dataset and identify promising directions, not to perform the entire analysis in one step.
