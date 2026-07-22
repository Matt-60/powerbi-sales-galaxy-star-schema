# Power BI Data Modeling — End-to-End Portfolio Project

Most Power BI projects that suffer from bad performance or wrong numbers don't actually have a bad report — they have a **bad data model** underneath. This project simulates a messy dataset of 23 raw tables full of real-world chaos (duplicate tables, mixed grains, many-to-many relationships, inconsistent naming, header/detail structures) and rebuilds it into a healthy, well-documented **star schema** with protected numbers and row-level security.

`Power BI Desktop` · `Power Query (M)` · `DAX` · `Star Schema` · `RLS`

---

## 🎯 Business Goal

Sales, marketing, and finance teams need one trustworthy source of truth for revenue, orders, campaigns, and fulfillment — without duplicated numbers, broken totals, or slow reports. This project rebuilds a real inherited data model so business users can safely answer questions like *"what were total sales by region and product last quarter"* or *"how long does it take from order to payment"* — with confidence the numbers won't silently break as the model grows.

## 🏗️ Final Data Model

- **Dimensions:** `dim_customer`, `dim_product`, `dim_campaign`, `dim_geo`, `dim_order_flags` (junk), `dim_date`
- **Facts:** `fact_sales`, `fact_inventory`, `fact_campaign_spend`, `fact_promotion_coverage` (factless), `fact_order_process` (accumulating snapshot), `fact_sales_targets`
- **Support:** `security` table for RLS
- Clean star schema — single-direction filters, no fact-to-fact relationships, all shared context routed through dimensions

<img width="1134" height="616" alt="star schema" src="https://github.com/user-attachments/assets/c8d87768-b3b4-4e37-b41a-afebcf879de8" />

## 🧱 Key Concepts Demonstrated

Star schema design · grain identification & grain-mismatch resolution · header/detail modeling · accumulating snapshot & factless facts · junk & role-playing dimensions · data enrichment · surrogate vs. natural keys · Row-Level Security (`USERPRINCIPALNAME()`) · naming standards & single source of truth

<details>
<summary><b>🧭 Full build process — Prepare → Dimensions → Facts → Polish (click to expand)</b></summary>

**1. Prepare**
- Explored all 23 raw tables to understand the business, entities, and candidate dimensions/facts
- Organized Power Query into folders: `Stage`, `Dimensions`, `Facts`, `Support`

**2. Build Dimensions**
- Consolidated related tables into single, clean dimensions (Customer, Product)
- Resolved grain mismatches before merging (avoiding row fan-out)
- Dropped unnecessary columns, hash keys, source-system noise
- Built a junk dimension for low-cardinality flags (channel, status, priority)
- Enriched data with manually mapped lookup values (channel codes → friendly names)

**3. Build Facts**
- Split header/detail tables (orders + order lines) into a clean fact at the lowest grain
- Built Sales, Inventory, Campaign Spend facts, plus a factless fact (Promotion Coverage)
- Built an accumulating snapshot fact (Order Fulfillment: order → ship → deliver → invoice → pay)
- Validated every merge against a protected "total sales" number to catch silent breaks
- Connected all facts to shared dimensions only — never fact-to-fact

**4. Polish**
- Enforced naming conventions (`snake_case`, `dim_`/`fact_` prefixes, `_key` suffix)
- Standardized date formats and aggregation behavior
- Auto-generated `dim_date` via `CALENDARAUTO()`, connected as a shared/role-playing dimension
- Built a centralized measures table with reusable DAX
- Implemented Row-Level Security based on sales region
- Final validation of key totals before handoff

</details>

## 📚 Key Takeaways

1. Always explore and understand the business/grain before modeling.
2. State the grain of every table out loud before connecting anything.
3. Define and follow naming standards from day one.
4. Every column must earn its place — remove what reporting doesn't need.
5. Protect your numbers — validate totals after every structural change.
6. Always aim for a star schema; never connect fact tables directly to each other.

---

*Built as a hands-on portfolio piece to demonstrate real-world data modeling skills for Business Intelligence and Analytics roles.*
