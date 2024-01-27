---
title: "[TIL] SQL 5주차 (2) SQL 기초를 뗐다!"
excerpt: "Window Functions, Date Formats, Pivot View practice"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-27T22:54
---

## Window Function (윈도우 함수)

- [MySQL 공식 문서 Window Functions](https://dev.mysql.com/doc/refman/8.0/en/window-functions.html)

### 기본 구조

```sql
<window_function>(<expression>) over (partition by 그룹 기준 컬럼 order by 정렬 기준)
```

- window_function: `SUM`, `AVG`, `ROW_NUMBER` 등
- window function 과 `over` 는 짝꿍!
- expression: 계산에 사용되는 컬럼 또는 표현식
- `partition by`: `group by` 기능. 특정 컬럼 값을 기준으로 데이터를 그룹화
- `order by`: 윈도우 함수의 계산에 영향을 미침
- 윈도우 함수를 사용하면 특정 열에 대한 누적 합계, 이동 평균, 순위 등을 계산할 수 있으며, 데이터 분석 및 리포팅 작업에서 유용하게 활용된다.

### 실습 1. N번째까지의 대상을 조회하고 싶을 때, Rank

- `RANK` 특정 기준으로 순위를 매겨주는 기능. RANK 함수는 인수를 필요로 하지 않는다. ex. `RANK() OVER (PARTITION BY` ~

> 음식 타입별로 주문 건수가 가장 많은 상점 3개씩 조회하기

```sql
select cuisine_type,
	restaurant_name,
	cnt_order,
	ranking
from
(
	select cuisine_type,
		restaurant_name,
		cnt_order,
		rank() over (partition by cuisine_type order by cnt_order desc) ranking
	from
	(
		select cuisine_type,
			restaurant_name,
			count(1) cnt_order
		from food_orders
		group by 1, 2
	) a -- 음식 종류별, 식당별 주문 수
) b -- 윈도우 함수 RANK 를 사용해 a 결과에 대한 순위 반환
where ranking between 1 and 3 -- b 결과에서 3위까지 반환
```

- Results
  | cuisine_type | restaurant_name | cnt_order | ranking |
  | :----------- | :-------------------------- | :-------- | :------ |
  | American | Shake Shack | 219 | 1 |
  | American | Blue Ribbon Fried Chicken | 96 | 2 |
  | American | Five Guys Burgers and Fries | 29 | 3 |
  | Chinese | RedFarm Broadway | 59 | 1 |
  | Chinese | RedFarm Hudson | 55 | 2 |
  | Chinese | Han Dynasty | 46 | 3 |
  | French | Balthazar Boulangerie | 10 | 1 |
  | French | LExpress | 6 | 2 |
  | French | Le Grainne Cafe | 2 | 3 |

### 실습 2. 전체에서 차지하는 비율, 누적합 구하기 SUM

- `SUM` 누적합이 필요하거나 카테고리별 합계 컬럼과 원본 컬럼을 함께 이용할 때 유용하다.

> 각 음식점의 주문건이 해당 음식 타입에서 차지하는 비율을 구하고, 주문건이 낮은 순으로 정렬 했을 때 누적 합 구하기

```sql
select cuisine_type,
	restaurant_name,
	cnt_order,
	sum(cnt_order) over (partition by cuisine_type) sum_cuisine,
	sum(cnt_order) over (partition by cuisine_type order by cnt_order) cum_cuisine
from
(
select cuisine_type,
	restaurant_name,
	count(1) cnt_order
from food_orders
group by 1, 2
) a
group by 1, cnt_order
```

## 날짜 형식과 조건 FORMAT

- `date()` : 시간을 무시하고 날짜 부분만 문자열로 반환 (ex. date(’2024-01-27 20:06:10’) → ‘2024-01-27’)
- `date_format()` : 날짜를 특정 형식으로 포맷팅. 문자열로 반환 (ex. date_format(’2024-01-27 20:06:10’, ‘%Y-%m-%d %H’:%i:%s) → ‘2024-01-27 20:06:10’)
  - date_format(’2024-01-27’, ‘%w’) → 6 (토요일)
  - %w 은 요일(weekday)이며 일요일을 0으로 반환한다. (일:0, 월:1, 화:2, 수:3…)

### 실습 - 연도별 3월의 주문건수 구하기

```sql
-- 내가 작성한 쿼리
select date_format(date, '%Y') year,
	date_format(date, '%m') month,
	count(1) cnt_orders
from payments
where date_format(date, '%m') = 3
group by 1, 2
order by 1

-- 해설 쿼리
select date_format(date(date), '%Y') year,
	date_format(date(date), '%m') month,
	date_format(date(date), '%Y%m') yearmonth,
	count(1) cnt_order
from food_orders f
inner join payments p on f.order_id = p.order_id
where date_format(date(date), '%m') = 3
group by 1, 2, 3
order by 1
```

food_orders에 있는 order_id 등을 같이 조회해야하는 조건이 없어서 join 없이 작성했다.

## SQL 마지막 숙제! 야호!

> 음식 타입별, 연령별 주문건수 pivot view 만들기

```sql
-- 내가 작성한 쿼리
select cuisine_type,
	max(if(age='10', cnt, 0)) '10',
	max(if(age='20', cnt, 0)) '20',
	max(if(age='30', cnt, 0)) '30',
	max(if(age='40', cnt, 0)) '40',
	max(if(age='50', cnt, 0)) '50'
from
(
select cuisine_type,
case
	when age between 10 and 19 then '10'
	when age between 20 and 29 then '20'
	when age between 30 and 39 then '30'
	when age between 40 and 49 then '40'
	when age between 50 and 59 then '50'
end age
, count(1) cnt
from food_orders f
inner join customers c on f.customer_id = c.customer_id
where age between 10 and 59
group by 1, 2
) a
group by 1
```

```sql
-- 정답 쿼리
-- 내가 작성한 쿼리
select cuisine_type,
	max(if(age=10, cnt, 0)) '10대',
	max(if(age=20, cnt, 0)) '20대',
	max(if(age=30, cnt, 0)) '30대',
	max(if(age=40, cnt, 0)) '40대',
	max(if(age=50, cnt, 0)) '50대'
from
(
select cuisine_type,
case
	when age between 10 and 19 then 10
	when age between 20 and 29 then 20
	when age between 30 and 39 then 30
	when age between 40 and 49 then 40
	when age between 50 and 59 then 50
end age
, count(1) cnt
from food_orders f
inner join customers c on f.customer_id = c.customer_id
where age between 10 and 59
group by 1, 2
) a
group by 1
```

- 서브쿼리 WHERE 절에서 when then 결과값을 숫자로 한 것과 문자열로 정의한 것이 달랐다. age 데이터는 원래 숫자이기 때문에 내가 작성한 것처럼 when then 결과 값을 ‘10’ 처럼 문자열로 정하면 불필요한 형변환이 일어날 수 있다. 숫자로 결과 값을 정해야 데이터 타입이 일치한다. 또한 숫자로 저장하면 비교 연산이나 정렬이 문자열보다 더 빠르게 처리된다.
- 🚨 **조건문을 작성할 때는 참조하려는 데이터 타입에 유의하여 작성하기!**
  - 특히 나이와 같은 수치적인 값은 보통 숫자이므로 숫자로 표현하는 것이 더 효율적인 방법이다.
