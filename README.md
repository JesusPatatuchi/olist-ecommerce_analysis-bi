# 🛒 Olist E-Commerce Analytics: Customer Retention, RFM Segmentation & Market Basket Analysis

An end-to-end data analytics project exploring **100,000+ Brazilian e-commerce orders** from Olist (2016–2018). Built with **Google BigQuery (SQL)** for ETL, cohort matrix modeling, and statistical customer segmentation, paired with a multi-page interactive **Looker Studio Executive Dashboard**.

---

## 📌 Executive Summary & Key Business Findings

* **The Retention Dilemma (One-and-Done Model):** The 20-month cohort retention analysis revealed a steep drop-off after the initial purchase, with an average Month-1 ($M_1$) retention rate of **~0.4%–1.5%**. Olist acts primarily as an acquisition pipeline rather than a recurring customer ecosystem.
* **Geographical & Logistics Concentration:** Over **60% of total GMV and volume** is concentrated in the southeastern region of Brazil (**São Paulo, Rio de Janeiro, and Minas Gerais**), indicating where logistics, shipping speed, and fulfillment optimizations have the highest operational leverage.
* **RFM Customer Segmentation:** The RFM scoring revealed a heavy concentration of users in *At Risk* and *Lost / Hibernating* segments due to low purchase frequency, while top-tier *Champions* and *Loyal Customers* generate a disproportionate share of high-margin orders.
* **Market Basket Affinity:** Clear product co-occurrence patterns emerged within the **Home & Living cluster** (`bed_bath_table` + `furniture_decor` + `housewares`) and the **Family/Baby segment** (`baby` + `toys` + `cool_stuff`), highlighting clear opportunities for automated cart bundling and cross-selling campaigns.

---

## 📊 Interactive Looker Studio Dashboard Architecture (4 Pages)

The executive dashboard is organized into 4 dedicated business perspectives:

### 1. Executive Sales & Geospatial Performance
* **High-Level KPIs:** Total Orders (96,211), Gross Merchandise Value (R$ 13.18M), Average Order Value (R$ 137.00).
* **Time-Series Analysis:** Monthly dual-axis trend comparing order volume vs. gross revenue.
* **Geospatial Intelligence:** Regional state density map covering all 27 Brazilian federal units (ISO 3166-2).

![Executive Overview](assets/dashboard_p1_overview.png)

---

### 2. Cohort Retention Heatmap (Full Matrix)
* **20-Month Triangular Retention Matrix:** Complete historical cohort retention pivot table tracking customer longevity from $M_0$ through $M_{20}$.
* **Granular Precision:** 2-decimal percentage heat map highlighting peak retention and repurchase behavior over time.

![Cohort Retention Matrix](assets/dashboard_p2_cohort_matrix.png)

---

### 3. Retention Curves & Behavioral Trends
* **Decay Curve Modeling:** Average customer retention decay curve across relative month numbers.
* **Retention Metrics Breakdown:** Active user counts and cohort size dynamics across monthly lifecycles.

![Retention Curves](assets/dashboard_p3_retention_curves.png)

---

### 4. RFM Segmentation & Market Basket Affinity
* **RFM Distribution:** Customer segment treemap (*Champions*, *Loyal*, *At Risk*, *Lost / Hibernating*).
* **Pareto Value Analysis:** Customer share vs. Revenue contribution grouped column chart.
* **Behavioral Matrix:** Recency vs. Monetary value scatter plot with dynamic sizing.
* **Cross-Selling Opportunities:** Top 10 product category affinities from self-join market basket analysis.

![RFM & Market Basket](assets/dashboard_p4_rfm_marketbasket.png)

---

## 🛠️ Tech Stack & Architecture

* **Cloud Data Warehouse:** Google BigQuery
* **Data Transformation & Modeling:** Standard SQL (Common Table Expressions, Window Functions `NTILE()`, Self-Joins, Aggregate Metrics)
* **Business Intelligence & Reporting:** Looker Studio
* **Dataset:** Olist Brazilian E-Commerce Public Dataset (~100k orders)

```
Raw CSV / Olist Tables (BigQuery)
        │
        ▼
[ BigQuery Modeling & Staging ]
  ├── stg_orders_consolidated    ──> Sales KPIs & Geo Analytics
  ├── fct_customer_rfm           ──> RFM Scoring & Customer Segments
  ├── fct_retention_cohorts      ──> 20-Month Triangular Cohort Heatmap
  ├── fct_retention_rfm          ──> Average Retention Decay Curves
  └── fct_market_basket          ──> Self-Join Product Affinity Analysis
        │
        ▼
[ Looker Studio 4-Page Executive Dashboard ]
```

---

## 🧠 Key SQL Methodologies

### 1. Market Basket Self-Join Affinity
Prevented duplicate pairs and mirror combinations using strict alphabetical comparison and English category translations:
```sql
SELECT
  a.product_category_name AS category_a,
  b.product_category_name AS category_b,
  COUNT(DISTINCT a.order_id) AS times_bought_together
FROM order_categories a
JOIN order_categories b 
  ON a.order_id = b.order_id 
 AND a.product_category_name < b.product_category_name
GROUP BY 1, 2
ORDER BY times_bought_together DESC;
```

### 2. 20-Month Cohort Retention Matrix
Calculated user activity relative to their first purchase date:
```sql
SELECT
  a.cohort_month,
  c.total_cohort_users AS cohort_size,
  a.month_number,
  COUNT(DISTINCT a.customer_unique_id) AS active_users,
  ROUND(COUNT(DISTINCT a.customer_unique_id) / c.total_cohort_users, 4) AS retention_ratio
FROM user_activities a
JOIN cohort_size c ON a.cohort_month = c.cohort_month
GROUP BY 1, 2, 3
ORDER BY a.cohort_month ASC, a.month_number ASC;
```

### 3. RFM Percentile Ranking
Segmented customer behavior dynamically into percentiles:
* **Recency (R):** Days since the latest order date.
* **Frequency (F):** Distinct orders per unique customer.
* **Monetary (M):** Total historical spend per unique customer.

---

## 💡 Strategic Recommendations for Growth

1. **Automated Post-Purchase Re-engagement:** Trigger category-based retention emails at **15, 30, and 45 days** post-delivery with targeted discounts on high-affinity complementary categories.
2. **Dynamic Cart Bundling:** Bundle top co-purchased items (e.g., *Bed Bath Table* + *Furniture Decor*) with checkout bundle incentives.
3. **Win-Back Campaigns for At-Risk Customers:** Re-engage high-historical-value users inactive for >180 days with targeted promotional discounts.
