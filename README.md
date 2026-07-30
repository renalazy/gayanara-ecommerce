# Gayanara E-Commerce — Data Analytics Portfolio Project

End-to-end SQL analytics project on **Gayanara**, a fictional Indonesian online fashion store.
The dataset is deliberately messy (duplicate emails, inconsistent product categories, missing
review text), so the work covers the full analyst loop: **audit → clean → answer business
questions → segment customers → present**.

**Author:** Renaldy Bilal Setyawan
**Stack:** DuckDB · JupySQL (`%%sql`) · Python (pandas) · Jupyter

---

## Dataset

Source: [NgulikData — Gayanara: Toko Fashion Online](https://ngulikdata.com) (5 tables, 10,586 rows, ~736 KB).

| Table | Rows | Grain |
|---|---|---|
| `customers` | 800 | one row per customer account |
| `orders` | 3,000 | one row per order (payment, courier, promo, status) |
| `order_items` | 4,986 | one row per product line inside an order |
| `products` | 300 | one row per SKU (category, material, price) |
| `reviews` | 1,500 | one row per product review |

Raw CSVs live in `Dataset/gayanara/`. The analytical database `gayanara.db` is **not** tracked —
it is rebuilt from those CSVs by Task 0 of notebook 01.

## Roadmap

| Phase | Scope | Status |
|---|---|---|
| **1 — Data Quality & Foundation** | null audit, duplicate customers, orphaned records, category standardization | ✅ Done → [`01_Data_Quality_Audit.ipynb`](01_Data_Quality_Audit.ipynb) |
| **2 — Ad-Hoc Business Insights** | top products vs. dead stock, payment & courier mix, revenue by city, promo effectiveness, courier return rates | 🔜 Next |
| **3 — Advanced Analytics** | RFM segmentation, cohort retention heatmap | ⏳ Planned |
| **4 — Final Deliverables** | executive sales dashboard, case-study write-up & deck | ⏳ Planned |

Full detail: [`Gayanara_Project_Roadmap.pdf`](Gayanara_Project_Roadmap.pdf).

## Phase 1 findings

- **Nulls** — `customers.phone` 5.0% (40 rows), `products.material` 3.0% (9 rows),
  `reviews.review_text` 8.0% (120 rows). Only `phone` materially constrains SMS campaigns;
  `material` gets an `'Unknown'` default, missing review text is normal behaviour.
- **Duplicate customers** — 3 email addresses map to 2 `customer_id`s each. Must be deduplicated
  before any customer-level metric (LTV, RFM) to avoid double-counting revenue.
- **Category chaos** — 14 distinct values from mixed Indonesian/English terms and casing
  (*Aksesoris* vs *Accessories*, *Dress* vs *dress*). Collapsed to **6** clean categories via a
  `CASE WHEN` mapping, persisted as a new `products.category_clean` column.
- **Referential integrity** — 0 orphaned rows across 4,986 `order_items` → `products`, so
  `INNER JOIN` is safe for all revenue and product-mix work.

## Reproduce

```bash
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
jupyter lab
```

Then run `01_Data_Quality_Audit.ipynb` top to bottom — Task 0 ingests the CSVs and creates
`gayanara.db` locally.

## Repo layout

```
├── 01_Data_Quality_Audit.ipynb    # Phase 1: audit + cleaning
├── Dataset/gayanara/*.csv         # raw source data
├── Gayanara_Project_Roadmap.pdf   # project plan (4 phases)
├── images/                        # screenshots used in write-ups
└── requirements.txt
```
