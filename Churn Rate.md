## Churn Rate: Persentase pelanggan yang berhenti menggunakan produk atau layanan Anda dalam periode waktu tertentu.

### In the initial cohort analysis, we need to determine what data are needed for the analysis.
- Agg value : count(distinct buyer_id)
- Unique identifier : buyer_id
- Initial start date : first_post_date
- Period : month

```sql
WITH user_activity AS (
  SELECT
    DISTINCT buyer_id,
    MIN(DATE_TRUNC(DATE(transaction_date), MONTH)) OVER (PARTITION BY buyer_id) AS first_txn_date,
    DATE_TRUNC(DATE(transaction_date), MONTH) AS running_txn_date
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
    COUNT(DISTINCT buyer_id) AS number_user
  FROM
    cohort_sizes
  WHERE
    first_txn_date >= '2020-01-01'
  GROUP BY
    1, 2, 3
  ORDER BY
    1, 2
),
churn_table AS (
  SELECT
    rt.first_txn_date,
    rt.diff_month,
    rt.cohort_size,
    rt.number_user,
    LAG(rt.number_user) OVER (PARTITION BY rt.first_txn_date ORDER BY rt.diff_month) AS previous_user
  FROM
    retention_table AS rt
)
SELECT
  ct.first_txn_date,
  ct.diff_month,
  (ct.previous_user - ct.number_user) / ct.cohort_size AS churn_rate
FROM
  churn_table AS ct
WHERE
  ct.previous_user IS NOT NULL
ORDER BY
  1, 2;
```
