# Ecommerce Customer Churn Analysis

[![Python](https://img.shields.io/badge/Python-3.10%2B-blue.svg)](https://www.python.org/)
[![Jupyter](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![SQLite](https://img.shields.io/badge/SQL-SQLite%20%28zero%20setup%29-green.svg)](https://www.sqlite.org/)
[![Data](https://img.shields.io/badge/Data-Kaggle-yellow.svg)](https://www.kaggle.com/code/ankitverma2010/e-commercecustomerchurn/input)

SQL-first exploratory analysis of ecommerce churn. All logic runs as SQL inside
a single Jupyter notebook (`notebook.ipynb`) on an in-memory SQLite copy of
`Data.csv`. No DB server, no migration tooling, no opaque ORM.

## Files

| File             | Description                                                          |
|------------------|----------------------------------------------------------------------|
| `notebook.ipynb` | 27 raw SQL cells: schema/data checks, cleaning, 3 derived bins, 17 BBQs |
| `Data.csv`       | Source dataset — 5,630 customers, 20 columns (Kaggle)                 |
| `README.md`      | This document                                                        |

## Data Source

- **Kaggle**: [`ankitverma2010/e-commercecustomerchurn/input`](https://www.kaggle.com/code/ankitverma2010/e-commercecustomerchurn/input)
- 5,630 rows, 20 columns; target `Churn` (0 = stayed, 1 = churned).
- 7 columns carry nulls: `Tenure`, `WarehouseToHome`, `HourSpendOnApp`,
  `OrderAmountHikeFromLastYear`, `CouponUsed`, `OrderCount`, `DaySinceLastOrder`.

## Quick Start

```bash
pip install pandas nbformat nbclient jupyter
jupyter notebook notebook.ipynb
```

Run all cells. Expect ~1–2 s. Outputs include: churn rate 16.84%,
5630 total customers, 0 duplicates, 7 null counts, 17 Q&A tables.

## Business Questions Answered

| #  | Question | Key Result |
|----|----------|------------|
| 1 | Overall churn rate | 16.84% |
| 2 | Churn by login device | Computer 19.83%, Phone 15.62% |
| 3 | Churn by city tier | Tier 3: 21.37% (highest) |
| 4 | Warehouse distance vs churn | Churn ↑ with distance |
| 5 | Churn rate by payment mode | Cash on Delivery: 24.90% |
| 6 | Typical tenure (churned) | 6 months: 32.42% |
| 7 | Gender | Male 17.73% vs Female 15.49% |
| 8 | App usage: churned vs retained | ~2.9 hrs (no meaningful diff) |
| 9 | Registered devices vs churn | 6 devices: 34.57% |
| 10 | Order category vs churn | Mobile Phone 27.40%, Grocery 4.88% |
| 11 | Satisfaction score vs churn | Score 5: 23.83% (highest) |
| 12 | Marital status | Single 26.73% vs Married 11.52% |
| 13 | Avg addresses (churned) | ≈ 4.5 |
| 14 | Complaints influence | Yes: 31.67% vs. No: 10.93% |
| 15 | Coupon usage | Churned: 1,622 vs. Retained: 8,044 |
| 16 | Days since last order (churned) | ~3.3 days |
| 17 | Cashback amount vs churn | $100–200: 18.91% (highest) |

## Method

1. Validate: total customers, duplicates, nulls.
2. Impute nulls with truncated column means (`CAST(AVG(col) AS INT)`).
3. Normalise inconsistent values (`Mobile Phone` ↔ `Mobile`, `COD` ↔ `Cash on Delivery`, typos 126/127).
4. Add derived columns `CustomerStatus`, `ComplainRecieved`, `WarehouseToHomeRange`,
   `TenureRange`, `CashbackAmountRange`.
5. Run 17 `GROUP BY` analyses and single-value aggregates.

All logic is raw SQL. The only Python is the SQLite loader and a one-line `sql()` helper per cell.

## Porting to SQL Server / Azure SQL

| Notebook (SQLite) | T-SQL | Why |
|---|---|---|
| `ROUND(x, 2)` | `CAST(x AS DECIMAL(10,2))` | SQLite ignores DECIMAL precision |
| `CAST(AVG(col) AS INT)` | `AVG(col)` | T-SQL `AVG(int)` returns `int` |
| `COLLATE NOCASE` on status filters | device defaults (case-insensitive) | SQL Server collation is case-insensitive |
| `ALTER TABLE t ADD c TYPE` | `ALTER TABLE t ADD c TYPE` | Identical here |

Steps:
1. Import `Data.csv` into a table named `ecommercechurn` (headers match column names).
2. Open the converted script in SSMS / Azure Data Studio.
3. Execute top-to-bottom — **not idempotent**; re-import before a clean re-run.

## Known Inconsistencies / Caveats

- The original T-SQL marks Q13 as "churned customers" but filters `CustomerStatus = 'stayed'` (verified against the original script).
  The notebook corrects this to `CustomerStatus = 'Churned'` and states the corrected result (~4.5 addresses).
- Q5 groups *all* customers and ranks churn rate — i.e. "churn rate per payment mode",
  not "most-used mode among churned users". Filter `WHERE CustomerStatus = 'Churned'` first if you need the latter.
- The cleaning steps mutate the table in place; running cells in isolation on a fresh schema is not supported.

## Requirements

- Python 3.10+
- `pandas`, `nbformat`, `nbclient`, `jupyter` (no DB server needed — SQLite is in `sqlite3`, part of stdlib)