---
title: "[TIL] 엑셀보다 쉽고 빠른 SQL 2주차"
excerpt: "SUM, AVG, COUNT, DISTINCT, MIN, MAX, GROUP BY, ORDER BY"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-18T21:44
---

## 2-1 2주차 학습 목표

Category: `SQL`

- 두 컬럼의 합계 값, 전체 데이터의 갯수 구하기
- 범주별 계산하기
- 데이터 오름차순, 내림차순 정렬하기

## 2-2 엑셀 대신 SQL로 한번에 계산하기 (SUM, AVERAGE, COUNT, MIN, MAX)

Category: `SQL`

> SUM, AVERAGE

### 숫자 연산 예시

```sql
SELECT food_preparation_time,
		delivery_time,
		food_preparation_time + delivery_time as total_time
FROM food_orders
```

- 연산자 +, -, \*, / 등 사용 가능

### 합계와 평균 구하기

```sql
select sum(food_preparation_time) total_food_preparation_time,
		avg(delivery_time) avg_food_preparation_time
from food_orders
```

- 함수
  - 합계: `SUM(column)`
  - 평균: `AVG(column)`
- 풀이
  - food_preparation_time column의 `SUM` 합계를 구하고 alias를 total_food_preparation_time으로 설정
  - delivery_time column의 `AVG` 평균을 구하고 alias를 avg_food_preparation_time으로 설정

### SUM, AVG 실습

```sql
-- customers 테이블에서 age의 평균값을 구하고,
-- 해당 column의 alias를 average_of_age로 설정하기

SELECT AVG(age) as average_of_age
FROM customers

-- food_orders 테이블에서 price의 합계를 구하고,
-- 해당 column의 alias를 total_price로 설정하기

SELECT SUM(price) as total_price
FROM food_orders
```

> COUNT

### 갯수 구하기 예시

```sql
SELECT COUNT(1) count_of_orders,
	COUNT(distinct customer_id) count_of_customers
FROM food_orders
```

- `COUNT(*)` 혹은 `COUNT(1)`은 테이블 안의 모든 데이터의 갯수를 카운트하라는 의미이다.
- `COUNT(distinct column_name)`에서 `distinct`는 해당 column_name의 고유한 데이터 갯수를 카운트하라는 의미이다. 즉, 중복을 제거하는 함수이다.

### 갯수 구하기 실습

```sql
-- payments 테이블의 전체 데이터 갯수를 구하고, alias를 total_count로 설정
SELECT COUNT(1) as total_count
FROM payments

-- payments 테이블의 pay_type column의 고유한 데이터 갯수를 구하고,
-- alias를 count_of_pay_type으로 설정
SELECT COUNT(DISTINCT pay_type) as count_of_pay_type
FROM payments
```

> MIN, MAX

### 최대값, 최소값 구하기 예시

```sql
SELECT MIN(price) min_price,
			 MAX(price) max_price
FROM food_orders
```

- `MIN(column_name)` column_name의 데이터 값 중 최소값을 구하라.
- `MAX(column_name)` column_name의 데이터 값 중 최대값을 구하라.

### 최대값, 최소값 구하기 실습

```sql
-- food_orders 테이블에서 quantity column의 최소값과 최대값을 구하고,
-- 각 column의 alias를 min_quantity와 max_quantity로 설정하기

SELECT MIN(quantity) as min_quantity,
			 MAX(quantity) as max_quantity
from food_orders
```

## 2-3 [실습] WHERE 절로 원하는 데이터를 뽑고, 계산해보기

Category: `SQL`

### 1. [실습] 주문 금액이 30,000원 이상인 주문건의 갯수 구하기

> 🚀 Query를 작성하기 전, _1. 흐름을 정리하고, 2. 구문으로 만든 후, 3. 전체 구조로 합친다._

> 흐름 정리하기

1.  어떤 테이블에서 데이터를 가져올까? → food_orders 테이블에서 가져오자.
2.  어떤 컬럼을 이용할까? → 주문 금액이니 price 컬럼을 이용하자.
3.  어떤 조건을 지정해야 할까? → 30,000원 이상
4.  어떤 함수(수식)을 이용해야 할까? → 주문건의 갯수를 구해야하니 `COUNT` 함수

> 구문으로 만들기

1.  `FROM` food_orders
2.  `SELECT` price
3.  `WHERE` price `>=` 30000
4.  `SELECT` `COUNT`(price) → `COUNT(1)`도 가능

> 전체 구조로 합치기

```sql
SELECT COUNT(price) as "cnt_orders"
FROM food_orders
WHERE price >= 30000
```

| Results | cnt_orders |
| ------- | ---------- |
| 1       | 100        |

### 2. [실습] 한국 음식의 주문 당 평균 음식가격 구하기

```sql
SELECT AVG(price) avg_kr_price
FROM food_orders
WHERE cuisine_type = 'Korean'
```

| Results | avg_kr_price |
| ------- | ------------ |
| 1       | 14001.5385   |

## 2-4 범주별 연산을 한 번에 끝내기 (GROUP BY)

Category: `SQL`

### GROUP BY 예시

```sql
select cuisine_type,
			sum(price) sum_of_price
from food_orders
group by cuisine_type
```

- 해석: food_orders 테이블에서, cuisine_type 컬럼으로 그룹 지어(`GROUP BY`) 각 cuisine_type 데이터에 해당하는 price 컬럼 데이터의 합계(`SUM`)를 구하라.

  | Results | cuisine_type | sum_of_price |
  | ------- | ------------ | ------------ |
  | 1       | Korean       | 182,020      |
  | 2       | Japanese     | 7,663,130    |
  | 3       | Mexican      | 1,303,850    |
  | 4       | American     | 9,530,780    |

> ### 실습해보자! 👩🏻‍💻

#### 음식점별 주문 금액 최댓값 조회하기

```sql
select cuisine_type,
	    max(price) as max_price
from food_orders
group by cuisine_type
```

| Results | cuisine_type              | sum_of_price |
| ------- | ------------------------- | ------------ |
| 1       | Hangawi                   | 30,750       |
| 2       | Blue Ribbon Sushi Izakaya | 33,030       |
| 3       | Cafe Habana               | 31,380       |
| 4       | Blue Ribbon Fried Chicken | 19,400       |
| 5       | Dirty Bird to Go          | 29,390       |

#### 결제 타입별 가장 최근 결제일 조회하기

```sql
select pay_type,
	    max(date) as recent_date
from payments
group by pay_type
```

| Results | pay_type | recent_date |
| ------- | -------- | ----------- |
| 1       | card     | 2021-03-21  |
| 2       | cash     | 2021-05-06  |

## 2-5 Query 결과를 정렬하여 업무에 바로 사용하기 (ORDER BY)

Category: `SQL`

> `ORDER BY` 함수를 사용하면 데이터가 변경되거나 삭제되어도 동일한 순서로 출력할 수 있다!

### ORDER BY 예시

```sql
select cuisine_type,
		sum(price) sum_of_price
from food_orders
group by cuisine_type
order by sum(price)
```

- `ORDER BY` 특정 컬럼을 기준으로 정렬해줘!
- `SUM(price)` 오름차순으로 정렬된다.
- `ORDER BY` default는 오름차순. `ASC` Ascending
- 내림차순은 `DESC` Descending

> ### 실습해보자! 👩🏻‍💻

#### 1. 음식점별 주문 금액 최댓값 조회하기 - 최댓값 기준으로 내림차순 정렬

```sql
select restaurant_name,
		max(price) max_price
from food_orders
group by restaurant_name
order by max(price) desc
```

| Results | restaurant_name   | max_price |
| ------- | ----------------- | --------- |
| 1       | Pylos             | 35,410    |
| 2       | Han Dynasty       | 34,190    |
| 3       | Blue Ribbon Sushi | 33,370    |
| 4       | Nobu Next Door    | 33,370    |

#### 2. 고객을 이름 순으로 오름차순으로 정렬하기

```sql
select *
from customers
order by name
```

| Results | customer_id | name   | email                     | gender | age |
| ------- | ----------- | ------ | ------------------------- | ------ | --- |
| 1       | 232359      | 강도연 | http://hdcjcgshgmail.com/ | female | 78  |
| 2       | 110461      | 강도연 | mailto:lnldqztx@naver.com | male   | 5   |
| 3       | 105068      | 강도연 | mailto:hgmdmpnp@naver.com | female | 49  |
| 4       | 117810      | 강도원 | http://wqmpzcqbgmail.com/ | male   | 85  |

```sql
select *
from customers
order by gender, age, name
```

- 정렬 기준은 여러 컬럼이 될 수 있다. 쉼표로 구분한다.

| Results | customer_id | name   | email                     | gender | age |
| ------- | ----------- | ------ | ------------------------- | ------ | --- |
| 1       | 301618      | 강윤진 | mailto:wiviukmg@daum.net  | female | 1   |
| 2       | 220693      | 김서우 | mailto:orylqykh@daum.net  | female | 1   |
| 3       | 42274       | 박도호 | http://rpbuxguggmail.com/ | female | 1   |
| 4       | 374826      | 박민서 | mailto:iplshijb@daum.net  | female | 1   |

> 데이터에 동명이인이 많아 호기심으로 `COUNT` 함수를 써봤다.

```sql
select count(distinct name)
from customers
-- 중복되는 이름을 걸러내면 724명
```

| Results | count(distinct name) |
| ------- | -------------------- |
| 1       | 724                  |

## 2-6 SQL 구조 마스터 - WHERE, GROUP BY, ORDER BY로 완성되는 SQL 구조

Category: `SQL`

### SQL 문의 기본 구조

```sql
select
from
where
group by
order by
```

### 구조 맞춰보기 퀴즈! 🚀

```sql
-- Quiz. 1
select cuisine_type,
        sum(delivery_time) total_delivery_time
from food_orders
where day_of_the_week='Weekend'
group by cuisine_type
order by sum(delivery_time) desc
```

- 해석: food_orders 테이블에서, cuisine_type 컬럼과 delivery_time 컬럼 데이터의 합계를 각각 가져오는데 delivery_time 합계 컬럼의 별명은 total_delivery_time으로 정하겠다. day_of_the_week이 Weekend 인 조건을 가져와라. 가져온 데이터는 cuisine_type 컬럼으로 분류하고, delivery_time 합계의 내림차순으로 정렬하라.

  | Results | cuisine_type | total_delivery_time |
  | ------- | ------------ | ------------------- |
  | 1       | American     | 9355                |
  | 2       | Japanese     | 7544                |
  | 3       | Italian      | 4700                |
  | 4       | Chinese      | 3630                |
  | …       | …            | …                   |
  | 14      | Vietnamese   | 100                 |

```sql
-- Quiz. 2
select age, count(name) count_of_name
from customers
where age between 20 and 40
group by age
order by age
```

- 해석: customers 테이블에서 age 컬럼과 name 컬럼을 가져오는데, name 컬럼은 데이터의 개수를 센 값을 가져오고, 별명을 count_of_name으로 하겠다. age가 20 이상 40 이하인 조건인 경우만 가져와라. age로 분류하고, age 오름차순으로 정렬해라.

  | Results | age | count_of_name |
  | ------- | --- | ------------- |
  | 1       | 21  | 13            |
  | 2       | 22  | 27            |
  | 3       | 23  | 8             |
  | 4       | 24  | 13            |
  | …       | …   | …             |
  | 18      | 39  | 9             |

## 2-7 2주차 끝 & 숙제 안내

Category: `SQL`

> ✅ 음식 종류별 가장 높은 주문 금액과 가장 낮은 주문 금액을 조회하고, 가장 낮은 주문 금액 순으로 (내림차순) 정렬하기

```sql
select cuisine_type,
		min(price) min_price,
		max(price) max_price
from food_orders
group by cuisine_type
order by min(price) desc
```

- `select` 에 min, max 만 가져오라고 할 경우, 결과값에 min, max 컬럼에 1개 데이터씩 1열만 생성되어 내림차순 정리가 의미가 없어지므로 cusine_type 컬럼으로 분류해본다.

  | Results | cuisine_type | min_price | max_price |
  | ------- | ------------ | --------- | --------- |
  | 1       | Spanish      | 12,130    | 29,100    |
  | 2       | French       | 11,980    | 29,250    |
  | 3       | Southern     | 7,380     | 31,430    |
  | 4       | Thai         | 6,690     | 32,930    |
  | …       | …            | …         | …         |
  | 14      | Japanese     | 4,470     | 33,370    |
