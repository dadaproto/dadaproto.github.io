---
title: "[TIL] SQL 4주차 끝! + SQL 연습 문제 복습"
excerpt: "LEFT JOIN, INNER JOIN, SEGMENTATION 실습, AVG(), WHERE 절에서 서브쿼리"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-25T20:25
---

## 필요한 데이터가 서로 다른 테이블에 있을 때 조회하기 (JOIN)

- JOIN은 엑셀의 Vlookup과 유사하다.
- 두 테이블이 공통으로 갖고 있는 컬럼을 기준으로 필요한 값을 가져온다.
- 공통 컬럼을 기준으로 두 테이블을 합쳐서 각 테이블ㄴ에서 필요한 데이터를 조회할 수 있다.
- LEFT JOIN: 공통 컬럼 (키값) 을 기준으로 하나의 테이블에 값이 없더라도 모두 조회된다.
- INNER JOIN: 공통 컬럼 (키값) 을 기준으로 두 테이블에 모두 있는 값만 조회한다.

```sql
-- 기본 형태
select *
from table_a
join table_b on table_a.column1 = table_b.column1

-- table에 alias 설정
select *
from table_a a
join table_b b on a.column1 = b.column1

-- 컬럼을 불러올 때
select a.column_a, b.column_b
from table_a a
join table_b b on a.column1 = b.column1
```

## 실습: JOIN 으로 두 테이블의 데이터 조회하기

> 1. 한국 음식의 주문별 결제 수단과 수수료율 조회하기

- 조회 컬럼: 주문 번호, 식당 이름, 주문 가격, 결제 수단, 수수료율
- 결제 정보가 없는 경우도 포함하여 조회 (LEFT JOIN)
  >

```sql
select f.order_id, f.restaurant_name, f.price, p.pay_type, p.vat
from food_orders f
left join payments p on f.order_id = p.order_id
where f.cuisine_type = 'Korean'
```

> 2. 고객의 주문 식당 조회하기

- 조회 컬럼: 고객 이름, 연령, 성별, 주문 식당
- 고객명으로 정렬, 중복 없도록 조회
  >

```sql
-- 해설 없이 작성해본 코드

select c.name, c.age, c.gender, f.restaurant_name
from customers c
left join food_orders f on c.customer_id = f.customer_id
group by c.customer_id
order by name
```

- 🤔 조인하는 테이블의 순서에 따라 어떤 차이가 있을까?
  - 일반적으로는 테이블 순서가 결과에 영향을 미치지 않지만 LEFT JOIN 혹은 RIGHT JOIN을 사용하는 경우 순서가 중요할 수 있다. 또 이름이 같은 테이블이 있는 경우에도 순서에 따라 결과에 영향을 미칠 수 있다.

```sql
-- 해설

select distinct c.name, c.age, c.gender, f.restaurant_name
from food_orders f
left join customers c on f.customer_id = c.customer_id
order by c.name
```

- 해설처럼 쿼리를 작성하고 실행했을 때는 customer_id가 NULL인 데이터도 반환했다.
- 해설 쿼리에서 LEFT JOIN을 INNER JOIN으로 변경하면 NULL 데이터는 뜨지 않는다.
- customers 테이블을 왼쪽에 배치했을 때도 NULL 데이터는 뜨지 않는다.
- SELECT 문에서도 `distinct` 를 활용해 중복을 제거할 수 있다.

## 실습: JOIN으로 두 테이블의 값 연산하기

> 1. 주문 가격과 수수료율을 곱하여 주문별 수수료 구하기 (수수료율이 있는 경우만 조회)

```sql
select f.order_id,
	f.restaurant_name,
	f.price,
	p.vat,
	(f.price * p.vat) fee
from food_orders f
inner join payments p on f.order_id = p.order_id
```

-

> 2. 50세 이상 고객의 연령에 따라 경로 할인율을 적용하고, 음식 타입별로 원래 가격과 할인 적용 가격 합을 구하기 (할인: 나이-50\*0.005, 고객 정보 없는 경우 포함, 할인 금액 큰 순서대로 정렬)

```sql
select a.cuisine_type, sum(a.price), sum(discount_result), sum(a.price - discount_result) discount
from
(
select f.cuisine_type, f.price,
f.price - f.price * (c.age - 50) * 0.005 as discount_result
from food_orders f
left join customers c on f.customer_id = c.customer_id
where c.age >= 50
) a
group by 1
order by 4 desc
```

- 일반적으로 곱하기 연산할 때처럼 column_name(a+b) 라고 표기하면 에러가 난다. 함수 호출로 인식하기 때문인 것 같다. column_name\* (a+b) 로 연산자를 반드시 포함할 것!
- JOIN으로 생성된 테이블이 서브쿼리 안에 있을 때 메인 쿼리에서 컬럼을 호출하려면 subquery_alias.column_name 으로 표기하거나 그냥 column_name으로 표기

## 4주차 끝! 숙제!

> 식당별 평균 음식 주문 금액과 주문자의 평균 연령을 기반으로 Segmentation 하기

- 평균 음식 주문 금액 기준: 5000 / 10000 / 30000 / 30000 < a
- 평균 연령: ~20대 / ~30대 / ~40대 / 50대 ≤ b
- 두 테이블 모두에 데이터가 있는 경우만 조회, 식당 이름 순으로 오름차순 정렬

```sql
select restaurant_name,
	case
		when avg_price <= 5000 then 'pg1'
		when avg_price > 5000 and avg_price <= 10000 then 'pg2'
		when avg_price > 10000 and avg_price <= 30000 then 'pg3'
		when avg_price > 30000 then 'pg4'
	end price_range,
	case
		when age <= 29 then 'ag1'
		when age between 30 and 39 then 'ag2'
		when age between 40 and 49 then 'ag3'
		when age >= 50 then 'ag4'
	end age_range
from
(
select restaurant_name,
	avg(price) avg_price,
	avg(age) age
from food_orders f
inner join customers c on f.customer_id = c.customer_id
group by 1
) t
order by 1
```

- 평균 구할 때는 avg()!

## 프로그래머스 SQL 코테 연습 문제

> [문제: 즐겨찾기가 가장 많은 식당 정보 출력하기](https://school.programmers.co.kr/learn/courses/30/lessons/131123#qna)

```sql
-- 결과는 같지만 틀렸다고 채점된 쿼리
select food_type,
	rest_id,
	rest_name,
	max(favorites) favorites
from rest_info
group by food_type
order by favorites desc

-- 수정한 쿼리
select food_type,
	rest_id,
	rest_name,
	favorites
from rest_info
where (food_type, favorites) in
    (
        select food_type, max(favorites)
        from rest_info
        group by food_type
    )
order by food_type desc
```

두 쿼리의 실행 결과가 같은데 왜 틀릴까? 아무리 뜯어봐도 모르겠어서 해설을 찾아봤다. 첫번째 쿼리에서는 ‘food_type’과 ‘max(favorites)’는 유효한 값을 출력하지만 ‘rest_id’와 ‘rest_name’은 행의 첫번째 값을 반환할 수 있다는 것. 즉 데이터에 따라 데이터가 섞인 결과값이 나올 수 있다는 것으로 이해했다.

```sql
-- ChatGPT가 작성한 쿼리
SELECT
    food_type,
    rest_id,
    rest_name,
    favorites
FROM
    rest_info r1
WHERE
    (food_type, favorites) IN (
        SELECT
            food_type,
            MAX(favorites) AS max_favorites
        FROM
            rest_info r2
        WHERE
            r1.food_type = r2.food_type
        GROUP BY
            food_type
    )
ORDER BY
    food_type DESC;
```

> [문제: 식품분류별 가장 비싼 식품의 정보 조회하기](https://school.programmers.co.kr/learn/courses/30/lessons/131116)

```sql
select category,
    price,
    product_name
from food_product p1
where (category, price) in
    (
        select category, max(price)
        from food_product p2
        where p1.category = p2.category
    ) and category in ('과자', '국', '김치', '식용유')
order by 2 desc
```

바로 유사한 문제가 나와서 복습할 수 있었다. 굿~

- where 절에서 조건에 맞는 테이블을 만들어 기존 테이블에 조인하는 개념
