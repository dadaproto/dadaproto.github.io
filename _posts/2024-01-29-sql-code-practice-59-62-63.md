---
title: "[TIL] SQL - 어쩌다 보니 자동차 문제 모음!"
excerpt: "MAX() in SELECT, DATEDIFF + 1, Order of execution of a Query"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-29T20:55
---

## 자동차 대여 기록에서 대여중 / 대여 가능 여부 구분하기

```sql
select car_id,
max(case
    when '2022-10-16' between start_date and end_date then '대여중'
    else '대여 가능'
end) availability
from car_rental_company_rental_history
group by 1
order by 1 desc
```

해당 쿼리에서 **`MAX`** 함수가 사용된 이유는 **`CASE`** 문의 결과가 '대여중' 또는 '대여 가능' 두 가지 값 중 하나이기 때문이다. 이 **`CASE`** 문은 '2022-10-16'이 해당 차량의 대여 기간에 속하는지 여부를 확인하고, 결과로 '대여중' 또는 '대여 가능' 중 하나를 반환한다. 따라서 각 **`car_id`**에 대해 이 결과 중 최댓값을 찾아 대여 가능 여부를 나타낸다.

이렇게 하면 각 차량에 대해 '대여중'이 하나라도 있으면 해당 차량은 '대여중'으로 표시되고, 그렇지 않으면 '대여 가능'으로 표시된다. 이러한 접근 방식은 대여 기간 중 어떤 시점에라도 대여 중인 경우를 확인하기 위해 사용될 수 있다.

## 자동차 대여 기록에서 장기/단기 대여 구분하기

```sql
SELECT HISTORY_ID,
    CAR_ID,
    DATE_FORMAT(START_DATE, '%Y-%m-%d') START_DATE,
    DATE_FORMAT(END_DATE, '%Y-%m-%d') END_DATE,
CASE
   WHEN DATEDIFF(END_DATE, START_DATE) + 1 >= 30 THEN '장기 대여'
   ELSE '단기 대여'
END RENT_TYPE
FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY
WHERE DATE_FORMAT(START_DATE, '%Y-%m') = '2022-09'
ORDER BY HISTORY_ID DESC
```

평소에는 당연한데 막상 코드 쓸 때 갑자기 바보 된 것처럼 실수하는 날짜 계산

- 날짜 계산할 때 이후 날짜 - 이전 날짜 `+1` (이 +1 을 안해서 한참 해맸던…)

그 밖에 처음에 쿼리를 쓸 때 메인 쿼리 부분을 깔끔하게 쓰고 싶어서 모든 와랄라랄(?) 한 부분을 다 서브 쿼리로 보냈었는데, 그럴 때 서브 쿼리에 연산이 있고 같은 연산을 메인 쿼리에서 반복해야 하는 경우라면 중복 연산을 피하기 위해 와랄라 해도 메인 쿼리로 빼는 것이 낫다.

## 자동차 평균 대여 기간 구하기

```sql
SELECT CAR_ID,
    AVERAGE_DURATION
FROM
(
    SELECT CAR_ID,
        ROUND(AVG(DATEDIFF(END_DATE, START_DATE)+1), 1) AVERAGE_DURATION
    FROM CAR_RENTAL_COMPANY_RENTAL_HISTORY
    GROUP BY 1
) A
WHERE AVERAGE_DURATION >= 7
ORDER BY 2 DESC, 1 DESC
```

SELECT 문에서 집계 함수가 쓰였을 때 WHERE 조건절에서 해당 집계 함수를 조건절로 사용하면 에러가 난다. (HAVING 에는 쓸 수 있음) 이는 이전에 기록했듯이 SQL에서 쿼리가 실행되는 순서가 ‘FROM에서 테이블 선택 → WHERE 에서 조건 필터링 → SELECT 에서 필터링 된 값을 호출’ 이렇게 되기 때문. 하지만 서브쿼리 A에서 쓰인 집계 함수는 메인쿼리에서 AVERAGE_DURATION이라는 컬럼명과 해당 값이 SELECT 문에서 호출되어 반환되기 때문에 WHERE 절에서 컬럼명으로 사용 가능해진다.
