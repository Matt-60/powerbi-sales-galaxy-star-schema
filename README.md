# Power BI Data Modeling — Galaxy Schema from 23 Messy Tables

Most Power BI projects with slow reports or wrong numbers don't have a bad report — they have a **bad data model** underneath. This project takes a deliberately messy workbook of 23 raw sheets (duplicate tables, mixed grains, header/detail splits, split shipments, duplicate payments, inconsistent naming) and rebuilds it into a documented **galaxy schema**: 6 fact tables sharing conformed dimensions, 20 governed measures and row-level security.

`Power BI Desktop` · `PBIP / TMDL` · `Power Query (M)` · `DAX` · `Galaxy Schema` · `RLS`

---

## 🎯 Business Goal

The dataset represents an international B2B distributor of electronics and apparel selling to retail chains and businesses on credit terms (Net 15–60), managed by account managers across 5 regions.
Sales, marketing, finance and operations need one trustworthy model for revenue, targets, campaigns, stock and fulfillment — without duplicated numbers or broken totals. Questions the model answers:

- *What were net sales and margin by region and product last quarter, and how do they compare to target?*
- *Which promoted products didn't sell at all during the campaign?*
- *What is our closing stock and sell-through by category?*
- *How many days does it take from order to shipment and to payment?*


## 🏗️ Data Model

<img width="1143" height="614" alt="image" src="https://github.com/user-attachments/assets/883f8b48-39e2-4392-9fe5-a04eeed73f2f" />

| Table | Type | Grain |
|---|---|---|
| `fact_sales` | Transaction fact | One order line |
| `fact_order_process` | Accumulating snapshot | One order (order → ship → deliver → invoice → pay dates) |
| `fact_inventory` | Periodic snapshot | Product × month (stock snapshot) |
| `fact_campaign_spend` | Transaction fact | Campaign × day |
| `fact_promotion_coverage` | Factless fact | Campaign × promoted product |
| `fact_sales_targets` | Plan fact (coarser grain) | Month, company level |
| `dim_customer`, `dim_product`, `dim_geo`, `dim_campaign` | Dimensions | One row per member |
| `dim_order_flags` | Junk dimension | Unique combination of channel / status / priority / ship mode |
| `dim_date` | Date dimension (DAX, `CALENDARAUTO`) | One day; role-playing via inactive relationships |
| `security` | RLS mapping (hidden) | User e-mail → region |

**Rules applied:** every relationship is many-to-one and single-direction, there are no fact-to-fact relationships, all shared context goes through conformed dimensions, Auto date/time is off, and keys, PII and raw fact columns are hidden — users only see dimensions and measures.

## 🔧 Power Query Structure

Power Query here is the ETL layer, but nothing is persisted between steps, so this is **layered Power Query, not a medallion architecture**:

| Query group | Purpose | Loaded |
|---|---|---|
| `00_parameters` | `SourceFile` — path to `raw_sales_tables.xlsx` | No |
| `01_staging` | One query per source sheet: headers, data types, names. No business logic | No |
| `02_model\dims`, `02_model\facts` | Joins, grain fixes, surrogate keys, unknown member — the tables the model uses | Yes |

## 🧠 Key Design Decisions

<details>
<summary><b>Click to expand</b></summary>

- **Split shipments → no fan-out.** 10 orders ship in two parcels. `shipments` is grouped to one row per order (first `ShipDate`; `DeliveryDate` = when the *last* parcel arrives, blank if any is still in transit; `ShipmentCount`) before it is joined to `fact_order_process`.
- **Duplicate payments.** 5 invoices have an extra payment of exactly half the amount. Payments are matched to invoices on `InvoiceID + Amount` and grouped to one row per invoice, so each order has one pay date.
- **Gross vs net sales.** `total_sales` is the source `LineTotal` (qty × price, before discount) and is the protected reconciliation number. `net_sales` applies the line discount and is the revenue measure used everywhere else.
- **Targets at a coarser grain.** Targets exist only per month for the whole company. `target_revenue` returns blank below month level or when sliced by product, customer, geography, flags or campaign, instead of showing a misleading repeated total.
- **Semi-additive stock.** `inventory_units` returns the closing balance of the last month in the period; stock is never summed over time.
- **Unknown member.** Order lines and inventory rows whose product is missing from the product master map to `product_key = -1` ("Unknown / retired product"), so totals still reconcile and the issue is visible in reports.
- **Role-playing dates.** `dim_date` is active on order date; ship, delivery, invoice and pay dates are inactive relationships used through `USERELATIONSHIP`. The same pattern is used for ship-to vs bill-to city on `dim_geo`.
- **Isolated dimensions.** `dim_campaign` filters only campaign spend and promotion coverage. Campaign impact on sales goes through the factless fact (`TREATAS`), not through a fake relationship.

</details>

## 📐 Measures

20 measures in `_measure`, grouped in display folders. A field parameter (`Parameter`) lets report users switch the metric on a visual.

<details>
<summary><b>Full list (click to expand)</b></summary>

| Folder | Measure | Definition |
|---|---|---|
| 01 Sales | `total_sales` | Gross sales, `SUM(line_total)` — protected total |
| | `net_sales` | Sales after line discount |
| | `total_quantity` | Units sold |
| | `total_orders` | Distinct orders |
| | `avg_order_value` | Net sales / orders |
| | `total_cost` | Quantity × unit cost |
| | `gross_margin_pct` | (Net sales − cost) / net sales |
| | `net_sales_bill_to` | Net sales by bill-to city (`USERELATIONSHIP`) |
| 02 Customers | `total_active_customers` | Customers with at least one order line |
| 03 Targets | `target_revenue` | Monthly company target (blank at unsupported grain) |
| | `target_attainment_pct` | Gross sales / target |
| 04 Marketing | `campaign_spend` | Campaign spend |
| | `ctr_pct` | Clicks / impressions |
| | `marketing_spend_pct_of_sales` | Spend / net sales — drill-across two facts via `dim_date` |
| 05 Promotions | `promoted_product_sales` | Net sales of products covered by the selected campaign (`TREATAS`) |
| | `promoted_products_without_sales` | Promoted products with zero sales — factless-fact analysis |
| 06 Inventory | `inventory_units` | Closing stock (semi-additive) |
| | `sell_through_pct` | Sold / (sold + closing stock) |
| 07 Fulfillment | `avg_order_to_ship` | Avg days order → first shipment |
| | `avg_order_to_pay` | Avg days order → payment (paid orders) |

</details>

## 🔒 Row-Level Security

Role **Regional access**: `LOWER(security[user_email]) = LOWER(USERPRINCIPALNAME())`. The `security` table filters `dim_customer[region]`, so a regional user sees only their customers' **sales and orders** (`fact_sales`, `fact_order_process`).

Targets, inventory and campaigns are company-level data with no customer or region attribute, so they are intentionally **not** restricted by this role. The mapping assumes one region per user.

## ✅ Model Validation

The report contains one page, **Model Validation**, used to reconcile the model after every structural change: card totals vs `SUM(line_total)`, a year → quarter → month breakdown of sales, orders and quantity, and a trend chart driven by the metric field parameter.


## ▶️ How to Run

1. Clone the repo and open `Data_modeling_sales.pbip` in Power BI Desktop (PBIP / TMDL format — the model is fully diffable in Git).
2. **Transform data → Edit parameters** → set `SourceFile` to the full path of `raw_sales_tables.xlsx` in your clone.
3. **Refresh**.
4. To test RLS: **Modeling → View as → Regional access** with **Other user** set to an e-mail from the `security` sheet.

## 📚 Key Takeaways

1. Understand the business and state the grain of every table before connecting anything.
2. Fix grain mismatches (split shipments, duplicate payments) *before* joining, or totals silently inflate.
3. Not every number belongs at every level — targets and stock need measures that respect their grain.
4. Keep dimensions isolated; answer cross-process questions with shared dimensions and DAX, never fact-to-fact joins.
5. Protect a reconciliation number and validate it after every change.
6. Follow naming standards from day one (`snake_case`, `dim_` / `fact_`, `_key`).

---

*Built as a hands-on portfolio piece to demonstrate real-world data modeling skills for BI and Analytics Engineering roles.*
