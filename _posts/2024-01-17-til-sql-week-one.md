---
title: "[TIL] 엑셀보다 쉽고 빠른 SQL 1주차"
excerpt: "SQL이란? 데이터베이스와 대화를 하기 위한 언어이다."
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-17T18:00
---

## 1-1 오늘 배울 것

Category: `SQL`

- SQL이란?
  - 데이터베이스와 대화를 하기 위한 언어이다.
  - SQL이 데이터베이스에게 값을 요청하기 위한 문법이라고 한다면, Query는 요청하는 질의를 의미한다.
- 강의 목표
  - _SQL로 데이터를 뽑아보자!_
  - SQL의 기본 구조와 문법
  - 실습을 통해 SQL을 사용해보기
  - 숙제 잊지 말기!
- 강의를 들을 때 다짐
  - 복잡해 보여도 일단 시작해보기
  - 실수하거나 SQL 문이 실행되지 않아도 실망하거나 포기하지 않

## 1-2 함께 시작하기

Category: `SQL`

- SQL 실습 프로그램: 🦫[**DBeaver**](https://dbeaver.io/download/) → Community Version

  ![Screenshot 2023-12-20 at 3.03.40 PM.png](/docs/assets/images/Screenshot_2023-12-20_at_3.03.40_PM.png){: .align-center width="70%"}

- Connection Settings 설정

- `Test Connection` → Download MySQL driver files

  ![Screenshot](/docs/assets/images/Screenshot_2023-12-20_at_3.11.05_PM.png){: .align-center width="70%"}

  ![Screenshot](/docs/assets/images/Screenshot_2023-12-20_at_3.11.22_PM.png){: .align-center width="70%"}

- 데이터베이스는 ‘데이터가 저장 되어있는 큰 폴더’이다.
- ‘데이터베이스’ 폴더 내에 ‘테이블’이라는 파일이 있다.
- 엑셀과 비슷하게 생긴 ‘테이블’

  ```sql
  -- 예시
  Database
    ㄴ table
    -- food_orders
      ㄴ column
      -- order_id, customer_id, restaurant_name, ...
  ```

## 1-3 SQL 데이터 조회하기

Category: `SQL`

- SQL 기본 명령어 익히기 (`Select`, `From`)

  - Open SQL Script

    ```sql
    select *
    from food_orders

    -- select는 데이터를 가져오는 기본 명령어, 데이터를 조회하는 모든 Query에 사용됨
    -- *은 모든 컬럼을 가져온다
    -- from은 데이터를 가져올 테이블을 특정해주는 문법
    ```

  - 스크립트 실행 단축키: `^ + enter`
  - 직접 해보자! 👩🏻‍💻

    ```sql
    -- payments 테이블 데이터 조회하기
    select *
    from payments

    -- customers 테이블 데이터 조회하기
    select *
    from customers
    ```

## 1-4 필요한 항목만 뽑아서 사용하기

Category: `SQL`

- column을 지정해 필요한 데이터만 조회하기

  ```sql
  select restaurant_name, addr
  -- ex. select column1, column2
  from food_orders
  ```

- column의 별명(alias) 주기

  ```sql
  select restaurant_name as "음식점", addr address
  from food_orders

  -- column1 as alias1
  -- column2 alias2

  select order_id as ord_no,
         restaurant_name "식당 이름"
  from food_orders
  ```

  - 🚨 alias 유의 사항

    | 구분 | 영문, 언더바 | 특수문자, 한글                  |
    | ---- | ------------ | ------------------------------- |
    | 방법 | 별명만 적기  | “별명”으로, 큰 따옴표 안에 적기 |
    | 예시 | ord_no       | “ord no”, “주문번호”            |

---

### SQL - Case Sensitive or Insensitive?

> 🚀 DBeaver에서 tab으로 자동 완성 기능을 사용했을 때, select가 SELECT로 대문자 처리되는데 강의에서는 항상 소문자로 표기하여 Case 관련 내용을 찾아보았다.

- _SQL은 대소문자를 구별하지 않는다(Case Insensitive)._
  _다만, 레코드 내용은 대소문자를 구분한다._
- Stackoverflow에서 [14년 전에 작성된 관련 스레드](https://stackoverflow.com/questions/608196/why-should-i-capitalize-my-sql-keywords-is-there-a-good-reason)를 찾아 댓글을 읽어봤다.

  - 키워드를 소문자로 작성하는게 더 보기 좋다는 의견도 많은데, 대문자로 표기하는 것이 가독성 면에서 월등하다는 댓글도 많이 보였다.
  - 문법 상으로는 문제가 없으며 개인의 선호 차이.
  - 대문자 표기하는 사람은 40 넘었다는 댓글 ㅋㅎㅎ
  - 대문자 표기 방식으로 잘 쓰여진 SQL 구문을 예시로 든 댓글이 인상적이었다.

---

- 직접 해보자! 👩🏻‍💻

  ```sql
  -- 1
  SELECT order_id ord_no, price "가격", quantity "수량"
  FROM food_orders

  -- 2
  SELECT name "이름", email "e-mail"
  FROM customers

  -- e-mail은 언더바가 아니므로 큰 따옴표 안에 적는다.
  ```

  ![# 1](/docs/assets/images/Screenshot_2023-11-20_at_6.13.29_PM.png){: .align-center width="40%"}
  ![# 2](/docs/assets/images/Untitled.png){: .align-center width="40%"}

## 1-5 조건에 맞는 데이터 필터링 하기 (WHERE)

Category: `SQL`

- WHERE 문법을 사용해 필요한 데이터만 가져올 수 있다.
  - 🚨 주의할 점! 값을 입력할 때 문자열은 작은 따옴표 씌워 표기! 값은 대소문자 구분함!
- 고객 테이블에서 나이가 21세인 고객만 필터링
  ```sql
  SELECT *
  FROM customers
  WHERE age = 21
  ```
  ![Screenshot](/docs/assets/images/Screenshot_2024-01-17_at_5.09.49_PM.png){: .align-center width="70%"}
- 고객 테이블에서 성별이 남자인 데이터만 필터링
  ```sql
  SELECT *
  FROM customers
  WHERE gender = 'male'
  ```
  ![Screenshot](/docs/assets/images/Screenshot_2024-01-17_at_5.24.58_PM.png){: .align-center width="70%"}
- 주문 내역 테이블에서 한식 데이터만 필터링
  ```sql
  SELECT *
  FROM food_orders
  WHERE cuisine_type = 'Korean'
  ```
  ![Screenshot](/docs/assets/images/Screenshot_2024-01-17_at_5.24.22_PM.png){: .align-center width="70%"}
- 결제 방식 테이블에서 카드 결제 데이터만 필터링
  ```sql
  SELECT *
  FROM payments
  WHERE pay_type='card'
  ```
  ![Screenshot](/docs/assets/images/Screenshot_2024-01-17_at_5.16.32_PM.png){: .align-center width="70%"}

## 1-6 필터링을 할 때 유용한 표현 알아보기 (비교연산, BETWEEN, IN, LIKE)

Category: `SQL`

- 비교 연산자
  - a `>=` b : a는 b보다 크거나 같다
  - a `<>` b : a는 b와 같지 않다
  - a `>` b : a는 b보다 크다
  - a `<` b : a는 b보다 작다

```sql
-- 고객의 나이가 21세 이상인 조건으로 필터링
SELECT *
FROM customers
WHERE age>=21
```

![Screenshot 2024-01-17 at 5.27.24 PM.png](/docs/assets/images/Screenshot_2024-01-17_at_5.27.24_PM.png){: .align-center width="70%"}

- SQL 자체에서 제공되는 다양한 조건 필터
  - `BETWEEN`: A와 B 사이
  - `IN`: ‘포함’하는 조건 추가
  - `LIKE`: 완전히 똑같지는 않지만, 비슷한 값을 조건으로 주기
- `BETWEEN` → `BETWEEN` a `and` b
  ```sql
  -- 고객의 나이가 21세 이상 23세인 조건으로 필터링
  SELECT *
  FROM customers
  WHERE age BETWEEN 21 and 23
  ```
  ![Screenshot 2024-01-17 at 5.38.08 PM.png](/docs/assets/images/Screenshot_2024-01-17_at_5.38.08_PM.png){: .align-center width="70%"}
- `IN` → `IN` (1, 2, 3) / `IN` (’a’, ‘b’) 문자열은 작은 따옴표 표기
  ```sql
  -- 고객의 나이가 21세, 25세, 27세인 조건으로 필터링
  SELECT *
  FROM customers
  WHERE age in (21, 25, 27)
  ```
  ![Screenshot 2024-01-17 at 5.39.36 PM.png](/docs/assets/images/Screenshot_2024-01-17_at_5.39.36_PM.png){: .align-center width="70%"}
  ```sql
  -- 고객의 이름이 김하원, 정현준인 조건으로 필터링
  SELECT *
  FROM customers
  WHERE name in ('김하원', '정현준')
  ```
  ![Screenshot 2024-01-17 at 5.41.39 PM.png](/docs/assets/images/Screenshot_2024-01-17_at_5.41.39_PM.png){: .align-center width="70%"}
- `LIKE` → `LIKE` ’시작문자%’ / `LIKE` ’%포함문자%’ / `LIKE` ’%마침문자’

  ```sql
  -- 성이 '김'씨인 조건으로 필터링
  SELECT *
  FROM customers
  WHERE name like '김%'

  -- 이름에 '민'이 포함되는 조건으로 필터링
  WHERE name LIKE '%민%'

  -- 이름이 '호'로 끝나는 조건으로 필터링
  WHERE name LIKE '%호'
  ```

- 실습해보자!

  ```sql
  -- 나이가 40세 이상인 조건으로 필터링
  SELECT *
  FROM customers
  WHERE age>=40

  -- 주문 금액이 15,000 이하인 조건으로 필터링
  SELECT *
  FROM food_orders
  WHERE price<=17000

  -- 주문 금액이 20,000 이상 30,000 사이
  SELECT *
  FROM food_orders
  WHERE price BETWEEN 20000 and 30000

  -- B로 시작하는 상점의 주문 조회
  SELECT *
  FROM food_orders
  WHERE restaurant_name LIKE 'B%'
  ```

## 1-7 여러 개의 조건으로 필터링하기 (논리연산)

Category: `SQL`

- 여러 필터링 조건을 하나의 Query 문에서 적용할 수 있다.
- 여러 조건을 적용할 때 사용되는 연산이 ‘논리연산’이다.
- 논리연산의 종류

| 논리연산자 | 의미   | 예시                                                     |
| ---------- | ------ | -------------------------------------------------------- |
| AND        | 그리고 | age>20 and gender=’female’ → 나이가 20세 이상이고, 여성  |
| OR         | 또는   | age>20 or gender=’female’ → 나이가 20세 이상이거나, 여성 |
| NOT        | 아닌   | not gender=’female’ → 여성이 아닌                        |

- `AND` 논리연산자 사용하기

  ```sql
  -- 21세 이상 남성
  SELECT *
  FROM customers
  WHERE age>=21
  AND gender='male'

  -- 30000원 이상 한식
  SELECT *
  FROM food_orders
  WHERE cuisine_type='KOREAN'
  AND price >= 30000
  ```

- `OR` 논리연산자 사용하기

  ```sql
  -- 결제 내역 테이블에서 카드 결제이거나 부가세 0.2 이하인 조건
  SELECT *
  FROM payments
  WHERE pay_type='card'
  OR vat <= 0.2
  ```

## 1-8 에러메세지에 당황하지 않고 스스로 문제 해결해보기

Category: `SQL`

- 에러는 고수도 언제나 겪는 것! 에러메시지를 만날 때마다 잘 읽고 해결해보자!
- 🔥 자주 만날 수 있는 에러메세지
  - 테이블 명을 다르게 적었을 때 → _테이블이 존재하지 않는다._
  - 컬럼 명을 다르게 적었을 때 → _필드명이 잘못 되었다._
  - 필터링 조건에서 문자열에 작은 따옴표를 씌우지 않았을 때 → _‘문자열’에 문제가 있다._

## 1-9 1주차 끝 & 숙제 안내

Category: `SQL`

> ✅ 상품 준비시간이 20~30분 사이인, 한국 음식점의 식당명과 고객번호 조회하기

- Hint

  - 조회해야 할 컬럼 특징
  - ‘사이’ 조건 : `BETWEEN`
  - 특정 조건 지정 : =
  - 복수의 조건 지정 : `AND`

- 도전!

  ```sql
  SELECT restaurant_name "식당명", customer_id "고객번호"
  FROM food_orders
  WHERE food_preparation_time BETWEEN 20 AND 30
  AND cuisine_type = 'Korean'
  ```

- 스스로 정답 풀이 👩🏻‍💻
  - 특정 column 조회 → `SELECT` column_name
  - 테이블 선택 → `FROM` table_name
  - 사이 조건 지정 → `WHERE` column_name `BETWEEN` 20 `AND` 30
  - 복수 조건 연결 → `WHERE` 조건1 `AND` 조건2

## 더 알아보기

Category: `SQL`

- [**What is MySQL?**](https://dev.mysql.com/doc/refman/8.0/en/what-is-mysql.html)
  - **MySQL is a database management system.**
    - A database is a structured collection of data. ex) shopping list, picture gallery, etc.
    - To **add**, **access**, and **process** data stored in a computer database, you need a database management system such as MySQL server.
    - Database Management System play a central role in computing, as standalone utilities, or as part of other applications.
  - **MySQL databases are relational.**
    - The logical model, with objects such as databases, tables, views, rows, and columns, offers a flexible programming environment.
    - SQL stands for Structured Query Language. most common standardized language used to access databases.
    - fun fact: SQL standard has been evolving since 1986 → 1992 “SQL-92” → 1999 “SQL:1999” → current version of standard “SQL:2003”
  - **MySQL software is Open Source.**
  - **The MySQL Database Server is very fast, reliable, scalable, and easy to use.**
    - MySQL Server can run on a desktop or laptop, alongside other applications, web servers, and so on.
    - MySQL Server was originally developed to handle large databases much faster. Its connectivity, speed, and security make MySQL Server highly suited for accessing database on the Internet.
  - **MySQL Server works in client/server or embedded systems.**
    - client/server system — multithreaded SQL server that supports different back ends, client programs and libraries, administrative tools, and a wide range of application programming interfaces (APIs).
    - embedded multithreaded library that can be linked to the application to get a smaller, faster, easier-to-manage standalone product.
  - MySQL Heatwave?
    - fully managed database service, powered by the HeatWave in-memory query accelerator.
    - The only cloud service that combines _transactions, real-time analytics across data warehouses and data lakes, and machine learning in one MySQL Database._
    - Available on OCI, AWS, and Azure.
- [The Main Features of MySQL](https://dev.mysql.com/doc/refman/8.0/en/features.html)

  - MySQL is written in C and C++.
  - Data Types

    ```sql
    FLOAT, DOUBLE, CHAR, VARCHAR, BINARY,
    VARBINARY, TEXT, BLOB, DATE, TIME, DATETIME,
    TIMESTAMP, YEAR, SET, ENUM, OpenGIS spatial types.

    Fixed-length and variable-length string types.
    ```
