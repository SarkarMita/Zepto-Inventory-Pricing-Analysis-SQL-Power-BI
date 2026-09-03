# Zepto Inventory & Pricing Analysis

A data analysis project where I explored Zepto's product catalog data using **SQL** for cleaning and querying, and then built an interactive **Power BI dashboard** on top of it to visualize the findings.

![Dashboard Screenshot](dashboard_screenshot.png)

---

## Why I did this project

I wanted to work with real-world style e-commerce data and answer some practical business questions — things like: which categories run out of stock the most, how deep are the discounts actually, and which products offer the best value for money. Zepto's product listing data (available on Kaggle) was a good fit for this because it has pricing, discount, stock, and weight information for thousands of SKUs.

This was also a chance for me to practice the full analyst workflow — starting from raw data, cleaning it in SQL, running exploratory queries, and then turning those findings into a dashboard that anyone (non-technical included) could look at and understand in a few seconds.

---

## Dataset

The dataset has about **3,732 rows**, each representing a product listing on Zepto, spread across **14 categories** (Fruits & Vegetables, Biscuits, Dairy, Personal Care, and so on). Each row includes:

| Column | What it means |
|---|---|
| `sku_id` | Unique ID for each listing |
| `category` | Product category |
| `name` | Product name |
| `mrp` | Maximum retail price |
| `discountPercent` | Discount % applied |
| `discountedSellingPrice` | Final selling price after discount |
| `availableQuantity` | Stock quantity currently available |
| `weightInGms` | Product weight |
| `outOfStock` | Whether it's currently out of stock |
| `quantity` | Order/pack quantity |

One thing worth mentioning upfront — this is **catalog data, not sales/transaction data**. There's no "units sold" or order history here, so everything in this project is about pricing, discounting, and stock availability, not actual revenue.

---

## Step 1: Cleaning and exploring the data in SQL

Before touching Power BI, I ran the raw CSV through PostgreSQL to clean it up and get a feel for the data. The full script is in [`Zepto_SQL_Solved.sql`](Zepto_SQL_Solved.sql). Here's what I did, in order:

**1. Set up the table and did a basic sanity check**
Checked row count, looked at a sample, and checked for null values across every column.

**2. Checked for duplicate product listings**
```sql
SELECT name, COUNT(sku_id) AS "Number of sku"
FROM zepto
GROUP BY name
HAVING COUNT(sku_id) > 1
ORDER BY COUNT(sku_id) DESC;
```
Turned out a lot of product names repeat — usually because the same product is listed multiple times for different pack sizes.

**3. Removed junk rows**
A few products had `mrp = 0`, which doesn't make sense for a real listing, so I dropped those.

**4. Fixed the pricing units**
The `mrp` and `discountedSellingPrice` columns were stored in **paise**, not rupees (e.g., 2500 instead of ₹25.00). I converted both to rupees:
```sql
UPDATE zepto
SET mrp = mrp/100.0,
discountedSellingPrice = discountedSellingPrice/100.0;
```

**5. Ran some actual business questions**, including:
- Top 10 products with the highest discount %
- Products with high MRP that are still out of stock
- Estimated revenue by category (as a proxy metric, since there's no real sales data)
- Products priced above ₹500 with less than 10% discount
- Top 5 categories by average discount
- Price-per-gram for products over 100g, to find the best value-for-money items
- Grouping products into Low / Medium / Bulk weight categories
- Total inventory weight per category

These queries gave me a good first read on the data before I moved to building anything visual.

---

## Step 2: Building the Power BI dashboard

Once the data was clean, I connected it to Power BI and built out a dashboard focused on three things: **catalog composition**, **discounting behavior**, and **inventory/stock health**.

### KPI Cards
A quick-glance summary row at the top: Total SKUs, Distinct Products, Category Count, Avg MRP, Avg Selling Price, Avg Discount %, Out-of-Stock Rate, Total Discount Value, plus a second row for Total Available Quantity, Avg Weight, and In-Stock Count.

### Charts
- **SKU Volume by Category** — which categories have the most products listed
- **Out-of-Stock Rate by Category** — which categories run out of stock most often, color-coded (green = healthy stock, red = high risk)
- **Category Summary Matrix** — a single table combining SKU count, avg discount, avg MRP, and OOS% per category, with conditional formatting
- **Discount % Distribution** — a histogram showing how discounts are spread across the whole catalog
- **Category Share of Total SKUs** — a donut chart showing catalog composition

### Filters
Added a category dropdown and a stock-status filter so anyone viewing the dashboard can slice the data themselves instead of just looking at static numbers.

---

## What I found

Here's the summary of what the data actually shows, in plain terms:

**Discounting is shallow, not aggressive.** The average discount across the whole catalog is only around **7.6%**. Barely 3 products out of the entire catalog cross 50% off. About 65% of all products get less than 10% discount. So Zepto isn't running big flash-sale style discounting — it's more of a "slightly cheaper, consistently" approach.

**Stockouts are a real problem, but only in certain categories.** Overall, about **12% of listings** are out of stock at any given time — roughly 1 in 8 products. But it's not evenly spread:
- **Biscuits** is the worst offender, with almost **29% out of stock**.
- **Beverages** and **Dairy, Bread & Batter** aren't far behind, both around **21-22%**.
- **Personal Care** and **Paan Corner** are the best-stocked, at only around **6% OOS**.

**Small categories tend to struggle more.** Biscuits has one of the smallest product ranges (147 SKUs) but the highest stockout rate — that combination usually points to under-inventoried categories that need either more variety or better restocking.

**Bigger categories dominate the catalog.** Cooking Essentials and Munchies each have 500+ SKUs, while categories like Meats, Fish & Eggs are much smaller (just 63 SKUs).

**Pricing data looks clean.** Discounted price never exceeds MRP anywhere in the dataset — a good sign there's no data integrity issue there. Cheaper products tend to follow a tighter, more consistent discount pattern, while pricier products show a bit more variation in how much discount they get.

---

## Tools used
- **PostgreSQL** — data cleaning and exploratory SQL queries
- **Power BI Desktop** — dashboard building and DAX measures
- **Dataset** — Zepto product catalog data (via Kaggle)

---

## Files in this repo
- `Zepto_SQL_Solved.sql` — all the SQL cleaning and analysis queries
- `zepto.pbit` — the Power BI template file (open in Power BI Desktop, connect your own data source)
- `dashboard_screenshot.png` — a preview of the finished dashboard

---

## A note on limitations
This is catalog/listing data, not sales data — so anything that sounds like "revenue" in this project is an estimate (selling price × available quantity), not real transaction revenue. If actual order-level data was available, the next step would be building a proper sales performance view on top of this.

---

Feel free to open the `.pbit` file in Power BI Desktop to explore the dashboard yourself, or check out the SQL file to see the full data cleaning and querying process.
