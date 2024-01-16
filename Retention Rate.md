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
  FROM ferrous-acronym-390114.Transaction_PaDi.transaction_join    
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
        (number_user / cohort_size)*100 as retention_rate
  FROM retention_table  
  order by 1,2;
````

----

### Monthly Retention Rate
 ```sql
 WITH monthly_purchases AS (
  SELECT
    buyer_id,
    DATE_TRUNC(DATE(transaction_date), MONTH) AS month,
    COUNT(DISTINCT transaction_date) AS purchase_count
  FROM
    ferrous-acronym-390114.Transaction_PaDi.transaction_join
  GROUP BY
    buyer_id, month
),

retained_customers AS (
  SELECT
    a.buyer_id,
    a.month
  FROM
    monthly_purchases a
  JOIN
    monthly_purchases b ON a.buyer_id = b.buyer_id
                          AND DATE_TRUNC(DATE_ADD(b.month, INTERVAL 1 MONTH), MONTH) = a.month
  WHERE
    a.purchase_count > 0
)

SELECT
  a.month,
  COUNT(DISTINCT a.buyer_id) AS retained_customers,
  COUNT(DISTINCT b.buyer_id) AS total_customers,
  (COUNT(DISTINCT a.buyer_id) / COUNT(DISTINCT b.buyer_id))*100 AS retention_rate
FROM
  retained_customers a
JOIN
  monthly_purchases b ON a.month = b.month
GROUP BY
  a.month
ORDER BY
  a.month ASC;
```


