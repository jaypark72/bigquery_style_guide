## SELECT 컬럼 리스트
SQL의 가장 기본적인 `SELECT` 문장의 컬럼 리스트는 아래의 지침에 따라 작성한다.
- `SELECT *` 로 전체 컬럼을 가져오기 보다는 필요한 컬럼만을 `SELECT` 리스트에 열거하도록 한다.
- `SELECT` 리스트에 등장하는 컬럼이 복수개일 경우 한 라인에 하나씩 작성하는 것을 기본으로 한다.
-
```sql
SELECT station_id,
       name,
       status,
       latitude,
       longitude,
  FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```
- 의미적으로 관련이 있거나 시야 범위내에서 한 눈에 읽히고 이해될 수 있다면 80자를 넘지 않는 선에서 하나의 라인에 복수의 컬럼들을 기술할 수 있다.
- 주석을 사용하여 컬럼의 의미를 부연 설명할 수 있다.
- 마지막 컬럼명의 뒤에 콤마(,)를 추가한다.

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
- 컬럼 별칭(alias)을 포함하는 경우에도 위의 내용들이 동일하게 적용된다.
- 컬럼 별칭은 `AS` 키워드 다음에 위치시키며 별칭의 시작 위치는 따로 정렬시키지 않는다.

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
       longitude,  -- 마지막 컬럼 뒤 콤마(,) 추가
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

뒤따르는 쉼표를 지원하지 않는 SQL 언어에서는 앞선 쉼표를 사용해서 SELECT 리스트에 새로운 컬럼을 추가해도 직전 라인에 영향을 주지 않도록 하였으나 BigQuery에서는 뒤따르는 쉼표를 지원하므로 이를 기본 스타일로 삼는다.

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
     , name     -- 아래 컬럼 추가로 변경되는 내용이 없음
     , status   -- 신규컬럼 추가
  FROM ...
```

이외에 범용 프로그래밍 언어나 동적(_dynamic_) SQL을 사용하여 `SELECT` 문의 컬럼 리스트를 자동 생성할 때 뒤따르는 쉼표를 사용할 경우 리스트의 마지막 컬럼에 쉼표가 생략되도록 예외 처리할 필요가 없어 구현 로직이 단순해지는 장점이 있다.


#### 컬럼 순서 관례 (_column order conventions_)
- 기본키(PK, Primary Key)에 해당하는 컬럼명을 먼저 위치시킨다.
- (optional) 파티션된 테이블 사용시 파티션 컬럼명은 마지막에 위치시킨다. 운영중에 스키마 변경으로 인한 컬럼 추가시는 예외가 된다.
- `GROUP BY` 를 동반한 집계 쿼리에서는 차원(dimension)에 해당하는 컬럼명을 먼저 나열한 후 지표(metric)에 해당하는 컬럼명을 이어 기술한다.
