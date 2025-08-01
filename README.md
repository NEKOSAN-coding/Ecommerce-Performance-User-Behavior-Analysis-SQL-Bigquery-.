# 📊 Project Title: Google Analytics Ecommerce SQL Analysis  
**Author:** [Hà Minh Khuê]  
**Date:** 2025-07-31  
**Tools Used:** SQL (Google BigQuery)

---

## 📑 Table of Contents  
- 📌 Background & Overview  
- 📂 Dataset Description & Data Structure  
- 🧠 Design Thinking Process  
- 📊 Key Insights & Visualizations  
- 🔎 Final Conclusion & Recommendations  

---

## 📌 Background & Overview  

### 📖 What is this project about?
- This project analyzes e-commerce performance using Google Analytics sample data via SQL.
- The key objectives are:  
  ✔️ Understand monthly user behavior (visits, pageviews, transactions)  
  ✔️ Evaluate bounce rate by traffic source  
  ✔️ Identify weekly/monthly revenue patterns  
  ✔️ Differentiate between purchaser and non-purchaser behaviors  
  ✔️ Analyze product funnel performance (view → cart → purchase)  

### 👤 Who is this project for?
✔️ Junior Data Analysts & BI learners  
✔️ E-commerce business strategists  
✔️ Recruiters assessing SQL proficiency  

---

## 📂 Dataset Description & Data Structure

### 📌 **Data Source**  
📌 **Data Source:** The sample data is from **Google Analytics 4 (GA4)**, exported to **BigQuery**, including user activity data from the **Google Merchandise Store** e-commerce website.

### 📌 **Data Size**  
- **Dataset:** `ga4_obfuscated_sample_ecommerce`

### 📌 **How to Access the Data**
1. Log in to your **Google Cloud Platform** account and create a new project.  
2. Open the **BigQuery Console** and select your project.  
3. Click on **"Add Data"** in the navigation panel, then choose **"Search a project"**.  
4. In the search bar, enter the project ID: `bigquery-public-data.google_analytics_sample.ga_sessions` and press **Enter**.  
5. Click on the `ga_sessions_` table to explore its structure and data.

---


## ⚒️ Main Process

### 🧪 Query 01: Total Visits, Pageviews, and Transactions by Month (Jan–Mar 2017)

**📌 Description & Purpose:**  
Aggregates core engagement metrics to analyze monthly user behavior across Q1 2017.

**🧾 SQL Code:**
```sql
SELECT
  format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
  SUM(totals.visits) AS visits,
  SUM(totals.pageviews) AS pageviews,
  SUM(totals.transactions) AS transactions
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`
WHERE _TABLE_SUFFIX BETWEEN '0101' AND '0331'
GROUP BY month
ORDER BY month;
```
** Query result**
<img width="868" height="244" alt="image" src="https://github.com/user-attachments/assets/500ec482-3288-4a4e-8c30-4650cc0d85de" />

**🔍 Insight:**  
March 2017 showed the highest engagement across visits, pageviews, and transactions.

---

### 🧪 Query 02: Bounce Rate by Traffic Source (July 2017)

**📌 Description & Purpose:**  
Calculates bounce rate per traffic source to assess quality of traffic and user engagement.

**🧾 SQL Code:**
```sql
SELECT
  trafficSource.source,
  COUNT(totals.visits) as total_visits,
  COUNT(totals.bounces) as total_num_of_bounces,
  ROUND((COUNT(totals.bounces)/COUNT(totals.visits))*100,3) as bounce_rate
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*` 
GROUP BY trafficSource.source
ORDER BY total_visits DESC;
```

**🔍 Insight:**  
Google had the most traffic but also a high bounce rate (~51.6%), indicating potential misalignment of user intent.

---

### 🧪 Query 03: Revenue by Traffic Source by Week and Month (June 2017)

**📌 Description & Purpose:**  
Breaks down revenue by traffic source across both weekly and monthly time frames for deeper temporal insights.

**🧾 SQL Code:**
```sql
WITH 
month_data AS (
  SELECT
    "Month" as time_type,
    format_date("%Y%m", parse_date("%Y%m%d", date)) as month,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) hits,
    UNNEST(product) p
  WHERE p.productRevenue IS NOT NULL
  GROUP BY 1,2,3
),
week_data AS (
  SELECT
    "Week" as time_type,
    format_date("%Y%W", parse_date("%Y%m%d", date)) as week,
    trafficSource.source AS source,
    SUM(p.productRevenue)/1000000 AS revenue
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201706*`,
    UNNEST(hits) hits,
    UNNEST(product) p
  WHERE p.productRevenue IS NOT NULL
  GROUP BY 1,2,3
)
SELECT * FROM month_data
UNION ALL
SELECT * FROM week_data
ORDER BY time_type;
```

**🔍 Insight:**  
Direct traffic generated the most revenue in both weekly and monthly aggregates.

---

### 🧪 Query 04: Avg. Pageviews by Purchaser vs Non-Purchaser (June–July 2017)

**📌 Description & Purpose:**  
Compares engagement behavior by examining average pageviews between users who made a purchase and those who did not.

**🧾 SQL Code:**
```sql
WITH 
purchaser_data AS (
  SELECT
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    SUM(totals.pageviews)/COUNT(DISTINCT fullvisitorid) AS avg_pageviews_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(product) product
  WHERE _TABLE_SUFFIX BETWEEN '0601' AND '0731'
    AND totals.transactions >= 1
    AND product.productRevenue IS NOT NULL
  GROUP BY month
),
non_purchaser_data AS (
  SELECT
    format_date("%Y%m",parse_date("%Y%m%d",date)) as month,
    SUM(totals.pageviews)/COUNT(DISTINCT fullvisitorid) AS avg_pageviews_non_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_2017*`,
    UNNEST(hits) hits,
    UNNEST(product) product
  WHERE _TABLE_SUFFIX BETWEEN '0601' AND '0731'
    AND totals.transactions IS NULL
    AND product.productRevenue IS NULL
  GROUP BY month
)
SELECT
  pd.*, avg_pageviews_non_purchase
FROM purchaser_data pd
FULL JOIN non_purchaser_data USING(month)
ORDER BY pd.month;
```

**🔍 Insight:**  
Non-purchasers viewed more pages on average, possibly indicating friction in the conversion process.

---

### 🧪 Query 05: Avg. Transactions per Purchasing User (July 2017)

**📌 Description & Purpose:**  
Calculates how many transactions were made on average by each unique purchasing user.

**🧾 SQL Code:**
```sql
SELECT
  format_date("%Y%m",parse_date("%Y%m%d",date)) AS month,
  SUM(totals.transactions)/COUNT(DISTINCT fullvisitorid) AS Avg_total_transactions_per_user
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) hits,
  UNNEST(product) product
WHERE totals.transactions >= 1
  AND product.productRevenue IS NOT NULL
GROUP BY month;
```

**🔍 Insight:**  
On average, a purchasing user completed about 4.16 transactions in July 2017.

---

### 🧪 Query 06: Avg. Revenue per Session for Purchasers (July 2017)

**📌 Description & Purpose:**  
Evaluates average spend per visit for sessions that included a transaction.

**🧾 SQL Code:**
```sql
SELECT
  format_date("%Y%m",parse_date("%Y%m%d",date)) AS month,
  (SUM(product.productRevenue)/SUM(totals.visits))/1000000 AS avg_revenue_by_user_per_visit
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) hits,
  UNNEST(product) product
WHERE product.productRevenue IS NOT NULL
  AND totals.transactions >= 1
GROUP BY month;
```

**🔍 Insight:**  
Purchasing sessions in July 2017 generated an average revenue of ~$43.85 per visit.

---

### 🧪 Query 07: Other Products Purchased with "YouTube Men's Vintage Henley"

**📌 Description & Purpose:**  
Identifies co-purchased products by users who bought a specific product to uncover bundling opportunities.

**🧾 SQL Code:**
```sql
WITH buyer_list AS (
  SELECT DISTINCT fullVisitorId
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE product.v2ProductName = "YouTube Men's Vintage Henley"
    AND totals.transactions >= 1
    AND product.productRevenue IS NOT NULL
)
SELECT
  product.v2ProductName AS other_purchased_products,
  SUM(product.productQuantity) AS quantity
FROM `bigquery-public-data.google_analytics_sample.ga_sessions_201707*`,
  UNNEST(hits) AS hits,
  UNNEST(hits.product) AS product
JOIN buyer_list USING(fullVisitorId)
WHERE product.v2ProductName != "YouTube Men's Vintage Henley"
  AND product.productRevenue IS NOT NULL
GROUP BY other_purchased_products
ORDER BY quantity DESC;
```

**🔍 Insight:**  
Popular co-purchased items include Google Sunglasses and Hero Tees, revealing potential upsell combos.

---

### 🧪 Query 08: Product Funnel (View → Add to Cart → Purchase) – Jan–Mar 2017

**📌 Description & Purpose:**  
Builds a product-level conversion funnel to evaluate the effectiveness of each stage in the purchase process.

**🧾 SQL Code:**
```sql
WITH product_data AS (
  SELECT
    format_date('%Y%m', parse_date('%Y%m%d',date)) AS month,
    COUNT(CASE WHEN eCommerceAction.action_type = '2' THEN product.v2ProductName END) AS num_product_view,
    COUNT(CASE WHEN eCommerceAction.action_type = '3' THEN product.v2ProductName END) AS num_add_to_cart,
    COUNT(CASE WHEN eCommerceAction.action_type = '6' AND product.productRevenue IS NOT NULL THEN product.v2ProductName END) AS num_purchase
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_*`,
    UNNEST(hits) AS hits,
    UNNEST(hits.product) AS product
  WHERE _TABLE_SUFFIX BETWEEN '20170101' AND '20170331'
    AND eCommerceAction.action_type IN ('2','3','6')
  GROUP BY month
)
SELECT *,
  ROUND(num_add_to_cart / num_product_view * 100, 2) AS add_to_cart_rate,
  ROUND(num_purchase / num_product_view * 100, 2) AS purchase_rate
FROM product_data;
```

**🔍 Insight:**  
Conversion rates improved monthly, with March achieving a ~12.6% view-to-purchase rate.

---

## 📊 Key Insights & Visualizations  

> ⚠️ *Power BI dashboards not included in this SQL-only project. However, key findings are listed below.*

### 📌 Analysis 1: User Activity Trends
- March 2017 had the highest number of transactions (993).
- Google was the largest traffic source with high bounce rate (51.5%).

### 📌 Analysis 2: Revenue Contribution
- Direct traffic brought in the highest revenue in June 2017: ~$97,000.
- Revenue trends showed weekly spikes corresponding with product sales.

### 📌 Analysis 3: Purchaser vs. Non-Purchaser Behavior
- Non-purchasers had surprisingly higher average pageviews than purchasers.
- Average transactions per purchasing user in July = ~4.16.

### 📌 Analysis 4: Product Funnel Drop-off
- Product view to add-to-cart rates range: 28%–37%  
- Conversion from view to purchase: ~8–13%, improving over time

---

## 🔎 Final Conclusion & Recommendations  

👉🏻 Based on the insights and findings above, we recommend the ecommerce stakeholder team to consider:

### 📌 Key Takeaways:
✔️ Targeted campaigns should focus on low-bounce traffic sources like YouTube or Analytics Referrals.  
✔️ Investigate why non-purchasers consume more content—possible interest, but drop-off at checkout.  
✔️ Improve cart-to-purchase flow as many users drop off after adding items to cart.  
✔️ Consider weekly promotions to align with spikes in purchase behavior.  
✔️ Continue monitoring product performance at SKU level to boost co-purchase opportunities.
