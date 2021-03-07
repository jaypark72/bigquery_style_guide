# BigQuery Style Guide

#### written by Jaeseok Park (jaeseok.park@gmail.com)

- [BigQuery Style Guide](#bigquery-style-guide)
      - [written by Jaeseok Park (jaeseok.park@gmail.com)](#written-by-jaeseok-park-jaeseokparkgmailcom)
  - [일반 원칙 (_General Principle_)](#일반-원칙-general-principle)
    - [핵심원칙 (_core principle_)](#핵심원칙-core-principle)
    - [기본 스타일 가이드 (_basic style guide_)](#기본-스타일-가이드-basic-style-guide)
    - [예시 코드 구조 (_examples_)](#예시-코드-구조-examples)
  - [이름짓기와 사용하기 (_Naming Conventions_)](#이름짓기와-사용하기-naming-conventions)
    - [이름 기본 가이드](#이름-기본-가이드)
    - [기타 스타일 가이드](#기타-스타일-가이드)
    - [참고 (_references_)](#참고-references)
  - [컬럼 이름 (_column naming_)](#컬럼-이름-column-naming)
      - [컬럼 이름 기본 가이드](#컬럼-이름-기본-가이드)
      - [접두어와 접미어 (_prefix, suffix_)](#접두어와-접미어-prefix-suffix)
      - [컬럼 별칭 (_column_alias_)](#컬럼-별칭-column_alias)
  - [테이블 이름 (_table naming_)](#테이블-이름-table-naming)
  - [문장 구조화 하기 (_Statement Structure_)](#문장-구조화-하기-statement-structure)
    - [SELECT 절](#select-절)
      - [앞선 쉼표와 뒤따르는 쉼표 (_leading comma vs. trailing comma_)](#앞선-쉼표와-뒤따르는-쉼표-leading-comma-vs-trailing-comma)
      - [컬럼 순서 관례 (_column order conventions_)](#컬럼-순서-관례-column-order-conventions)
    - [FROM 절](#from-절)
      - [영속 테이블 이름과 임시 테이블 이름 (_permanent and temporary table name_)](#영속-테이블-이름과-임시-테이블-이름-permanent-and-temporary-table-name)
      - [서브쿼리 (_subqueries_)](#서브쿼리-subqueries)
    - [WHERE 절](#where-절)
      - [논리 연산자의 혼용 (_mixed logical operators_)](#논리-연산자의-혼용-mixed-logical-operators)
    - [GROUP BY 절](#group-by-절)
    - [공통 테이블 표현식 _Common Table Expression, CTEs_](#공통-테이블-표현식-common-table-expression-ctes)
    - [Join 절](#join-절)
  - [Window Function](#window-function)
  - [User Defined Function](#user-defined-function)
  - [참고 (_references_)](#참고-references-1)

이 문서에서는 빅데이터를 다루는 엔지니어(data engineer)와 데이터 분석가(data analyst), 데이터 과학자(data scientist)가 BigQuery 언어로 코드 작성시의 지침(guide)과 스타일(style)을 제시하고 있다.

## 일반 원칙 (_General Principle_)
### 핵심원칙 (_core principle_)
- 읽기 쉽고 유지보수가 용이한 코드를 최우선으로 한다.
- 문장 구조와 코드 스타일의 일관성을 유지하여 가독성이 높은 코드가 되도록 한다.
- 중언부언이나 군더더기 없는 간결한 코드가 되도록 힘쓴다. (reduce duplication and redundancy to make your code succinct)
- 코드로 표현하지 못하는 맥락은 주석을 이용하여 이해를 돕는다.

### 기본 스타일 가이드 (_basic style guide_)
- 줄바꿈과 들여쓰기를 적절히 사용하여 문장이 의미 단위로 읽힐 수 있도록 만든다.
- 문장(statement)의 각 절(clause)은 새로운 라인에서 시작하며 각 절의 시작 키워드가 오른쪽 정렬이 되도록 한다. `DDL`문은 예외적으로 왼쪽 정릴시킨다.
- 줄바꿈이 지나쳐 좁고 길쭉한 (*skinny*) 구조가 되는 경우 연관된 코드 뭉치를 하나의 라인으로 합칠 수 있다. 
- 반대로 한 라인의 길이가 80 글자 내외의 시야 범위를 벗어나는 경우 적절히 줄바꿈을 하여 가로가 지나치게 길어지지 (*flat and wide*) 않도록 라인의 넓이를 유지한다.
- 들여쓰기는 탭이 아닌 공백을 이용하며 2칸 들여쓰기를 기본으로 한다.
- 언어에서 제공하는 장치들을 이용하여 반복을 최소화한다. CTE(Common Table Expression)와 UDF(User Defined Function) 등은 코드의 모듈화를 돕는 장치들로 중복을 제거하는데 유용하다.

### 예시 코드 구조 (_examples_)
SQL 문장은 아래와 같이 세분화할 수 있다. 각 문장에 대해 스타일 가이드를 적용한 예시들을 먼저 살펴보도록 하자.

- `SEL` statement - `SELECT`
- `DDL` statement - `CREATE`, `ALTER`, `DROP`
- `DML` statement - `INSERT`, `UPDATE`, `DELETE`, `TRUNCATE`, `MERGE`
- `DCL` statement - BigQuery에서는 Data Constraint Language 구문를 지원하지 않음.

`SELECT`는 `DML`의 한 종류이지만 다른 `DML` 문과 달리 테이블의 내용을 변경하지 않는다는 점에서 구분되기도 한다.

- `SELECT` statement 예시

```sql
/* SELECT 쿼리 예시 e*/
SELECT station_id,
       name,
       status,
       latitude, longitude,   -- 연관 컬럼들을 하나의 라인에 표기한 예
       ST_DISTANCE (
         ST_GEOGPOINT(longitude, latitude), ST_GEOGPOINT(-0.118092, 51.509865)
       ) AS distance_from_city_centre,
       COUNT(1) AS cnt,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations` s
  LEFT JOIN `bigquery-public-data.austin_bikeshare.bikeshare_trips` t
    ON s.station_id = t.start_station_id
 WHERE s.status IN ('active')
   AND s.latitude > 15.0
 GROUP BY 1, 2, 3, 4, 5
HAVING cnt > 1
 LIMIT 1000
;
```

- `DDL` statement 예시
 
DDL문의 시작 키워드는 예외적으로 왼쪽 정렬을 시킨다. 

```sql
-- https://cloud.google.com/bigquery/docs/reference/standard-sql/data-definition-language#creating_a_new_table

CREATE TABLE my_dataset.new_table (
  x INT64 OPTIONS(description = 'An optional INTEGER field'),
  y STRUCT<
    a ARRAY<STRING> OPTIONS(description = 'A repeated STRING field'),
    b BOOL
  >,
)
PARTITION BY _PARTITIONDATE
OPTIONS(
  expiration_timestamp = TIMESTAMP '2025-01-01 00:00:00 UTC',
  partition_expiration_days = 1,
  description = 'a table that expires in 2025, with each partition living for 24 hours',
  labels=[("org_unit", "development")]
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

- `DML` statement 예시

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
      INSERT(product, quantity, supply_constrained, comments)
      VALUES(product, quantity, true, 
             ARRAY<STRUCT<created DATE, comment STRING>>[
               (DATE('2016-01-01'), 'comment1')
             ])
 WHEN NOT MATCHED THEN
      INSERT(product, quantity, supply_constrained)
      VALUES(product, quantity, false)
```

----
## 이름짓기와 사용하기 (_Naming Conventions_)

### 이름 기본 가이드
이 장에서는 문장을 이루는 가장 기본적인 단위인 키워드와 식별자 이름에 대한 내용을 다룬다.

식별자(_identifier_)는 문장내에서 유일하게 구별되어 인식되는 이름으로 아래 항목들을 포함한다.
1. 언어의 문법구조를 기술하는 토큰(_token_) - ex. `SELECT`, `JOIN`, `FROM`, ...
2. 테이블, 컬럼의 명칭이나 별칭(_alias_)
3. 기본 제공 함수 또는 사용자 정의 함수 이름 - ex. `SUM()`, `STRPOS()`, ...

[키워드(_keyword_)](https://cloud.google.com/bigquery/docs/reference/standard-sql/lexical#reserved_keywords)는 예약된 식별자(_reserved identifier_) 로서 SQL 명세에 의해 그 쓰임이 이미 정해져 있는 식별자를 말한다.

본 가이드에서는 아래 서술된 관례에 따라 이름을 정의하고 표기하도록 한다.
- 키워드는 대문자로 그 외의 식별자는 소문자로 작성하는 것을 권장한다.
- 기본 제공 함수 혹은 내장 함수(_built-in function_)는 대소문자를 구분하지 않으나 대문자로 작성하는 것을 기본으로 한다.
- 사용자 정의 함수(UDF, _user-defined function_)는 소문자를 사용하여 함수 이름을 정의하도록 한다. UDF 이름은 대소문자를 구분한다.
- 여러 단어로 이루어진 식별자는 camelCase이 아닌 snake_case의 형태로 작성토록 한다.

대소문자의 구분은 문장내에서 각 이름의 역할이 드러나도록 하는데 도움을 준다. 대문자 `SELECT`, `FROM` 등의 키워드는 문장의 구조를 표현하는데 주로 사용함으로써 소문자로 표기되는 개체(_entity_)나 속성(_attribute_)의 식별자 이름과 시각적으로 구분되도록 한다. 이는 전체적인 문장 구조의 파악을 용이하게 한다.

내장 함수와 달리 사용자 정의 함수의 이름에 소문자를 사용하는 이유는 사용자 정의 함수를 영속 함수 (_persistent function_) 로 정의하는 경우 데이터셋 이름을 포함한 한정된 함수 이름 (_qualified function name_)을 사용해야 하는데 이때 함수를 한정시키기 위해 사용하는 프로젝트 이름이나 데이터셋의 이름이 소문자로 기술되기 때문에 하나의 의미 단위내에서 대소문자가 혼용되지 않게 하기 위해서이다. 이를 통해 아래에 설명할 `FQTN (Fully Qualified Table Name)` 테이블 이름 규칙과의 일관성도 유지할 수 있다.

```sql
-- 완전 한정된 함수 이름의 예 - fully qualified function name
SELECT bqutils.fn.last_day('2021-03-07')
```

기본 관례에 따른 예시는 아래와 같다.

:smile: 권장 (_recommend_)
```sql
SELECT first_name, last_name FROM my_dataset.my_table
```
:disappointed: 기피 (_avoid_)
```sql
select firstName, lastName from myDataset.myTable
-- OR
SELECT FIRST_NAME, LAST_NAME FROM MY_DATASET.MY_TABLE
```

### 기타 스타일 가이드
- 키워드가 부득이하게 컬럼의 이름으로 사용되는 경우 backtick(`)을 이용하여 감싸준다 (_quoted identifiers_).
```sql
SELECT 'asia-northeast3' AS `from` FROM ...
```
- 문자열 리터럴(_string literal_)은 단일따옴표(')를 사용한다.
- 상수(_constant_)값을 갖는 변수 이름은 대문자로 한다.

```sql
DECLARE START_DATE DATE DEFAULT '2021-02-19'
```

### 참고 (_references_)
- [Lexical structure and syntax in Standard SQL](https://cloud.google.com/bigquery/docs/reference/standard-sql/lexical)

----
## 컬럼 이름 (_column naming_)

#### 컬럼 이름 기본 가이드
컬럼이름은 `camelCase`가 아닌 소문자를 이용하여 `snake_case` 로 표현하며 [헝가리언 표기](https://en.wikipedia.org/wiki/Hungarian_notation) 나 의미를 유추하기 힘든 지나치게 축약된 접두어(_prefix_), 접미어(_suffix_)의 사용은 자제한다.

기준정보와 같이 팀내에서 표준으로 관리되는 속성들이 있는 경우에는 해당 표준에서 정의된 명명규칙을 우선한다. 예를 들어, 국가코드등 기준정보로 사용되는 컬럼의 이름은 표준에서 정의된 명명 규칙을 차용해서 사용하도록 한다.
```sql
SELECT cnty_cd, mcc, mnc, FROM ...
```

배열형이나 복수의 의미를 지니는 컬럼인 경우는 이름을 복수형으로 만든다. 

```sql
SELECT hits,          -- ARRAY<STRUCT<>> 형
       totals.visits, -- INT64형, 총 세션 횟수
       totals.hits,   -- INT64형, 세션내 조회수
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801` 
;
```

#### 접두어와 접미어 (_prefix, suffix_)
접두어나 접미어는 이름에 부가적인 의미(semantics)를 부여하기 위한 목적으로 사용 가능하나 지나치게 축약된 약어나 과도한 사용은 오히려 가독성을 떨어뜨릴 수 있어 사용에 신중해야 한다.

아래는 몇가지 사용 가능한 예시이다.

- Boolean 형의 경우 `is_`, `has_`, 또는 `does_`의 접두어를 사용할 수 있다.
- Date형은 `_dt` 접미어를 붙일 수 있다.
- DateTime이나 Timestamp형은 `_at` 접미어가 가능하다.
- `user_id`, `device_id` 처럼 식별자(_id_) 접미어의 사용이 가능하다.

```sql
SELECT is_active,
       registered_dt,
       updated_at,
  FROM ...       
```

#### 컬럼 별칭 (_column_alias_)
컬럼의 별칭 이름은 컬럼 이름 규칙에 준하여 작성한다. 별칭은 목적에 따라서 컬럼 이름보다 상세히 작성되는 경우도 있고 반대로 간략하게 축약된 형태로 사용하기도 한다. 

아래와 같이 컬럼의 의미를 명확하 하기 위해서 긴 이름의 별칭을 사용할 수 있다.

```sql
SELECT fname AS first_name,
       lname AS last_name,  
  FROM ...
```

반면에 컬럼 이름이 유도된 컬럼(_derived column_) 또는 계산된 컬럼(_calculated column_)의 연산식에 반복적으로 사용되어지는 경우는 간결함을 위해서 축약된 이름을 사용할 수 있다.

```sql
WITH locations AS (
  SELECT x_position,
         y_position,
         city_centre_x_position,
         city_center_y_position,
    FROM ...
)
SELECT SQRT(
         (city_center_x_position - x_position) * (city_center_x_position - x_position) +
         (city_center_y_position - y_position) * (city_center_y_position - y_position)
       ) AS distince,  -- calculated(or derived) column
  FROM locations
```
위와 같이 기본 컬럼 (_base column_) 으로부터 새로운 컬럼을 파생시키는 연산식에서 컬럼 이름이 반복적으로 등장하여 라인이 길어지는 경우 아래와 같이 간결하게 연산식이 표현되도록 축약된 별칭을 사용하는 것이 가능하다.

```sql
WITH location AS (
  SELECT x_position AS x,
         y_position AS y,
         city_centre_x_position AS cx,
         city_center_y_position AS cy,
    FROM ...
)
SELECT SQRT((cx - x) * (cx - x) + (cy - y) * (cy - y)) AS distince,
  FROM location
```

----
## 테이블 이름 (_table naming_)

기본적으로는 소문자를 사용하되 팀내 과제별 명명 규칙이 별도로 정해져 있는 경우 그것을 우선한다. 테이블 이름은 대소문자를 구분한다.

- dash(-)가 아닌 underscore(_)를 사용하여 snake_case 이름이 되도록 한다.
- 과제별 명명규칙에 의해 이미 생성되어 운영되고 있는 데이터셋과 테이블 이름들은 가이드의 권고를 따르지 않더라도 As-Is 이름을 그대로 사용토록 한다.

■ Fully Qualified Table Name (FQTN)

> '_완전하게 한정된 테이블 이름_ 또는 _정규화된 테이블 이름_'의 의미는 그 이름을 가지고 실행컨텍스트와 상관없이 유일하게 테이블를 특정할 수 있다는 말이다.
> 예를 들어,  `customer.devices` 라는 _일부 한정된 테이블 이름_ 을 사용하는 경우 쿼리를 실행시키는 프로젝트에 따라서 서로 다른 `customer.devices` 테이블에 접근이 될 수 있다.

BigQuery로 DWH(Data Ware House)를 구축할 경우 데이터가 저장된 프로젝트와 BigQuery Job을 실행시켜주는 프로젝트가 다를 수 있기 때문에 실행컨텍스트의 영향을 받지 않도록 정규화된 형태의 테이블 이름을 사용하도록 한다. 

이외에, 단일 문장(_single statement_)의 쿼리를 실행하는 경우가 아닌 BigQuery 스크립트로 복수의 문장(_multiple statements_)을 실행하는 경우에는 GCP Project 이름에 포함된 dash(-)가 뺄셈 등의 다른 의미로 해석이 되지 않도록 _정규화된 테이블 이름_ 을 backtick(`)을 사용하여 감싸준다. 

이때 dash가 포함된 프로젝트 이름만 backtick으로 감싸주어도 오류가 발생하지는 않으나 FQTN 전체를 backtick을 감싸주어 의미적으로 하나의 단위임을 명시적으로 나타내도록 한다. 

:smile: 권장 (_recommend_)
```sql
SELECT * FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```
:disappointed: 기피 (_avoid_)
```sql
SELECT * FROM `bigquery-public-data`.austin_bikeshare.bikeshare_stations
```

----
## 문장 구조화 하기 (_Statement Structure_)

### SELECT 절
가장 기본적인 문장인 `SELECT` 문은 아래의 가이드를 따른다.
- `SELECT *` 로 전체 컬럼을 가져오기 보다는 필요한 컬럼만을 `SELECT` 리스트에 기술하도록 한다.
- `SELECT` 리스트에 등장하는 컬럼이 복수개일 경우 한 라인에 하나씩 작성하는 걸을 기본으로 한다.
```sql
SELECT station_id,
       name,
       status,
       latitude,
       longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```
- 의미적으로 관련이 있거나 시야 범위내에서 한 눈에 읽히고 이해될 수 있다고 하면 80자를 넘지 않는 선에서 하나의 라인에 복수의 컬럼들을 기술할 수 있다.
- 주석을 사용하여 컬럼의 의미를 설명할 수 있다.
```sql
SELECT station_id,
       name,
       status,
       latitude, longitude,  -- 하나의 컬럼 뭉치(Column Family)로 의미적으로 묶임
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
;
-- 또는
SELECT station_id, name, status, latitude, longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
;
```
- 컬럼 별칭(alias)을 포함하는 경우에도 상기 내용들이 동일하게 적용된다.
- 컬럼 별칭은 `AS` 키워드 다음에 위치시키며 별칭의 시작 위치는 별도로 정렬시키지 않는다.

:smile: 권장 (_recommend_)
```sql
SELECT station_id AS station_id,
       name AS station_name,
       status AS bike_status,
       latitude,
       longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```
:disappointed: 기피 (_avoid_)
```sql
SELECT station_id         AS station_id,
       name               AS station_name,
       status             AS bike_status,
       latitude,
       longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```

#### 앞선 쉼표와 뒤따르는 쉼표 (_leading comma vs. trailing comma_)
BigQuery에서는 `SELECT` 리스트의 마지막 컬럼 뒤에 콤마(,)를 허용하기 때문에 본 가이드에서는 뒤따르는 쉼표를 사용한다.

:smile: 권장 _recommend_
```sql
SELECT station_id,
       name,
       status,
       latitude,
       longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```
:disappointed: 기피 _avoid_
```sql
SELECT station_id
     , name
     , status
     , latitude
     , longitude
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```

뒤따르는 쉼표를 지원하지 않는 SQL 언어에서는 앞선 쉼표를 사용해서 SELECT 리스트에 새로운 컬럼을 추가해도  직전 라인에 영향을 주지 않도록 하였으나 BigQuery에서는 뒤따르는 쉼표를 지원하므로 이를 기본 스타일로 삼는다.
```sql
-- 뒤따르는 쉼표 사용
SELECT station_id,
       name
  FROM ...

-- 컬럼 추가시
SELECT station_id,
       name,    -- 아래 때문에 comma가 추가됨.
       status   -- 신규컬럼 추가
  FROM ...
```

```sql
-- 앞선 쉼표 사용
SELECT station_id
     , name
  FROM ...

-- 컬럼 추가시
SELECT station_id
     , name     -- 아래에 의해 변경되는 내용이 없음
     , status   -- 신규컬럼 추가
  FROM ...
```

#### 컬럼 순서 관례 (_column order conventions_)
- 기본키(PK, Primary Key)에 해당하는 컬럼명을 먼저 위치시킨다.
- (optional) 파티션된 테이블 사용시 파티션 컬럼명은 마지막에 위치시킨다. 운영중에 스키마 변경으로 인한 컬럼 추가시는 예외가 된다.
- `GROUP BY` 를 동반한 집계 쿼리에서는 차원(dimension)에 해당하는 컬럼명을 먼저 나열한 후 지표(metric)에 해당하는 컬럼명을 기술한다.

### FROM 절
`FROM` 절에는 테이블 또는 이에 상응하는 서브쿼리가 위치하게 된다. 서브쿼리에 대한 대한 스타일은 추후 다시 언급하도록 한다.

- `FROM` 절은 `SELECT` 리스트의 다음 라인에 위치하며 [테이블이름](#테이블-이름-(_table-naming_)) 은 `FROM` 키워드 다음에 이어서 기술한다.
- `FROM` 키워드는 `SELECT` 키워드에 오른쪽 정렬(right-align)이 되도록 위치시킨다.

#### 영속 테이블 이름과 임시 테이블 이름 (_permanent and temporary table name_)
- 테이블 이름으로는 영속 테이블(_permanent table_)과 임시 테이블(_temporary table_) 또는 CTE(_common table expression_) 구문에 의해 정의된 이름을 사용할 수 있다. 
- 영속 테이블 이름은 FQTN 형식을 사용하며 임시 테이블이나 CTE 이름은 `snake_case` 형식을 사용한다.
- 임시테이블이나 CTE이름은 테이블의 사용목적이 명확히 들어나도록 상세하게 작성한다.

``` sql
-- Permanent Table 
SELECT * FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`;
```
```sql
-- Temporara Table (ex. CTAS 구문으로 임시테이블 생성)
CREATE TEMP TABLE tmp_table AS SELECT ... ;
SELECT * FROM tmp_table;
```
```sql
-- CTE - named subquery
WITH cte_name AS ( -- snake_case 규칙에 따라 기술
  SELECT ...
)
SELECT * FROM cte_name;
```

#### 서브쿼리 (_subqueries_)
영속 테이블(Permanent Table) 이 아닌 서브쿼리를 사용하여 `FROM` 절을 구성할 경우에 아래와 같이 다양한 스타일이 가능하다.  이 가이드에서는 `from_style_#2` 를 사용한 구성을 권장한다.

``` sql
-- 테이블 서브쿼리 스타일 #1 (from_style_#1)
SELECT *
  FROM (SELECT station_id,
               name,
               status
          FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`)

-- 테이블 서브쿼리 스타일 #2 (from_style_#2)
SELECT *
  FROM (
    SELECT station_id,
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
`from_style_#1` 은 동일한 수준(level or depth)에 위치한 두 서브쿼리 `a`, `b`의 들여쓰기가 서로 달라져 있다. 반면에 `from_style_#2` 은 두 서브쿼리가 동일한 들여쓰기를 유지하면서 동시에 테이블 별칭(alias)이 라인의 끝에 위치하지 않고 앞쪽에 위치함으로써 `ON`절에서 시선을 크게 이동시키지 않고서도 확인 가능하다.

### WHERE 절
`WHERE` 절에는 결과셋(result set)에 남겨질 열(row)들을 선택할 조건을 기술한다. `WHERE` 절의 조건은 아래의 내용에 따라 작성한다.

- 비교 연산자의 앞 뒤로 공백을 두도록 한다.
- 비동등연산자는 `!=`와 `<>` 모두 사용가능하다. [SQL-1999](http://web.cecs.pdx.edu/~len/sql1999.pdf)에서는 `<>`를 표준으로 정의하고 있다.
- 조건절은 `AND`와 `OR` 논리 연산자에 의해 복수개가 연결(_chaining_)될 수 있다. 
- 복수개의 조건을 기술하는 경우는 하나의 라인에 하나의 조건이 기술될 수 있도록 한다.
- `BETWEEN`이나 `IN` 연산자를 사용하여 복수개의 조건을 묶을 수 있는 경우는 연산자를 사용하여 조건을 간결하게 표현한다.

```sql
SELECT station_id,
       name,
       status,
       latitude, longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
 WHERE status = 'active'
   AND latitude > 135.0   -- AND 로 조건 연결 
;
```

#### 논리 연산자의 혼용 (_mixed logical operators_)
테이블 조회시 `AND`와 `OR`를 혼용하여 사용하는 경우에는 아래의 가이드에 따라 코드를 구조화한다.

1. 주어진 테이블에서 `AND` 조건으로 결과셋(_result set_)을 줄여나가는지 반대로 `OR` 조건으로 대상을 늘려나가는지를 보고 최상위 조건절의 구조를 먼저 파악한다.
2. 각 상위 조건들이 다시 세부적인 조건으로 나뉘는 경우에 괄호를 이용하여 해당 세부조건을 묶는다. 괄호를 통해서 연산자 우선순위에 따른 부작용을 없앤다.
3. 세부 조건에 등장하는 `AND` 또는 `OR` 연산자는 라인의 끝에 위치시키도록 한다.
4. 논리 연산을 통해 3 단계 이상의 깊이에서 논리연산이 이루어지는 경우 분배 법칙을 이용하여 깊이를 낮출 수 있다.

```sql
-- 최상위 조건들을 AND로 묶고 세부조건이 OR로 묶이는 경우
SELECT station_id,
       name,
       status,
       latitude, longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
 WHERE status = 'active'
   AND (latitude > 135.0 OR
        longitude > 80.0) 
   AND ...
;
-- 또는, 최상위 조건들을 OR로 묶고 세부조건을 AND로 묶는 경우
SELECT station_id,
       name,
       status,
       latitude, longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
 WHERE status = 'active'
    OR (latitude > 135.0 AND longitude > 80.0) -- 의미단위로 묶어서 한 줄에 표현
    OR ...    
```
```sql
SELECT *
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
 WHERE status = 'active'
    OR (latitude > 135.0 AND (longitude = 80.0 OR longitude = 90.0))
    OR ...
-- 위에서와 같이 3 단계 이상의 깊이에서 논리연산 이루어지는 경우 분배법칙을 이용하여 아래와 같이 결과적으로 동치인 문장으로 풀어쓸 수 있다. a · (b + c) = (a · b) + (a · c)
 WHERE status = 'active'
    OR (latitude > 135.0 AND longitude = 80.0)
    OR (latitude > 135.0 AND longitude = 90.0)
    OR ...
```

### GROUP BY 절

- 집계함수에 의해서 만들어지는 지표(metric) 컬럼은 별칭(alias)를 사용하여 반드시 이름을 부여한다.
- `GROUP BY` 리스트에는 컬럼의 별칭이나 컬럼 순서(ordinal position)를 사용할 수 있으므로 컬럼이름을 반복적으로 사용하기 보다는 컬럼 순서를 사용하도록 한다.
- 특히, 기본컬럼에서 `CASE`문 등으로 유도된 컬럼(derived column)이 차원으로 사용되는 경우에는 반드시 별칭이나 컬럼 순서를 사용하여 코드가 반복되어 사용되지 않도록 한다.ss
- `HAVING` 절에서는 컬럼 별칭 사용이 가능하다.

```sql
SELECT station_id,
       name AS station_name,
       status,
       CASE
         WHEN ST_DISTANCE (
                ST_GEOGPOINT(longitude, latitude), 
                ST_GEOGPOINT(-0.118092, 51.509865)
              ) < 7000000 THEN 'downtown'
         ELSE 'outskirt'
       END AS downtown_or_outskirt,
       COUNT(1) AS cnt,    -- 집계함수에 의해 유도된 metric 컬럼에 별칭 부여
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
 WHERE s.status IN ('active')
   AND s.latitude > 135.0
 GROUP BY 1, 2, 3
HAVING cnt > 1
;
```

아래 예시처럼 `SELECT` 리스트에 유도된 컬럼을 정의하고 해당 컬럼을 차원으로 `GROUP BY` 집계를 하는 경우 `GROUP BY` 리스트에서 코드가 반복되어 문장이 길어지고 향후 유지보수가 어려워지게 된다.

:smile: 권장 (_recommend_)
```sql
SELECT CASE
         WHEN is_male = TRUE THEN 'Male'
         WHEN is_male = FALSE THEN 'Female'
       END AS gender,
       COUNT(1) AS cnt,
  FROM `bigquery-public-data.samples.natality`
 GROUP BY 1
HAVING cnt > 1000
; 
```

:disappointed: 기피 (_avoid_)
```sql
SELECT CASE
         WHEN is_male = TRUE THEN 'Male'
         WHEN is_male = FALSE THEN 'Female'
       END,
       COUNT(1) AS cnt,
  FROM `bigquery-public-data.samples.natality`
 GROUP BY CASE
            WHEN is_male = TRUE THEN 'Male'
            WHEN is_male = FALSE THEN 'Female'
          END
;
```

### 공통 테이블 표현식 _Common Table Expression, CTEs_
Common Table Expression 은 쿼리에 의한 임시 결과셋(result set)에 이름을 부여하여 다른 곳에서 참조할 수 있도록 한 표현방식이다. 다시 말해 서브쿼리의 결과셋에 이름을 부여한 Named 서브쿼리라 할 수 있다. CTE는 복잡한 쿼리 문장을 간결하게 구조화하는데 핵심적인 역할을 한다.

CTEs와 관련한 기본적인 가이드는 2단계 이상 중첩되는 서브쿼리가 존재하는 경우에는 무조건 CTE 구문으로 바꾸고 CTE에 부여한 이름을 사용하여 전체 쿼리 구조가 적절한 깊이(2 depth)를 유지 하도록 한다.

아래는 CTEs를 이용하여 유도된 컬럼이 반복되지 않도록 구조를 변경한 예시이다. 중복은 코드 길이를 증가시키고 유지보수를 어렵게 하기 때문에 서브쿼리나 CTE를 이용하여 코드가 반복되지 않도록 만드는 것이 좋다.

:disappointed: 기피 (_avoid_)
```sql
SELECT name AS station_name,
       ST_DISTINCE (
         ST_GEOPOINT(longitude, latitude), ST_GEOPOINT(-0.118092, 51.509865)
       ) AS distance_from_city_centre_m
  FROM `bigquery-public-data.london_bicycles.cycle_stations`
 WHERE ST_DISTINCE (
         ST_GEOPOINT(longitude, latitude),
         ST_GEOPOINT(-0.118092, 51.509865)
       ) <= 500
 ORDER BY distance_from_city_centre_m
```

:smile: 권장 (_recommend_)
```sql
WITH stations (
  SELECT name AS station_name,
         ST_DISTINCE (
           ST_GEOPOINT(longitude, latitude), ST_GEOPOINT(-0.118092, 51.509865)
         ) AS distance_from_city_centre_m
    FROM `bigquery-public-data.london_bicycles.cycle_stations`
)
SELECT * 
  FROM stations
 WHERE distance_from_city_centre_m <= 500
 ORDER BY distance_from_city_centre_m
```
`FROM`절에 직접 서브쿼리를 사용하는 경우 아래쪽에 위치한 서브쿼리를 먼저 읽고 다시 위쪽의 쿼리를 읽어야 하는 반면, CTE를 사용하는 경우는 위에서부터 자연스럽게 아래로 코드 리딩이 가능한 장점이 있다.


2번 이상 중첩된 서브쿼리를 가진 경우에도 CTE를 사용하여 코드의 깊이를 한 단계로 유지하면서 단계별 중간 테이블을 만들어낼 수 있다.

:disappointed: 기피 _avoid_

```sql
SELECT *
  FROM (
    SELECT * 
      FROM (
        SELECT * 
          FROM base_table
      ) AS first_table   -- 설명을 위해서 별칭 사용
  ) AS second_table 
```

:smile: 권장 _recommend_

```sql
WITH first_table (  -- 가장 안쪽의 서브쿼리
SELECT * 
  FROM base_table
),
second_table ( -- 상위 서브쿼리
SELECT *
  FROM first_table
)
SELECT *
  FROM second_table
;
```

### Join 절

- `INNER`와 `OUTER` 키워드는 생략되더라도 의미적으로 혼동을 주지 않기 때문에 생략한다. 내부조인은 `JOIN`, 외부조인은 `LEFT JOIN`, `OUTER JOIN`, Cartesian Product는 `CROSS JOIN`을 사용한다.
- `ON` 절은 `JOIN`절 바로 아래에 작성하며 결합조건이 여러 개 등장하는 경우는 `WHERE` 절을 참고하여 복수개의 결합 조건을 기술한다.
- 테이블 별칭을 사용하여 긴 테이블 이름이 반복되지 않도록 한다.

:disappointed: 기피 (_avoid_)
```sql
SELECT m.*,
       d.*,
  FROM masters
  LEFT OUTER JOIN details
    ON masters.id = details.id
```

:smile: 권장 (_recommend_)
```sql
SELECT m.*,
       d.*,
  FROM masters AS m
  LEFT JOIN details AS d
    ON m.id = d.id
   AND m.status = d.status
```

아래는 지금까지의 가이드에 따라 실제 분석용 쿼리를 구조화한 예시이다. 쿼리는 아래 링크의 내용을 사용하였다.
- https://cloud.google.com/life-sciences/docs/how-tos/interval-joins

```sql
#standardSQL
WITH variants AS (
  -- Retrieve the variants in this cohort, flattening by alternate bases and
  -- counting affected alleles.
  SELECT REPLACE(reference_name, 'chr', '') as reference_name,
         start_position,
         end_position,
         reference_bases,
         alternate_bases.alt AS alt,
         (SELECT COUNTIF(gt = alt_offset+1) FROM v.call, call.genotype gt) AS num_variant_alleles,
         (SELECT COUNTIF(gt >= 0) FROM v.call, call.genotype gt) AS total_num_alleles,
    FROM `bigquery-public-data.human_genome_variants.platinum_genomes_deepvariant_variants_20180823` v,
         UNNEST(alternate_bases) alternate_bases WITH OFFSET alt_offset
),
intervals AS (
  -- Define an inline table that uses five rows
  -- selected from silver-wall-555.TuteTable.hg19.
  SELECT * 
    FROM UNNEST([
           STRUCT<Gene STRING, Chr STRING, 
                  gene_start INT64, gene_end INT64,
                  region_start INT64, region_end INT64>
           ('PRCC',  '1', 156736274, 156771607, 156636274, 156871607),
           ('NTRK1', '1', 156785541, 156852640, 156685541, 156952640),
           ('PAX8',  '2', 113972574, 114037496, 113872574, 114137496),
           ('FHIT',  '3',  59734036,  61238131,  59634036,  61338131),
           ('PPARG', '3',  12328349,  12476853,  12228349,  12576853)
         ])
),
gene_variants AS (
  --
  -- JOIN the variants with the genomic intervals overlapping
  -- the genes of interest.
  --
  -- The JOIN criteria is complicated because the task is to see if
  -- an SNP overlaps an interval.  With standard SQL you can use complex
  -- JOIN predicates, including arbitrary expressions.
  SELECT reference_name,
         start_position,
         reference_bases,
         alt,
         num_variant_alleles,
         total_num_alleles,
    FROM variants v
    JOIN intervals i
      ON v.reference_name = i.Chr
     AND i.region_start <= v.start_position
     AND i.region_end >= v.end_position 
)
--
-- And finally JOIN the variants in the regions of interest
-- with annotations for rare variants.
SELECT DISTINCT 
       Chr,
       annots.Start AS Start,
       Ref,
       annots.Alt,
       Func,
       Gene,
       PopFreqMax,
       ExonicFunc,
       num_variant_alleles,
       total_num_alleles,
  FROM `silver-wall-555.TuteTable.hg19` AS annots
  JOIN gene_variants AS vars
    ON vars.reference_name = annots.Chr
   AND vars.start_position = annots.Start
   AND vars.reference_bases = annots.Ref
   AND vars.alt = annots.Alt
 WHERE PopFreqMax <= 0.01 -- Retrieve annotations for rare variants only.
 ORDER BY Chr, Start
;
```
----
## Window Function
`TBD`

## User Defined Function
`TBD`

---
## 참고 (_references_)

- [SQL Style Guide](https://www.sqlstyle.guide/)
- [Mazur's SQL Style Guide](https://github.com/mattm/sql-style-guide)
- [Kickstarter SQL Style Guide](https://gist.github.com/fredbenenson/7bb92718e19138c20591)
- [GitLab SQL Style Guide](https://about.gitlab.com/handbook/business-ops/data-team/platform/sql-style-guide/)
