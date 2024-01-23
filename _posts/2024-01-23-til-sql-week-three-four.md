---
title: "[TIL] SQL 4주차 강의 시작 + SQL 연습 문제 복습"
excerpt: "데이터 타입 오류, CASTING, Subquery, JOIN"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-23T22:22
---

## Data Type Error

### 데이터 타입 오류가 발생하는 원인들

1. **데이터 타입 불일치**: 열에 저장된 값과 쿼리에서 비교하거나 조작하려는 값의 데이터 타입이 일치하지 않을 때 발생. 예를 들어, 문자열과 숫자를 비교하거나 연산하는 경우에 발생할 수 있다.

   ```sql
   -- 잘못된 예: 정수형 age를 문자열 '30'과 비교하여 오류 발생

   SELECT * FROM table1 WHERE age = '30'
   ```

2. **NULL 값 처리:** NULL 값이 예상되는 곳에서 NULL이 아닌 값이 사용되거나, 그 반대의 경우에도 데이터 타입 오류가 발생할 수 있다. NULL 값과 관련된 함수나 연산을 사용할 때 주의!

   ```sql
   -- 잘못된 예: 정수형을 NULL과 비교할 때는 'IS NULL' 사용해야 함

   SELECT * FROM table1 WHERE age = NULL
   ```

3. **캐스팅 오류:** 데이터 타입을 변환(Casting)하는 도중에 발생. 예를 들어, 문자열을 숫자로 변환할 때 숫자 형식이 아닌 문자열이 포함되어 있으면 오류가 발생할 수 있다.

   ```sql
   -- 잘못된 예: 문자열을 정수형으로 변환할 수 없기 때문에 에러 발생

   SELECT CAST('abc' as INT)
   ```

4. **날짜와 시간 형식:** 날짜나 시간 형식의 값들을 정확하게 입력하지 않았을 때 발생. 데이터베이스 시스템에 따라 날자와 시간 형식에 정확한 맞춤이 필요하다.

   ```sql
   -- 잘못된 예: name이 문자형(VARCHAR)일 때
   -- 날짜 형식으로 변환한 값과 비교하여 오류 발생

   SELECT *
   FROM table1
   WHERE DATE_FORMAT(NOW(), '%Y-%m-%d') = name
   ```

5. **집계 함수 사용:** 집계 함수를 사용할 때, 해당 함수의 반환 값의 데이터 타입을 확인해야 한다. 종종 그룹화된 열을 대상으로 하는 집계 함수의 결과는 해당 열의 데이터 타입이 아닐 수 있다.

   ```sql
   -- 잘못된 예: rating에 1~5 사이의 숫자 외에 'A' 등의 문자가
   -- 섞여있다고 가정했을 때, 이 상태에서 AVG 함수를 사용하면
   -- 문자열 'A'를 정수로 변환할 수 없기 때문에 오류 발생

   SELECT cuisine_type, AVG(rating) AS average_rating
   FROM food_orders
   GROUP BY cuisine_type
   ```

### 캐스팅(CASTING)이란?

> 데이터를 한 데이터 타입에서 다른 데이터 타입으로 변환하는 프로세스

```sql
-- 예시 1 숫자로 변경

SELECT
	avg(cast(if(rating='Not given', '1', rating) as decimal)) as avg_rate
FROM food_orders

-- 예시 2 문자로 변경

SELECT concat(restaurant_name, '-', cast(order_id as char))
FROM food_orders
```

- 예시 2에서 cast를 쓰지 않아도 실행 결과 값은 같다. MySQL에서는 CONCAT 함수로 문자열과 숫자를 합칠 때 자동으로 형 변환이 이루어지기 때문이다. 하지만 다른 데이터베이스 시스템에서는 제공되지 않는 기능일 수 있으므로 cast 기억하기~!

## 3주차 숙제

> 다음의 조건으로 배달시간이 늦었는지 판단하는 값을 만들기

- 주중: 25분 이상
- 주말: 30분 이상
  >

```sql
SELECT order_id, day_of_the_week, delivery_time,
CASE
	WHEN day_of_the_week='Weekday' AND delivery_time >= 25 THEN 'Late'
	WHEN day_of_the_week='Weekend' AND delivery_time >= 30 THEN 'Late'
	ELSE '-'
END AS "delivery_delay"
FROM food_orders
```

- 위 쿼리를 활용해 지역별 배달 지연 횟수 카운트 해보기

```sql
SELECT substr(addr, 1, 2) addr,
count(*)
FROM food_orders
WHERE
(day_of_the_week='Weekday' AND delivery_time >= 25)
or (day_of_the_week='Weekend' AND delivery_time >= 30)
GROUP BY 1
```

- Results
  경기 309
  서울 72
  대전 22
  부산 35
  광주 30
  인천 34
  울산 10
  대구 35
  세종 7

### CASE 문에서 생성된 컬럼을 WHERE 절에서 직접 참조할 수 있을까?

- 일반적으로 SELECT 문에서 작성한 CASE 조건문에서 생성된 컬럼을 WHERE 절에서 직접 참조할 수 없다. SQL 쿼리의 논리적 실행 순서 때문이다.
- SQL 쿼리의 논리적 실행 순서
  1. **`FROM`**: 데이터를 추출할 테이블이나 뷰를 지정
  2. **`WHERE`**: 조건을 설정하여 특정 행을 필터링
  3. **`SELECT`**: 선택된 열과 행을 추출
  4. **`GROUP BY`**: 그룹화를 통해 집계 함수를 사용
  5. **`HAVING`**: 그룹화된 데이터에 대한 조건을 지정
  6. **`ORDER BY`**: 정렬을 수행
- PostgreSQL에서는 CASE 문을 WHERE 절에서 사용할 수 있는 FILTER 절을 지원한다고 함.
- 경우에 따라 CASE 문을 사용하는 대신 필요한 조건을 WHERE 절에 직접 작성하는 것이 더 효율적일 수 있다.

## 여러 번의 연산을 한 번의 SQL 문으로 수행하기 (Subquery)

### Subquery란?

- SQL 문 내부에 포함된 또 다른 SQL 문이다. 주로 SELECT 문에서 사용하는 경우가 많지만 어떤 부분에서든 사용될 수 있다.
  ```sql
  SELECT column1, column2, ...
  FROM table_name
  WHERE column_name operator
  	(
  	SELECT column_name
  	FROM another_table
  	WHERE condition
  	) as subquery
  ```

### Subquery가 필요한 경우는?

- 여러번의 연산을 수행해야 할 때
- 조건문에 연산 결과를 사용해야 할 때
- 조건에 Query 결과를 사용하고 싶을 때

### Subquery를 사용했을 때의 장점은?

- 복잡한 조건 처리
- 가독성 향상
- 유연한 결과 제어
- 부분 집합 처리
- 동적 데이터 사용: 주 쿼리의 결과에 따라 동적으로 서브쿼리의 조건 설정 가능
- 복잡한 데이터 변환: 데이터를 변환하거나 특정 기준에 따라 계산된 값을 활용하여 주 쿼리에 적용할 수 있다.

### Subquery 사용시 고려할 부분은?

- 성능 문제: 중첩되거나 복잡한 로직 때문에 성능에 영향. 대용량 데이터셋을 활용할 때 비효율적인 서브쿼리가 쿼리의 실행 시간을 증가시킬 수 있다.
- 가독성 및 유지보수
- 조인(JOIN) 사용 가능한지 확인: 조인을 사용하는 것이 성능면에서 더 효율적일 수 있음. 특히 큰 테이블 간의 관계를 표현할 때~
- 인덱스 활용: 서브쿼리를 사용하면 주 쿼리에서 인덱스의 효율적 활용이 어려울 수 있다. 적절한 인덱스를 사용해 조인 조건을 설정하는 것이 성능 향상에 도움이 될 수 있다.
- 데이터베이스 종속성: 데이터베이스 시스템마다 서브쿼리 처리 방식이 다를 수 있다. 어떤 데이터베이스는 조인이 더 효과적, 어떤 데이터베이스는 서브쿼리가 더 효과적일 수 있다.
- 쿼리 최적화: 지금 이 순간 서브쿼리가 가장 적절한가? 생각해보기

### Subquery 예시

```sql
select order_id, restaurant_name, if(over_time>=0, over_time, 0) over_time
from
(
select order_id, restaurant_name, food_preparation_time-25 over_time
from food_orders
) a
```

- 서브쿼리에서 ‘food_preparation_time-25’ 에 ‘over_time’ alias를 붙여줬고, 이 값을 주 쿼리에서 가져다 쓸 수 있다.
- 서브쿼리를 쓰지 않았다면 if 조건문을 사용해야 해 쿼리가 훨씬 복잡해졌을 것이다.

## SQL 코테 연습 문제 복습 (JOIN, LEFT JOIN)

강의에서는 아직 진도가 나가지 않았지만, 홀린 듯 코테 연습 문제를 풀면서 새로운 개념을 선행 학습하고 있다. 오늘은 JOIN을 활용하는 문제가 많았다.

- JOIN은 데이터베이스에서 테이블끼리 연결한다.
- JOIN은 일치하는 행만 반환하고, 일치하는 행이 없는 경우 결과에서 제외한다.
- LEFT JOIN은 왼쪽 테이블의 모든 행을 포함하고, 오른쪽 테이블과 일치하는 행이 없는 경우에도 왼쪽 테이블의 모든 행이 결과에 포함된다. 오른쪽 테이블의 일치하지 않는 부분은 NULL 값으로 채워진다.

> [문제: 카테고리 별 도서 판매량 집계하기](https://arc.net/l/quote/kuqcvabx)

```sql
-- 1
SELECT b.category, sum(s.sales) total_sales
FROM book AS b
JOIN book_sales AS s USING (BOOK_ID)
WHERE year(s.sales_date) = 2022 AND month(s.sales_date) = 1
GROUP BY category
ORDER BY category

-- 2
SELECT category, sum(sales)
FROM book
JOIN book_sales using (book_id)
WHERE sales_date like '2022-01%'
GROUP BY category
ORDER BY category

-- 3
SELECT category, sum(sales)
FROM book
JOIN book_sales on book.book_id = book_sales.book_id
WHERE sales_date like '2022-01%'
GROUP BY category
ORDER BY category
```

- 조금씩 다르게 적어 Submit 했고, 결과는 셋 다 같다.
- JOIN한 테이블에 alias가 필요한 경우는 두 테이블에 같은 이름을 가진 컬럼이 있을 경우

### JOIN ON, JOIN USING

- `JOIN ON`
  - 특정 조건에 따라 두 테이블을 연결할 때 사용
  - 연결 조건이 복잡하거나 여러 컬럼 사용할 때 유용
  - `JOIN table2 on table1.column_name = table2.column_name`
- `JOIN USING`
  - 두 테이블 간의 특정 컬럼 이름이 같을 때 사용
  - 간단한 조인을 수행할 때 편리
  - `JOIN table2 using (column_name)`
