### FROM 절
`FROM` 절에는 테이블 또는 이에 상응하는 서브쿼리가 위치하게 된다. 서브쿼리에 대한 대한 스타일은 다음 장에서 다시 언급하도록 한다.

- `FROM` 절은 `SELECT` 리스트의 다음 라인에 위치하며 테이블 이름은 `FROM` 키워드 다음에 이어서 기술한다.
- `FROM` 키워드는 `SELECT` 키워드에 오른쪽 정렬(right-aligned)이 되도록 위치시킨다.

#### 영구 테이블 이름과 임시 테이블 이름 (_permanent and temporary table name_)
- `FROM` 키워드 다음에 등장하는 테이블 이름으로는 영구 테이블(_permanent table_)과 임시 테이블(_temporary table_) 이름 또는 CTE(_common table expression_) 구문에 의해 정의된 이름을 사용할 수 있다. 
- 영구 테이블 이름은 `FQTN` 형식을 사용하며 임시 테이블이나 CTE 이름은 `snake_case` 형식을 사용한다.
- 임시 테이블이나 CTE 이름은 테이블의 사용목적이 명확히 들어나도록 상세하게 작성한다.

``` sql
-- Permanent Table 
SELECT * 
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`;
```
```sql
-- Temporara Table (ex. CTAS 구문으로 임시테이블 생성)
CREATE TEMP TABLE tmp_table AS SELECT ... ;

SELECT * FROM tmp_table;
```
```sql
-- CTEs - named subquery
WITH cte_name AS ( -- snake_case 규칙에 따라 기술
  SELECT ... -- 왼쪽 두칸 들여쓰기를 한다.
    FROM ... 
)
SELECT * FROM cte_name;
```

#### 서브쿼리 (_subqueries_)
테이블 이름이나 CTE 이름이 서브쿼리를 사용하여 `FROM` 절을 구성할 경우에 아래와 같이 다양한 스타일이 가능하다.  이 가이드에서는 `from_style_#2` 를 사용한 구성을 권장한다.


:disappointed: 기피 (_avoid_)
``` sql
-- 테이블 서브쿼리 스타일 #1 (from_style_#1)
SELECT *
  FROM (SELECT station_id,
               name,
               status
          FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`)
```
:smile: 권장 (_recommend_)
```sql
-- 테이블 서브쿼리 스타일 #2 (from_style_#2)
SELECT *
  FROM (
    SELECT station_id, -- FROM 키워드로부터 두 칸 들여쓰기
           name,
           status,
      FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
  )
```

본 가이드에서 `from_style_#2` 를 선호하는 이유는 아래와 같다. 우선 위의 쿼리를 `JOIN`을 이용하여 확장해 보도록 하자.

``` sql
/* JOIN을 이용하여 쿼리를 확장한 예시 */
-- 테이블 서브쿼리 스타일 #1 (from_style_#1)
SELECT *
  FROM (SELECT station_id,
               name,
               status
          FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`) AS a
  LEFT JOIN (SELECT station_id,
                    name,
                    status
               FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`) AS b
    ON a.station_id = b.station_id
;
-- 테이블 서브쿼리 스타일 #2 (from_style_#2)
SELECT *
  FROM (
    SELECT station_id,
           name,
           status,
      FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
  ) AS a
  LEFT JOIN (
    SELECT station_id,
           name,
           status,
      FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
  ) AS b
    ON a.station_id = b.station_id
;
 ```
`from_style_#1` 은 동일한 수준(level or depth)에 위치한 두 서브쿼리 `a`, `b`의 들여쓰기가 서로 달라져 있다. 반면에 `from_style_#2` 은 두 서브쿼리가 동일한 들여쓰기를 유지하면서 동시에 테이블 별칭(alias)을 라인의 끝이 아닌 앞쪽에 위치함으로써 `ON`절에서 시선을 크게 이동시키지 않고서도 확인 가능하다.

