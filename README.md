# Cart-Abandonment-Pattern-Analysis





<h1 align="center">🛒 Cart Abandonment Pattern Analysis</h1>
<p align="center"><b>Why do customers add products to their cart and then leave without buying?</b><br>
Python + SQL (DuckDB) + Power BI analysis of 55M e-commerce events and 1.66M cart sessions</p>

<p align="center">
  <img src="https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white" alt="Python">
  <img src="https://img.shields.io/badge/Pandas-150458?style=for-the-badge&logo=pandas&logoColor=white" alt="Pandas">
  <img src="https://img.shields.io/badge/SQL-DuckDB-FFF000?style=for-the-badge&logo=duckdb&logoColor=black" alt="DuckDB SQL">
  <img src="https://img.shields.io/badge/Power_BI-F2C811?style=for-the-badge&logo=powerbi&logoColor=black" alt="Power BI">
</p>

<p align="center">
  📄 <a href="report/Cart_Abandonment_Pattern_Analysis_Report.pdf"><b>Read the 6-page report</b></a> &nbsp;|&nbsp;
  📓 <a href="Cart%20Abandonment%20Pattern%20Analysis%20EDA.ipynb"><b>EDA notebook</b></a> &nbsp;|&nbsp;
  🧾 <a href="SQL/cart_abandonment_sql_analysis.sql"><b>SQL analysis</b></a> &nbsp;|&nbsp;
  📊 <a href="Dashboard"><b>Dashboard</b></a>
</p>

---

## 🎯 Business Problem

Many e-commerce shoppers add products to their cart but never complete the purchase. This project answers: **why are customers abandoning their carts, and which customer, product and timing factors are most strongly associated with it?** The goal is to show a product team where to investigate first.

## ⚡ Headline Results

| 55.3M | 57.8% | 89.4% | -19.5 pts |
|:---:|:---:|:---:|:---:|
| events processed in chunks to isolate 1.66M cart sessions | cart abandonment (960K of 1.66M cart sessions) | of cart sessions hold a single product; 60.2% of them are abandoned | abandonment for repeat users (6+ cart sessions) vs one-time users |

> **💡 Key insight:** Abandonment depends less on *what* is in the cart than on *who* is buying and *how engaged* they are. One-time visitors abandon **65.4%** of carts versus **45.9%** for repeat users, single-product carts abandon **60.2%** versus about **30%** for carts with 4+ products, and sessions that stay within a single minute abandon **63.9%**. **Each 1-point reduction in abandonment is worth about 16,600 extra purchase sessions a month.**

## 📊 Dashboard Preview

<p align="center"><img src="Executive Overview.png" width="780" alt="Executive overview dashboard page"></p>
<p align="center">
  <img src="Abandonment Drivers.png" width="385" alt="Abandonment drivers dashboard page">
  <img src="Customer & Session Behavior.png" width="385" alt="Customer and session behavior dashboard page">
</p>
<p align="center"><img src="Funnel Analysis.png" width="780" alt="Funnel analysis dashboard page"></p>

The 4-page Power BI report covers: **Executive Overview** (KPI cards, Category / Date / Brand slicers, daily trends), **Funnel Analysis**, **Abandonment Drivers** (category, brand, price, product count) and **Customer & Session Behavior**.

## 🗄️ Data and Method

**Dataset:** Kaggle *E-commerce Behavior Data from Multi-Category Store*, February 2020 file (7.15 GB, 55,318,565 events). [Source](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store)

| Step | Result |
|---|---|
| Scan the full file in chunks (500K rows) | 51.2M views, 2.89M carts, 1.20M purchases; 4.23M users; 13.5M sessions |
| Keep cart events and flag each session as purchased / abandoned | 2,885,608 cart events |
| Remove exact duplicates | 229,080 removed (7.9%), leaving **2,656,528** cart rows |
| Handle missing values | Category missing on 7.67% of rows, brand on 10%: kept and labelled "Unknown" |
| Add time features | Date, hour (UTC), weekday |

A cart session is **converted** if the same session also contains a purchase, otherwise **abandoned**: 700,091 converted (42.17%) and 960,019 abandoned (57.83%).

### The purchase funnel

89.5% of browsing sessions never reach the cart, and 57.8% of cart sessions never buy. Cart abandonment is the more actionable loss because these customers already showed intent.

## 📈 Key Findings

### 1. Repeat users and bigger carts convert better


| Signal | Result |
|---|---|
| One-time vs 6+ cart-session users | 65.4% vs 45.9% abandonment |
| 1 product vs 2 vs 4-5 products in cart | 60.2% vs 39.8% vs 30.0% |
| Share of carts with a single product | 89.4% (1,484,011 sessions) |

### 2. Instant sessions rarely convert

77.9% of cart sessions stay within a single minute (63.9% abandoned); sessions with 3-10 minutes of cart activity abandon only 30-31%. Purchasers average 1.85 cart events vs 1.42 for abandoners.

### 3. Low-priced items are abandoned most

Items under $50 abandon **65.6%** and $50-100 items **60.0%**, versus roughly 50-53% above $100.

### 4. Category and brand: read volume before rate
The most extreme rates come from small volumes (smartphones 93.8% on 4,263 sessions; Silverlit 97.8% on 1,831), and many top brands are tire brands, so brand partly reflects product type. Among **high-volume** items, Unknown category (68.9%), shoes (63.5%), bicycles (61.3%), Sony (61.0%) and Bosch (62.4%) run above the 57.8% average, while Samsung (45.3%) and Oppo (40.8%) run far below.

> ⚠️ **Data caution:** `construction.tools.light` holds 40% of carts, yet Apple and Samsung items appear under it in the sample, so the category label looks mis-coded.

### 5. Timing

Abandonment is about 56% from 05:00-12:00 UTC and 59-62% from 14:00-18:00 UTC. A weekday view suggests Saturday is worst (62.3%), but 1 February (a Saturday) alone shows 74.5% abandonment on about 104K sessions, and the other Saturdays average roughly 56%. The dashboard also shows an outage-like dip around 26-27 February, so weekday and daily trends need checking for data-boundary effects.

## 🧮 SQL Analysis (DuckDB)

[`cart_abandonment_sql_analysis.sql`](SQL/cart_abandonment_sql_analysis.sql) answers **10 business questions**: overall abandonment, products per cart, price, session duration, cart events, category, brand, hour of day and day of week. Techniques: CTEs, `COUNT(DISTINCT)`, CASE-based banding, `DATE_DIFF`, `EXTRACT`, `DAYNAME`, subqueries and minimum-volume filters (10,000 sessions) so small segments do not distort results.

## ✅ Recommendations

| # | Action | Evidence |
|---|---|---|
| 1 | **Test cross-sell on single-product carts** (related items, free-shipping thresholds) | 89.4% of carts; 60.2% abandon vs 39.8% for 2 products |
| 2 | **Win back one-time visitors** (guest checkout, trust signals, cart reminders) | 65.4% vs 45.9% for repeat users; 28% of cart sessions |
| 3 | **Re-engage instant sessions** (capture email or save the cart) | 77.9% of sessions stay within one minute; 63.9% abandon |
| 4 | **Review sub-$100 baskets** (shipping fees, minimums, bundles) | 65.6% and 60.0% vs about 50% above $100 |
| 5 | **Fix catalog data, then review weak high-volume segments** | 7.7% of carts have no category; shoes, bicycles, Sony, Bosch run above average |
| 6 | **Run recovery campaigns at 14:00-18:00 UTC** | 59-62% abandonment vs about 56% in the morning |

## 🔍 Limitations

- Findings show **association, not causation** (repeat buyers may simply return more often).
- The dataset has **no checkout-step, device or traffic-source fields**, so the exact friction point and device or channel effects cannot be measured here.
- One month of data (February 2020), with start-of-month and late-February anomalies.

## 🚀 Next Steps

- [ ] Instrument checkout steps, device type and traffic source, then rerun the analysis by channel
- [ ] Design A/B tests for recommendations 1-3 with abandonment rate as the success metric
- [ ] Compare several months and exclude data-boundary days to confirm weekday and daily patterns

## 📁 Repository Structure

```
Cart-Abandonment-Pattern-Analysis/
├── README.md
├── Cart Abandonment Pattern Analysis EDA.ipynb    # chunked Python cleaning + EDA
├── Cart Abandonment Pattern SQL Analysis.ipynb    # DuckDB SQL analysis
├── Cart_Abandonment_EDA_Summary.pdf               # written EDA walkthrough
├── SQL/
│   └── cart_abandonment_sql_analysis.sql
├── Dashboard/                                     # Power BI file and page screenshots
├── images/                                        # charts used in this README
└── report/
    └── Cart_Abandonment_Pattern_Analysis_Report.pdf
```

## ▶️ How to Reproduce

1. Download `2020-Feb.csv` from the [Kaggle dataset](https://www.kaggle.com/datasets/mkechinov/ecommerce-behavior-data-from-multi-category-store).
2. Set `file_path` in the EDA notebook to your local copy and run it. It processes the file in chunks and writes `february_cart_events.csv`.
3. Run the SQL notebook (or the `.sql` script) with `pip install duckdb`; it reads `february_cart_events.csv` directly.
4. Open the dashboard in Power BI Desktop.

## 👤 About Me

**Vicky Jeeru**: Data Analyst & Power BI Developer, Hyderabad, India

[![LinkedIn](https://img.shields.io/badge/LinkedIn-0A66C2?style=flat&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/vicky-jeeru-257a18281/)
[![Portfolio](https://img.shields.io/badge/Portfolio-13A89E?style=flat&logo=googlechrome&logoColor=white)](https://vickyjeeru.my.canva.site/)
[![Email](https://img.shields.io/badge/Email-D14836?style=flat&logo=gmail&logoColor=white)](mailto:jeeruvicky51@gmail.com)

*Open to Data Analyst and BI roles. If this project was useful, a ⭐ is appreciated.*
