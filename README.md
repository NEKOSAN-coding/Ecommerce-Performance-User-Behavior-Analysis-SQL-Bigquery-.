# 📊 Project Title: Google Analytics Ecommerce SQL Analysis  
**Author:** [Your Name]  
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

### 📌 Data Source
- Source: Google BigQuery Public Dataset: `bigquery-public-data.google_analytics_sample`  
- Size: Multiple tables with millions of rows covering Google Merchandise Store  
- Format: SQL tables with nested fields (arrays, records)

---

### 📊 Data Structure & Relationships  

#### 1️⃣ Tables Used
- `ga_sessions_2017*`: main sessions data from January to July 2017  

#### 2️⃣ Table Schema & Data Snapshot  
- The table includes nested fields like `hits`, `hits.product`, `trafficSource`, `totals`.

| Column Name           | Data Type | Description                                  |
|-----------------------|-----------|----------------------------------------------|
| date                  | STRING    | Date of the session                          |
| totals.visits         | INTEGER   | Number of visits per session                 |
| totals.pageviews      | INTEGER   | Pageviews during the session                 |
| totals.transactions   | INTEGER   | Transaction count                            |
| hits.productRevenue   | INTEGER   | Product revenue per product (in micros)      |
| hits.eCommerceAction  | RECORD    | Includes action_type: view, add_to_cart, buy |
| trafficSource.source  | STRING    | User source: direct, google, youtube, etc.   |

#### 3️⃣ Data Relationships
- One-to-many between sessions and hits  
- One-to-many between hits and products  
👉🏻 ER diagram not available as this is flat JSON-based schema, but logical nesting applies.

---

## 🧠 Design Thinking Process  

1️⃣ **Empathize**: Understand the data format (nested structure), usage of Google Analytics terms  
2️⃣ **Define POV**: SQL practice aligned with ecommerce KPIs  
3️⃣ **Ideate**: What user and product behavior patterns can be extracted  
4️⃣ **Prototype & Review**: Build SQL queries → validate output → optimize filtering

---

### ⚒️ Main Process

#### ✅ Data Cleaning & Preprocessing  
- Extract year, month, and week using `PARSE_DATE` and `FORMAT_DATE`  
- Use `_TABLE_SUFFIX` for filtering by month/date  
- Handle NULLs and filter revenue with `productRevenue IS NOT NULL`

#### ✅ Exploratory Data Analysis (EDA)
- Used aggregate functions: `SUM`, `COUNT`, `ROUND`, `GROUP BY`, `WITH`  
- Applied `UNNEST()` to access nested arrays: `hits`, `product`  

#### ✅ SQL Query Highlights  
- Query 1: Monthly visit/pageview/transaction summary  
- Query 2: Bounce rate calculation per source  
- Query 3: Revenue by month and week  
- Query 4: Avg. pageviews: purchasers vs. non-purchasers  
- Query 5: Avg. transactions per user  
- Query 6: Avg. spend per session  
- Query 7: Co-purchase behavior  
- Query 8: Product funnel conversion (view → cart → purchase)  

👉🏻 Each query includes expected output format and business logic  
👉🏻 Advanced CTE usage (`WITH`) and derived metrics like conversion rate

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
