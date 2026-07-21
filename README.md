# Power BI Data Modeling — End-to-End Portfolio Project

Transforming a chaotic, real-world-style "nightmare" dataset into a clean, trustworthy **star schema** in Power BI — following the same step-by-step process used on real client projects.

## 📌 Project Overview

Most Power BI projects that suffer from bad performance or wrong numbers don't actually have a bad report — they have a **bad data model** underneath. This project simulates a messy dataset containing 23 raw tables full of typical real-world chaos: duplicate tables, mixed grains, many-to-many relationships, inconsistent naming, technical IDs, junk columns, and header/detail transactional structures.

The goal: turn this mess into a healthy, well-documented **star schema** ready for reporting — with protected numbers, clear standards, and row-level security.

## 🎯 Objectives

- Explore and understand a raw, undocumented dataset before making any changes
- Design and build clean dimension and fact tables using Power Query
- Apply a consistent modeling standard across the entire model
- Protect key business numbers throughout every transformation
- Implement Row-Level Security (RLS)
- Deliver a final model that is easy to build reports on top of

## 🧭 Process / Phases

**1. Prepare**
- Explore all 23 raw tables to understand the business, entities, and candidate dimensions/facts
- Organize Power Query into folders: `Stage`, `Dimensions`, `Facts`, `Support`

**2. Build Dimensions**
- Consolidate related tables into single, clean dimensions (e.g. Customer, Product)
- Resolve grain mismatches before merging (avoiding row fan-out / duplication)
- Drop unnecessary columns, hash keys, and source-system noise
- Build a junk dimension for low-cardinality flags (order channel, status, priority)
- Enrich data with manually mapped lookup values (e.g. channel codes → friendly names)

**3. Build Facts**
- Split classic **header/detail** transactional tables (orders + order lines) into a clean fact at the lowest grain, pulling context up into dimensions
- Build multiple fact tables: Sales, Inventory, Campaign Spend, and a **factless fact** (Promotion Coverage) for tracking many-to-many relationships without measures
- Build an **accumulating snapshot fact** (Order Fulfillment) to track process milestones (order → ship → deliver → invoice → pay)
- Validate every merge against a protected "total sales" number to catch silent data breaks
- Connect all facts to shared dimensions only — **never fact-to-fact**

**4. Polish**
- Enforce naming conventions (`snake_case`, `dim_`/`fact_` prefixes, `_key` suffix for surrogate keys)
- Standardize date formats and number aggregation behavior
- Auto-generate a `dim_date` table using `CALENDARAUTO()` and connect it as a shared/role-playing dimension
- Build a centralized measures table with core, reusable DAX measures (avoiding duplicate/conflicting logic across reports)
- Implement Row-Level Security based on sales region
- Final validation of key totals before handoff

## 🧱 Key Concepts Demonstrated

- Star schema design principles
- Grain identification and grain mismatches during merges
- Header/detail (transaction header + line item) modeling pattern
- Accumulating snapshot fact tables
- Factless fact tables
- Junk dimensions
- Role-playing dimensions
- Data enrichment / manual mapping tables
- Surrogate vs. natural (business) keys
- Row-Level Security (RLS) with `USERPRINCIPALNAME()`
- Data model governance: naming standards, column pruning, single source of truth

## 🏗️ Final Data Model

- **Dimensions:** `dim_customer`, `dim_product`, `dim_campaign`, `dim_geo`, `dim_order_flags` (junk), `dim_date`
- **Facts:** `fact_sales`, `fact_inventory`, `fact_campaign_spend`, `fact_promotion_coverage` (factless), `fact_order_process` (accumulating snapshot), `fact_sales_targets`
- **Support:** `security` table for RLS
- Clean star schema — single-direction filters, no fact-to-fact relationships, all shared context routed through dimensions

## 🛠️ Tools Used

- Power BI Desktop
- Power Query (M)
- DAX

## 📚 Key Takeaways

1. Always explore and understand the business/grain before modeling.
2. State the grain of every table out loud before connecting anything.
3. Define and follow naming standards from day one.
4. Every column must earn its place — remove what reporting doesn't need.
5. Protect your numbers — validate totals after every structural change.
6. Always aim for a star schema; never connect fact tables directly to each other.

---

*This project was built as a hands-on portfolio piece to demonstrate real-world data modeling skills for Business Intelligence and Analytics roles.*
