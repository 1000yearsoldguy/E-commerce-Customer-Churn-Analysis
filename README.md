# Ecommerce Customer Churn Analysis

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![SQLite](https://img.shields.io/badge/SQL-SQLite%20%28zero%20setup%29-green.svg)](https://www.sqlite.org/)
[![Data](https://img.shields.io/badge/Data-Kaggle-yellow.svg)](https://www.kaggle.com/code/ankitverma2010/e-commercecustomerchurn/input)

SQL-first exploratory analysis of ecommerce churn. Every cleaning and analysis
step is a raw SQL statement run against an in-memory SQLite copy of `Data.csv`.
Python only loads the CSV and executes the statements — no DB server, no ORM,
no query builder.

## Files

| File | Description |
|------|-------------|
| `notebook.ipynb` | The whole analysis: 68 cells (44 code, 24 markdown) — cleaning pipeline, then 17 questions, each with a SQL cell and a chart |
| `Data.csv` | Source dataset — 5,630 customers, 20 columns (Kaggle) |
| `README.md` | This document |

## Data Source

- **Kaggle**: [`ankitverma2010/e-commercecustomerchurn/input`](https://www.kaggle.com/code/ankitverma2010/e-commercecustomerchurn/input)
- 5,630 rows, 20 columns; target `Churn` (0 = stayed, 1 = churned).
- 948 customers churned — a **16.84%** churn rate.
- 7 columns carry nulls: `Tenure`, `WarehouseToHome`, `HourSpendOnApp`,
  `OrderAmountHikeFromLastYear`, `CouponUsed`, `OrderCount`, `DaySinceLastOrder`.

## Quick Start

```bash
pip install pandas matplotlib seaborn jupyter
jupyter notebook notebook.ipynb
```

Run all cells top-to-bottom. Expect ~1–2 s. Outputs include 17 result tables,
17 charts, and the cleaned table used by every query.

## How the Notebook Works

1. **Load** — `pandas.read_csv("Data.csv").convert_dtypes()` writes the CSV to
   an in-memory SQLite table `ecommercechurn`. `convert_dtypes` keeps
   whole-number columns as `INTEGER`, so `SUM`/`AVG` return numbers, not strings.
2. **`sql()` helper** — one function for every cell: `SELECT` returns a
   DataFrame, any other statement returns the affected row count.
3. **Clean** — one `UPDATE ... COALESCE` fills all 7 null columns with their
   truncated means (`CAST(AVG(col) AS INT)`), then duplicate `CustomerID`s are
   checked.
4. **Normalise** — readable status columns (`CustomerStatus`, `ComplainRecieved`),
   category fixes (`Mobile` → `Mobile Phone`, `mobile phone` → `Phone`,
   `COD` → `Cash on Delivery`), and two distance typos (`127` → `27`, `126` → `26`).
5. **Derive** — `WarehouseToHomeRange`, `TenureRange`, `CashbackAmountRange` as
   `CASE`-based buckets for questions 4, 6 and 17.
6. **Analyse** — 17 `GROUP BY` queries plus single-value aggregates, each
   followed by a matplotlib/seaborn chart that re-runs the same query.

All logic is raw SQL; the only Python is the loader, the `sql()` helper, and the
chart code.

## Business Questions Answered

| # | Question | Key Result |
|---|----------|------------|
| 1 | Overall churn rate | 16.84% (948 of 5,630) |
| 2 | Churn by login device | Computer 19.83%, Phone 15.62% |
| 3 | Churn by city tier | Tier 3: 21.37%, Tier 2: 19.83%, Tier 1: 14.51% |
| 4 | Warehouse distance vs churn | Rises with distance: far 20.90%, very close 13.51% |
| 5 | Churn by payment mode | Cash on Delivery 24.90% highest, Credit Card 12.86% lowest |
| 6 | Typical tenure of churned customers | 6 months: 32.42%, 1 year: 9.85%, >2 years: 0.00% |
| 7 | Gender difference | Male 17.73% vs Female 15.49% |
| 8 | App usage: churned vs retained | 2.90 h vs 2.89 h — no meaningful difference |
| 9 | Registered devices vs churn | 6 devices: 34.57%, 1 device: 9.36% |
| 10 | Order category vs churn | Mobile Phone 27.40% highest, Grocery 4.88% lowest |
| 11 | Satisfaction score vs churn | Score 5: 23.83% highest, score 1: 11.51% lowest |
| 12 | Marital status | Single 26.73%, Divorced 14.62%, Married 11.52% |
| 13 | Avg addresses (churned) | 4.47 |
| 14 | Complaints vs churn | Yes: 31.67% vs No: 10.93% (~2.9×) |
| 15 | Coupon usage | Churned: 1,622 total (≈1.71 each), retained: 8,044 (≈1.72 each) |
| 16 | Days since last order (churned) | 3.28 |
| 17 | Cashback amount vs churn | Moderate ($100–200): 18.91%, high: 10.72%, very high: 6.41% |

## Notable Findings

- **Tenure is the strongest signal.** A third of customers in their first six
  months churn; nobody past two years does.
- **Complaints nearly triple churn risk** (31.67% vs 10.93%).
- **Satisfaction score runs backwards.** Score 5 churns most (23.83%) — worth
  treating as a data-quality question rather than a business insight.
- **Coupon totals mislead.** Churned customers used fewer coupons in total only
  because there are fewer of them; usage per customer is essentially identical.
- **Cashback is not monotonic.** Moderate cashback has the highest churn, and
  the "low" bucket has just 12 customers, so its 0.00% is noise.

## Caveats

- The notebook fills nulls *before* printing the per-column null report, so that
  report reads zero by design; the 7 affected columns are listed above.
- Q5 ranks payment modes by churn *within each mode*. It does not answer "which
  mode is most used by churned customers" — filter `CustomerStatus = 'Churned'`
  before grouping for that.
- Q13 corrects an error in the original T-SQL, which filtered
  `CustomerStatus = 'stayed'` while asking about churned customers.
- Cleaning mutates the table in place, so cells assume they run in order on a
  freshly loaded `Data.csv`.

## Requirements

- Python 3.10+
- `pandas`, `matplotlib`, `seaborn`, `jupyter` (SQLite ships with Python's stdlib)
