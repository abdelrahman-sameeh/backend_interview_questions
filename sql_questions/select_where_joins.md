sql order queries
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT

```sql
  SELECT customer_name, COUNT(*) as 'orders_count', SUM(price) as 'total_spent'
  from orders
  where price > 100
  GROUP BY customer_name
  HAVING COUNT(*) > 2
  ORDER BY total_spent DESC
  LIMIT 5;
```
