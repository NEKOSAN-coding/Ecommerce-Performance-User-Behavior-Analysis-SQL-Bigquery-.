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

### 📖 What is this project about? What business questions does it aim to answer?

This project leverages **SQL** and **Google BigQuery** to explore key performance metrics of an e-commerce website, using real GA4-exported data from the **Google Merchandise Store**. It aims to answer critical business questions across customer behavior, traffic source effectiveness, and purchase funnel conversion.

The analysis focuses on:

✔️ Monitoring monthly trends in visits, pageviews, and transactions  
✔️ Breaking down revenue by traffic source at both weekly and monthly levels  
✔️ Comparing engagement levels between purchasers and non-purchasers  
✔️ Mapping the customer journey from product view → add-to-cart → purchase

Through this, the project uncovers actionable insights to support data-driven decisions in e-commerce strategy, UX optimization, and marketing performance.

---

### 👤 Who is this project for?

✔️ **Data Analysts** – for SQL practice and behavior-driven segmentation  
✔️ **Marketing Teams** – to evaluate campaign traffic quality and ROI  
✔️ **E-commerce Managers & Stakeholders** – to optimize customer experience and conversion  
✔️ **Business Intelligence Teams** – to build metrics for strategic performance tracking
  

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
**💡 Query result:**

<img width="868" height="244" alt="image" src="https://github.com/user-attachments/assets/500ec482-3288-4a4e-8c30-4650cc0d85de" />

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
**💡 Query result:**

<img width="854" height="413" alt="image" src="https://github.com/user-attachments/assets/a15dbf65-e1e1-463e-bcd6-04797959b899" />

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
**💡 Query result:**

<img width="986" height="412" alt="image" src="https://github.com/user-attachments/assets/d379862f-be8b-46da-b78b-0aeeb594f68c" />

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
**💡 Query result:**

<img width="865" height="210" alt="image" src="https://github.com/user-attachments/assets/1d102b0d-d3d9-418f-b2f0-ee1c36e35e4b" />

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
**💡 Query result:**

<img width="679" height="179" alt="image" src="https://github.com/user-attachments/assets/e0262de6-85a0-4fa0-9114-61ae683e2d1c" />

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
**💡 Query result:**

<img width="670" height="177" alt="image" src="https://github.com/user-attachments/assets/68a1ca15-056f-4a86-bb7c-5d333d7719ea" />

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
**💡 Query result:**

<img width="694" height="408" alt="image" src="https://github.com/user-attachments/assets/2522fe2c-5c3a-4282-b553-be7787a81219" />

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
**💡 Query result:**

<img width="1110" height="245" alt="image" src="https://github.com/user-attachments/assets/91c61b1d-7f61-4a82-b974-78e2edb07131" />

---

## 📌 Key Takeaways, Insights & Recommendations

### 🟡 Key Takeaways:
- March 2017 recorded **69,931 visits** and **993 transactions**, the highest in Q1.
- Email traffic from `mail.google.com` generated over **$2.56M in revenue**, dominating all sources.
- Purchasers in July made an average of **4.16 transactions** and spent **$43.86 per session**.
- Non-purchasers viewed up to **334 pages on average**, 3× more than purchasers.

---

### 🔍 Insights:
- March outperformed January and February in visits, pageviews, and transactions—indicating peak campaign or seasonal impact.
- Users who made purchases in July completed **over 4 transactions** and spent an average of **$43.86 per session**.
- Visitors from social sources like YouTube had high bounce rates (~66%), showing low engagement compared to direct or referral traffic.
- Common co-purchases included branded lifestyle products like sunglasses and t-shirts, indicating cross-category shopping behavior.

---

### ✅ Recommendations:
- **Replicate March strategies** (campaigns, UX changes, or promotions) across other months or product categories.
- **Double down on email marketing**, especially transactional or automated flows, which generated the highest revenue.
- **Improve conversion paths** for high-pageview, non-converting users—optimize product pages, loading speed, and checkout flow.
- **Implement product bundling and cross-sell strategies** based on co-purchase behavior to increase average order value (AOV).

