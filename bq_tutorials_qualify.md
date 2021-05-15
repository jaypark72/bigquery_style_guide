# QUALIFY 

BigQuery의 `QUALIFY`절은 `HAVING`이 집계함수(aggregate function)의 결과값을 필터링하는 것과 유사하게 분석함수(analytic function)의 결과를 걸러내는 목적으로 사용된다.

- [Produce table](https://cloud.google.com/bigquery/docs/reference/standard-sql/analytic-function-concepts#produce_table)

```sql
SELECT * 
  FROM (
    SELECT item,
           RANK() OVER (PARTITION BY category ORDER BY purchases DESC) as rank
     FROM Produce
    WHERE Produce.category = 'vegetable'
  )
 WHERE rank <= 3; 
```
```sql
 SELECT item,
        RANK() OVER (PARTITION BY category ORDER BY purchases DESC) as rank
   FROM Produce
  WHERE Produce.category = 'vegetable'
QUALIFY rank <= 3
```
