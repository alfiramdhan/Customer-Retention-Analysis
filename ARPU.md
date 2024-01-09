### Average Revenue Per User (ARPU): Rata-rata pendapatan yang dihasilkan per pelanggan dalam periode waktu tertentu

```sql
WITH user_activity AS (
  SELECT
    DISTINCT buyer_id,
    MIN(DATE_TRUNC(DATE(transaction_date), MONTH)) OVER (PARTITION BY buyer_id) AS first_txn_date,
    DATE_TRUNC(DATE(transaction_date), MONTH) AS running_txn_date,
    SUM(revenue) OVER (PARTITION BY buyer_id) AS total_revenue
  FROM
    ferrous-acronym-390114.padi_umkm.transaction
),
cohort_sizes AS (
  SELECT
    *,
    DATE_DIFF(running_txn_date, first_txn_date, month) AS diff_month,
    COUNT(DISTINCT buyer_id) OVER (PARTITION BY first_txn_date) AS cohort_size
  FROM
    user_activity
),
retention_table AS (
  SELECT
    first_txn_date,
    diff_month,
    cohort_size,
    COUNT(DISTINCT buyer_id) AS number_user,
    SUM(total_revenue) AS total_revenue
  FROM
    cohort_sizes
  WHERE
    first_txn_date >= '2020-01-01'
  GROUP BY
    1, 2, 3
  ORDER BY
    1, 2
)
SELECT
  first_txn_date,
  diff_month,
  total_revenue / cohort_size AS arpu
FROM
  retention_table
ORDER BY
  1, 2;
```
