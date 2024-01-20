---
title: "[TIL] 프로그래머스 SQL 문제 풀이"
excerpt: "WHERE 문에서 조건을 연결하는 논리 연산자, IFNULL(MySQL)과 CASE 문 사용 방법"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-19T19:00
---

## 이름에 el이 들어가는 동물 찾기 (WHERE, AND)

> 문제 [[링크]](https://school.programmers.co.kr/learn/courses/30/lessons/59047): 이름에 "EL"이 들어가는 개의 아이디와 이름을 조회하고 이름 순으로 정렬하기.

### Query

```sql
SELECT ANIMAL_ID, NAME
FROM ANIMAL_INS
WHERE NAME like '%el%' AND ANIMAL_TYPE = 'Dog'
ORDER BY NAME
```

### Results

| ANIMAL_ID | NAME   |
| :-------- | :----- |
| A354597   | Ariel  |
| A355753   | Elijah |
| A373219   | Ella   |
| ...       | ...    |

### Learned

> WHERE, AND

- 처음에 `WHERE` 절을 쉼표로 연결하여 에러가 발생했다.
- `WHERE` 절의 조건들은 `AND`혹은 `OR`로 연결!

## NULL 처리하기 (IFNULL, CASE)

> 문제 [[링크]](https://school.programmers.co.kr/learn/courses/30/lessons/59410): 동물의 생물 종, 이름, 성별 및 중성화 여부를 아이디 순으로 조회하고, 이름이 없는 동물의 이름은 "No name"으로 표시하기.

### Query

```sql
-- IFNULL
SELECT ANIMAL_TYPE, IFNULL(NAME, 'No name')

-- CASE
SELECT ANIMAL_TYPE,
CASE WHEN NAME IS NULL THEN "No name"
ELSE NAME END NAME,
SEX_UPON_INTAKE
FROM ANIMAL_INS
ORDER BY ANIMAL_ID
```

### Results

| ANIMAL_TYPE | NAME    | SEX_UPON_INTAKE |
| :---------- | :------ | :-------------- |
| Cat         | Sugar   | Neutered Male   |
| ...         | ...     | ...             |
| Dog         | No name | spayed Female   |
| Dog         | Sniket  | Neutered Male   |
| ...         | ...     | ...             |

### Learned

> CASE

- `CASE`: 조건부 로직을 적용하는 데 사용되며 `CASE` 문의 시작을 나타내는 키워드
- `WHEN condition THEN result`: 조건을 평가하고 해당하는 경우에 결과를 반환. 여러 `WHEN` 절을 사용할 수 있다.
- `ELSE result`: 모든 조건이 거짓인 경우에 반환할 기본 결과
- `END`: `CASE` 문의 끝을 나타내는 키워드
- `CASE`는 데이터 값을 범주(category)로 변환하거나 특정 조건에 따라 다른 계산을 수행하는 데 활용할 수 있다.

## 경기도에 위치한 식품창고 목록 출력하기

> 문제 [[링크]](https://school.programmers.co.kr/learn/courses/30/lessons/131114): FOOD_WAREHOUSE 테이블에서 경기도에 위치한 창고의 ID, 이름, 주소, 냉동시설 여부를 조회하기. 이때 냉동시설 여부가 NULL인 경우, 'N'으로 출력하고, 결과는 창고 ID를 기준으로 오름차순 정렬

### Query

```sql
SELECT WAREHOUSE_ID,
    WAREHOUSE_NAME,
    ADDRESS,
    IFNULL(FREEZER_YN, 'N') FREEZER_YN
FROM FOOD_WAREHOUSE
WHERE ADDRESS LIKE '경기%'
ORDER BY WAREHOUSE_ID
```

### Results

| WAREHOUSE_ID | WAREHOUSE_NAME | ADDRESS                                 | IFNULL(FREEZER_YN, 'N') |
| :----------- | :------------- | :-------------------------------------- | :---------------------- |
| WH0001       | 창고\_경기1    | 경기도 안산시 상록구 용담로 141         | Y                       |
| WH0003       | 창고\_경기2    | 경기도 이천시 마장면 덕평로 811         | N                       |
| WH0004       | 창고\_경기3    | 경기도 김포시 대곶면 율생중앙로205번길  | N                       |
| WH0012       | 창고\_경기7    | 경기도 수원시 권선구 오목천로152번길 40 | N                       |
| WH0032       | 창고\_경기9    | 경기도 안양시 만안구 전파로 3           | N                       |

### Learned

> IFNULL

- `IFNULL`은 NULL 값을 다른 값으로 다른 값으로 대체할 때 사용된다.
- 기본 구문은 `IFNULL(expr1, expr2)`이며 `expr1`은 평가할 표현식, `expr2`는 `expr1`이 NULL인 경우 반환될 값이다.
- `IFNULL`은 MySQL에서 사용되는 특정 함수이며, 다른 데이터베이스 시스템에서는 비슷한 기능을 하는 함수가 존재할 수 있다. (ex. `COALESCE`)
