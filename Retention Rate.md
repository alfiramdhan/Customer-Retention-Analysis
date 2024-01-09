## Retention Rate: Persentase pelanggan yang tetap menggunakan produk atau layanan Anda dalam periode waktu tertentu.

### In the initial cohort analysis, we need to determine what data are needed for the analysis.
- Agg value : count(distinct user_id)
- Unique identifier : user_id
- Initial start date : first_post_date
- Period : month


````sql
WITH user_activity AS(
  SELECT
        DISTINCT buyer_id,
        MIN(DATE_TRUNC(DATE(transaction_date),MONTH)) OVER(PARTITION BY buyer_id) as first_txn_date,
        DATE_TRUNC(DATE(transaction_date),MONTH) as running_txn_date
  FROM ferrous-acronym-390114.padi_umkm.transaction     
),
cohort_sizes AS(
  SELECT *,
        DATE_DIFF(running_txn_date, first_txn_date, month)as diff_month,
        COUNT(DISTINCT buyer_id) OVER(PARTITION BY first_txn_date) cohort_size
  FROM user_activity      
),
retention_table AS(
  SELECT
        first_txn_date,
        diff_month,
        cohort_size,
        COUNT(DISTINCT buyer_id)as number_user
  FROM cohort_sizes
  WHERE  first_txn_date >= '2020-01-01'     
  GROUP BY 1,2,3
  ORDER BY 1,2
)
  SELECT
        first_txn_date,
        diff_month,
        number_user / cohort_size as retention_rate
  FROM retention_table  
  order by 1,2;
````
