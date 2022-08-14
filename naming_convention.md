# 이름짓기와 사용하기 (_Naming Conventions_)

이 장에서는 문장을 이루는 가장 기본적인 단위인 키워드와 식별자 이름에 대한 내용을 다룬다.

## 명명 기본 가이드

식별자(_identifier_)는 문장내에서 유일하게 구별되어 인식되는 이름으로 아래 항목들을 포함한다.
1. 언어의 문법구조를 기술하는 토큰(_token_) - ex. `SELECT`, `JOIN`, `FROM`, ...
2. 테이블, 컬럼의 명칭이나 별칭(_alias_)
3. 기본 제공 함수 또는 사용자 정의 함수 이름 - ex. `SUM()`, `STRPOS()`, ...

[키워드(_keyword_)](https://cloud.google.com/bigquery/docs/reference/standard-sql/lexical#reserved_keywords) 는 예약된 식별자(_reserved identifier_) 로서 SQL 명세에 의해 그 쓰임이 이미 정해져 있는 이름을 말한다.

본 가이드에서는 아래 서술된 관례에 따라 이름을 정의하고 표기하는 것을 권장한다.
- 키워드는 대문자로 그 외의 식별자는 소문자로 작성하도록 한다.
- 기본 제공 함수 혹은 내장 함수(_built-in function_)는 대소문자를 구분하지 않으나 대문자로 작성하는 것을 기본으로 한다.
- 사용자 정의 함수(UDF, _user-defined function_)는 소문자를 사용하여 함수 이름을 정의하도록 한다. UDF 이름은 대소문자를 구분한다.
- 여러 단어로 이루어진 식별자는 `camelCase`가 아닌 `snake_case`의 형태로 작성토록 한다.

위의 관례에 따른 예시는 다음과 같다.

:smile: 권장 (_recommend_)
```sql
SELECT first_name, last_name FROM my_dataset.my_table
```
:disappointed: 기피 (_avoid_)
```sql
select firstName, lastName from myDataset.myTable -- camel case 사용
-- OR
SELECT FIRST_NAME, LAST_NAME FROM MY_DATASET.MY_TABLE -- 식별자로 대문자 사용
```

내장 함수와 달리 사용자 정의 함수의 이름으로 소문자를 사용하는 이유는 다음과 같다.

사용자 정의 함수를 영속 함수 (_persistent function_) 로 정의하는 경우 데이터셋 이름을 포함한 **한정된 함수 이름** (_qualified function name_)을 사용해야 한다.  이때 함수를 한정시키기 위해 사용하는 프로젝트 이름이나 데이터셋의 이름이 소문자로 작성되기 때문에 하나의 의미 단위내에서 대소문자가 혼용되지 않게 하기 위해서이다. 

이를 통해 아래에 설명할 **FQTN (Fully Qualified Table Name)** 테이블 이름 규칙과의 일관성도 유지할 수 있다.

:smile: 권장 (_recommend_)
```sql
-- 완전 한정된 함수 이름의 예 - fully qualified function name
SELECT bqutils.fn.last_day('2021-03-07')
```

:disappointed: 기피 (_avoid_)
```sql
-- Fully Qualified Function Name에 대소문자가 혼용
SELECT bqutils.fn.LAST_DAY('2021-03-07')
```

대소문자의 구분은 문장내에서 각 이름이 제 역할을 드러내도록 하는데 도움을 준다. 

대문자 `SELECT`, `FROM` 등의 키워드는 주로 문장 구조를 표현하는데 사용되며 소문자로 표기되는 개체(_entity_)나 속성(_attribute_)의 식별자 이름과 시각적으로 구분이 된다. 이는 전체적인 문장 구조의 파악을 용이하게 하여 가독성을 높여준다.

## 기타 스타일 가이드
- 키워드가 부득이하게 컬럼의 이름으로 사용되는 경우 backtick(`)을 이용하여 감싸준다 (_quoted identifiers_).
```sql
-- from 예약어가 컬럼명으로 사용하는 경우
SELECT 'asia-northeast3' AS `from` FROM ...
```
- 문자열 리터럴(_string literal_)은 단일따옴표(')를 사용한다.
- 상수(_constant_)값을 갖는 변수 이름은 대문자로 한다.

```sql
-- BigQuery Scripting에서의 변수 선언
DECLARE START_DATE DATE DEFAULT '2021-02-19'
```

## 참고 (_references_)
- [Lexical structure and syntax in Standard SQL](https://cloud.google.com/bigquery/docs/reference/standard-sql/lexical)

----
# 컬럼 이름 짓기와 사용하기 (_Column Naming_)

## 컬럼 이름 기본 가이드
컬럼이름은 `camelCase`가 아닌 소문자를 이용하여 `snake_case` 로 표현하며 [헝가리언 표기](https://en.wikipedia.org/wiki/Hungarian_notation) 나 의미를 유추하기 힘든 지나치게 축약된 접두어(_prefix_), 접미어(_suffix_)의 사용은 자제한다.

기준정보와 같이 팀내에서 표준으로 관리되는 속성들이 있는 경우에는 해당 표준에서 정의된 명명규칙을 우선한다. 예를 들어, 국가코드등 기준정보로 사용되는 컬럼의 이름은 표준에서 정의된 명명 규칙을 차용해서 사용하도록 한다.
```sql
SELECT cnty_cd, mcc, mnc, csc, FROM meta_table
```

배열형이나 복수의 의미를 지니는 컬럼인 경우는 이름을 복수형으로 하여 집합체(collection) 임을 암시토록 한다.  아래 예시의 `visits` 과 같은 복수형의 이름은 `number_of_visit` 보다 간결하게 횟수를 표현하는 컬럼명으로 자주 애용된다.

```sql
SELECT hits,          -- ARRAY<STRUCT<>> 형
       totals.visits, -- INT64형, 총 세션 횟수
       totals.hits,   -- INT64형, 세션내 조회수
  FROM `bigquery-public-data.google_analytics_sample.ga_sessions_20170801` 
;
```

## 접두어와 접미어 (_prefix, suffix_)
접두어나 접미어는 이름에 부가적인 의미(semantics)를 부여하기 위한 목적으로 사용 가능하나 지나치게 축약된 약어나 과도한 사용은 오히려 가독성을 떨어뜨릴 수 있어 사용에 신중해야 한다.

아래는 몇가지 사용 가능한 예시이다.

- Boolean 형의 경우 `is_`, `has_`와 같은 접두어를 사용할 수 있다.
- Date형은 `_dt` 접미어를 붙일 수 있다.
- DateTime이나 Timestamp형은 `_at` 접미어가 가능하다.
- `user_id`, `device_id` 처럼 식별자(_id_) 접미어의 사용이 가능하다.

```sql
SELECT is_active,
       registered_dt,
       updated_at,
  FROM ...       
```

## 컬럼 별칭 (_column_alias_)
컬럼의 별칭 이름은 컬럼 이름 기본 가이드에 준하여 작성한다. 별칭 앞의 `AS` 키워드 는 생략 가능하나 본 가이드에서는 명시적으로 사용하는 것을 기본으로 한다.

별칭은 목적에 따라서 컬럼 이름보다 상세히 작성되는 경우도 있고 반대로 간략하게 축약된 형태로 사용하기도 한다. 

아래와 같이 컬럼의 의미를 명확하 하기 위해서는 긴 이름의 별칭을 사용할 수 있다.

```sql
SELECT fname AS first_name,
       lname AS last_name,  
  FROM ...
```

반면에 **유도된 컬럼**(_derived column_) 또는 **계산된 컬럼**(_calculated column_)을 만들어 내는 연산식에서 **기본 컬럼**(_base column_) 이름이 반복적으로 사용되어지는 경우는 간결함을 위해서 축약된 이름을 사용하는 경우도 있다.

:disappointed: 기피 (_avoid_)
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
위의 예시에서 테이블의 **기본 컬럼**(_base column_) 으로부터 새로운 컬럼을 파생시키는 연산식에 **기본 컬럼** 이름이 반복적으로 등장하여 라인이 길어지고 있다. 이를 아래와 같이 축약된 별칭을 사용하게 되면 연산식이 간결하게 표현된다.

:smile: 권장 (_recommend_)
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
# 테이블 이름 짓기와 사용하기 (_Table Naming_)

## 테이블 이름 기본 가이드
테이블 이름은 소문자 사용을 기본으로 하되 영구 테이블(_permanent table_)에 대한 팀내 명명 규칙이 별도로 정해져 있는 경우 그것을 우선한다.  

다만, 임시테이블(_temporary table_)이나 공통 테이블 표현식(_CTEs_) 이름의 경우는 소문자를 사용하도록 한다. 테이블 이름은 대소문자를 구분한다.

- 여래 개의 단어를 조합하여 이름를 지을 경우 dash(-)가 아닌 underscore(_)를 사용하여 snake_case 이름이 되도록 한다.
- 과제별 명명규칙에 의해 이미 생성되어 운영되고 있는 데이터셋과 테이블 이름들은 가이드의 권고를 따르지 않더라도 As-Is 이름을 그대로 사용토록 한다.

■ Fully Qualified Table Name (FQTN)

> '_완전하게 한정된 테이블 이름_ 또는 _정규화된 테이블 이름_'의 의미는 그 이름을 가지고 실행컨텍스트와 상관없이 유일하게 테이블를 특정할 수 있다는 말이다.
> 예를 들어,  `customer.devices` 라는 _일부 한정된 테이블 이름_ 을 사용하는 경우 쿼리를 실행시키는 프로젝트에 따라서 각기 다른 `customer.devices` 테이블에 접근이 될 수 있다.

BigQuery로 DWH(Data Ware House)를 구축하는 경우 데이터가 저장된 프로젝트와 BigQuery Job을 실행시켜주는 프로젝트가 다를 수 있기 때문에 실행컨텍스트의 영향을 받지 않도록 정규화된 형태의 테이블 이름을 사용하도록 한다. 테이블 이름에 프로젝트 명을 생략하는 경우 BigQuery Job 이 수행되고 있는 프로젝트 이름을 암묵적으로 사용한다.

이외에, 단일 문장(_single statement_)의 쿼리를 실행하는 경우가 아닌 BigQuery 스크립트(_script_)로 복수의 문장(_multiple statements_)을 실행하는 경우에는 GCP Project 이름에 포함된 dash(-)가 뺄셈 등의 다른 의미로 해석이 되지 않도록 _정규화된 테이블 이름_ 을 backtick(`)을 사용하여 감싸준다.  

이때 dash가 포함된 프로젝트 이름만 backtick으로 감싸주어도 오류가 발생하지는 않으나 FQTN 전체를 backtick을 감싸주어 의미적으로 하나의 단위임을 명시적으로 나타내도록 한다.

:smile: 권장 (_recommend_)
```sql
SELECT * FROM `bigquery-public-data.austin_bikeshare.bikeshare_stations`
```
:disappointed: 기피 (_avoid_)
```sql
SELECT * FROM `bigquery-public-data`.austin_bikeshare.bikeshare_stations
```

별표(`*`)를 포함하는 와일드카드(_wildcard_) 테이블 이름도 backtick(`)으로 감싸주어야 한다.

```sql
/* Valid standard SQL query with wildcard table name*/
SELECT max
  FROM `bigquery-public-data.noaa_gsod.gsod*`
 WHERE max <> 9999.9 # code for missing data
   AND _TABLE_SUFFIX = '1929'
 ORDER BY max DESC
;
```
