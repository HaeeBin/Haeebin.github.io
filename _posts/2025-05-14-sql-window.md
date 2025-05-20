---
title: "윈도우 함수"
excerpt: "윈도우 함수"

categories:
  - SQL
tags:
  - [sql, SQL WINDOW함수]

permalink: /SQL/SQL-window/

toc: true
toc_sticky: true

date: 2025-05-14
last_modified_at: 2025-05-14
---

# Window(Analytics) 함수
## 1. 윈도우 함수(Window/Analytics Function)란?
- 데이터를 그룹화하지 않고도 집계 값을 계산하거나 순위를 매길 때 사용
  - `GROUP BY`처럼 결과를 줄이지 않고, 행마다 연산 결과를 새 컬럼으로 반환
- 일부분씩 보면서 계산할 수 있게 해주는 함수. 그룹 내 순위, 누적 합계, 평균 등을 계산할 때 유용
- 데이터 분석에 강력한 도구를 제공. 데이터를 더 세밀하게 처리할 수 있도록 도와줌
<br>

## 2. 윈도우 함수의 구성

```
RANK() OVER (PARTITION BY 학년 ORDER BY 키 DESC) AS 학년 별 키 순위
```

- 윈도우 함수의 함수 명 : `RANK()`, `LAG()`, `SUM()` 등
- OVER 키워드(~에 걸쳐서, ~에 대해서) : 윈도우 함수임을 선언
- OVER 절 안의 `PARTITION BY`, `ORDER BY`
  - `PARTITION BY` : 데이터를 특정 기준으로 나눈다.
  - `ORDER BY` : 윈도우 내에서 행의 순서를 지정한다.
- 윈도우 프레임(윈도우 함수가 작동하는 범위)
<br><br>

## 3. 윈도우 함수의 종류
- 탐색 함수 : `LEAD`, `LAG`, `FIRST_VALUE`, `LAST_VALUE`
- 번호 함수 : `RANK`, `ROW_NUMBER`, `DENSE_RANK`, `NTILE` 등
- 집계 함수 : 집계 함수들, `AVG`, `COUNT`, `SUM`, `MAX`, `MIN`, ...
<br><br>

## 4. 탐색 함수
- Row들을 탐색하며 값을 찾는 함수. 정렬이 항상 필요

### LEAD(expr, offset, default) : 후속 행
- 다음 행의 값을 반환
- `LEAD(컬럼명, 2)` : 다다음 행의 값을 반환

```
SELECT
  value,
  LEAD(value) OVER (ORDER BY date) AS next_val
FROM dataset;
```
<br><br>

### LAG(expr, offset, default) : 이전 행
- 이전 행의 값을 반환
- `LAG(컬렴명, 2)` : 2번 전 행의 값을 반환

```
SELECT
  value,
  LAG(value) OVER (ORDER BY date) AS prev_val
FROM dataset;
```
<br><br>

### FIRST_VALUE(expr) : 첫 번째 값
- 지정한 그룹의 첫 번째 값

```
-- ex) 상점별 첫 날 매출을 구함
FIRST_VALUE(sales) OVER (PARTITION BY store ORDER BY date)
```
<br><br>

### LAST_VALUE(expr) : 마지막 값
- 지정한 그룹의 마지막 값
- 주의) : 마지막의 기준은 데이터가 최신화가 되면 바뀜. 즉, 최신 방문 월 구하는 쿼리를 실행할 때마다 달라질 수 있는 값
- ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING 설정 권장
  - "전체 그룹 번위 모두 포함". 즉, PARTITION BY로 묶인 그룹 전체를 대상으로 집계한다는 의미
  - 저걸 따로 작성하지 않는다면 빅쿼리는 기본적으로 `ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW`를 **자동 설정함**
  - 이 경우 현재 행까지의 범위만 보기 때문에 **진짜 마지막 값이 아니라 현재까지 중 가장 마지막 값을 반환**

> `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING`
- 현재 행을 기준으로 어느 범위까지 계산할 것인가를 명확히 지정
- `ROWS BETWEEN {시작 범위} AND {끝 범위}` : 기본 형식
  - `UNBOUNDED PRECEDING` : 윈도우의 첫 행부터 시작
  - `CURRENT ROW` : 현재 행만 포함
  - `UNBOUNDED FOLLOWING` : 윈도우의 마지막 행까지 포함

```
-- 잘못된 예시 (기본 프레임) : 현재 행까지에서 가장 마지막 값을 반환
SELECT LAST_VALUE(sales) OVER (
  PARTITION BY store ORDER BY date
) AS wrong_last;

-- 정확한 마지막 값 : 그룹 내의 전체 행 중에 마지막 값을 반환
SELECT LAST_VALUE(sales) OVER (
  PARTITION BY store ORDER BY date
  ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
) AS correct_last;
```
<br><br>

### FIRST_VALUE, LAST_VALUE의 NULL 처리
- 일반적인 집계 함수들에서 NULL을 처리하는 방식
  - GROUP BY와 사용되는 SUM, AVG 집계 함수는 **NULL을 제외해서 연산**
- 윈도우 함수의 FIRST_VALUE, LAST_VALUE에선 기본적으로 **NULL을 포함해서 연산**
  - 파티션 내의 처음/마지막 값이 NULL이면 NULL을 반환
- NULL을 제외하고 싶으면 `IGNORE NULLS`

```
SELECT
  *,
  FIRST_VALUE(numbers IGNORE NULLS) OVER(ORDER BY date) AS first_num,
  LAST_VALUE(numbers IGNORE NULLS) OVER(ORDER BY date) AS last_num
FROM raw_data;
``` 
<br><br>

### 탐색 함수를 활용해서 할 수 있는 것
- USER가 앱/웹에 접속한 후, 어떤 화면으로 이동했는지 알 수 있음
  - 즉, 시간의 흐름에 따른 행동을 볼 수 있음. 다음 Row의 Page를 확인
- 한 USER의 앱 로그 상에서 직전 이벤트와 현재 이벤트가 동일한 것들을 필터링할 수 있음
  - 같은 페이지를 연속으로 접근한 경우 하나로 처리해서 퍼널을 구할 때 활용 가능
- 리텐션 쿼리를 작성할 때 기준점을 만들 수 있음. (유저의 첫 접속일)
<br><br>

## 5. 번호 함수
- 탐색 함수와 유사. 번호 지정(번호표를 주는 행위)를 하는 함수들

### ROW_NUMBER() : 행의 순서
- 정렬 기준에 따라 각 행에 고유 번호 부여

```
-- ex) dept 별 salary 많은 순서대로 순서 부여.  
-- 단, 동일 salary 나와도 순차적으로 부여. 
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC)
```
<br><br>

### RANK() : 순위
- 동일한 값은 동일 순위, 그 다음 순위는 건너뜀 (1, 2, 2, 4...)

```
-- ex) score 많은 순서대로 부여. 
-- 같은 score나오면 같은 숫자 부여. 다음 나오는 숫자는 숫자 하나 건너뛰고 부여. 
RANK() OVER (ORDER BY score DESC)
```
<br><br>

### DENSE_RANK() : rank와 비슷하지만 건너뛰지 않음
- 동일한 값은 같은 순위, 건너뛰지 않음 (1, 2, 2, 3...)

```
-- ex) score 많은 순서대로 부여. 
-- 같은 score나오면 같은 숫자 부여. 다음 나오는 숫자는 건너뛰지 않고 부여. (1, 2, 2, 3, ...)
DENSE_RANK() OVER (ORDER BY score DESC)
```
<br><br>

### NTILE(n) : n개의 구간으로 나눔
- n개의 구간으로 균등하게 행을 나눔

```
NTILE(4) OVER (ORDER BY score)
```
<br><br>

### ROW_NUMBER와 RANK의 차이
- 중복 값을 어떻게 처리하는 지의 차이
- `RANK` : 중복이 있으면 공동 N으로 처리하고 그 다음 값은 패스. 
  - 공동 2등이 있으면 3등은 없고 4등이 나옴.
- `ROW_NUMBER` : 중복이 있으면 랜덤으로 숫자 부여
<br><br>

### ROW_NUMBER와 RANK 중 어떤 것을 선택해야 할까?
- 파티션 내에서 고유한 순서가 필요한 경우 : `ROW_NUMBER`
- 파티션 내에서 동일한 값을 가진 행에 대해 동일한 순서를 부여해야 할 경우 : `RANK`
- 상위 30%의 그룹화를 위해 랭킹을 뽑고 싶은 경우 : `RANK`
<br><br>

- `ROW_NUMBER`를 사용하면 순서가 보장되지 않는 것 아닐까?
  - 특정 value 기준으로는 동일한 값이 생길 수 있고, 그런 경우 쿼리를 실행할 때마다 값이 달라질 수 있음
  - 이 경우 `ORDER BY`에 id 추가하면 고정된 값을 얻을 수 있음.

```
-- ex
ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salary DESC, id)
```
<br><br>

## 6. 집계 분석 함수
- `GROUP BY` 없이 누적 합/평균/최대값 등을 구할 수 있음
- 여러 값을 가지고 계산하는 함수
  - `AVG`, `COUNT`, `SUM`, `MAX`, `MIN`, ...
<br><br>

### 윈도우 함수 vs GROUP BY 비교
- GROUP BY : 여러 Row의 값을 집계해서 반환
- 윈도우 함수 : 각각의 Row에 값을 계산해서 반환
<br><br>

### SUM, AVG, COUNT, MIN, MAX
- 각각 누적 합계, 누적 평균, 누적 개수, 누적 최소 값, 누적 최대 값

```
-- ex) store 별 누적 매출액
SUM(sales) OVER (PARTITION BY store ORDER BY date)
```
<br><br>

## 7. 데이터 범위 지정 - Frame 설정
- 데이터의 범위를 제한해서 연산하고 싶은 경우
- 윈도우 안에서 실제로 계산 대상이 되는 행 범위 지정 (기본은 현재 행까지)
- 명시해야 하는 이유 : LAST_VALUE, AVG, RANK 등의 정확한 계산을 위해 꼭 필요
- ex) 이동 평균
  - 어떤 것이 방향성을 가지고 움직일 때, 이동하면서 구해지는 평균
  - 예를 들어, 특정 row에서 이전 3번째부터 이후 2번째의 데이터의 평균
- ex) rowdml 3개월 전 데이터부터 현재 row까지 합계
<br><br>

### Frame 설정 방법
#### 1) ROWS
- 물리적인 행의 수를 기준으로 경계를 지정
  - ex) 이전 행, 이후 3개의 행
- ROWS Frame를 더 많이 사용
<br><br>

#### 2) RANGE
- 논리적인 값의 범위를 기준으로 지정
  - ex) 값의 3일 전, 3일 후
<br><br>

#### Frame의 시작과 끝 지점 명시
- `PRECEDING` : 현재 행 기준으로 이전 행
  - ex) `UNBOUNDED PRECEDING` : 맨 처음 행부터 포함
- `CURRENT ROW` : 현재 행
- `FOLLOWING` : 현재 행 기준으로 이후 행
- `UNBOUNDED` : 처음부터 또는 끝까지(사전적 의미 : 묶이지 않고 제한되지 않음)
  - ex) `UNBOUNDED FOLLOWING` : 맨 마지막 행까지 포함

```
-- ex) 현재 행과 그 앞 뒤 한 행 씩을 포함해서 평균 
AVG(col) OVER (PARTITION BY product_type ORDER BY timestamp
ROWS BETWEEN 1 PRECEDING AND 1 FOLLOWING)

-- ex) 처음부터 현재 행까지의 평균
AVG(col) OVER (PARTITION BY product_type ORDER BY timestamp
ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW)
```
<br><br>

## 8. 윈도우 함수 조건 설정 - QUALIFY
### QUALIFY란?
- 윈도우 함수 결과에 조건을 거는 절
  - `WHERE`은 행 값에 조건
  - `HAVING`은 집계 결과에 조건
  - `QUALIFY`는 윈도우 함수 결과에 조건
- 즉, `RANK()`, `ROW_NUMBER()`, `DENSE_RANK()`같은 윈도우 함수를 필터링할 때 쓰는 전용 절
- MySQL, PostgreSQL은 지원하지 않음

```
-- 기본 형식
SELECT ...
  WINDOW ...
FROM your_table
(WHERE) 테이블 조건
QUALIFY <윈도우 함수 결과 조건>
```
<br><br>

### QUALIFY를 왜 쓸까?
- 기존 SQL에서는 윈도우 함수 결과를 필터링하려면 서브쿼리를 써야 했음

```
-- ex) 기존 방식
SELECT *
FROM (
  SELECT *,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_time DESC) AS rn
  FROM user_logs
)
WHERE rn = 1


-- ex) QUALIFY 적용
SELECT *
FROM user_logs
QUALIFY ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_time DESC) = 1
```
<br><br>

### 자주 쓰는 예제
- 1) 유저별 마지막 접속 기록

```
SELECT *
FROM user_logs
QUALIFY ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY login_time DESC) = 1;
```
<br><br>

- 2) 제품별 매출 상위 3위

```
SELECT *
FROM sales
QUALIFY RANK() OVER (PARTITION BY product_id ORDER BY sales_amount DESC) <= 3;
```
<br><br>

- 3) 유저별 가장 이른 구매 상품

```
SELECT user_id, product_id, purchase_time
FROM purchase_log
QUALIFY FIRST_VALUE(product_id) OVER (PARTITION BY user_id ORDER BY purchase_time) = product_id
```