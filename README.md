# Customer Shopping Behavior Analysis

An end-to-end retail analytics project: **Python** for cleaning, feature engineering and hypothesis testing, **PostgreSQL** for 18 business queries, and a **dashboard** for stakeholder reporting. It turns 3,900 purchase records into specific, testable recommendations on merchandising, discounting, subscriptions and loyalty.

![Python](https://img.shields.io/badge/Python-pandas%20%7C%20SciPy%20%7C%20seaborn-blue)
![SQL](https://img.shields.io/badge/PostgreSQL-18%20queries-336791)
![Dashboard](https://img.shields.io/badge/Dashboard-BI-orange)

---

## Table of Contents

1. [Business Problem](#business-problem)
2. [Headline Findings](#headline-findings)
3. [Dataset](#dataset)
4. [Tech Stack](#tech-stack)
5. [Repository Structure](#repository-structure)
6. [Methodology](#methodology)
7. [Key Insights](#key-insights)
8. [SQL Query Catalogue](#sql-query-catalogue)
9. [Statistical Test: Do Discounts Lift Order Value?](#statistical-test-do-discounts-lift-order-value)
10. [Dashboard](#dashboard)
11. [Recommendations](#recommendations)
12. [Caveats and Limitations](#caveats-and-limitations)
13. [How to Reproduce](#how-to-reproduce)
14. [Author](#author)

---

## Business Problem

A retail company wants to understand customer shopping behavior across demographics, product categories, seasons, shipping and payment preferences, discounts, ratings and repeat purchases, in order to **improve sales, customer satisfaction and retention**.

## Headline Findings

| Metric | Value |
| --- | --- |
| Total revenue | $233,081 |
| Purchases analysed | 3,900 |
| Average purchase amount | $59.76 |
| Average review rating | 3.75 / 5 |
| Orders with a discount | 1,677 (43.0%) |
| Subscribers | 1,053 (27.0%) |

**The core story: this is a volume business, not a ticket-size business.** Spend per order stays within roughly $57 to $62 across almost every cut of the data (gender, category, age group, subscription, discount). Growth has to come from more orders, a better product mix and stronger retention.

1. **Clothing and Accessories carry the business:** 44.7% and 31.8% of revenue (76.6% combined).
2. **Gender gap is a volume effect:** men generate 67.7% of revenue because they place 68.0% of orders; per-order spend is $59.54 (male) vs $60.25 (female).
3. **Subscription is not a loyalty lever yet:** subscribers spend the same as non-subscribers ($59.49 vs $59.87), and only 27.6% of repeat buyers subscribe, versus a 27.0% base rate.
4. **Discounts do not raise order value:** Welch t = -1.1122, p = 0.2661.
5. **Quality is uniformly middling:** best-rated item (Gloves) is 3.86 against a 3.75 average; three of the five best-selling items (Blouse, Pants, Shirt) rate below average.

## Dataset

- **File:** `customer_shopping_behavior.csv` (3,900 rows, 18 columns, one row per customer purchase; every Customer ID is unique)
- **Source:** `[PLACEHOLDER: add dataset source / license]`

| Group | Columns |
| --- | --- |
| Demographics | Customer ID, Age, Gender, Location, Subscription Status |
| Purchase details | Item Purchased, Category, Purchase Amount (USD), Size, Color, Season |
| Behavior | Review Rating, Shipping Type, Discount Applied, Promo Code Used, Previous Purchases, Payment Method, Frequency of Purchases |

**Data quality:** 37 missing values in `Review Rating` (0.9%); no duplicate records.

## Tech Stack

| Layer | Tools |
| --- | --- |
| Analysis | Python, pandas, NumPy, SciPy, seaborn, matplotlib |
| Database | PostgreSQL (loaded via SQLAlchemy + psycopg2) |
| Reporting | Dashboard (`[PLACEHOLDER: BI tool name]`) |

## Repository Structure

Files marked *(placeholder)* are suggested names, not files from the project.

```text
.
├── README.md
├── data/
│   └── customer_shopping_behavior.csv
├── notebooks/
│   └── Customer_Shopping_Behavior_Analysis.ipynb
├── sql/
│   └── customer_behavior_sql_queries.sql
├── dashboard/
│   └── Dashboard.png
├── reports/
│   └── Customer Shopping Behavior Analysis.docx      (placeholder)
└── images/                                           (placeholder)
    └── sql_screenshots/                              (placeholder: SQL_Q1_screenshot ... SQL_Q18_screenshot)
```

## Methodology

### 1. Python: preparation and testing

- **Load and inspect:** `info()`, `describe()`, category and color counts, missing-value and duplicate checks.
- **Impute ratings:** the 37 missing `Review Rating` values were filled with the **median rating of each product category**.
- **Standardise columns:** snake_case names; `purchase_amount_(usd)` renamed to `purchase_amount`.
- **Engineer features:**
  - `age_group`: quartile bins of age via `pd.qcut` (Young Adult, Adult, Middle-aged, Senior), which gives ages 18-31, 32-44, 45-57 and 58-70.
  - `frequency_of_purchases_days`: purchase frequency mapped to days (Weekly 7, Fortnightly and Bi-Weekly 14, Monthly 30, Quarterly and Every 3 Months 90, Annually 365).
- **Remove redundancy:** `discount_applied` and `promo_code_used` matched in every row, so `promo_code_used` was dropped.
- **Test:** Welch's two-sample t-test on purchase amount, discounted vs non-discounted orders.
- **Load to PostgreSQL:** the cleaned DataFrame was written to a `customer` table for SQL analysis.

### 2. SQL: business questions

Eighteen queries using aggregation, CTEs, window functions (`ROW_NUMBER`, `DENSE_RANK`, `SUM() OVER`), subqueries and `CASE` segmentation. See the [catalogue](#sql-query-catalogue).

### 3. Dashboard

KPI cards (customers, average purchase, average rating, total revenue), subscription split, revenue and sales by category and by age group, with filters for gender, subscription status, category and shipping type.

## Key Insights

### Revenue, category and demographics

| Category | Revenue | Orders | Avg. order | Share of revenue |
| --- | --- | --- | --- | --- |
| Clothing | $104,264 | 1,737 | $60.03 | 44.7% |
| Accessories | $74,200 | 1,240 | $59.84 | 31.8% |
| Footwear | $36,093 | 599 | $60.26 | 15.5% |
| Outerwear | $18,524 | 324 | $57.17 | 7.9% |

| Gender | Revenue | Orders | Avg. order | Share of orders |
| --- | --- | --- | --- | --- |
| Male | $157,890 | 2,652 | $59.54 | 68.0% |
| Female | $75,191 | 1,248 | $60.25 | 32.0% |

- Men and women buy the **same category mix** (Clothing is about 45% of each gender's orders).
- **Age groups contribute almost evenly:**

  | Age group | Revenue |
  | --- | --- |
  | Young Adult | $62,143 |
  | Middle-aged | $59,197 |
  | Adult | $55,978 |
  | Senior | $55,763 |

- Each category has a different lead age group (Q12): Clothing, Young Adult (479); Accessories, Adult (321); Footwear, Middle-aged (183); Outerwear, Senior (85).
- **Top items per category (Q8):** Jewelry (171), Blouse (171) and Pants (171), Sandals (160), Jacket (163). Outerwear has only two items (Jacket, Coat).
- **Seasonality is mild:** Spring 999, Fall 975, Winter 971, Summer 955 orders; Fall has the highest average order ($61.56). Clothing is the top category in every season. Pants is the one all-season staple (first in Summer, second in Winter).

### Loyalty and subscriptions

| Group | Customers | Share | Revenue | Avg. spend |
| --- | --- | --- | --- | --- |
| Subscribers | 1,053 | 27.0% | $62,645 | $59.49 |
| Non-subscribers | 2,847 | 73.0% | $170,436 | $59.87 |

- **Repeat buyers (more than 5 prior purchases):** 2,518 non-subscribers vs 958 subscribers, so most loyal customers are still unsubscribed.
- Purchase frequency is nearly identical for subscribers and non-subscribers (each of the seven bands holds 13.3% to 15.2% of its group, Q16).
- **Segmentation (Q7):** Loyal 3,116 (79.9%), Returning 701 (18.0%), New 83 (2.1%). Prior purchases are spread evenly from 1 to 50, so the "Loyal" threshold (more than 10) is too low to be selective.

### Discounts

- 839 of 1,677 discounted orders (50.0%) exceeded the overall average spend (Q2).
- Most discounted items (Q6): Hat 50.00%, Sneakers 49.66%, Coat 49.07%, Sweater 48.17%, Pants 47.37%.
- Discount users have a similar purchase history to non-users: 25.74 vs 25.06 average prior purchases (Q17).

### Product quality and fulfillment

| Top-rated item (Q3) | Avg. rating |
| --- | --- |
| Gloves | 3.86 |
| Sandals | 3.84 |
| Boots | 3.82 |
| Hat | 3.80 |
| Skirt | 3.78 |

- **High volume, below-average rating (Q15):** nine items are flagged; the weakest are Shirt (3.62), Blouse (3.68), Scarf and Shorts (3.71), Pants (3.72) and Coat (3.73). Dress, Sunglasses and Socks sit at 3.75.
- **Shipping (Q4):** Express $60.48 vs Standard $58.46. The $2.02 gap was not significance-tested.

## SQL Query Catalogue

Full script: [`customer_behavior_sql_queries.sql`](sql/customer_behavior_sql_queries.sql)

| # | Business question | Techniques |
| --- | --- | --- |
| Q1 | Total revenue, male vs female | `GROUP BY`, `SUM` |
| Q2 | Discount users who spent above average | Subquery |
| Q3 | Top 5 products by average rating | `AVG`, `ORDER BY`, `LIMIT` |
| Q4 | Average spend, Standard vs Express shipping | `WHERE IN`, `AVG` |
| Q5 | Subscribers vs non-subscribers: spend and revenue | `GROUP BY` |
| Q6 | Top 5 products by discount rate | Conditional aggregation |
| Q7 | New / Returning / Loyal segmentation | CTE, `CASE` |
| Q8 | Top 3 products per category | CTE, `ROW_NUMBER` |
| Q9 | Repeat buyers (more than 5 purchases) and subscription | `WHERE`, `GROUP BY` |
| Q10 | Revenue by age group | `GROUP BY` |
| Q11 | Category popularity by gender | `GROUP BY` |
| Q12 | Lead age group per category | CTE, `DENSE_RANK` |
| Q13 | Top 3 products per season | CTE, `DENSE_RANK` |
| Q14 | Category share of total revenue | Window `SUM() OVER ()` |
| Q15 | High volume but below-average rating | CTE, subqueries |
| Q16 | Purchase frequency by subscription status | Window `PARTITION BY` |
| Q17 | Prior purchases of discount users | `AVG`, `SUM` |
| Q18 | Most popular category per season | CTE, `DENSE_RANK` |

Result screenshots: `[PLACEHOLDER: images/sql_screenshots/SQL_Q1_screenshot ... SQL_Q18_screenshot]`

## Statistical Test: Do Discounts Lift Order Value?

H0: mean purchase amount is equal with and without a discount (Welch's two-sample t-test, unequal variances).

| Orders | n | Mean | Median | IQR | Std. dev. |
| --- | --- | --- | --- | --- | --- |
| Discount applied | 1,677 | $59.28 | $60.00 | $38 to $80 | $23.61 |
| No discount | 2,223 | $60.13 | $60.00 | $39 to $81 | $23.74 |

**Result:** t = -1.1122, p = 0.2661. We cannot reject H0: discounts show no measurable lift in order value, while giving up margin on 43.0% of orders.

## Dashboard

![Customer Behavior Dashboard](dashboard/Dashboard.png)

KPIs: 3,900 customers, $59.76 average purchase, 3.75 average rating, $233K total revenue. Filters: gender, subscription status, category, shipping type.

> **Known inconsistency:** the dashboard's age-group panels (for example Senior $88,480 and Young Adult $34,630) do not match the notebook and SQL results, which use quartile bins (Young Adult $62,143, Senior $55,763). The SQL and notebook figures are used throughout this README. Align the dashboard's age bands before external use.

## Recommendations

| Priority | Action | Evidence |
| --- | --- | --- |
| 1 | Fix or reposition Blouse, Shirt, Pants and Jeans (fit, quality, sizing content) before promoting them further | Three of the top five sellers rate below average; Jeans rates 3.65 |
| 2 | Replace blanket discounts with a holdout test on Hat, Sneakers, Coat, Sweater and Pants | 43.0% of orders discounted; no lift (p = 0.27) |
| 3 | Target the 2,518 unsubscribed repeat buyers with a frequency-based subscription offer | Subscribers do not spend more; repeat buyers subscribe at the base rate |
| 4 | Feature the best-rated items (Gloves, Sandals, Boots, Hat) in campaigns | Ratings 3.78 to 3.86 |
| 5 | Build an acquisition and engagement plan for women | 32.0% of orders, per-order spend equal to men |
| 6 | Recalibrate loyalty tiers (for example by quartiles of prior purchases) | 79.9% of customers are "Loyal" under the current rule |
| 7 | Align the dashboard's age bands with the SQL quartiles | Dashboard and SQL age-group figures disagree |

## Caveats and Limitations

- **Observational snapshot:** no causal or A/B lift is proven; the t-test shows similar spend, not what would happen if a discount were removed.
- **Purchasers only:** the data cannot show conversion or incremental volume from discounts or subscriptions.
- **One row per customer:** no time dimension, so trends over time cannot be assessed.
- **Small gaps are untested:** for example Express vs Standard ($2.02) and differences between age groups.
- **Feature-engineering choices matter:** age groups are sample quartiles, "Loyal" depends on a threshold of 10 prior purchases, and the frequency mapping treats Fortnightly and Bi-Weekly (14 days) and Quarterly and Every 3 Months (90 days) as identical.
- **Imputed ratings:** 37 missing ratings were filled with category medians.

## How to Reproduce

1. **Clone** the repository: `git clone [PLACEHOLDER: repo URL]`
2. **Install** dependencies: `pip install pandas numpy scipy seaborn matplotlib sqlalchemy psycopg2-binary`
3. **Run** `notebooks/Customer_Shopping_Behavior_Analysis.ipynb` (reads `customer_shopping_behavior.csv`).
4. **Create** a PostgreSQL database named `customer_behavior`. The notebook loads the cleaned data into a table called `customer`.
   - The notebook currently hardcodes database credentials. Replace them with environment variables or your own values before running, and never commit real passwords.
5. **Run** the queries in `sql/customer_behavior_sql_queries.sql` against the `customer` table.

## Author

`[PLACEHOLDER: Name | LinkedIn | Email | Portfolio]`
