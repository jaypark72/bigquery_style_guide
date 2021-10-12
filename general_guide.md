# 일반 지침 (_General Guide_)

본 문서에서는 빅데이터를 다루는 엔지니어, 데이터 분석가와 데이터 과학자 등이 BigQuery 언어로 코드 작성시 참고할 수 있는 지침과 스타일을 제시한다.

## 핵심원칙 (_core principle_)
- 코드 스타일의 일관성을 유지하여 가독성이 높은 코드가 되도록 한다.
- 유지보수가 용이하도록 문장 구조를 유연하게 구하도록 한다.
- 중언부언이나 군더더기 없는 간결한 코드가 되도록 힘쓴다.
- 코드만으로 표현하지 못하는 맥락은 주석을 추가하여 이해를 돕는다.

> [주의] 본 지침은 BigQuery Scripting 이나 BigQuery ML 등 표준 SQL을 넘어선 BigQuery 확장 문법까지 고려되었으므로 일부 스타일은 타 SQL 구현체에 어울리지 않을 수 있다.

## 공통 스타일 가이드 (_common style guide_)
- 줄바꿈과 들여쓰기를 적절히 사용하여 문장이 의미 단위로 읽힐 수 있도록 만든다.
- 들여쓰기는 탭이 아닌 공백을 이용하며 2칸 들여쓰기를 기본으로 한다.
- 문장(statement)의 각 절(clause)은 새로운 라인에서 시작하며 각 절의 시작 키워드가 오른쪽 정렬이 되도록 한다.
  - 오른쪽 정렬된 단어들의 세로 기준선을 따라 왼편에는 주로 문장의 구조를 파악하는데 필요한 단어들이 등장하며 오른편에는 비즈니스 로직이 나타난다.
  ![Styling Concept](images/styling_concept.png)

  - `DDL`(Data Definition Language) 문의 시작 키워드는 예외적으로 왼쪽 정렬시킨다. `DDL`문을 시작하는 `CREATE`나 `ALTER`보다 길이가 긴 단어들이 이어서 등장하기 때문이지만 `DDL`문의 구조가 단순하여 전체적인 스타일의 일관성을 해치지는 않는다. 
- 줄바꿈이 지나쳐 좁고 길쭉한 (*skinny*) 구조가 되는 경우 연관된 코드 뭉치를 하나의 라인으로 합칠 수 있다. 
- 반대로 한 라인의 길이가 80 글자 내외의 시야 범위를 넘어서는 경우 가로로 지나치게 길어지지 (*flat and wide*) 않도록 줄바꿈을 통해 적절한 폭을 유지한다.
- 언어에서 제공하는 장치들을 이용하여 반복을 최소화한다. **공통 테이블 표현식** `CTE(Common Table Expression)`와 **사용자 정의 함수** `UDF(User Defined Function)` 등은 코드의 모듈화를 돕는 장치들로 중복을 제거하는데 유용하다.


## 스타일 가이드를 적용한 예시 (_examples_)
SQL 문장은 아래와 같이 세분화할 수 있다. 각 문장에 대해 스타일 가이드를 적용한 예시들을 몇가지 살펴보도록 하자.

- `SEL` statement - `SELECT`
- `DDL`(Data Definition Language) statement - `CREATE`, `ALTER`, `DROP`
- `DML`(Data Manipulation Language) statement - `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `MERGE`
- `DCL`(Data Constraint Language) statement - BigQuery에서는 Data Constraint Language 구문를 지원하지 않는다.

`SELECT`는 `DML`의 한 종류이지만 다른 `DML` 문과 달리 테이블의 내용을 변경하지 않는다는 점에서 구분되기도 한다.

### `SELECT` statement 예시

```sql
/* 각 절의 시작 키워드인 SELECT, FROM, LEFT, WHERE, GROUP 등을 오른쪽 정렬 */
SELECT station_id,
       name,
       status,
       latitude, longitude,  -- 연관 컬럼들을 같은 라인에 나열하는 것도 가능
       ST_DISTANCE(
         ST_GEOGPOINT(longitude, latitude), ST_GEOGPOINT(-0.118092, 51.509865)
       ) AS distance_from_city_centre,
       COUNT(1) AS cnt,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations` s
  LEFT JOIN `bigquery-public-data.austin_bikeshare.bikeshare_trips` t
    ON s.station_id = t.start_station_id
 WHERE s.status IN ('active') AND s.latitude > 15.0
 GROUP BY 1, 2, 3, 4, 5
HAVING cnt > 1
 LIMIT 1000
;
```

### `DDL` statement 예시
 
`DDL`문은 `SEL` 문장과 달리 시작 키워드를 왼쪽으로 정렬을 시킨다. 

```sql
-- https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#creating_a_new_table

CREATE TABLE my_dataset.new_table (
  x INT64           OPTIONS (description = 'An optional INTEGER field'),
  y STRUCT<
    a ARRAY<STRING>   OPTIONS (description = 'A repeated STRING field'),
    b BOOL            OPTIONS (description = 'A BOOL field')
  >                 OPTIONS (description = 'A struct field'),
)
PARTITION BY _PARTITIONDATE
OPTIONS (
  expiration_timestamp = TIMESTAMP '2025-01-01 00:00:00 UTC',
  partition_expiration_days = 1,
  description = 'a table that expires in 2025, with each partition living for 24 hours',
  labels = [("org_unit", "development")]
)
;
```
```sql
-- https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#examples_6

ALTER TABLE mydataset.mytable 
SET OPTIONS (
  expiration_timestamp = TIMESTAMP_ADD(CURRENT_TIMESTAMP(), INTERVAL 7 DAY),
  description = "Table that expires seven days from now"
)
;
``` 

### `DML` statement 예시

```sql
-- https://cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax#insert_using_explicit_values

INSERT dataset.Inventory (
         product,
         quantity
       )
VALUES ('top load washer', 10),
       ('front load washer', 20),
       ('dryer', 30),
       ('refrigerator', 10),
       ('microwave', 20),
       ('dishwasher', 30),
       ('oven', 5)
;
```
```sql
-- https://cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax#update_using_joins

UPDATE dataset.Inventory
   SET quantity = quantity + 
                  (SELECT quantity 
                     FROM dataset.NewArrivals
                    WHERE Inventory.product = NewArrivals.product),
       supply_constrained = false
 WHERE product IN (SELECT product FROM dataset.NewArrivals)
;

UPDATE dataset.Inventory i
   SET quantity = i.quantity + n.quantity,
       supply_constrained = false
  FROM dataset.NewArrivals n
 WHERE i.product = n.product
;
```

```sql
-- https://cloud.google.com/bigquery/docs/reference/standard-sql/dml-syntax#merge_examples

MERGE dataset.DetailedInventory T
USING dataset.Inventory S
   ON T.product = S.product
 WHEN NOT MATCHED AND quantity < 20 THEN 
      INSERT (product, quantity, supply_constrained, comments)
      VALUES (
        product, quantity, true, 
        ARRAY<STRUCT<created DATE, comment STRING>>[
          (DATE('2016-01-01'), 'comment1')
        ]
      )
 WHEN NOT MATCHED THEN
      INSERT (product, quantity, supply_constrained)
      VALUES (product, quantity, false)
```

### BigQuery ML Example

```sql
# Model Training
CREATE OR REPLACE MODEL `bqml_tutorial.penguins_model`
OPTIONS ( 
  model_type = 'linear_reg',
  input_label_cols = ['body_mass_g']
) AS
WITH train AS (
  SELECT *
    FROM `bigquery-public-data.ml_datasets.penguins`
   WHERE body_mass_g IS NOT NULL
)
SELECT * FROM train
;

# Model Evaluation
WITH train AS (
  SELECT *  
    FROM `bigquery-public-data.ml_datasets.penguins` 
   WHERE body_mass_g IS NOT NULL
)
SELECT * FROM ML.EVALUATE(MODEL `bqml_tutorial.penguins_model`, TABLE train)
;

# Prediction
WITH test AS (
  SELECT * 
    FROM `bigquery-public-data.ml_datasets.penguins`
   WHERE body_mass_g IS NOT NULL
     AND island = "Biscoe"
)
SELECT * FROM ML.PREDICT(MODEL `bqml_tutorial.penguins_model`, TABLE test)
;
```