---
title: "[TIL] SQL 5주차 (1)"
excerpt: "L유효하지 않은 데이터 처리 방법, Pivot Table"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-26T20:24
---

## 조회한 데이터에 아무 값이 없다면?

- 사례
  - 숫자형 데이터에 문자열 데이터(ex. not given)가 포함된 경우
  - JOIN 했을 때 NULL 데이터가 포함되어 있는 경우
- 해결 방법
  - if 조건문을 사용해 해당 값을 연산에서 제외하기 → NULL, 0 등으로 변환
  - where 절에서 NULL 데이터가 포함된 행 제외하기
  - 사용할 수 없는 값 대신 다른 값을 대체해서 사용
    - 다른 값이 있을 때: if(rating≥1, rating, 대체값)
    - NULL 값일 때 coalesce(age, 대체값)

💡 짚고 가기! ‘같지 않다’ : ≠, <>, NOT

> COALESCE

COALESCE 함수는 여러 값 중에서 첫 번째로 나오는 NULL이 아닌 값을 반환하는 함수이다. 즉, 여러 값 중에서 첫 번째로 NULL이 아닌 값을 찾아주는 역할을 한다.

```sql
-- 예시
SELECT COALESCE(column_name, 'Default') AS result
FROM your_table;
```

- **`column_name`**이 NULL이면 'Default'를 반환하고, NULL이 아니면 실제 값을 반환한다. 따라서 NULL 값에 대해 대체(default) 값을 설정하는 용도로 사용할 수 있다.

## 조회한 데이터가 상식적이지 않은 값을 가지고 있을 때?

- 사례
  - 고객의 나이가 주로 20대 이상 50대 이하인데 2세 등의 데이터가 있는 경우
  - 주문 일자가 주로 2020~2024년인데 1920년 등의 데이터가 있는 경우
- 해결 방법
  - 조건문으로 값의 범위 지정하기
  ```sql
  -- 예시
  select name, age,
  CASE
  	when age < 15 then 15
  	when age >= 80 then 80
  	else age
  END re_age
  from customers
  ```

## 실습: SQL로 Pivot Table 만들어보기

### 피봇 테이블(Pivot Table)이란?

**Pivot Table**은 표 형태의 데이터를 원하는 형식으로 재구성하는 것을 의미한다. (일반적으로 행에서 열로 변환!) 데이터베이스에서는 **`PIVOT`**이라는 명령문을 사용하여 피벗 테이블을 생성할 수 있다. 피벗 테이블은 주로 집계된 데이터를 다양한 관점에서 쉽게 분석하기 위해 사용된다. (ex. 보고용 요약 테이블)

> 음식점별 시간별 주문건수 Pivot Table 뷰 만들기 (15~20시 사이, 20시 주문건수 기준 내림차순) → IF 문으로 만들기

```sql
-- 해답 쿼리
select restaurant_name,
	max(if(hh='15', cnt_order, 0)) "15",
	max(if(hh='16', cnt_order, 0)) "16",
	max(if(hh='17', cnt_order, 0)) "17",
	max(if(hh='18', cnt_order, 0)) "18",
	max(if(hh='19', cnt_order, 0)) "19",
	max(if(hh='20', cnt_order, 0)) "20"
from
(
select f.restaurant_name,
	substr(p.time, 1, 2) hh,
	count(1) cnt_order
from food_orders f
inner join payments p on f.order_id = p.order_id
where substr(p.time, 1, 2) between 15 and 20
group by 1, 2
) a
group by 1
order by 7 desc
```

- 서브쿼리에서 substr으로 시간값, count로 주문수를 뽑아서 이 데이터를 주 쿼리에서 사용했다.
- INNER JOIN을 사용하지 않으면, 두 테이블 간에 **`order_id`**가 일치하는 행만 선택되기 때문에 각 주문에 대한 시간 정보를 함께 사용할 수 없게 되어 원하는 결과를 얻을 수 없게 된다.
- Results

  | restaurant_name           | 15  | 16  | 17  | 18  | 19  | 20  |
  | :------------------------ | :-- | :-- | :-- | :-- | :-- | :-- |
  | Shake Shack               | 13  | 7   | 6   | 9   | 3   | 9   |
  | The Meatball Shop         | 4   | 2   | 7   | 3   | 4   | 9   |
  | Blue Ribbon Fried Chicken | 5   | 4   | 2   | 4   | 4   | 5   |
  | Han Dynasty               | 1   | 0   | 3   | 0   | 3   | 5   |
  | Parm                      | 2   | 2   | 3   | 4   | 2   | 4   |
  | TAO                       | 6   | 3   | 0   | 3   | 0   | 4   |
  | Blue Ribbon Sushi         | 4   | 7   | 4   | 1   | 4   | 4   |
  | Osteria Morini            | 0   | 1   | 1   | 0   | 1   | 3   |

> 성별, 연령별 주문 건수 Pivot Table 뷰 만들기 (나이는 10~59세 사이, 연령 순으로 내림차순)

```sql
select age,
	max(if(gender="male", cnt_order, 0)) "male",
	max(if(gender="female", cnt_order, 0)) "female"
from
(
select gender,
case
	when age between 10 and 19 then 10
	when age between 20 and 29 then 20
	when age between 30 and 39 then 30
	when age between 40 and 49 then 40
	when age between 50 and 59 then 50
end age,
	count(1) cnt_order
from food_orders f
inner join customers c on f.customer_id = c.customer_id
where age between 10 and 59
group by 1, 2
) a
group by 1
order by 1 desc
```

- 해설 보기 전에 세로축을 성별, 가로축을 연령으로 나눴었다. 다만 그렇게 했을 때는 max(if()) 구문을 age 기준으로 작성하기 때문에 쿼리가 좀 더 길어졌었다.
- 피봇테이블을 만들 때 메인 쿼리의 첫번째 컬럼명이 세로축, if문 안으로 들어가는 컬럼들이 세로축이 된다.
