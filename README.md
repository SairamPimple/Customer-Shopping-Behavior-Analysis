# Customer Shopping Behavior Analysis

Python + PostgreSQL + dashboard analysis of 3,900 retail purchases to find what drives revenue, retention and the return on discounts and subscriptions.

![Python](https://img.shields.io/badge/Python-pandas%20%7C%20SciPy%20%7C%20seaborn-blue)
![SQL](https://img.shields.io/badge/PostgreSQL-18%20queries-336791)

## Business Problem

A retailer wants to understand behavior across demographics, categories, seasons, shipping, payment, discounts, ratings and repeat purchases to improve sales, satisfaction and retention.

## Headline Results

| Revenue | Purchases | Avg. purchase | Avg. rating | Discounted orders | Subscribers |
| --- | --- | --- | --- | --- | --- |
| $233,081 | 3,900 | $59.76 | 3.75 / 5 | 1,677 (43.0%) | 1,053 (27.0%) |

**It is a volume business, not a ticket-size business:** spend per order stays at about $57 to $62 across gender, category, age, subscription and discount. Growth must come from more orders, better mix and retention.

- **Two categories carry revenue:** Clothing 44.7% and Accessories 31.8%.
- **Gender gap is volume:** men are 67.7% of revenue on 68.0% of orders; per-order spend is $59.54 (male) vs $60.25 (female).
- **Subscription is not a loyalty lever yet:** subscribers spend $59.49 vs $59.87 for non-subscribers; only 27.6% of repeat buyers subscribe (base rate 27.0%).
- **Discounts do not lift order value** (Welch t = -1.1122, p = 0.2661).
- **Quality is uniformly middling:** best item Gloves 3.86; Blouse, Pants and Shirt are top sellers rated below average.

## Tech Stack

Python (pandas, NumPy, SciPy, seaborn, matplotlib), PostgreSQL (SQLAlchemy, psycopg2), dashboard (`[PLACEHOLDER: BI tool]`).

## Approach

1. **Clean (Python):** imputed 37 missing ratings (0.9%) with category medians; snake_case columns; dropped `promo_code_used` (identical to `discount_applied` in every row).
2. **Engineer:** `age_group` (age quartiles: 18-31, 32-44, 45-57, 58-70) and `frequency_of_purchases_days`.
3. **Test:** Welch's t-test on discounted vs full-price spend.
4. **Query (PostgreSQL):** 18 business questions using CTEs, window functions, subqueries and `CASE`.
5. **Report:** dashboard with KPIs and filters for gender, subscription, category and shipping type.

## Key Findings

### Revenue and demographics

| Category | Revenue | Orders | Avg. order | Share |
| --- | --- | --- | --- | --- |
| Clothing | $104,264 | 1,737 | $60.03 | 44.7% |
| Accessories | $74,200 | 1,240 | $59.84 | 31.8% |
| Footwear | $36,093 | 599 | $60.26 | 15.5% |
| Outerwear | $18,524 | 324 | $57.17 | 7.9% |

| Gender | Revenue | Orders | Avg. order |
| --- | --- | --- | --- |
| Male | $157,890 | 2,652 | $59.54 |
| Female | $75,191 | 1,248 | $60.25 |

- Men and women buy the same category mix (Clothing about 45% of each).
- Age groups contribute almost evenly: Young Adult $62,143, Middle-aged $59,197, Adult $55,978, Senior $55,763.
- Seasonality is mild (Spring 999, Fall 975, Winter 971, Summer 955 orders). Clothing leads every season; Pants is the all-season staple.

### Loyalty and subscriptions

| Group | Customers | Revenue | Avg. spend |
| --- | --- | --- | --- |
| Subscribers | 1,053 (27.0%) | $62,645 | $59.49 |
| Non-subscribers | 2,847 (73.0%) | $170,436 | $59.87 |

- Repeat buyers (more than 5 prior purchases): **2,518 non-subscribers vs 958 subscribers.**
- Purchase frequency is nearly identical across both groups (13.3% to 15.2% per band).
- Segments: Loyal 3,116 (79.9%), Returning 701 (18.0%), New 83 (2.1%). Prior purchases are spread evenly from 1 to 50, so the "Loyal" cutoff (more than 10) is not selective.

### Discount effectiveness

| Orders | n | Mean | Median | IQR |
| --- | --- | --- | --- | --- |
| Discount applied | 1,677 | $59.28 | $60.00 | $38 to $80 |
| No discount | 2,223 | $60.13 | $60.00 | $39 to $81 |

**t = -1.1122, p = 0.2661:** no measurable lift in order value, while margin is given up on 43.0% of orders.

- 839 of 1,677 discounted orders (50.0%) exceeded the overall average spend.
- Most discounted items: Hat 50.00%, Sneakers 49.66%, Coat 49.07%, Sweater 48.17%, Pants 47.37%.
- Discount users have similar history to non-users (25.74 vs 25.06 prior purchases).

### Quality and fulfillment

- **Top rated:** Gloves 3.86, Sandals 3.84, Boots 3.82, Hat 3.80, Skirt 3.78.
- **High volume, below-average rating (Q15):** Shirt 3.62, Blouse 3.68, Scarf and Shorts 3.71, Pants 3.72, Coat 3.73 (Dress, Sunglasses, Socks sit at 3.75).
- **Shipping:** Express $60.48 vs Standard $58.46 (gap of $2.02, not significance-tested).

## Recommendations

| # | Action | Evidence |
| --- | --- | --- |
| 1 | Fix or reposition Blouse, Shirt, Pants, Jeans before promoting them | Three top-five sellers rate below average; Jeans 3.65 |
| 2 | Replace blanket discounts with a holdout test (Hat, Sneakers, Coat, Sweater, Pants) | 43.0% discounted, no lift (p = 0.27) |
| 3 | Offer a frequency-based subscription to the 2,518 unsubscribed repeat buyers | Subscribers do not spend more |
| 4 | Feature Gloves, Sandals, Boots, Hat in campaigns | Ratings 3.78 to 3.86 |
| 5 | Build an acquisition plan for women | 32.0% of orders, equal per-order spend |
| 6 | Recalibrate loyalty tiers (for example by quartiles) | 79.9% classed "Loyal" |
| 7 | Align dashboard age bands with SQL quartiles | Figures disagree (see below) |

## SQL Coverage (18 queries)

| Theme | Queries |
| --- | --- |
| Revenue and demographics | Q1 gender revenue, Q10 age-group revenue, Q11 category by gender, Q12 lead age group per category, Q14 category revenue share |
| Products and seasons | Q3 top-rated, Q8 top 3 per category, Q13 top 3 per season, Q15 high-volume low-rating, Q18 top category per season |
| Subscriptions and loyalty | Q5 subscribers vs not, Q7 segmentation, Q9 repeat buyers, Q16 frequency by subscription |
| Discounts and fulfillment | Q2 high-spending discount users, Q4 Standard vs Express, Q6 top discounted items, Q17 prior purchases by discount |

Result screenshots: `[PLACEHOLDER: images/sql_screenshots/SQL_Q1_screenshot ... SQL_Q18_screenshot]`

## Dashboard

![Customer Behavior Dashboard](dashboard/Dashboard.png)

> **Known inconsistency:** the dashboard's age-group panels (for example Senior $88,480) do not match the notebook and SQL (quartile bins, Senior $55,763). This README uses the SQL figures; align the dashboard bands before external use.

## Caveats

- Observational snapshot: no causal lift proven; a holdout or A/B test is needed.
- Purchasers only: conversion and incremental volume are not visible.
- One row per customer: no time dimension.
- Small gaps (Express vs Standard, age groups) are untested.
- Age groups are sample quartiles; "Loyal" depends on a 10-purchase threshold; Fortnightly and Bi-Weekly (14 days) and Quarterly and Every 3 Months (90 days) are mapped identically.

## Repository Structure

Names marked *(placeholder)* are suggestions, not project files.

```text
├── data/customer_shopping_behavior.csv
├── notebooks/Customer_Shopping_Behavior_Analysis.ipynb
├── sql/customer_behavior_sql_queries.sql
├── dashboard/Dashboard.png
├── reports/Customer Shopping Behavior Analysis.docx   (placeholder)
└── images/sql_screenshots/                            (placeholder)
```

## Run It

1. `pip install pandas numpy scipy seaborn matplotlib sqlalchemy psycopg2-binary`
2. Run the notebook; it loads the cleaned data into a `customer` table in a PostgreSQL database named `customer_behavior`.
3. Run `sql/customer_behavior_sql_queries.sql` against that table.

> The notebook hardcodes database credentials. Use environment variables and never commit real passwords.

**Dataset source:** `[PLACEHOLDER]` | **Author:** `[PLACEHOLDER: Name | LinkedIn | Email]`
