---
title: "[TIL] SQL 3주차! 그리고 SQL 문제 복습"
excerpt: "CASE, IF 조건문을 활용해 여러 조건에 따라 다른 수식 적용해보기 / IN 비교 연산자"
toc: true
categories:
  - WIL
tags:
  - WIL
last_modified_at: 2024-01-22T22:47
---

## 실습 - User Segmentation

> 10세 이상, 30세 미만의 고객의 나이와 성별로 그룹 나누기

```sql
select
case
	when (age between 10 and 19) and gender='male' then '10대 남성'
	when (age between 10 and 19) and gender='female' then '10대 여성'
	when (age between 20 and 29) and gender='male' then '20대 남성'
	when (age between 20 and 29) and gender='female' then '20대 여성'
END "고객 분류",
name, age, gender
from customers
where age between 10 and 29
```

- Results
  | 고객 분류 | name | age | gender |
  | --------- | ------ | --- | ------ |
  | 10대 남성 | 김도진 | 15 | male |
  | 10대 여성 | 정지은 | 15 | female |
  | 20대 남성 | 김하호 | 21 | male |
- CASE, WHEN, THEN, END 문법을 활용해 해당 조건에 해당하는 그룹을 나눠줄 수 있다.

> 음식 단가, 음식 종류 별로 음식점 그룹 나누기

```sql
-- 음식 종류만 나눠보기

SELECT
CASE
	WHEN cuisine_type='Korean' THEN '한식'
	WHEN cuisine_type in ('Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') THEN '아시아식'
	ELSE '기타'
END "음식 종류",
restaurant_name, IF(quantity = 1, price, price/quantity) "평균 금액"
FROM food_orders
```

- CASE 조건문을 사용해 음식 종류를 분류
- 기존의 price는 주문량인 quantity를 포함한 금액이므로, IF 조건문을 활용해 price를 quantity로 나누고 alias를 평균 금액으로 정의
- Results
  | 음식 종류 | cuisine_type | restaurant_name | 평균 금액 |
  | --------- | ------------ | ------------------------- | --------- |
  | 한식 | Korean | Hangawi | 30,750 |
  | 아시아식 | Japanese | Blue Ribbon Sushi Izakaya | 12,080 |
  | 기타 | Mexican | Cafe Habana | 12,230 |

```sql
-- 가상의 배달 음식 플랫폼의 메뉴 추천 페이지를 상상하면서 적어본 쿼리

SELECT
CONCAT(
CASE
	WHEN (price/quantity < 10000) THEN "만원의 행복"
	WHEN (price/quantity) BETWEEN 10000 AND 20000 THEN "2만원 이하"
	WHEN (price/quantity BETWEEN 20000 AND 40000) THEN "특별한 날"
END,
' ',
CASE
	WHEN cuisine_type='Korean' THEN '한식'
	WHEN cuisine_type='Japanese' THEN '일식'
	WHEN cuisine_type='Chinese' THEN '중식'
	WHEN cuisine_type in ('Thai', 'Vietnamese', 'Indian') THEN '아시안 요리'
	WHEN cuisine_type in ('American', 'Italian') THEN '양식'
	ELSE '기타'
END
) "추천 카테고리",
cuisine_type, restaurant_name, IF(quantity = 1, price, price/quantity) "평균 금액"
FROM food_orders
```

- 음식 카테고리와 가격 카테고리를 각각의 CASE 조건문으로 작성한 다음,CONCAT 문자열 함수로 두 개의 CASE 조건문을 결합해봤다. 이게 되네 헤헤. 중간에 깨알 띄어쓰기
- Results

  | 추천 카테고리          | cuisine_type | restrant_name      | 평균 금액 |
  | ---------------------- | ------------ | ------------------ | --------- |
  | 특별한 날 한식         | Korean       | Hangawi            | 30,750    |
  | 2만원 이하 아시안 요리 | Indian       | Anjappar Chettinad | 16,440    |
  | 만원의 행복 양식       | American     | Shake Shack        | 5,400     |

- 강의 코드 스니펫
  ```python
  select restaurant_name,
         price/quantity "단가",
         cuisine_type,
         order_id,
         case when (price/quantity <5000) and cuisine_type='Korean' then '한식1'
              when (price/quantity between 5000 and 15000) and cuisine_type='Korean' then '한식2'
              when (price/quantity > 15000) and cuisine_type='Korean' then '한식3'
              when (price/quantity <5000) and cuisine_type in ('Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') then '아시아식1'
              when (price/quantity between 5000 and 15000) and cuisine_type in ('Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') then '아시아식2'
              when (price/quantity > 15000) and cuisine_type in ('Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') then '아시아식3'
              when (price/quantity <5000) and cuisine_type not in ('Korean', 'Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') then '기타1'
              when (price/quantity between 5000 and 15000) and cuisine_type not in ('Korean', 'Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') then '기타2'
              when (price/quantity > 15000) and cuisine_type not in ('Korean', 'Japanese', 'Chinese', 'Thai', 'Vietnamese', 'Indian') then '기타3' end "식당 그룹"
  from food_orders
  ```

### 🤔 숫자를 처리하는 함수

> 실무에서 필요에 따라 소수점 이하를 생략하는 경우에 쓰일 만한 함수는 뭐가 있을까?

- ROUND(): 반올림
- CEIL() 또는 CEILING(): 올림
- FLOOR(): 내림

## 실습 - 조건에 따라 다른 수식 적용하기

> 지역과 배달 시간을 기반으로 배달 수수료 구하기

```sql
-- 해설 보기 전 직접 생각하며 적어본 쿼리

SELECT order_id,
	restaurant_name,
	substr(addr, 1, 2) "지역",
CEIL(
CASE
	WHEN delivery_time < 15 THEN price * IF(substr(addr, 1, 2)='서울', 0.06, 0.07)
	WHEN delivery_time between 15 and 20 THEN price * IF(substr(addr, 1, 2) in ('서울', '경기', '대전', '부산'), 0.07, 0.08)
	WHEN delivery_time between 21 and 25 THEN pric * 0.09
	WHEN delivery_time between 26 and 30 THEN price/quantity * 0.1
	ELSE price * 0.11
END) "수수료"
, delivery_time
, price
FROM food_orders
WHERE price between 15000 and 20000
ORDER BY 5, 4, addr
```

- 배달 시간이 15분 미만일 때, 지역이 서울이면 6%, 그 외 지역은 7%
  - IF 조건문 안에 substr 문자열 함수로 지역에 따라 수수료 달리 함
- 배달 시간이 15분 이상 20분 이하일 때, 지역이 서울, 경기, 대전, 부산이면 7%, 그 외 지역은 8%
- 배달 시간이 21분 이상 25분 이하일 때, 수수료 9%
- 배달 시간이 26분 이상 30분 이하일 때, 수수료 10%
- 그 외 조건이면 수수료 11%

```sql
-- 강의 자료 추가 조건 적용
-- 지역: 서울, 기타 - 서울은 시간 계산 * 1.1
-- 시간: 25분, 30분 - 25분 초과시 음식 가격의 5%, 30분 초과시 10%

SELECT order_id,
restaurant_name,
substr(addr, 1, 2) "지역",
CASE
	WHEN delivery_time > 30 THEN price * 0.1 * if(addr like %서울%), 1.1, 1)
	WHEN delivery_time > 25 THEN price * 0.05
	ELSE 0
END fee
FROM food_orders
ORDER BY 5 DESC
```

- fee 에서 WHEN delivery_time 30을 먼저 적으면 30분 초과의 데이터를 필터링한 이후 25분 초과 조건을 처리한다.

> 주문 시기와 음식 수를 기반으로 배달 할증료 구하기

```sql
-- 해설 보기 전 직접 생각하며 적어본 쿼리 1

SELECT
CASE
	WHEN day_of_the_week="Weekend" THEN price * 0.1 *
		CASE
			WHEN quantity > 3 THEN price * 0.05
			ELSE price * 0.02
		END
	WHEN day_of_the_week="Weekday" THEN price * 0.05 *
		CASE
			WHEN quantity > 3 THEN price * 0.05
			ELSE price * 0.02
		END
END fee,
quantity,
day_of_the_week "주문 시기"
FROM food_orders
```

- 주말에는 수수료 10%, 주문양이 3 초과일 때는 수수료 5%, 그 외 2%
- 평일에는 수수료 5%, 주문양이 3 초과일 때는 수수료 5%
- 중복이 많은 것 같아서 다시 도전!

```sql
-- 직접 생각하며 적어본 쿼리 222222

SELECT
CASE
	WHEN quantity > 3 THEN price * 0.05 *
	CASE
		WHEN day_of_the_week="Weekend" THEN 0.1
		WHEN day_of_the_week="Weekday" THEN 0.05
	END
	ELSE price * 0.02
END fee,
quantity,
day_of_the_week "주문 시기"
FROM food_orders
```

- 중복은 줄였지만 원하는 잘못 쓴 쿼리이다. ELSE에 들어간 가격은 주말과 주중 할증료를 반영하지 않는다.

```sql
-- 직접 생각하며 적어본 쿼리 33333

SELECT
CASE
	WHEN day_of_the_week="Weekend" THEN price * 0.1 * IF(quantity>3, 0.05, 0.02)
	WHEN day_of_the_week="Weekday" THEN price * 0.05 * IF(quantity>3, 0.05, 0.02)
END fee,
quantity,
day_of_the_week "주문 시기"
FROM food_orders
ORDER BY quantity desc
```

- 처음 작성한 쿼리와 흐름은 비슷한데 CASE 대신 IF 조건문을 사용했다.

자 그럼 이제 강사님의 쿼리를 살펴보자.

```sql
-- 추가 조건
-- 주문 시기: 평일 기본료 = 3000 / 주말 기본료 = 3500
-- 음식 수: 3개 이하면 할증 없음 / 3개 초과면 기본료 * 1.2

SELECT
CASE
	WHEN day_of_the_week="Weekday" THEN 3000 * IF(quantity>3, 1.2, 1)
	WHEN day_of_the_week="Weekend" THEN 3500 * IF(quantity>3, 1.2, 1)
END "배달할증료",
restaurant_name,
order_id,
day_of_the_week,
quantity
FROM food_orders
```

- 세번째로 작성해본 쿼리와 같은 구조였다. 굿!

## SQL 코테 연습 문제 복습

오늘은 13개의 문제를 풀었다! 야호! 며칠 동안 공부한 개념들이 계속 나와서 대부분 어렵지 않게 풀 수 있었는데, 마지막 문제에서 에러를 몇 번 봤다. 바로 `WHERE` 문에서 `IN` 비교 연산자를 사용했기 때문인데, 처음에 `WHERE ~ IN (’통풍시트’, ’열선시트’, 가죽시트’)` 이렇게 적어서 에러가 난 것이었다.

[[문제] 자동차 종류 별 특정 옵션이 포함된 자동차 수 구하기](https://school.programmers.co.kr/learn/courses/30/lessons/151137)

```sql
-- Solutions

SELECT CAR_TYPE, COUNT(*)
FROM CAR_RENTAL_COMPANY_CAR
WHERE OPTIONS like '%통풍시트%' or OPTIONS like '%열선시트%' or OPTIONS like '%가죽시트%'
GROUP BY CAR_TYPE
ORDER BY CAR_TYPE
```

- WHERE OPTIONS IN (a, b, c) 사용할 수 없는 이유
  - SQL에서 WHERE IN은 리스트에 지정된 값 중 하나와 정확히 일치하는 경우만 필터링하기 때문
  - 리스트에서 지정한 옵션 외에 데이터에 다른 옵션이 포함되어 있으면 결과값에 포함하지 않음

그 밖에 실행은 잘 됐지만 틀린 결과인 경우들이 있었는데, 정렬 조건이나 그룹 조건 등을 기재하지 않은 경우였다. 연습 문제라는 생각 때문에 Submit 버튼을 누르고 싶은 충동이 마구 일지만 앞으로 진짜 코테라 생각하고 문제를 꼼꼼히 읽고, 결과 값도 차근차근 검토한 후에 Submit하는 것을 목표로 해야겠다.
