## Monthly_New_Buyers
```sql
SELECT first_date,
      COUNT(buyer_id)as new_buyers
FROM (      
  SELECT
          DISTINCT buyer_id,
          MIN(DATE_TRUNC(DATE(transaction_date),MONTH)) OVER(PARTITION BY buyer_id) as first_date
  FROM ferrous-acronym-390114.Transaction_PaDi.transaction_join
  GROUP BY transaction_date, buyer_id) AA
GROUP BY 1 
ORDER BY 1;
```

## Monthly_Active_Buyers
```sql
SELECT DATE_TRUNC(DATE(transaction_date),MONTH)AS month,
       COUNT(DISTINCT buyer_id) AS active_buyers
FROM ferrous-acronym-390114.Transaction_PaDi.transaction_join
WHERE buyer_id IN (SELECT buyer_id
                  FROM (SELECT DISTINCT buyer_id, MIN(transaction_date) AS first_date
                        FROM ferrous-acronym-390114.Transaction_PaDi.transaction_join
                        GROUP BY buyer_id) subquery)
GROUP BY 1
-- HAVING month >= MIN(DATE_TRUNC(DATE(transaction_date),MONTH))
ORDER BY 1 ASC;
```
