# 🍽️ Restaurant Order Analysis: Culinary Data Science  

### An End-to-End SQL Project Exploring Customer Behavior and Menu Profitability

![SQL](https://img.shields.io/badge/Language-SQL-blue?style=for-the-badge&logo=mysql&logoColor=white)
![Data Analysis](https://img.shields.io/badge/Analysis-Exploratory%20Data%20Analysis-orange?style=for-the-badge)
![Business Intelligence](https://img.shields.io/badge/Domain-Hospitality%20%26%20F%26B-green?style=for-the-badge)
![Database](https://img.shields.io/badge/Database-MySQL%20%2F%20PostgreSQL-lightgrey?style=for-the-badge&logo=database&logoColor=white)


---

## 📌 Project Overview

The "**Taste of the World Cafe**" introduced a new international menu at the beginning of the year.  
Management wants to understand how customers are responding to the updated offerings and which menu items are driving profitability.

This project analyzes **12,266 transactional records** from Q1 (January–March 2023) to answer critical business questions:

- Which cuisines generate the most revenue?
- Which items are popular vs. profitable?
- Are there bulk ordering patterns?
- What is the highest-value order in the dataset?
- Where are there opportunities for strategic improvement?

Using advanced SQL techniques — including **CTEs, Window Functions, Subqueries, Aggregations, and Revenue Analysis** — this project translates raw operational data into actionable insights.

---

## 📊 Dataset Overview

- **Database:** MySQL  
- **Schema:** `restaurant_db`  
- **Timeframe:** January – March 2023  
- **Records:** 12,266  
- **Tables:** `menu_items`, `order_details`

### `menu_items`
| Column | Description |
|--------|------------|
| menu_item_id | Unique dish identifier |
| item_name | Name of dish |
| category | Cuisine category |
| price | Dish price |

### `order_details`
| Column | Description |
|--------|------------|
| order_id | Unique order identifier |
| order_date | Date of transaction |
| order_time | Time of transaction |
| item_id | Linked dish ID |

---

## 🎯 Objective 1: Understanding the Menu Structure

Before analyzing customer behavior, it’s important to understand the pricing architecture and composition of the menu itself.

---

### 🔎 Explore Menu Structure

We first inspect the table to understand available columns and data types.

```sql
SELECT * FROM menu_items;
```

---

### 📌 How Many Items Are on the Menu?

Understanding menu size helps contextualize demand concentration and pricing spread.

```sql
SELECT COUNT(*) AS total_menu_items
FROM menu_items;
```

---

### 💰 What Are the Least & Most Expensive Items?

Pricing extremes reveal premium positioning and potential margin drivers.  
Using `DENSE_RANK()` ensures ties are handled fairly.

```sql
WITH price_rank AS (
    SELECT 
        item_name,
        category,
        price,
        DENSE_RANK() OVER(ORDER BY price DESC) AS expensive_rank,
        DENSE_RANK() OVER(ORDER BY price ASC) AS cheapest_rank
    FROM menu_items
)
SELECT *
FROM price_rank
WHERE expensive_rank = 1 OR cheapest_rank = 1;
```

---

### 🍝 Italian Cuisine Pricing Analysis

Italian cuisine often carries strong demand in international restaurants.  
We evaluate its price range and positioning within the menu.

```sql
SELECT 
    COUNT(*) AS total_italian_dishes,
    MIN(price) AS cheapest_italian_item,
    MAX(price) AS most_expensive_italian_item,
    ROUND(AVG(price),2) AS avg_italian_price
FROM menu_items
WHERE category = 'Italian';
```

---

### 📊 Category Distribution & Pricing Strategy

This helps determine whether certain cuisines are positioned as premium, mid-tier, or budget offerings.

```sql
SELECT 
    category,
    COUNT(*) AS total_items,
    ROUND(AVG(price), 2) AS avg_dish_price
FROM menu_items
GROUP BY category
ORDER BY avg_dish_price DESC;
```

---

## 🎯 Objective 2: Understanding Order Patterns

After understanding pricing structure, we analyze transaction behavior and operational volume.

---

### 📅 What Is the Date Range?

This validates time coverage and ensures no data gaps.

```sql
SELECT 
    MIN(order_date) AS first_order,
    MAX(order_date) AS last_order
FROM order_details;
```

---

### 📦 How Many Orders & Items Were Sold?

This measures operational scale and throughput.

```sql
SELECT 
    COUNT(DISTINCT order_id) AS total_orders,
    COUNT(*) AS total_items_sold
FROM order_details;
```

---

### 🏆 Which Orders Had the Most Items?

Large item counts may indicate catering, group dining, or high-value customers.

```sql
SELECT 
    order_id,
    COUNT(*) AS num_items
FROM order_details
GROUP BY order_id
ORDER BY num_items DESC;
```

---

### 📈 How Many Orders Had More Than 12 Items?

Quantifying bulk orders helps assess catering potential.

```sql
SELECT COUNT(*) AS large_orders
FROM (
    SELECT order_id
    FROM order_details
    GROUP BY order_id
    HAVING COUNT(*) > 12
) AS bulk_orders;
```

---

## 🎯 Objective 3: Customer Behavior & Revenue Analysis

This is where pricing and order behavior intersect to generate business insight.

---

### 🔗 Combine Menu & Order Data

To connect revenue with item-level behavior:

```sql
SELECT 
    o.order_id,
    o.order_date,
    m.item_name,
    m.category,
    m.price
FROM order_details o LEFT JOIN menu_items m
     ON o.item_id = m.menu_item_id;
```

---

### 📊 Most & Least Ordered Items (Volume + Revenue)

Volume alone doesn’t tell the full story — revenue contribution is equally important.

```sql
SELECT 
    m.item_name,
    m.category,
    COUNT(*) AS times_ordered,
    SUM(m.price) AS total_revenue
FROM order_details o LEFT JOIN menu_items m
     ON o.item_id = m.menu_item_id
GROUP BY m.item_name, m.category
ORDER BY times_ordered DESC;
```

This reveals:
- Popular but low-priced items  
- Premium items with fewer but high-value purchases  
- Revenue concentration by dish  

---

### 💵 Top 5 Highest-Spending Orders

Identifying top spenders helps understand revenue concentration and customer value.

```sql
SELECT 
    o.order_id,
    SUM(m.price) AS total_spend
FROM order_details o LEFT JOIN menu_items m
     ON o.item_id = m.menu_item_id
GROUP BY o.order_id
ORDER BY total_spend DESC
LIMIT 5;
```

---

### 🥇 Inspect the Highest Spend Order

To understand purchasing composition and bundle patterns:

```sql
WITH highest_order AS (
    SELECT 
        o.order_id,
        SUM(m.price) AS total_spend
    FROM order_details o LEFT JOIN menu_items m
         ON o.item_id = m.menu_item_id
    GROUP BY o.order_id
    ORDER BY total_spend DESC
    LIMIT 1
)

SELECT 
    o.order_id,
    m.item_name,
    m.category,
    m.price
FROM order_details o LEFT JOIN menu_items m
     ON o.item_id = m.menu_item_id
WHERE o.order_id = (SELECT order_id FROM highest_order);
```

---

# ✅ Final Validation

## ❓ What Was the Most Expensive Order?

```sql
SELECT 
    MAX(order_total) AS highest_order_value
FROM (
    SELECT 
        o.order_id,
        SUM(m.price) AS order_total
    FROM order_details o LEFT JOIN menu_items m
         ON o.item_id = m.menu_item_id
    GROUP BY o.order_id
) AS order_totals;
```

---

## 📋 Analytical Bias Audit

| Stage | Risk | Mitigation |
|-------|------|------------|
| Data Scope | Seasonal bias (Q1 only) | Interpreted findings within limited timeframe |
| Ranking | Tie bias | Used `DENSE_RANK()` for fairness |
| Interpretation | Popularity bias | Compared volume and revenue |

---

## 💡 Strategic Recommendations

### 1️⃣ Premium Bundle Strategy for High-Revenue Categories
Italian cuisine shows strong pricing leverage and revenue potential.  
Introduce curated premium bundles (e.g., “Italian Dinner Experience”) that package appetizers, mains, and desserts to increase Average Order Value (AOV).

---

### 2️⃣ Menu Engineering Optimization
Segment items into:
- High Volume / High Revenue (Stars)
- High Volume / Low Revenue (Traffic Drivers)
- Low Volume / High Revenue (Premium Niche)
- Low Volume / Low Revenue (Candidates for removal)

Use this framework to:
- Reposition underperforming items  
- Adjust pricing where demand is inelastic  
- Improve menu layout prominence  

---

### 3️⃣ Corporate & Group Targeting Strategy
Orders with more than 12 items indicate group purchasing behavior.  
Develop:
- Corporate lunch packages  
- Event catering bundles  
- Pre-set group menus  

This captures predictable bulk revenue streams.

---

### 4️⃣ Revenue Concentration Monitoring
If a small percentage of orders contributes disproportionately to revenue, consider:
- Loyalty programs for high spenders  
- Personalized offers  
- Upsell prompts at checkout  

---

## 📈 Skills Demonstrated

- Advanced SQL (CTEs, Window Functions, Subqueries)
- Revenue Analysis & KPI Computation
- Customer Segmentation
- Menu Engineering Concepts
- Business Strategy Translation
- Analytical Bias Awareness
- Data Storytelling

---

## 📚 Credits

The dataset and initial project inspiration were adapted from guided SQL case studies provided by **Maven Analytics**.

This implementation, analysis approach, business framing, and strategic recommendations were independently developed and expanded upon as part of my SQL portfolio work.

🔗 https://mavenanalytics.io/

---

## 👩‍💻 Author

**Ayushi Gajendra**  
SQL • Data Analytics • Business Intelligence  

