# Gayanara E-Commerce — Data Analytics Portfolio Project

End-to-end analytics project on **Gayanara**, a fictional Indonesian online fashion store.
The dataset ships deliberately messy — duplicate emails, mixed-language product categories,
missing review text — so the work covers the full analyst loop: **audit → clean → model →
segment → visualize → present**.

**Author:** Renaldy Bilal Setyawan
**Stack:** DuckDB · JupySQL (`%%sql`) · Python (pandas) · Tableau Public

🔗 **[Live Dashboard (Tableau Public)](https://public.tableau.com/shared/4BX3MHD2Z?:display_count=n&:origin=viz_share_link)** · **[Presentation Deck](https://docs.google.com/presentation/d/14xbmuDezL6l8d0TN5l4qujirpH5VXvgP/edit?usp=share_link&ouid=105502671421642459213&rtpof=true&sd=true)** · **[Portfolio](https://renalazy.github.io/portofren/)**

---

## ⚠️ About this dataset

Gayanara is a **synthetically generated** dataset (source: [NgulikData](https://ngulikdata.com)).
It is used to demonstrate analytical workflow, not to produce real business conclusions.

Two findings in this project are artifacts of the generating process rather than business
signal, and are labelled as such in the notebooks:

- **Category distribution** — all six categories fall within 14% of each other in revenue.
  Real apparel catalogs are far more concentrated.
- **Retention curve** — flat at ~10% from Month 1 to Month 12 with no decay, indicating
  independent random purchase draws rather than customer lifecycle behaviour.

Distinguishing genuine signal from generator artifact is treated as part of the analysis.

---

## Dataset

| Table | Rows | Grain |
|---|---|---|
| `customers` | 800 | one row per customer account |
| `orders` | 3,000 | one row per order (payment, courier, promo, status) |
| `order_items` | 4,986 | one row per product line inside an order |
| `products` | 300 | one row per SKU (category, material, price) |
| `reviews` | 1,500 | one row per product review |

Raw CSVs live in `Dataset/gayanara/`. The analytical database `gayanara.db` is **not** tracked —
it is rebuilt from those CSVs by Task 0 of notebook 01.

---

## Metric definitions

Every figure in this project resolves to one of these. They reconcile exactly.

| Metric | Definition | Value |
|---|---|---|
| **Gross revenue** | `SUM(order_items.subtotal_idr)`, excl. cancelled & returned | Rp 1,296,768,000 |
| **Net revenue** | Gross less order-level discounts | **Rp 1,265,340,379** |
| **Completed orders** | Orders not cancelled or returned | **2,521** |
| **Average order value** | Net revenue ÷ completed orders | **Rp 501,920** |
| **Units sold** | `SUM(order_items.quantity)` | 5,442 |

Net revenue reconciles to the penny against `orders.total_amount_idr`. Shipping is excluded
as a pass-through cost. Category-level revenue is reported gross, because order-level
discounts cannot be attributed to individual line items.

---

## Project phases

| Phase | Scope | Output |
|---|---|---|
| **1 — Data Quality Audit** | null audit, duplicate customers, orphaned records, category standardization | [`01_Data_Quality_Audit.ipynb`](01_Data_Quality_Audit.ipynb) |
| **2 — Exploratory Analysis** | revenue trends, category performance, RFM segmentation, geography, promo impact, courier evaluation, cohort retention | [`02_Exploratory_Data_Analysis_EDA.ipynb`](02_Exploratory_Data_Analysis_EDA.ipynb) |
| **3 — BI Delivery** | executive dashboard (7 views, date filter) | Tableau Public |
| **4 — Communication** | stakeholder deck with recommendations and limitations | `Gayanara_-_Performance_Review.pptx` |

---

## Key findings

**Growth is volume-led, not price-led.**
Monthly net revenue grew roughly 3× from 2022 to early 2025 (Rp 21.8M → Rp 66.3M). Over the
same period monthly orders rose from ~32 to 100+, while AOV showed no directional trend —
fluctuating between Rp 390K and Rp 610K with no upward movement. The business is acquiring
customers, not increasing basket size.

**Rp 31.4M in discounts produced no measurable basket lift.**
A naive comparison of `total_amount_idr` shows promo orders with a *lower* AOV — but that
field is recorded net of discount, so the discount is subtracted from promo orders and then
read as smaller baskets. Comparing like-for-like on gross basket value, promo orders average
Rp 515,199 versus Rp 513,830 for organic orders: a 0.3% difference, effectively zero.
Recommendation is threshold-based promos, which structurally require a larger basket to unlock.

**Anteraja's cancellation rate warrants investigation, not suspension.**
At 14.33% (44 of 307 orders) it leads all couriers, against a fleet average near 10%. But the
sample is thin, and Anteraja simultaneously has the *lowest* return rate (2.61%) — pointing
toward pre-dispatch friction rather than handling damage. Recommendation is a dispatch-time
audit and a controlled volume shift, not a contract decision on this evidence.

**The customer base is cold.**
Median recency across 787 customers is 145 days. 343 fall into Lost/Inactive and 135 are
At Risk VIPs. Note that the Lost threshold (120 days) sits *below* median recency, so the
segment is large partly by construction — stated explicitly in the notebook.

**Retention does not decay.**
Pooled across all cohorts, repeat purchase rate holds between 8.1% and 12.6% from Month 1 to
Month 12 (M1: 9.15%, M6: 10.32%, M12: 10.40%). Real e-commerce retention drops steeply in
Month 1 and then flattens. A flat curve means tenure carries no information — the signature
of independent random draws, and evidence that this dataset cannot support churn analysis.

---

## Limitations

- **No margin data**, so promo analysis measures revenue impact only, not true ROI. The
  comparison is observational; establishing incrementality would require a holdout test.
- **Courier sample sizes** — Anteraja (307 orders) and Pos Indonesia (161) carry too little
  volume for confident rate comparison. No significance testing was performed.
- **No lifecycle dynamics** in the generated data, so churn drivers and cohort quality are
  out of scope.
- **3 duplicate email addresses** were identified and deliberately *not* merged; `customer_id`
  is retained as the grain to preserve the source system's definition of a customer.
- **RFM boundaries are fixed business rules**, not statistical clusters. A quantile-based
  scoring approach would adapt as the customer base grows.
- **Scope:** Jan 2022 – Feb 2025. February 2025 may be a partial month; the apparent revenue
  spike should not be read as trend.

---

## Reproduce

```bash
git clone https://github.com/renalazy/gayanara-ecommerce.git
cd gayanara-ecommerce

python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt

jupyter lab
```

Run `01_Data_Quality_Audit.ipynb` top to bottom first — Task 0 ingests the CSVs and creates
`gayanara.db` locally. Then run `02_Exploratory_Data_Analysis_EDA.ipynb`, which reads that
database and writes six aggregated CSVs to `dashboard_exports/` for Tableau.

Notebooks resolve paths relative to the repo root, so launch Jupyter from there.

---

## Repo layout

```
├── 01_Data_Quality_Audit.ipynb            # Phase 1: audit + cleaning
├── 02_Exploratory_Data_Analysis_EDA.ipynb # Phase 2: analysis + exports
├── Dataset/gayanara/                      # raw source CSVs
│   ├── customers.csv
│   ├── orders.csv
│   ├── order_items.csv
│   ├── products.csv
│   └── reviews.csv
├── dashboard_exports/                     # aggregated CSVs → Tableau
│   ├── dashboard_monthly_revenue.csv
│   ├── dashboard_geospatial.csv
│   ├── dashboard_promo_roi.csv
│   ├── dashboard_courier_eval.csv
│   ├── dashboard_rfm_segments.csv
│   └── dashboard_cohort_retention.csv
├── images/                                # dashboard screenshots
├── Gayanara_-_Performance_Review.pptx     # stakeholder deck
├── requirements.txt
└── README.md
```