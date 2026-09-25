# Data Analyst Assessment


# Superstore Profitability Analysis

**Data Analyst Assessment — VirtuBox Infotech**
**Prepared by:** Gungun Verma

## 1. Objective

The goal of this project is to identify where the Superstore business is generating
sales but not converting it into proportional profit, and to give management
concrete, measurable actions to fix it.

## 2. Dataset

**Source:** Sample Superstore dataset (Tableau Public)

The dataset is structured across three related tables:


| Table | Rows | Columns | Description |
|---|---|---|---|
| Orders | 10,194 | 21 | Transaction-level data — dates, customer segment, geography, product hierarchy, sales, discount, profit |
| People | 4 | 2 | Maps each Region to its Regional Manager |
| Returns | 296 | 2 | Flags which Order IDs were returned |

This dataset was chosen because it requires relational joining across multiple
tables and offers enough business dimensions (sales, profit, discount, shipping,
geography, returns) for meaningful multi-angle analysis.

## 3. Business Problem

Despite steady sales growth, certain products, regions, and customer segments may
be silently eroding profitability through heavy discounting and high return rates.
This analysis identifies where the business is losing money despite strong sales,
and which regions/managers need to act on it.

## 4. Methodology

### Step 1 — Data Cleaning (Python / Pandas / Google Colab)
- Checked for missing values (none found)
- Removed duplicate rows
- Converted Order Date and Ship Date to proper datetime format
- Created calculated fields: Delivery Days, Profit Margin %
- Merged the People table (Region → Manager) and Returns table (Order ID →
  Returned flag) into the main Orders data
- Flagged outlier orders (discount > 70% or profit loss > $500)

### Step 2 — Exploratory Analysis
- Aggregated Sales and Profit by Category, Region, and Sub-Category
- Checked the correlation between Discount and Profit
- Calculated return rates by category

### Step 3 — Insight Generation
Every insight is structured as evidence → business impact → recommendation, so
each finding directly supports a management decision.

### Step 4 — Dashboard
Built an interactive Google Looker Studio dashboard with KPI scorecards, category
and region profit/sales charts, a sub-category profit table, and a region filter —
designed for a non-technical management audience.

### Step 5 — Presentation
Summarized findings into a 7-slide deck: Business Problem → Methodology →
Findings → Deep-Dive → Recommendations → Impact → Limitations.

## 5. Code

```python
import pandas as pd

# Load all 3 linked tables
orders = pd.read_excel('sample_-_superstore.xls', sheet_name='Orders')
people = pd.read_excel('sample_-_superstore.xls', sheet_name='People')
returns = pd.read_excel('sample_-_superstore.xls', sheet_name='Returns')

# 1. Check missing values
print(orders.isnull().sum())

# 2. Remove duplicates
orders = orders.drop_duplicates()

# 3. Fix data types
orders['Order Date'] = pd.to_datetime(orders['Order Date'])
orders['Ship Date'] = pd.to_datetime(orders['Ship Date'])

# 4. Calculated field — delivery time
orders['Delivery Days'] = (orders['Ship Date'] - orders['Order Date']).dt.days

# 5. Calculated field — profit margin %
orders['Profit Margin %'] = (orders['Profit'] / orders['Sales'] * 100).round(2)

# 6. Merge with People (Region -> Manager)
orders = orders.merge(people, on='Region', how='left')

# 7. Merge with Returns (flag returned orders)
orders['Returned'] = orders['Order ID'].isin(returns['Order ID']).map({True: 'Yes', False: 'No'})

# 8. Flag outliers (extreme discount or heavy loss)
orders['Is Outlier'] = (orders['Discount'] > 0.7) | (orders['Profit'] < -500)

# Save processed, analysis-ready dataset
orders.to_csv('processed_superstore.csv', index=False)

# --- Exploratory Analysis ---
print(orders.groupby('Category')[['Sales', 'Profit']].sum())
print(orders.groupby('Region')[['Sales', 'Profit']].sum())
print(orders.groupby('Sub-Category')['Profit'].sum().sort_values())
print(orders[['Discount', 'Profit']].corr())
print(orders.groupby('Category')['Returned'].apply(lambda x: (x == 'Yes').mean()))
```

## 6. Key Finding

Furniture generates the second-highest sales ($754K) but the lowest profit
($19.7K, a 2.6% margin) — almost entirely driven by one sub-category, **Tables**,
which alone loses $17.8K. Office Supplies and Technology both convert sales to
profit at roughly 17% margins.

## 7. Limitations

- No cost/COGS data was available, so Profit Margin % was used as a proxy rather
  than a true cost breakdown
- The Returns table only records return status, not the reason or date
- The relationship between discount and profit is a correlation (-0.22), not a
  proven causal link

## 8. AI Usage

Claude AI was used to help structure the analytical approach, write and debug the
Pandas cleaning/joining code, and organize insights into the required reporting
format. All key numbers (profit margins, category/region aggregates) were
verified against the raw data, and AI-suggested insights were reworded to ensure
full understanding of each finding. This readme was enhanced through AI
