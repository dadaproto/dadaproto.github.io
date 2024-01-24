---
title: "[TIL] SQL 4주차 + SQL 연습 문제 복습"
excerpt: "SUBQUERY로 segmentation 실습, MONTH(), FLOOR()"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-24T21:10
---

## [실습] User Segmentation 과 조건별 수수료를 Subquery 로 결합해 보기

> 1. 음식점의 평균 단가별 segmentation을 진행하고, 그룹에 따라 수수료 연산하기 (수수료: 5000원 미만 0.05%, 20000원 미만 1%, 30000원 미만 2%, 30000 이상 3%)

```sql
-- 평균 단가 없이 주문 금액의 수수료 금액 결과

SELECT restaurant_name, price*fee_ratio fee
FROM
(
SELECT restaurant_name, price,
	CASE
		when price < 5000 then 0.005
		when price between 5000 and 19999 then 0.01
		when price between 20000 and 29999 then 0.02
		else 0.03
	END fee_ratio
FROM food_orders
) a
ORDER BY fee
```

```sql
-- 평균 단가별 수수료

SELECT restaurant_name, price_per_order*fee_ratio fee
FROM
(
SELECT restaurant_name, price_per_order,
	CASE
		when price_per_order < 5000 then 0.005
		when price_per_order between 5000 and 19999 then 0.01
		when price_per_order between 20000 and 29999 then 0.02
		else 0.03
	END fee_ratio
FROM
(
SELECT restaurant_name, avg(price/quantity) price_per_order
FROM food_orders
GROUP BY 1
) a
) b
```

- 🚨 주 쿼리에서 서브쿼리의 결과를 참조할 때, 서브쿼리의 SELECT 문에서는 반드시 컬럼명을 포함해야 한다.
  - 이 때 서브쿼리에 alias를 부여하지 않으면 ‘Unknown Field column_name(주 쿼리에서 참조하려고 한 서브쿼리의 컬럼명) in field list’ 와 같은 에러가 발생할 수 있다.
- 서브 쿼리가 포함된 쿼리를 읽을 때는 가장 안에 있는 서브 쿼리부터 읽는다.
- DBeaver에서 서브 쿼리 부분만 드래그하여 선택적으로 실행할 수 있다.

> 2. 음식점의 지역과 평균 배달시간으로 segmentation 하기 (CASE 문 사용하기 / 평균 배달 시간 조건: 20분, 30분, 30분 초과)

```sql
-- 해설 보지 않고 혼자 작성해 본 쿼리

SELECT restaurant_name, location, avg_delivery_time,
CASE
	WHEN avg_delivery_time <= 20 THEN 'fast'
	WHEN avg_delivery_time > 20 and avg_delivery_time <= 30 THEN 'normal'
	WHEN avg_delivery_time > 30 THEN 'slow'
END avg_deliver_speed
FROM
(
SELECT restaurant_name,
	substr(addr, 1, 2) location,
	avg(delivery_time) avg_delivery_time
FROM food_orders
GROUP BY 1
) a
```

어떤 조건이 선행 조건인지, 판단하고 쿼리 구조를 생각하는게 은근히 까다롭다고 느꼈다. 일단 해설을 보지 않고 주어진 조건별로 각각 쿼리를 짜보고 메인쿼리와 서브쿼리를 재배치하면서 최적의 상태를 찾아봤다. 다행히 alias 외에는 해설과 거의 같은 결과!

서브쿼리를 작성할 때는, 메인 테이블에서 베이스가 될 미니 테이블을 뽑고 그걸 바탕으로 쿼리를 전개해나간다는 개념으로 이해하면 될 것 같다.

## [실습] 복잡한 연산을 Subquery 로 수행하기

> 1. 음식 타입별, 지역별 총 주문 수량과 음식점 수를 연산하고, 주문 수량과 음식점 점포수 별 수수료율 산정하기 (점포수 ≥ 5, 주문수 ≥ 30 → 0.05 / 점포수 ≥ 5, 주문수 < 30 → 0.08 / 점포수 < 5, 주문수 ≥ 30 → 0.1 / 점포수 < 5, 주문수 < 30 → 0.02)

```sql
select cuisine_type, location, total_restaurant, total_quantity,
case
	when total_restaurant >= 5 and total_quantity >= 30 then 0.005
	when total_restaurant >= 5 and total_quantity < 30 then 0.008
	when total_restaurant < 5 and total_quantity >= 30 then 0.01
	when total_restaurant < 5 and total_quantity < 30 then 0.02
end fee
from
(
select cuisine_type,
	substr(addr, 1, 2) location,
	count(distinct restaurant_name) total_restaurant,
	sum(quantity) total_quantity
from food_orders
group by 1, 2
) a
```

- 음식 종류, 지역, 점포수, 주문수를 서브쿼리로 뽑고
- 메인쿼리에 CASE 조건문으로 수수료율 쿼리 작성

🤔 그런데 문제 조건 중에 ‘지역별’은 왜 있는거지…? 지역별이 있어 group by에 지역을 넣었는데 해설에서는 딱히 쓰이지 않았다.

> 2. 음식점의 총 주문수량과 주문 금액을 연산하고, 주문 수량을 기반으로 수수료 할인율 구하기 (할인 조건: 수량 ≤ 5 → 0.1, 수량 > 15 & 총 주문금액 ≥ 300000 → 0.005, else 0.01)

```sql
SELECT restaurant_name,
	total_quantity,
	total_price,
CASE
	when total_quantity <= 5 then 0.1
	when total_quantity > 15 and total_price >= 300000 then 0.005
	else 0.01
END discount_rate
FROM
(
SELECT restaurant_name,
	sum(quantity) total_quantity,
	sum(price) total_price
FROM food_orders
GROUP BY 1
) a
```

- 문제를 풀 때 세부 조건을 꼼꼼히 읽고 검토하는 것 잊지 말기!

## 프로그래머스 SQL 코테 연습 문제 복습

> [문제: 1. **3월에 태어난 여성 회원 목록 출력하기**](https://school.programmers.co.kr/learn/courses/30/lessons/131120)

```sql
SELECT MEMBER_ID,
	MEMBER_NAME,
	GENDER,
	DATE_FORMAT(DATE_OF_BIRTH, '%Y-%m-%d') DATE_OF_BIRTH
FROM MEMBER_PROFILE
WHERE MONTH(DATE_OF_BIRTH) = 3 AND GENDER = 'W' AND NOT TLNO IS NULL
ORDER BY MEMBER_ID
```

- MySQL에서는 일반적으로 문자열과 숫자를 비교할 때 자동 형변환을 수행한다. 예를 들어 ‘3’(문자열) 과 3(숫자)를 비교 연산하면 자동으로 문자열을 숫자로 변환하여 비교한다.
- WHERE 절에서 날짜 함수 `MONTH(DATE_OF_BIRTH)='3'` 이라고 작성했을 때도 같은 결과가 나오고 submit 했을 때도 correct가 떴는데, 정확히 쓰자면 `MONTH(DATE_OF_BIRTH)=3` 이라고 숫자로 표기해야 한다.

> [문제: 가격대 별 상품 개수 구하기](https://school.programmers.co.kr/learn/courses/30/lessons/131530)

주요 조건 중,

- 만원 단위의 가격대 별로 상품 개수를 출력하기
- 가격대 정보는 각 구간의 최소금액(10,000원 이상 ~ 20,000 미만인 구간인 경우 10,000)으로 표시하기

이 두 부분 때문에 재밌는 문제였다. 그 동안 CASE 조건문을 사용해 분류하는 경우가 많았는데, 구간을 나누는 숫자가 딱 떨어지기 때문에 조건을 일반화할 수 있어서 CASE 문 없이 수식으로 분류할 수 있었다.

```sql
SELECT FLOOR(PRICE/10000)*10000 PRICE_GROUP,
COUNT(*) PRODUCTS
FROM PRODUCT
GROUP BY PRICE_GROUP
ORDER BY PRICE_GROUP
```

- FLOOR()는 올림!
