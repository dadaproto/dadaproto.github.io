---
title: "[TIL] 엑셀보다 쉽고 빠른 SQL 3주차 (1)"
excerpt: "COUNT, WHERE, HAVING, REPLACE, SUBSTRING, CONCAT, LIMIT"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-19T20:00
---

## SQL 코드카타 복습 (COUNT, WHERE, HAVING)

Category: `SQL`

어제 SQL 코테 문제를 푸는데 `COUNT`의 개념이 좀 헷갈려서 문제도 다시 들여다보고, 개념에 대해서도 좀 더 찾아봤다.

> [SQL 문제: 동명 동물 수 찾기](https://school.programmers.co.kr/learn/courses/30/lessons/59041)

```sql
-- 동물 보호소에 들어온 동물 이름 중 두 번 이상 쓰인 이름과
-- 해당 이름이 쓰인 횟수를 조회하는 SQL문을 작성해주세요.
-- 이때 결과는 이름이 없는 동물은 집계에서 제외하며, 결과는 이름 순으로 조회해주세요.

SELECT NAME, COUNT(1) COUNT
FROM ANIMAL_INS
WHERE NAME IS NOT NULL
GROUP BY NAME
HAVING COUNT(1) >= 2
ORDER BY NAME
```

- 이 문제에서는 `COUNT`(1)로 작성할 때와 `COUNT`(NAME)으로 작성했을 때 결과가 같았다.

> COUNT, WHERE, HAVING

- `COUNT`(\*) 혹은 `COUNT`(1)은 전체 행의 갯수를 NULL을 포함해 카운트한다.
- `COUNT`(column_name)은 특정 컬럼의 NULL을 제외하고 카운트한다.
- `COUNT`(`distinct` column_name)은 특정 컬럼의 중복을 제외하고 카운트한다.
- `WHERE` 절은 SQL 쿼리에서 특정 조건을 만족하는 행을 필터링하는 데 사용된다. `WHERE` 절은 행 단위의 조건을 지정하여 특정 조건을 만족하는 행만을 결과로 반환한다. 반면에 `HAVING` 절은 집계 함수(`COUNT`)와 함께 그룹화된 결과에 조건을 적용하는 데 사용된다.

### COUNT(\*)과 COUNT(1)의 차이

- [Difference Between SQL COUNT(\*), COUNT(1), COUNT(column_name) In SQL Server](<https://www.dbblogger.com/post/difference-between-sql-count-count-1-count-column_name-in-sql-server#:~:text=The%20COUNT(*)returns%20the,total%20records%20in%20that%20table.&text=The%20COUNT(1)%20function%20replaces,is%20also%20replaced%20by%201>)
- ChatGPT의 답변

![Untitled](https://prod-files-secure.s3.us-west-2.amazonaws.com/31878edc-82e4-4e36-9b84-b78d5b5292cf/e7071baf-927a-4884-a45f-5b3b2e74c7dd/Untitled.png)

## 3-1 오늘 배울 것

Category: `SQL`

### 복습 과제

```sql
-- 음식 가격의 평균을
select cuisine_type, avg(price) avg_price, quantity
-- 주문 테이블에서
from food_orders
-- 주문 수량이 1건인 주문건의
where quantity = 1
-- 음식 종류별로 조회하여
group by cuisine_type
-- 음식 가격이 높은 순서대로 정렬하기
order by avg(price) desc
```

### 3주차 학습 목표

- 문자 데이터 포맷 변환
- 시간 포맷
- Data Type 오류 해결

## 3-2 업무 필요한 문자 포맷이 다를 때, SQL로 가공하기 (REPLACE, SUBSTRING, CONCAT)

Category: `SQL`

### 특정 문자를 다른 문자로 바꾸기 (REPLACE)

> 실습: 식당 명의 ‘Blue Ribbon’을 ‘Pink Ribbon’으로 바꾸기

```sql
select restaurant_name "원래 상점명",
		replace(restaurant_name, 'Blue', 'Pink') "바뀐 상점명"
from food_orders
where restaurant_name like '%Blue Ribbon%'
```

- `REPLACE`(column_name, old_string, new_string): 특정 컬럼의 특정 문자열을 다른 문자열로 바꾸는 함수
  | Results | 원래 상점명               | 바뀐 상점명               |
  | ------- | ------------------------- | ------------------------- |
  | 1       | Blue Ribbon Sushi Izakaya | Pink Ribbon Sushi Izakaya |
  | 2       | Blue Ribbon Fried Chicken | Pink Ribbon Fried Chicken |

> 실습: ‘문곡리’가 들어간 주소를 ‘문가리’로 바꿔보자.

```sql
select addr,
		replace(addr, '문곡리', '문가리') "바뀐 주소"
from food_orders
where addr like '%문곡리%'
```

- 해석: `REPLACE` 함수를 이용해 addr 컬럼에서 ‘문곡리’ 문자열을 ‘문가리’ 문자열로 변경하고 해당 컬럼의 별명을 “바뀐 주소”로 설정. 바뀐 주소만 파악할 수 있게 addr 컬럼에서 ‘문곡리’를 포함하는 데이터만 가져와라.
  | Results | addr                         | 바뀐 주소                    |
  | ------- | ---------------------------- | ---------------------------- |
  | 1       | 경기도 평택군 고덕면 문곡리  | 경기도 평택군 고덕면 문가리  |
  | 2       | 세종특별자치시 부강면 문곡리 | 세종특별자치시 부강면 문가리 |

### SUBSTRING, SUBSTR

> 실습

```sql
select addr "원래 주소",
		substr(addr, 1, 2) "시도"
from food_orders
where addr like '%서울특별시%'
```

- `SUBSTR`(column_name, start, length): 특정 컬럼의 데이터에서 문자열의 시작과 길이를 지정해 해당 문자열을 반환하는 함수
- `SUBSTRING`, `SUBSTR` 둘 다 사용 가능하다.

### CONCAT

> 실습: 서울시에 있는 음식점을 ‘[서울] 음식점명’ 이라고 수정

```sql
select restaurant_name "원래 이름"
		addr "원래 주소"
		concat('[', substr(addr, 1, 2), ']', restaurant_name) "바뀐 이름"
from food_orders
where addr like '%서울%'
```

- `CONCAT`(string1, string2, …): 여러 문자열을 결합하는 함수이다.
  - 특수문자는 작은 따옴표 씌워 표기
  - `SUBSTRING` 등 [문자열 함수](https://dev.mysql.com/doc/refman/8.0/en/string-functions.html)를 포함할 수 있다. (ex. `CHAR_LENGTH`,
  - column_name으로 column 데이터를 포함할 수 있다.

### 더 알아보기

> 🤔 `CONCAT`은 몇 개의 인수를 가질 수 있을까?

- 데이터베이스 시스템에 따라 다를 수 있지만, MySQL에서는 최대 255개의 인수를 가질 수 있다.

> 🤔 `SUBSTR` 함수를 잘 살펴보니 SQL은 문자열의 시작을 1부터 센다!

- Python, JavaScript, C, C++, JAVA 등은 0에서 시작한다. (0-based indexing)
- 인덱싱을 0부터 시작하는지, 1부터 시작하는지는 언어의 설계 및 표준에 따라 다르다.

> `CHAR_LENGTH`와 `LENGTH`

- `CHAR_LENGTH` 함수는 문자열의 문자 수를 반환한다. 이때, 멀티바이트 문자도 하나의 문자로 간주한다.
  ```sql
  SELECT CHAR_LENGTH('안녕하세요') AS char_length_result;
  -- 결과: 5
  ```
- `LENGTH` 함수는 문자열의 바이트 수를 반환한다. 이때, 멀티바이트 문자도 해당 문자의 바이트 수를 계산한다.
  ```sql
  SELECT LENGTH('안녕하세요') AS length_result;
  -- 결과: 15
  ```
  - UTF-8에서 각 문자가 3바이트이므로 총 15바이트이다.
- 영어 문자열에서는 일반적으로 두 함수의 결과가 같다.
  ```sql
  SELECT CHAR_LENGTH('Hello') AS char_length_result,
  		LENGTH('Hello') AS length_result;
  -- 결과: 5, 5
  ```

## 3-3 [실습] 문자 데이터 바꾸고, GROUP BY 사용하기

Category: `SQL`

> 서울 지역의 음식 타입별 평균 음식 주문 금액 구하기 (출력: ‘서울’, ‘타입’, ‘평균 금액’)

```sql
select substr(addr, 1, 2) "지역",
		cuisine_type "타입",
		avg(price) "평균 금액"
from food_orders
where addr like '%서울%'
group by cuisine_type
order by avg(price)
```

- 풀이
  - `SELECT` addr 컬럼 데이터의 첫번째 글자부터 두글자 추출, cuisine_type, price 평균
  - `FROM` food_orders 테이블
  - `WHERE` addr 컬럼에서 ‘서울’을 포함하는 데이터만
  - `GROUP BY` cuisine_type 카테고리로 분류
    ```sql
    -- select 컬럼 순 숫자로 표기할 수 있다
    -- group by substr(addr, 1, 2), cuisine_type와 동일

    group by 1, 2
    ```
  - `ORDER BY` 평균 금액 오름차순 정렬
- Results
  | 지역 | 타입          | 평균 금액 |
  | ---- | ------------- | --------- |
  | 서울 | Vietnamese    | 12,130    |
  | 서울 | Spanish       | 12,565    |
  | 서울 | Mediterranean | 14,678    |
  | …    | …             | …         |

> 이메일 도메인 별 고객 수와 평균 연령 구하기

```sql
select substr(email, 10) "도메인",
		count(*) "고객 수",
		avg(age) "평균 나이"
from customers
group by 1
```

- 풀이
  - `SELECT`
    - `substr`(email, 10): 현재 데이터에는 편의상 id가 모두 8글자이므로, 도메인 확인을 위해 10번째 글자를 시작으로 한다. 도메인의 길이가 각각 다를 때 (가져오고자 하는 문자열의 길이를 특정할 수 없을 때) 안전하게 큰 숫자를 입력하거나, 3번째 인수를 생략할 수 있다.
    - `count`(\*) 전체 행의 갯수를 센다.
    - `avg`(age) 평균 나이
  - `FROM` customers 테이블에서 데이터를 가져온다.
  - `GROUP BY` 1 즉 도메인으로 카테고리를 나눈다.
- Results
  | 도메인      | 고객 수 | 평균 나이 |
  | ----------- | ------- | --------- |
  | hanmail.com | 310     | 49.5323   |
  | daum.net    | 266     | 48.6015   |
  | mail.com    | 296     | 50.5473   |
  | naver.com   | 305     | 50.2525   |

> ‘[지역(시도)] 음식점 이름 (음식 종류)’ 컬럼을 만들고, 총 주문 건수 구하기

```sql
select concat('[', substr(addr, 1, 2), '] ', restaurant_name, ' (', cuisine_type, ')') "[지역] 음식점 (음식종류)",
		count(*) "주문 건수"
from food_orders
group by 1
order by 2 desc
```

- Results
  | [지역] 음식점 (음식종류)                    | 주문 건수 |
  | ------------------------------------------- | --------- |
  | [경기] Shake Shack (American)               | 131       |
  | [경기] Blue Ribbon Sushi (Japanese)         | 69        |
  | [경기] Blue Ribbon Fried Chicken (American) | 64        |
  | [경기] The Meatball Shop (Italian)          | 55        |
  | …                                           | …         |

## 코드카타에서 배운 것 (LIMIT)

### LIMIT

```sql
-- 예시
SELECT column1, column2, ...
FROM my_table
LIMIT row_count;

-- OFFSET과 함께 사용하면 특정 위치부터 선택할 수 있다.
SELECT *
FROM my_table
LIMIT 10 OFFSET 20;
```

- `LIMIT`는 `SELECT` 문의 일부로 사용되는 절(clause)로 결과 집합을 제한하는 데 사용되는 구문. 주로 특정 개수의 행을 선택하거나 특정 범위의 행을 선택할 때 사용된다.
