---
title: "[TIL] 엑셀보다 쉽고 빠른 SQL 3주차 (2)"
excerpt: "조건문 (IF, CASE)"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-21T21:53
---

## 조건문

Category: `SQL`

### IF 함수

```sql
IF(condition, value_if_true, value_if_false)
```

- SELECT 문에서 조건부로 값을 선택하거나 표현하는 데 사용
- 주로 MySQL에서 사용되며, 간단한 조건에 따라 값을 선택할 때 사용
- 문자열 함수 `REPLACE`와 같이 사용했을 때 예시
  ```sql
  select addr "원래 주소",
         if(addr like '%평택군%', replace(addr, '문곡리', '문가리'), addr) "바뀐 주소"
  from food_orders
  where addr like '%문곡리%'
  ```
- 잘못된 데이터 gmail.com을 @gmail.com으로 수정하기 위해 `REPLACE`와 결합한 예시
  ```sql
  select substring(if(email like '%gmail%', replace(email, 'gmail', '@gmail'), email), 10) "이메일 도메인",
         count(customer_id) "고객 수",
         avg(age) "평균 연령"
  from customers
  group by 1
  ```

### CASE 함수

```sql
select
case
	when condition1 then result1
	when condition2 then result2
	...
	else result
end
```

- IF 함수처럼 SELECT 문에서 조건부로 값을 선택하거나 표현하는 데 사용
- 보다 복잡한 조건에 따라 값을 선택하고자 할 때 사용
- WHEN, THEN 조건을 사용

```sql
select
case
	when cuisine_type = 'Korean' then '한식'
	when cuisine_type in ('Japanese', 'Chinese') then '아시아'
	else '기타'
end "음식타입",
	cuisine_type
from food_orders
```

- CASE 문 전체를 SELECT 문이나 다른 문장 내에서 사용할 때, 그 결과에 alias를 지정할 수 있다.

```sql
select order_id,
       price,
       quantity,
       case when quantity=1 then price
            when quantity>=2 then price/quantity end "음식 단가"
from food_orders
```

- ELSE 는 생략 가능하다.

### 🤔 조건문의 쓰임은?

- 새로운 카테고리를 만들 때
- 연산식을 적용할 조건을 지정할 때
- 다른 문법 안에서 적용하기
  - IF, CASE 문 안에 문법이나 연산을 넣을 수도 있지만 다른 문법 안에 조건문을 넣을 수도 있다. ex. `CONCAT(IF(condition, value1, value2))`
