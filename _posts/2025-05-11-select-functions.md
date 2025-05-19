---
title: "SELECT, 여러 함수 정리"
excerpt: "날짜/시간 함수, 문자열 함수, 수학 함수, SELECT"

categories:
  - SQL
tags:
  - [SQL 날짜/시간함수, SQL 문자열함수, SQL 수학함수, select, sql]

permalink: /SQL/SQL-functions/

toc: true
toc_sticky: true

date: 2025-05-11
last_modified_at: 2025-05-11
---

# 쿼리를 작성에 있어서 생각해볼 것
## 1. 쿼리를 작성하기 전
- 데이터가 어떻게 저장되어 있을까
- 어떤 데이터들이 저장되어 있을까
- 컬럼의 의미는 무엇일까
- 어떻게 데이터를 가공해야 할까
=> 생각해봐야 함

## 2. 쿼리를 작성하는 중
- 쿼리 실행 Output 형태를 예상해보기
=> 매번 실행하고 확인하는 것이 아니라 지금 쿼리가 어떻게 될 지 예상해보고 실제랑 어떻게 다른지 확인!

<br>

# 데이터베이스
## 1. OLTP
- Online Transaction Processing
- 거래를 하기 위해 사용되는 데이터베이스
- MySQL, Oracle, PostgreSQL 등
  - 보류, 중간 상태가 없음
  - 거래를 완료하거나 취소하거나 : 데이터가 무결하다
  - 데이터의 추가(INSERT), 데이터의 변경(UPDATE) 많이 발생함
- SQL을 사용해서 데이터를 추출할 수 있지만 분석을 위해 만든 데이터베이스가 아니기 때문에 쿼리 속도가 느릴 수 있음

<br>

## 2. OLAP
- OLTP로 데이터 분석을 하다가 기능 부족 이슈로 인해 OLAP 등장
- Online Analytical Processing
  - 분석을 위한 기능 제공
<br>
- 데이터 웨어하우스
  - 데이터를 한 곳에 모아서 저장
  - 여러 곳에 저장된 데이터 예시
  - Daatabase, 웹(크롤링), 파일, API 결과 등

<br>

## 3. BigQuery
### BigQuery란?
- 머신러닝, 지리정보 분석, 비즈니스 인텔리전스와 같은 기본 제공 기능을 이용해 데이터를 관리하고 분석할 수 있게 해주는 데이터 웨어하우스
- Google Cloud 서비스를 통해 제공되기 때문에 설치할 필요가 없음
- 회사에서 앱이나 웹에서 Firebase, Google Analytics 4를 활용할 경우 많이 사용함
- 또한, 적은 비용(인력 등)으로 운영을 하기 위해 많이 사용함

### BigQuery 장점
- SQL을 사용해 쉽게 데이터를 추출할 수 있음
- OLAP 도구 -> 속도가 빠름(단, 그만큼 돈을 지불)
- Firebase, Google Analytics 4의 데이터를 쉽게 추출할 수 있음
  - 사용 기기, 위치(시 단위까지 표현), OS 버전, 이벤트 행동 획득 가능(별도의 로깅 필요)
- Google에서 인프라를 관리하는 데이터 웨어하우스이기 때문에 서버(컴퓨터)를 띄울 필요 없음

<br>

# SQL
## 1. 기본적인 SQL 쿼리 구조

```
SELECT col1, col2, ...
FROM table
WHERE <조건>
```

- SELECT : Table에 저장되어 있는 컬럼 선택
- FROM : 데이터를 확인할 Table 명시
- WHERE : FROM에 명시된 Table에 저장된 데이터를 필터링(조건 설정)

<br>

## 2. 집계, 그룹화 - COUNT, DISTINCT, GROUP BY
### (1) 집계 : GROUP BY
- 같은 값끼리 모아서 그룹화
- 특정 컬럼을 기준으로 나머지 컬럼을 집계하는 데 주로 사용
  - ex) MAX, COUNT, MIN, AVG 등 집계 함수와 함께 사용
- 집계할 컬럼을 `SELECT`에 명시하고 그 컬럼을 꼭 `GROUP BY`에 작성

```
SELECT 집계할 컬럼1, 집계 함수, ...
FROM Table
GROUP BY 집계할 컬럼1
```
<br>

### (2) 중복 데이터 제거 : DISTINCT
- 컬럼 이름 앞에 `DISTINCT`를 붙이면 중복된 값은 1개만 출력됨

```
SELECT 
	집계할 컬럼,
    COUNT(DISTINCT count할 컬럼)
FROM Table
GROUP BY 
	집계할 컬럼
```
<br>

### (3) 조건 설정 : WHERE
- `Table`에 조건을 설정하고 싶은 경우 사용

```
SELECT 
	컬럼1, 컬럼2,
    COUNT(컬럼1) AS col1_count
FROM Table
WHERE
	컬럼1 >= 3
```
<br>

### (4) 그룹 조건 설정 : HAVING
- `GROUP BY`한 후 조건을 설정하고 싶은 경우 사용

```
SELECT 
	컬럼1, 컬럼2,
    COUNT(컬럼1) AS col1_count
FROM Table
GROUP BY 컬럼1, 컬럼2
HAVING
	col1_count > 3
```
<br>

### (5) 조회 데이터 정렬 : ORDER BY
- 특정 컬럼의 데이터를 기준으로 오름차순/내림차순 조회할 때 사용
- `ORDER BY`는 쿼리 맨 마지막에 작성. 중간에 필요없음
  - 서브 쿼리할 때 사용하면 실행 시간이 더 오래걸림
- `DESC` : 내림차순 / `ASC` : 오름차순(보통 default로 되어있음)

```
SELECT 
	col
FROM Table
ORDER BY col DESC
```
<br>

### (6) 출력 개수 제한 : LIMIT
- 쿼리문의 결과 Row 수를 제한하고 싶은 경우 사용
- 쿼리문의 제일 마지막에 작성

```
SELECT 
	col
FROM Table
ORDER BY col DESC
LIMIT 5
```
<br>

## 3. 데이터 변환하기
### (1) 자료 타입 변환 : CAST
- 데이터 타입을 변경할 때 `CAST` 사용

```
SELECT 
	CAST(1 AS STRING) # 숫자 1을 문자1로 변경
```
<br>

- 더 안전하게 데이터 타입 변경하기 : `SAFE_CAST`
  - `SAFE_`가 붙은 함수는 변환이 실패할 경우 NULL 반환

```
SELECT 
	SAFE_CAST("SQL" AS INT64) # 결과 NULL로 출력됨
```
<br>

### (2) 수학 함수
- 수학 함수는 수학 연산(평균, 표준편차, 코사인 등)이 존재
- 나누기같은 경우 `x / y` 대신 `SAFE_DIVIDE`사용하기
  - ex) `SAFE_DIVIDE(x, y)`
  - 그냥 나누기할 경우 x, y 중 하나라도 0인 경우 zero error 발생
  
![](https://velog.velcdn.com/images/haeebin/post/b337dc14-221d-4206-9782-e33addf05855/image.png)
<br>

#### 절댓값 구하기 : ABS
- 절댓값을 반환할 때 사용
- 함수 인자에 컬럼명뿐만 아니라 **식을 입력할 수도 있음**

```
SELECT
	col1, col2,
	ABS(col1 - col2) AS abs_diff
FROM table
```
<br>

- 자료형의 범위를 넘으면 산술 오버플로 에러가 발생함
  - 단, BigQuery에서는 오버플로우 에러가 발생하지 않음
    - BigQuery는 기본 정수 타입으로 INT64 (64비트 signed integer) 를 사용
    - `ABS()`는 입력값의 절댓값을 반환하며, 내부적으로 자동 형변환 또는 넉넉한 자료형 처리를 통해 오버플로우가 발생하지 않도록 설계되어 있음
    - 예외적으로 FLOAT64 또는 NUMERIC도 지원하며, 이 경우도 모두 절댓값 연산은 안전하게 처리됨

```
-- PostgreSQL
SELECT ABS(-2147483648)		# expression을(를) 데이터 형식 int(으)로 변환하는 중 산술 오버플로 오류가 발생했습니다.

-- BigQuery
SELECT ABS(-2147483648)		# 2147483648 출력
```
<br>

#### 양수/음수 판단 : SIGN
- 지정한 값이나 식의 양수, 음수, 0을 판단해 1, -1, 0 반환
- SQL Server의 경우 입력 자료형에 따라 반환형이 다름
  - `int`, `money`, `decimal`등
- BigQuery의 경우 `float`, `int`, `numeric` 등 모두 가능하지만 항상 `INT64` 반환. 자료형 통일

```
SELECT SIGN(-125), SIGN(0), SIGN(564)
-- -1, 0, 1 출력
```
<br>

#### 지정한 숫자보다 크거나 같은 값 구하기 : CEIL, CEILING
- 소수점을 올려 지정한 값보다 크거나 같은 최소 정수 반환

```
SELECT CEIL(123.45), CEILING(-123.45)
-- 124, -123 출력
```
<br>

#### 지정한 숫자보다 작거나 같은 값 구하기 : FLOOR
- 소수점을 내려 지정한 값보다 작거나 같은 최대 정수 반환

```
SELECT FLOOR(123.45), FLOOR(-123.45)
-- 123, -124 출력
```
<br>

#### 소수점 반올림 하기: ROUND
- 소수점 반올림한 값을 구함
- 매개변수는 2개인데 첫 번째 매개변수에 값을 넣고, 두 번째에 소수 자리 수를 넣음

```
SELECT ROUND(123.4567), ROUND(123.4567, 2)
-- 123, 123.46 출력
```
<br>

#### 로그 함수: LOG
- 로그 계산. 기본 계산은 자연로그(`ln`), 두 번째 인자로 밑 지정 가능

```
-- 기본 형식
LOG(float_expression [, base])

-- ex) ln(10), log10(100)
SELECT LOG(10), LOG(100, 10);
```
<br>

#### 지수 함수(e의 n 제곱값) : EXP
- `e`의 x 제곱 계산

```
SELECT EXP(2);  -- ≈ 7.389056
```
<br>

#### 거듭제곱 값, 제곱근 구하기 : POWER, SQRT
- `POWER(x, y)`는 x의 y 거듭제곱 값을 구할 때 사용

```
SELECT POWER(2, 3);  -- 8
```

- `SQRT`는 제곱근을 계산할 때 사용

```
SELECT SQRT(9), SQRT(4);  -- 3, 2
```
<br>

#### 난수 구하기 : RAND
- 0 이상 1 미만의 난수 생성

```
SELECT RAND();
```
<br>

#### 삼각함수 : SIN, COS, TAN, ATAN, PI()등
- 각도를 라디안 단위로 받아 삼각값 반환
- 결과는 모두 `FLOAT64` 자료형으로 반환

```
SELECT SIN(PI()/2), COS(PI()), TAN(PI()/4);
SELECT ATAN(1);  -- 역탄젠트 
```
<br>

### (3) 문자열 함수
- 문자열로 할 수 있는 대표적인 연산
  - 문자열 붙이기
  - 문자열 분리하기
  - 특정 단어 수정하기
  - 문자열 자르기
  - 영어 대문자 변환
<br>

#### 문자열 붙이기 : CONCAT
- 여러 문자열을 붙여서 하나의 문자열로 합침
- 주의사항
  - **NULL이 하나라도 포함되면 전체 결과가 NULL이 됨**
  - 대신 `CONCAT_WS(delimiter, ...)` 사용하면 NULL 무시 가능

```
-- 문법
CONCAT(string1, string2 [, ...])

-- ex) 'Hello', 'World' 사용해서 'Hello World' 출력하기
SELECT CONCAT('Hello', ' ', 'World')
```
<br>

#### 문자열 분리하기 : SPLIT
- 지정한 구분자를 기준으로 문자열을 배열로 나눔

```
-- 문법
SPLIT(문자열 원본, 나눌 기준이 되는 문자)

-- ex) 결과: ['apple', 'banana', 'grape']
SELECT SPLIT('apple,banana,grape', ',')
```
<br>

#### 특정 단어 수정하기 : REPLACE
- 문자열에서 특정 문자열을 다른 문자열로 바꿈

```
-- 문법
REPLACE(문자열 원본, 찾을 단어, 바꿀 단어)

-- ex) '결과: 'I loves SQL'
SELECT REPLACE('I loves coding', 'coding', 'SQL')
```
<br>

#### 문자열 자르기 : TRIM
- 문자열의 앞뒤 공백 또는 지정 문자 제거
- `TRIM(문자열 원본, 자를 단어)`

```
-- 문법
TRIM([LEADING | TRAILING | BOTH] [characters] FROM string)
TRIM(string) -- 간단하게 하면

-- ex
SELECT TRIM('   hello   ');          -- 'hello'
SELECT TRIM(BOTH 'x' FROM 'xxdatax'); -- 'data'
SELECT TRIM('Hello World', 'World');  -- 'Hello'
```
<br>

#### 영어 대문자 변환 : UPPER
- 문자열의 모든 알파벳을 대문자로 변환

```
-- 문법
UPPER(string)

-- ex) 결과 : 'BIGQUERY'
SELECT UPPER('bigquery')
```
<br>

### (4) 날짜/시간 함수
- 날짜 및 시간 타입에는 총 4개가 있음
  - `DATE` : 날짜를 표현하는 데이터 타입
    - ex) 2025-05-08
  - `DATETIME` : 날짜와 시간을 함께 표현하는 데이터 타입(타임존 정보는 없음)
    - ex) 2025-03-05 12:50:59.000
  - `TIME` : 시간을 표현하는 데이터 타입
    - ex) 13:55:20.000
  - `TIMESTAMP` : 날짜와 시간을 함께 표현하며, 타임존 정보까지 포함 
                 (Bigquery에서는 기본적으로 UTC 기준 시간이 표현)
    - ex) 2025-02-22 20:11:40.000UTC
<br>

- Timezone에서 GMT와 UTC는?
  - GMT : Greenwich Mean Time(한국 시간 : GMT +9)
    - 영국의 그리니치 천문대(경도 0도)를 기준으로 지역에 따른 시간의 차이를 조정하기 위해 생긴 시간의 구분선(1884년 채택)
    - 영국 근처에서 자주 활용
  - **UTC : Universal Time Coordinated**(한국 시간 : UTC+9)
    - 국제적인 표준 시간
    - 협정 세계시
    타임존이 존재함 = 특정 지역의 표준 시간대
  - TIMESTAMP 데이터는?
    - 시간 도장
    - UTC부터 경과한 시간을 나타내는 값
    - Timezone 정보 있음
<br>

- `millisecond`, `microsecond`
  - `millisecond(ms)`
    - 시간의 단위. 천 분의 1초(1,000ms = 1초)
    - 우리가 아는 초보다 더 짧은 시간 단위
    - 빠른 반응이 필요한 분야에서 사용(초보다 더 정확)
    - Millisecond -> TIMESTAMP -> DATETIME 으로 변경
  - `microsecond(μs)`
    - 1/1,000ms, 1/1,000,000초
  
  ```
  -- ex) 1704176819711ms =>  2024-01-02 15:26:59(DATETIME)
  SELECT 
    TIMESTAMP_MILLIS(1704176819711) AS milli_to_timestamp_value,
    TIMESTAMP_MICROS(1704176819711000) AS micro_to_timestamp_value,
    DATETIME(TIMESTAMP_MICROS(1704176819711000)) AS datetime_value;
  ```
<br>

- `TIMESTAMP`와 `DATETIME`의 차이

|  | TIMESTAMP | DATETIME |
| -- | -- | -- |
| 타임존 | UTC라고 나옴 | T가 나옴(Time을 의미) |
| 시간 차이 | 한국 시간 -9시간 | 한국 Zone 사용 시 한국 시간과 동일 |

<br>

#### 현재 시간을 다루기 : CURRENT_DATE / CURRENT_DATETIME / CURRENT_TIMESTAMP
- 현재 날짜를 `DATE` / `DATETIME` / `TIMESTAMP` 형식으로 반환
- 시간대는 선택 사항이며, 지정하지 않으면 UTC 기준으로 반환

```
-- 기본 형식
CURRENT_DATE()
CURRENT_DATE('Asia/Seoul')
CURRENT_DATETIME('Asia/Seoul')

-- ex) 결과 : 2025-05-11
SELECT CURRENT_DATE();    
SELECT CURRENT_DATE('Asia/Seoul');

-- ex) 결과 : 2025-05-11T10:40:31.436775
SELECT CURRENT_DATETIME('Asia/Seoul');  # timezone지정안하면 2025-05-11T01:41:07.836942

-- ex) 결과 : 2025-05-11 01:43:22.954613 UTC
SELECT CURRENT_TIMESTAMP();
```
<br>

#### 특정 부분 추출하기 : EXTRACT
- 날짜/시간 데이터에서 원하는 일부분(연, 월, 일, 시 등) 을 추출

```
-- 기본 형식
EXTRACT(YEAR FROM DATE '2025-05-10')
EXTRACT(HOUR FROM TIMESTAMP '2025-05-10 13:45:00')
EXTRACT(MONTH FROM <날짜 타입 컬럼명>)

-- ex) 
SELECT 
  EXTRACT(YEAR FROM TIMESTAMP '2025-05-10 14:23:45.678901') AS year, -- 2025
  EXTRACT(MILLISECOND FROM TIMESTAMP '2025-05-10 14:23:45.678901') AS ms, -- 678
  EXTRACT(DAYOFWEEK FROM DATE '2025-05-10') AS dow; -- 7
```

| part 키워드 | 설명 | 예시 값(2025-05-10 14:23:45.678901) | 설명 추가 |
| --- | --- | --- | --- |
| YREAR | 연도 추출 | 2025 | 4자리 연도 |
| MONTH | 월 추출 | 5 | 1 ~ 12 |
| DAY | 일 추출 | 10 | 1 ~ 31 |
| HOUR | 시 추출 (24시간 기준) | 14 | 0 ~ 23 |
| MINUTE | 분 추출 | 23 | 0 ~ 59 |
| SECOND | 초 추출(정수로 반환) | 45 | 0 ~ 59 |
| MILLISECOND | 밀리초 추출(소수점 3자리까지) | 678 | 0 ~ 999 |
| MICROSECOND | 마리크로초 추출(소수점 6자리까지) | 678901 | 0 ~ 999999 |
| DAYOFWEEK | 요일 추출(**1**=일요일 , **7**=토요일) | 7 (토요일이어서) | 요일 번호 반환 |
| DATE | 날짜 부분만 추출 (TIMESTAMP -> DATE) | 2025-05-10 | 시간 제거 |

<br>

#### Datetime 시간 자르기 : DATETIME_TRUNC
- 지정한 `DATETIME` 값을 `part` 단위로 내림(잘라냄) 해서 반환하는 함수
- `TIMESTAMP`에는 `TIMESTAMP_TRUNC` , `DATE`에는 `DATE_TRUNC` 사용
- 예를 들어, `DATETIME '2025-05-10 14:23:45'`를 `'HOUR'`로 자르면 → `2025-05-10 14:00:00`이 됨

```
-- 기본 형식
-- datetime_expression: 잘라낼 대상 (예: DATETIME 컬럼 또는 리터럴)
-- part: 자를 단위 (YEAR, MONTH, DAY, HOUR, MINUTE, SECOND, ...)
DATETIME_TRUNC(datetime_expression, part) 

-- ex) DATETIME 값: 2025-05-10 14:23:45.678
SELECT
  DATETIME_TRUNC(DATETIME '2025-05-10 14:23:45.678', YEAR), -- 2025-01-01T00:00:00
  DATETIME_TRUNC(DATETIME '2025-05-10 14:23:45.678', MONTH), -- 2025-05-01T00:00:00
  DATETIME_TRUNC(DATETIME '2025-05-10 14:23:45.678', HOUR); -- 2025-05-10T14:00:00
```

| part 키워드 | 설명 | 예시 값(2025-05-10 14:23:45) |
| --- | --- | --- |
| YREAR | 해당 연도의 1월 1일 00:00:00 | 2025-01-01 00:00:00 |
| QUARTER | 해당 분기의 시작 (1/4/7/10) | 2025-04-01 00:00:00 |
| MONTH | 해당 월의 1일 00:00:00 | 2025-05-01 00:00:00 |
| WEEK | 주의 시작 (일요일 기준) | 2025-05-04 00:00:00 | 
| DAY | 해당 일의 자정 | 2025-05-10 00:00:00 |
| HOUR | 해당 시각의 00분 00초 | 2025-05-10 14:00:00 |
| MINUTE | 해당 시각의 00초 | 2025-05-10 14:23:00 |
| SECOND | 	마이크로초 제거 | 2025-05-10 14:23:45 |

<br>

#### 문자열 -> 날짜 타입 변경 : PARSE_DATETIME
- 문자열로 저장된 DATETIME을 DATETIME 타입으로 바꾸고 싶은 경우 사용
- `DATE`로 변경 : `PARSE_DATE` , `TIMESTAMP`로 변경 : `PARSE_TIMESTAMP`
- 인자에 들어가는 format은 [Format Elements 문서](https://cloud.google.com/bigquery/docs/reference/standard-sql/format-elements#format_elements_date_time) 참고

```
-- 기본 형식
PARSE_DATE(format, string)
PARSE_DATETIME(format, strring)
PARSE_TIMESTAMP(format, string)

-- ex)
SELECT PARSE_DATE('%Y-%m-%d', '2025-05-10'); -- DATE 값 반환
SELECT PARSE_DATETIME('%Y-%m-%d %H:%M:%S', '2024-01-11 12:35:35'); -- DATETIME 값 반환
SELECT PARSE_TIMESTAMP('%Y-%m-%d %H:%M:%S', '2025-05-10 12:34:56'); -- TIMESTAMP 값 반환
```
<br>

#### 날짜 -> 문자열 타입 변경 : FORMAT_DATETIME
- DATETIME 타입 데이터를 지정한 형식의 문자열 데이터로 변환
- `DATE`타입일 경우 : `FORMAT_DATE` , `TIMESTAMP`일 경우 : `FORMAT_TIMESTAMP`

```
-- 기본 형식
FORMAT_DATE(format, date)
FORMAT_DATETIME(format, datetime)
FORMAT_TIMESTAMP(format, timestamp)

-- ex)
SELECT
	FORMAT_DATE('%Y-%m-%d', DATE '2025-05-10'), -- '2025-05-10'
    FORMAT_DATE('%B %d, %Y', DATE '2025-05-10'), -- 'May 10, 2025'
    FORMAT_DATETIME("%c", DATETIME "2024-01-11 12:35:35"), -- Thu Jan 11 12:35:35 2024
    FORMAT_TIMESTAMP('%F %T', TIMESTAMP '2025-05-10 12:00:00'), -- '2025-05-10 12:00:00'
```
<br>

#### 주어진 날짜의 마지막 날 : LAST_DAY
- 주어진 날짜를 기준으로 월, 분기, 해 등 기간의 마지막 날을 반환
- DATE, DATETIME, TIMESTAMP 모두 지원하지만, 반환 타입은 **DATE**
- `	TIMESTAMP`를 넣으면 날짜만 추출되어 계산

```
-- 기본 형식
-- date_expression: 기준이 되는 날짜 (DATE, DATETIME, TIMESTAMP 가능)
-- part: 자를 기준 단위 (MONTH, QUARTER, YEAR) 
------ 생략하면 기본값은 'MONTH'
LAST_DAY(date_expression [, part])

-- ex) 기준 날짜: 2025-05-10
SELECT
  LAST_DAY(DATE '2025-05-10') AS end_of_month, -- 2025-05-31
  LAST_DAY(DATE '2025-05-10', QUARTER) AS end_of_quarter, -- 2025-06-30
  LAST_DAY(DATE '2025-05-10', YEAR) AS end_of_year, -- 2025-12-31
  SELECT LAST_DAY(TIMESTAMP '2025-05-10 14:23:00'); -- 결과: 2025-05-31 (DATE 타입으로 반환)
```
<br>

#### 두 날짜의 차이 : DATETIME_DIFF
- `datetime1 - datetime2`의 차이를 원하는 단위로 반환
- date : DATE_DIFF , TIMESTAMP : TIMESTAMP_DIFF

```
-- 기본 형식
-- unit : YEAR, MONTH, DAY, WEEK 등 가능 (TIMESTAMP : MINUTE, MILLISECOND 등도 가능)
DATE_DIFF(date1, date2, unit)
DATETIME_DIFF(datetime1, datetime2, unit)
TIMESTAMP_DIFF(timestamp1, timestamp2, unit)

-- ex) 결과 : 9
SELECT DATE_DIFF(DATE '2025-05-10', DATE '2025-05-01', DAY);

-- 결과 : 550
SELECT TIMESTAMP_DIFF(
  TIMESTAMP '2025-05-10 10:00:05.500',
  TIMESTAMP '2025-05-10 10:00:00.000',
  MILLISECOND
);
```
<br>

#### 날짜 더하기 : DATE_ADD
- 날짜 또는 타임스탬프에 일정 기간을 더하거나 빼는 함수
- 유사 함수 : `DATE_SUB`, `TIMESTAMP_ADD`, `TIMESTAMP_SUB`

```
-- 기본 형식
DATE_ADD(date, INTERVAL x unit)

-- ex) 
SELECT DATE_ADD(DATE '2025-05-10', INTERVAL 5 DAY); -- '2025-05-15'

-- 2025-05-10 10:00:00 UTC
SELECT TIMESTAMP_SUB(TIMESTAMP '2025-05-10 12:00:00', INTERVAL 2 HOUR); 
```
