# Meridian Retail Co. — Customer Retention, Store Performance & Returns Analysis

An end-to-end data analytics project that simulates a real company engagement: three stakeholders with vague questions, raw and messy data, a cleaning phase, analysis, and recommendations.

> **Dataset note:** the data is simulated (generated with a Python script) to mimic a raw export from a retail transactional database, including realistic quality problems. No real company data was used.

## Business Context

Meridian Retail Co. is a mid-sized chain with 15 physical stores and an online store. Revenue has plateaued for two quarters, and three stakeholders each asked a question:

| Stakeholder | Original ask | Agreed definition (after clarifying questions) |
|---|---|---|
| CMO | "Which customers are at risk of churning?" | Loyalty members with no order in the last 60 days; list needs customer ID, email, and historical value |
| Head of Store Ops | "Which stores underperform, and is it traffic or basket size?" | Physical stores only; bottom 20% by revenue; basket size = dollar value per order |
| Finance Director | "Our return rate feels high" | % of orders that are returns, this year vs last year, by category |

## Data

- `orders_raw.csv` — 9,015 rows: order, customer, store, date, category, quantity, revenue, return flag
- `customers_raw.csv` — 1,240 rows: customer details, city, signup date, loyalty status, email

## Data Cleaning

| Issue found | Fix |
|---|---|
| Revenue stored as text with `$` / `USD` mixed in | Stripped symbols, converted to numbers |
| Order dates in 4 different formats | Parsed with exact formats (0 failures); avoided auto-guessing because day/month can be ambiguous |
| Category casing and trailing spaces | Standardized; 180 orders with missing category kept as "unknown" |
| 377 impossible quantities (0, negative, 999) | Marked invalid; excluded from revenue calculations because their revenue was computed from the bad quantity |
| `is_return` casing (`yes` / `Yes`) | Standardized |
| 40 duplicate customer IDs | Removed (1,240 → 1,200) |
| 15 duplicate order rows | Removed (9,015 → 9,000) |
| City casing, loyalty flag as `Y`/`N`/`Yes`/`No` | Standardized; 41 customers with unknown loyalty status left as unknown |
| ~8% of customers without email | Kept in the dataset; removed only from the campaign list, where an email is required |

Raw files were never overwritten; cleaned data was saved as separate files.

## Findings and Recommendations

### 1. Customer churn risk (CMO)
- 1,200 unique customers, 579 loyalty members. **293 loyalty members** have not ordered in 60+ days; **265** of them have a usable email.
- The 60-day rule flags **half of all customers (597 of 1,200)**, so it is not selective enough to target on its own.
- Loyalty members and non-members behave almost identically: 50.6% vs 49.1% inactive for 60+ days, and average spend of $860 vs $837. The loyalty program shows no measurable effect (this is a correlation, not proof that the program fails).
- **Recommendations:** prioritize the list by customer value; discuss a longer inactivity window (e.g. 90 days) with the CMO; run the campaign with a control group so its impact can be measured.

### 2. Store performance (Head of Store Ops)
- Bottom 20% by revenue: **ST02, ST04, ST14** (revenue cutoff ≈ $49.3K).
- Order counts are only ~2% lower (379 vs 387 orders). Average order value is lower ($124 vs $137), but the **median order value is identical ($87)**, so the gap comes from a few large orders.
- A shuffle test (randomly reassigning orders to stores) produced a best-to-worst store gap as large as the observed one about **25% of the time**, so the differences are within normal variation.
- **Recommendation:** do not act on store rankings from this data alone. Collect footfall, store size and staffing data and review a longer period.

### 3. Return rate (Finance Director)
- Overall return rate: **10.1%** across 9,000 orders.
- Like-for-like comparison (January–October of each year): **10.0% in 2024 vs 10.0% in 2025** — no increase.
- Category changes are small and mixed (Electronics 9.4% → 10.6%, Furniture 11.9% → 8.9%, Home Goods 10.2% → 10.6%, Kitchen 9.9% → 8.7%) and not clearly beyond normal variation.
- **Recommendation:** report returns as stable at about 10%; request an internal target or industry benchmark to judge whether 10% is high.

![Return rate by category](return_rate_by_category.png)

## Assumptions and Limitations
- "Today" is the latest order date in the data (2 October 2025), not the real current date.
- Returned orders are excluded from customer value and store revenue.
- Customers who never ordered are not flagged as at-risk (in this data every customer has at least one order).
- 2025 data stops in early October, so year-over-year comparisons use January–October of both years.
- The upper quantity cutoff of 20 is a judgment call; in a real project it would be confirmed with the business.
- Simulated data: patterns are not from a real company, so the business conclusions illustrate the method.

## Project Structure
```
meridian-retail-analysis/
├── data/
│   ├── orders_raw.csv
│   ├── customers_raw.csv
│   ├── orders_clean.csv
│   ├── customers_clean.csv
│   └── cmo_at_risk_campaign_list.csv
├── notebook/
│   └── meridian_analysis.ipynb       # replace with your notebook's name
├── return_rate_by_category.png
└── README.md
```

## Tools
Python · pandas · matplotlib · Jupyter Notebook
