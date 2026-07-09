# Delta Lake Assignment — Incremental Data Processing (SCD1)

## Objective

Performed incremental data processing using Delta Lake: load a customer dataset into a Delta
table, clean it, simulate an incremental batch of new/updated records, apply `MERGE`
operations to implement both **SCD Type 1** (overwrite, no history) and validate the results.

## Project Structure

```
delta-lake-assignment/
│
├── data/
│   ├── customer_master.csv          # Initial dataset (contains intentional nulls & duplicates)
│   └── customer_incremental.csv     # Incremental batch: updates to existing customers + new customers
│
├── notebooks/
│   └── merge_into_implementation.ipynb   # Full end-to-end notebook (PySpark + Delta Lake)
│
├── .gitignore
└── README.md
```

## Steps Performed (see the notebook for full code + output)

1. **Load dataset into a Delta table** — read `customer_master.csv`, inspect schema/row
   count, persist as a Delta table (bronze layer).
2. **Basic cleaning** — drop rows with nulls in required fields (`email`, `signup_date`)
   and remove duplicate `customer_id` rows.
3. **Incremental dataset** — `customer_incremental.csv` simulates a new batch: a few
   existing customers with changed `email`/`city`, plus brand-new customers.
4. **MERGE operations**
   - **SCD1**: `whenMatchedUpdate` + `whenNotMatchedInsert` — old values are overwritten,
     no history is kept.
5. **Validation** — row counts before/after, duplicate-key checks on SCD1.
6. **Final output** — display the resulting SCD1 table and a summary table of
   row counts at each stage.