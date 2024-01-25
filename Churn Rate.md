## Churn Rate: Persentase pelanggan yang berhenti menggunakan produk atau layanan Anda dalam periode waktu tertentu.

```sql
WITH monthly_purchases AS (
  SELECT
    buyer_id,
    DATE_TRUNC(DATE(transaction_date), MONTH) AS month,
    COUNT(DISTINCT order_id)as number_order
  FROM
    ferrous-acronym-390114.Transaction_PaDi.transaction_join
  GROUP BY
    buyer_id, month
),

churned_customers AS (
  SELECT
    a.buyer_id,
    a.month
  FROM
    monthly_purchases a
  LEFT JOIN
    monthly_purchases b ON a.buyer_id = b.buyer_id
                          AND DATE_TRUNC(DATE_ADD(b.month, INTERVAL 1 MONTH), MONTH) = a.month
  WHERE
    b.buyer_id IS NULL
)

SELECT
  c.month,
  COUNT(DISTINCT b.buyer_id) AS churned_customers,
  COUNT(DISTINCT c.buyer_id) AS total_customers,
  (COUNT(DISTINCT b.buyer_id) / COUNT(DISTINCT c.buyer_id)) * 100 AS churn_rate
FROM
  churned_customers b
JOIN
  monthly_purchases c ON b.month = c.month
GROUP BY
  c.month
ORDER BY
  c.month ASC;
```
