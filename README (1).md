# Zepto Inventory & Pricing Analysis — SQL + Power BI Project

![Dashboard Screenshot](Zepto%20Inventory%20%26%20Pricing%20Analysis%20Dashboard.png)

🔗 [View Dashboard Screenshot](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/Zepto%20Inventory%20%26%20Pricing%20Analysis%20Dashboard.png) &nbsp;|&nbsp; 📥 [Download Power BI File (.pbit)](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/zepto.pbit)

An end-to-end data analysis project on a product catalog dataset from **Zepto** (a quick-commerce grocery delivery platform). The project starts with data cleaning and exploratory analysis in **PostgreSQL**, and ends with an interactive **Power BI dashboard** built on top of the cleaned data — covering pricing, discounting behavior, and inventory/stock health.

---

## 📋 Project Overview

This project simulates a real-world e-commerce inventory analysis workflow — from raw, messy data to a fully interactive business dashboard. It's split into two parts:

1. **SQL Analysis** — table creation, data exploration, data cleaning, and business-focused querying using PostgreSQL.
2. **Power BI Dashboard** — connecting the cleaned dataset, building DAX measures, and designing visuals to surface insights around category performance, discounting, and stock availability.

---

## 🎯 Problem Statement

Zepto lists thousands of products across multiple grocery categories, with varying pricing, discounting, and stock-availability patterns. This project aims to answer:

- Which categories and products offer the deepest discounts, and how common are they really?
- Which categories run out of stock most often, and is there a pattern behind it?
- How does pricing (MRP vs. discounted price) behave across the catalog?
- Which products offer the best value for money, and are there any data quality issues worth flagging?

---

## 🛠️ Tools Used

- **PostgreSQL** — database, cleaning, and exploratory SQL queries
- **SQL** — DDL, DML, aggregation, filtering, and grouping
- **Power BI Desktop** — data modeling, DAX measures, and dashboard visualization

---

## 🗂️ Dataset

The `zepto` table represents product-level data with the following schema:

| Column | Type | Description |
|---|---|---|
| `sku_id` | SERIAL (PK) | Unique identifier for each SKU |
| `category` | VARCHAR(120) | Product category |
| `name` | VARCHAR(150) | Product name |
| `mrp` | NUMERIC(8,2) | Maximum retail price |
| `discountPercent` | NUMERIC(5,2) | Discount percentage offered |
| `availableQuantity` | INTEGER | Quantity available in stock |
| `discountedSellingPrice` | NUMERIC(8,2) | Final selling price after discount |
| `weightInGms` | INTEGER | Product weight in grams |
| `outOfStock` | BOOLEAN | Stock availability flag |
| `quantity` | INTEGER | Quantity per unit/pack |

The raw dataset has **3,732 rows** spanning **14 categories**, with **1,667 distinct product names** (many products are listed multiple times under different pack sizes).

---

## Part 1: SQL Data Analysis

Full script: [`Zepto SQL Solved.sql`](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/Zepto%20SQL%20Solved.sql)

### 1. Data Exploration

- Counted total rows and previewed sample data
- Checked for NULL values across all columns
- Listed distinct product categories
- Reviewed in-stock vs. out-of-stock product counts
- Identified products with multiple SKU entries:

```sql
SELECT name, COUNT(sku_id) AS "Number of sku"
FROM zepto
GROUP BY name
HAVING COUNT(sku_id) > 1
ORDER BY COUNT(sku_id) DESC;
```

### 2. Data Cleaning

- Removed rows with an MRP of ₹0 (invalid entries):
```sql
DELETE FROM zepto
WHERE mrp = 0;
```
- Converted `mrp` and `discountedSellingPrice` from paise to rupees:
```sql
UPDATE zepto
SET mrp = mrp/100.0,
discountedSellingPrice = discountedSellingPrice/100.0;
```

### 3. Business Analysis

The project answers the following questions with SQL queries:

1. **Top 10 best-value products** based on discount percentage
2. **High-MRP products that are out of stock** (MRP > ₹300)
3. **Estimated revenue per category** (based on discounted price × available quantity — a proxy metric, since there's no real transaction data)
4. **Premium, low-discount products** — MRP > ₹500 and discount < 10%
5. **Top 5 categories by average discount percentage**
6. **Best value-for-money products** — price per gram for items over 100g
7. **Weight-based product segmentation** — Low / Medium / Bulk categories
8. **Total inventory weight per category**

### 📈 Key Insights from SQL

- Revenue and discount patterns vary significantly across categories, highlighting where discounting is most aggressive and where it isn't.
- Several high-MRP products are frequently out of stock, pointing to potential demand-supply gaps for premium items.
- Price-per-gram analysis surfaces the most cost-efficient products for consumers by weight.
- Discounted price never exceeds MRP anywhere in the cleaned dataset — a good sign of data integrity.

---

## Part 2: Power BI Dashboard

Once the data was cleaned in SQL, I connected it to Power BI, built a data model, wrote DAX measures, and designed a dashboard to visualize the same business questions interactively.

File: [`zepto.pbit`](zepto.pbit)

### 🧮 DAX Measures

```DAX
Total SKUs = COUNTROWS('public zepto')

Distinct Products = DISTINCTCOUNT('public zepto'[name])

Category Count = DISTINCTCOUNT('public zepto'[category])

Avg MRP = AVERAGE('public zepto'[mrp])

Avg Selling Price = AVERAGE('public zepto'[discountedSellingPrice])

Avg Discount % = AVERAGE('public zepto'[discountPercent])

OOS Rate % = DIVIDE(
    CALCULATE(COUNTROWS('public zepto'), 'public zepto'[outofstock] = TRUE),
    COUNTROWS('public zepto')
)

Total Discount Value =
SUMX('public zepto', ('public zepto'[mrp] - 'public zepto'[discountedSellingPrice]))

Total Available Qty = SUM('public zepto'[availablequantity])

Avg Weight = AVERAGE('public zepto'[weightingms])

In Stock Count = CALCULATE(COUNTROWS('public zepto'), 'public zepto'[outofstock] = FALSE)

Deep Discount Count = CALCULATE(COUNTROWS('public zepto'), 'public zepto'[discountpercent] > 50)
```

### 📊 KPI Cards

A summary row at the top of the dashboard for an instant health check:

- Total SKUs
- Distinct Products
- Category Count
- Avg MRP
- Avg Selling Price
- Avg Discount %
- Out-of-Stock Rate
- Total Discount Value
- Total Available Quantity
- Avg Weight
- In Stock Count

### 📈 Visualizations

| Chart | Type | What it shows |
|---|---|---|
| SKU Volume by Category | Horizontal bar chart | Which categories have the most products listed |
| Out-of-Stock Rate by Category | Horizontal bar chart (color-coded) | Which categories run out of stock most often |
| Category Summary Matrix | Matrix with conditional formatting | SKU count, avg discount, avg MRP, and OOS% per category in one table |
| Discount % Distribution | Binned column chart (histogram-style) | How discounts are spread across the whole catalog |
| Category Share of Total SKUs | Donut chart | Catalog composition by category |

### 🎛️ Filters / Slicers

- **Category** dropdown slicer — filter the entire dashboard by category
- **Stock Status** slicer — isolate in-stock or out-of-stock products

### 💡 Dashboard Insights

**Discounting is shallow, not aggressive.** The average discount across the whole catalog is only around **7.6%**. Barely 3 products in the entire catalog cross 50% off, and about 65% of all products get less than 10% discount. Zepto isn't running big flash-sale style discounting — it's more of a "slightly cheaper, consistently" approach.

**Stockouts are a real problem, but only in certain categories.** Overall, about **12% of listings** are out of stock at any given time — roughly 1 in 8 products. But it's not evenly spread:
- **Biscuits** is the worst offender, with almost **29% out of stock**.
- **Beverages** and **Dairy, Bread & Batter** aren't far behind, both around **21-22%**.
- **Personal Care** and **Paan Corner** are the best-stocked, at only around **6% OOS**.

**Small categories tend to struggle more.** Biscuits has one of the smallest product ranges (147 SKUs) but the highest stockout rate — that combination usually points to an under-inventoried category that needs either more variety or better restocking.

**Bigger categories dominate the catalog.** Cooking Essentials and Munchies each have 500+ SKUs, while categories like Meats, Fish & Eggs are much smaller (just 63 SKUs).

**Pricing data looks clean.** Discounted price never exceeds MRP anywhere in the dataset. Cheaper products follow a tighter, more consistent discount pattern, while pricier products show a bit more variation in how much discount they get.

---

## 📁 Files in this Repository

| File | Description |
|---|---|
| [`Zepto.csv`](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/Zepto.csv) | Raw dataset — the original product catalog data |
| [`Zepto SQL Solved.sql`](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/Zepto%20SQL%20Solved.sql) | Full SQL script — table creation, data cleaning, and business queries |
| [`zepto.pbit`](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/zepto.pbit) | Power BI template file — open in Power BI Desktop |
| [`Zepto Inventory & Pricing Analysis Dashboard.png`](https://github.com/SarkarMita/Zepto-Inventory-Pricing-Analysis-SQL-Power-BI/blob/main/Zepto%20Inventory%20%26%20Pricing%20Analysis%20Dashboard.png) | Preview image of the finished dashboard |

---

## 🚀 How to Use

1. Clone or download this repository.
2. To explore the SQL analysis: run `Zepto SQL Solved.sql` in PostgreSQL (or any compatible SQL client) against the raw `Zepto.csv` dataset.
3. To explore the dashboard: open `zepto.pbit` in Power BI Desktop and connect it to your own copy of the cleaned dataset when prompted.

---

## 📌 Limitations

This is **catalog/listing data, not sales/transaction data** — there's no units-sold or order history in the source dataset. Anywhere "revenue" is mentioned in this project, it's an **estimated proxy** (discounted price × available quantity), not real sales figures. With actual order-level data, the natural next step would be building a true sales performance view on top of this same catalog data.
