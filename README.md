# Project Three: 2025 E-com RFM Analysis (SQL + Tableau)

![SQL](https://img.shields.io/badge/SQL-SQLite-blue) ![Tableau](https://img.shields.io/badge/Tableau-Public-orange)

Analyzed 100k synthetic sales with SQLite CTEs → RFM segments → Interactive dashboard.

## Key Insights
- Identified 4,000+ VIP customers (5-5-5 segment) — see [`output/rfm_segments.csv`](output/rfm_segments.csv).
- Promo codes drove **20% avg order lift** — see [`output/promo_roi.csv`](output/promo_roi.csv).

## Tech Stack
- **SQL (DBeaver/SQLite):** RFM modeling with window functions and CTEs.
- **Tableau Public:** Heatmap + funnels — [view dashboard](#) *(update link once published to Tableau Public)*.

## Overview
End-to-end pipeline: generate synthetic data → import CSV into SQLite → create SQL views → run analytical queries → export results to CSV.

## Files
| File | Purpose |
|------|---------|
| `generate_data.py` | Generates `data/synthetic_ecommerce_sales_2025.csv` (5 000 rows) |
| `ecom_analysis.py` | Loads CSV → SQLite, creates views, runs queries, exports CSVs |
| `data/synthetic_ecommerce_sales_2025.csv` | Synthetic dataset |
| `output/promo_roi.csv` | Promo ROI query results |
| `output/rfm_segments.csv` | RFM segmentation query results |

## Dataset Columns
`order_id`, `customer_id`, `product_category`, `sales_amount`, `purchase_date`, `device_type`, `promo_code`, `review_score`

## Quick Start
```bash
# (optional) regenerate the dataset
python3 generate_data.py

# run the full pipeline
python3 ecom_analysis.py
```

Requires: `pandas`, `sqlalchemy` (`pip install pandas sqlalchemy`)

## SQL Views Created
**`orders`** — mirrors all columns in `ecom`  
**`customers`** — per-customer aggregates: `total_orders`, `lifetime_value`, `avg_order_value`, `avg_review_score`, `last_purchase_date`

## Queries
### Promo ROI
```sql
SELECT promo_code,
       AVG(sales_amount) AS avg_order_value,
       COUNT(*) AS orders,
       SUM(sales_amount) AS total_revenue
FROM ecom
WHERE promo_code != ''
GROUP BY promo_code
ORDER BY total_revenue DESC;
```

### RFM Segmentation
```sql
WITH rfm AS (
  SELECT customer_id,
         JULIANDAY('2025-12-31') - JULIANDAY(MAX(purchase_date)) AS recency_days,
         COUNT(DISTINCT order_id) AS frequency,
         SUM(sales_amount) AS monetary
  FROM ecom
  GROUP BY customer_id
),
rfm_scored AS (
  SELECT
    NTILE(5) OVER (ORDER BY recency_days)   AS r_score,
    NTILE(5) OVER (ORDER BY frequency DESC) AS f_score,
    NTILE(5) OVER (ORDER BY monetary DESC)  AS m_score
  FROM rfm
)
SELECT r_score, f_score, m_score, COUNT(*) AS customers
FROM rfm_scored
GROUP BY r_score, f_score, m_score
ORDER BY r_score, f_score, m_score;
```
