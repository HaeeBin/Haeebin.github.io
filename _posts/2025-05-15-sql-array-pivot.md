---
title: "SQL 배열 함수와 PIVOT"
excerpt: "SQL ARRAY, STRUCT, UNNEST, PIVOT"

categories:
  - SQL
tags:
  - [sql, SQL ARRAY, SQL STRUCT, SQL UNNEST, SQL 배열함수, PIVOT]

permalink: /SQL/SQL-array-pivot/

toc: true
toc_sticky: true

date: 2025-05-15
last_modified_at: 2025-05-15
---

# ARRAY, STRUCT 다루기
## 1. ARRAY
### ARRAY란? 
- 같은 타입의 여러 값을 하나의 컬럼에 저장할 수 있는 자료형
- 하나의 행에 데이터 타입이 동일한 여러 값이 저장
- 배열로 저장할 때 저장 용량이 효율적
- BigQuery는 SQL임에도 배열을 지원해서 JSON 같은 복잡한 데이터를 유연하게 처리 가능

```
-- 기본 형식
[1, 2, 3]                          -- INT64 배열
['A', 'B', 'C']                   -- STRING 배열
ARRAY<STRING>                     -- 배열 자료형

-- ex) 
SELECT ['apple', 'banana', 'grape'] AS fruits;
```
<br>

### ARRAY 생성하기
1) 대괄호 사용
  
  ```
  SELECT [0, 1, 1, 2, 3, 5] AS some_numbers
  UNION ALL
  SELECT [2, 4, 8, 16, 32]
  UNION ALL
  SELECT [5, 10]
  ```
  <br>

2) ARRAY<> 사용 : ARRAY< 자료형>

  ```
  SELECT
    ARRAY<INT64>[0, 1, 3] AS some_numbers
  ```
  <br>

3) 배열 셍성 함수 사용

  ```
  SELECT
    output1,
    GENERATE_DATE_ARRAY('2024-01-01', '2024-02-01', INTERVAL 1 WEEK) AS
    GENERATE_ARRAY(1, 5, 2) AS output2
  ```
  <br>

4) ARRAY_AGG 함수 사용 : 여러 결과를 마지막에 배열로 저장하고 싶은 경우

  ```
  WITH programming_languages AS (
    SELECT "python" AS programming_language
    UNION ALL
    SELECT "go"
    UNION ALL
    SELECT "scala"
  )
  SELECT ARRAY_AGG(programming_language) AS output
  FROM programming_languages
  ```
  <br>

### ARRAY의 데이터 접근하기
- 배열에 접근하기 위해서는 `OFFSET`, `ORDINAL`을 사용
  - `OFFSET` : 0부터 시작
  - `ORDINAL` : 1부터 시작
- 단, 배열의 길이보다 큰 값을 지정하면 오류 발생
  - `Array index N is out of bounds (overflow)` : 방지하기 위해 `SAFE_` 항상 추가

```
-- 기본 형식
SELECT
  배열_컬럼[OFFSET/ORDINAL(숫자)]

-- ex)
SELECT arr[OFFSET(0)]   -- ['a', 'b', 'c']
```
<br>

## 2. STRUCT
- 하나의 값처럼 다루는 필드들의 묶음
- Python의 딕셔너리, JSON 오브젝트와 비슷
- 서로 다른 타입의 여러 값을 하나의 컬럼에 저장할 수 있는 자료형

```
-- ex)
SELECT STRUCT('apple' AS name, 1000 AS price) AS product;

-- ex) ARRAY + STRUCT 조합 (복잡한 JSON 유사 구조)
-- 결과 : ARRAY<STRUCT<name STRING, price INT64>>
SELECT [
  STRUCT('apple' AS name, 1000 AS price),
  STRUCT('banana' AS name, 800 AS price)
] AS products;
```
<br>

### STRUCT 생성하기
1) 소괄호() 사용
  
  ```
  -- 소괄호 사용 시 이름이 지정되어 있지 않음
  SELECT
    (1,2,3) AS struct_test
  ```
  <br>

2) STRUCT<>() 사용 : STRUCT<자료형>(데이터)

  ```
  SELECT
    STRUCT<hi INT64, hello INT64, awesome STRING>(1, 2, 'HI') AS struct_test
  ```
  <br>

### STRUCT의 값에 접근하기
- STRUCT 이름.key 형식

```
-- ex)
s.x, s.y            -- STRUCT('a' AS x, 1 AS y)
arr[OFFSET(0)].name     -- ARRAY<STRUCT<name, price>>
```

## 3. ARRAY, STRUCT의 관계
- ARRAY안에 STRUCT가 저장될 수 있음
- STRUCT안에 ARRAY 저장 가능
- STRUCT안에 STRUCT 가능
- ARRAY안에 ARRAY 가능
=> 유연하게 데이터를 저장할 수 있음
=> 중첩된 구조
<br>

## 4. UNNSET : 중첩된 데이터 구조를 풀기
### UNNEST란?
- Unnest의 뜻은 펼치다, 풀다라는 의미를 가짐
- 배열을 직접적으로 접근해서 사용하는 것보다, 독립적인 행으로 풀어서(평면화) 사용
- ARRAY에 대한 구조를 펼칠 수 있는 기능을 제공
- 즉, ARRAY의 요소를 독립적인 행으로 펼칠때 UNNEST를 사용

```
-- 기본 형식
-- UNNEST한 결과를 CROSS JOIN
SELECT
  a.column,
  alias_name
FROM Table_A AS a
CROSS JOIN UNNEST(ARRAY_Column) AS alias_name

-- CROSS JOIN은 생략하고 쉼표를 사용해도 괜찮음
SELECT
  a.column,
  alias_name
FROM Table_A AS a, UNNEST(ARRAY_Column) AS alias_name
```
<br>

### 예시
1) 유저가 구매한 제품 배열

```
SELECT
  user_id,
  ARRAY_AGG(product_id) AS purchased_products
FROM purchase_log
GROUP BY user_id;
```
<br>

2) JSON-like 테이블 구조

```
WITH orders AS (
  SELECT 1 AS order_id, [
    STRUCT('apple' AS name, 2 AS quantity),
    STRUCT('banana' AS name, 1 AS quantity)
  ] AS items
)
SELECT
  order_id,
  item.name,
  item.quantity
FROM orders, UNNEST(items) AS item;
```
<br>

## 5. 여러 배열 함수
### ARRAY_LENGTH()
- 배열의 **요소 개수(길이)**를 반환
- 주의
  - NULL 배열이면 NULL 반환
  - 빈 배열은 0

```
-- 기본 형식
ARRAY_LENGTH(array_expr)

-- ex) 결과 : 3
SELECT ARRAY_LENGTH(['a', 'b', 'c']) AS length;
```
<br>

### ARRAY_AGG()
- 여러 행의 값을 배열로 집계(aggregate)
- 주의
  - NULL 포함됨
  - 중복 제거하려면 `ARRAY(SELECT DISTINCT ...)` 사용

```
-- 기본 형식
ARRAY_AGG(expression [ORDER BY ...] [LIMIT n])

-- ex)
SELECT
  department,
  ARRAY_AGG(employee_name ORDER BY employee_name) AS names
FROM employees
GROUP BY department;
```
<br>

### ARRAY_TO_STRING()
- 배열을 문자열로 구분자(delimiter) 를 넣어 합치기

```
-- 기본 형식
ARRAY_TO_STRING(array_expr, delimiter [, null_text])

-- ex) 결과: 'A-B-C'
SELECT ARRAY_TO_STRING(['A', 'B', 'C'], '-') AS result;

-- NULL 처리 ex) 결과: 'A-x-C'
SELECT ARRAY_TO_STRING(['A', NULL, 'C'], '-', 'x');

```
<br>

### GENERATE_ARRAY()
- 숫자의 연속된 배열 생성
- 주의
  - step 생략 시 기본은 1
  - start > end이면 빈 배열 반환

```
-- 기본 형식
GENERATE_ARRAY(start, end [, step])

-- ex) [1, 2, 3, 4, 5]
SELECT GENERATE_ARRAY(1, 5) AS nums;

-- ex) [0, 2, 4, 6, 8, 10]
SELECT GENERATE_ARRAY(0, 10, 2) AS evens;
```
<br>

### ARRAY_REVERSE()
- 배열의 순서를 뒤집어 반환
- 실무 사용 예
  - 최근 방문 페이지를 최신순으로 정렬해서 보기
  - 누적값을 거꾸로 누적할 때

```
-- 기본 형식
ARRAY_REVERSE(array_expr)

-- ex) 결과: [3, 2, 1]
SELECT ARRAY_REVERSE([1, 2, 3]) AS reversed;
```
<br>

### ARRAY_CONCAT()
- 여러 배열을 이어 붙여 하나의 배열로 만듦
- 실무 사용 예
  - 복수 필드에 저장된 태그/카테고리를 하나의 배열로 합치기

```
-- 기본 형식
ARRAY_CONCAT(array1, array2)

-- ex) 결과: [1, 2, 3, 4]
SELECT ARRAY_CONCAT([1, 2], [3, 4]) AS merged;
```
<br>

### ARRAY(SELECT ...)
- 하위 쿼리의 결과를 배열로 반환
- `ARRAY_AGG()`와 비슷하지만 직접 SELECT를 배열로 감쌈
- 실무 사용 예
  - 조건이 있는 서브쿼리를 배열로 만들 때 (WHERE, LIMIT 등 가능)

```
-- 기본 형식
ARRAY(SELECT expression FROM table WHERE condition)

-- ex) 결과: ['a', 'b', 'c']
SELECT ARRAY(SELECT name FROM UNNEST(['a', 'b', 'c']) AS name) AS names;
```
<br>

### ARRAY_DISTINCT()
- 배열 내 중복값을 제거하고 고유한 값만 남김
- `ARRAY_AGG()`와 비슷하지만 직접 SELECT를 배열로 감쌈
- 실무 사용 예
  - 사용자 행동(클릭한 페이지 등) 중복 제거
  - 태그 배열의 중복 키워드 제거

```
-- 기본 형식
ARRAY_DISTINCT(array_expr)

-- ex) 결과: [1, 2, 3]
SELECT ARRAY_DISTINCT([1, 2, 2, 3, 3, 3]) AS distinct_values;
```
<br>

#### 예시 
- 페이지 방문 내역 중 고유한 페이지만 최신순으로 반환

```
SELECT
  ARRAY_REVERSE(
    ARRAY_DISTINCT(
      ARRAY(SELECT page FROM UNNEST(pages) AS page)
    )
  ) AS processed_pages
FROM my_table;
```
<br>

# PIVOT 
## 1. PIVOT이란?
- PIVOT의 뜻은 축을 중심으로 회전한다
- 테이블의 특정 컬럼 값을 열 이름으로 변환하여 가로 방향으로 요약 집계된 형태를 만드는 SQL 구문
<br>

## 2. PIVOT이 필요한 이유
1) 성능(퍼포먼스)
  - 자주 나오는 질문 : Python에서 처리해도 되지 않을까?
  - Row가 많을 경우 느려질 수 있음
  - 미리 데이터를 가공해서 ROW를 줄임
    - 네트워크, 데이터 처리 비용의 효율성
<br>

2) 데이터 시각화 도구에서 PIVOT한 형태를 지원
<br>

## 기본 문법

```
SELECT
  그룹컬럼,
  AGG_FUNC(IF(피벗기준컬럼 = '값1', 측정값, NULL)) AS 컬럼1,
  AGG_FUNC(IF(피벗기준컬럼 = '값2', 측정값, NULL)) AS 컬럼2,
  ...
FROM 테이블
GROUP BY 그룹컬럼
```

- `AGG_FUNC()` : 집계 함수 (예: SUM, COUNT, AVG, MAX)
- `<pivot_column>` : 열로 바꿀 대상 컬럼 (피벗 기준)
- `IN (...)` : 열로 만들 실제 값 목록
- `<group_column>` : 그룹화 기준 컬럼 (피벗에서 고정됨)
<br>

## 예시
### 예제 1 : 학생별 과목 점수를 가로 방향으로 피벗

```
SELECT
  student,
  MAX(IF(subject = '수학', score, NULL)) AS 수학,
  MAX(IF(subject = '영어', score, NULL)) AS 영어,
  MAX(IF(subject = '과학', score, NULL)) AS 과학
FROM scores
GROUP BY student
```

- `IF(subject = '수학', score, NULL)` → 수학일 때만 점수, 아니면 NULL
- `MAX(...)` → 학생-과목 조합은 하나뿐이므로 집계 함수로 하나의 값만 반환
- 결과적으로 subject가 행이 아니라 열로 바뀜
<br>

### 실무 활용 예시
- 유저별 이벤트 수 요약
  - `COUNT(IF(event_type = 'click', 1, NULL))`
- 월별 매출 요약
  - `SUM(IF(MONTH(date) = 1, revenue, NULL)) AS Jan_sales`
- 카테고리별 재고
  - `MAX(CASE WHEN category = '과일' THEN stock END)`