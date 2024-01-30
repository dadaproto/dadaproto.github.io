---
title: "[TIL] SQL 코테 연습 - 주로 JOIN과 필터링 문제"
excerpt: "WHERE IN, INNER JOIN, GROUP_CONCAT, WHERE 절에서 집계 함수, JOIN VS UNION"
toc: true
categories:
  - TIL
tags:
  - TIL
last_modified_at: 2024-01-30T22:00
---

## 헤비 유저가 소유한 장소 (WHERE IN, INNER JOIN)

```sql
-- WHERE 절로 필터링
SELECT ID,
    NAME,
    HOST_ID
FROM PLACES
WHERE HOST_ID IN
(
    SELECT HOST_ID
    FROM PLACES
    GROUP BY HOST_ID
    HAVING COUNT(*) > 1
)
ORDER BY ID
```

- WHERE 절로 필터링
  - 서브 쿼리
    - HOST_ID로 그룹화(`GROUP BY`) 했을 때 집계 함수 `COUNT(*)>1` 로 중복되는 HOST_ID가 있는 경우 필터링
  - 메인 쿼리
    - HOST_ID가 위 조건에 속하는 경우(`WHERE HOST_ID IN`)에만 값을 가져온다(`SELECT HOST_ID`)
- 직관적이라는 장점이 있으나 `IN` 절은 대량의 데이터에 대해 성능 저하가 발생할 수 있다.

```sql
-- INNER JOIN 으로 필터링
SELECT P1.ID,
    P1.NAME,
    P1.HOST_ID
FROM PLACES P1
INNER JOIN
(
    SELECT HOST_ID
    FROM PLACES
    GROUP BY HOST_ID
    HAVING COUNT(*) >= 2
) A ON P1.HOST_ID=A.HOST_ID
ORDER BY P1.ID
```

- INNER JOIN 으로 필터링
  - 서브 쿼리
    - PLACE 테이블에서 HOST_ID로 그룹화 했을 때 `HAVING`을 사용하여 집계 함수 `COUNT(*)>=2` 로 중복되는 HOST_ID가 있는 경우필터링
    - 위 조건으로 생성된 테이블 A의 HOST_ID와 메인 쿼리의 PLACES 테이블의 HOST_ID가 일치하는 경우로 `INNER JOIN` 한다.
- JOIN 연산은 대량의 데이터에 대해서도 효율적이지만, 구조가 복잡해질 수 있다.

## 우유와 요거트가 담긴 장바구니 (GROUP_CONCAT)

```sql
SELECT CART_ID
FROM
(
    SELECT CART_ID, GROUP_CONCAT(NAME) NAME
    FROM CART_PRODUCTS
    GROUP BY 1
) A
WHERE A.NAME LIKE '%MILK%' AND NAME LIKE '%YOGURT%'
```

- 서브쿼리에서 CART_ID로 그룹화하여 동일한 CART_ID를 갖는 NAME 값을 합친다.
- `GROUP_CONCAT`: 그룹화된 문자열을 합치는 함수

  ```sql
  -- 기본 구분자는 쉼표(,)이며 변경하고 싶을 때는 SEPARATOR 사용

  SELECT GROUP_CONCAT(column_name SEPARATOR ';')
  FROM your_table;
  ```

- CART_PRODUCTS
  | ID | CART_ID | NAME |
  | ---- | ------- | ------------ |
  | 3727 | 195 | Coffee |
  | 3728 | 195 | Ketchup |
  | 3729 | 195 | Pasta |
  | 3730 | 195 | Snack |
  | 3731 | 195 | Trash Bag |
  | 3732 | 195 | Laundry Care |
- Results
  | CART_ID | NAME |
  | ------- | ------------------------------------------------------ |
  | 195 | Coffee,Ketchup,Pasta,Snack,Trash Bag,Laundry Care,Soap |

## 조회수가 가장 많은 중고거래 게시판의 첨부파일 조회하기 (WHERE 절에서 집계 함수를 사용할 때)

```sql
SELECT FILE_PATH
FROM
(
    SELECT CONCAT('/home/grep/src/', B.BOARD_ID, '/', F.FILE_ID, F.FILE_NAME, F.FILE_EXT) FILE_PATH, VIEWS, FILE_ID
    FROM USED_GOODS_BOARD B
    JOIN USED_GOODS_FILE F ON B.BOARD_ID=F.BOARD_ID
    WHERE VIEWS = (SELECT MAX(VIEWS) FROM USED_GOODS_BOARD)
) A
ORDER BY A.FILE_ID DESC
```

- WHERE 절에서 집계함수를 사용할 때는 일반적으로 서브쿼리를 사용해야 한다. 집계 함수는 전체 데이터 집합을 기반으로 계산되는데, WHERE 절은 각 행의 조건을 검사하기 때문에 서브쿼리를 통해 집계된 결과를 가져와야 한다.
- 집계함수는 주로 HAVING 절에서 사용하지만 이 문제에서는 ‘조회수가 가장 많은 파일’을 찾아야 하는데 결과가 1개 이상일 수 있으며 GROUP BY와 HAVING을 사용했을 때 최대값을 필터링하지 못하므로 WHERE 절을 사용하는 게 낫다.

## 주문량이 많은 아이스크림들 조회하기 (JOIN, UNION)

문제에 JULY 테이블에는 다른 공장에서 출하한 경우가 있어 FLAVOR에 중복이 있을 수 있다는 조건이 있었다. 처음에 다른 컬럼을 키로 사용해 조금 헤맸었던 문제.

```sql
-- JOIN으로 합치기

SELECT FLAVOR
FROM
(
    SELECT J.FLAVOR FLAVOR,
        SUM(J.TOTAL_ORDER) + H.TOTAL_ORDER TOTAL_ORDER
    FROM FIRST_HALF H
    JOIN JULY J USING (FLAVOR)
    GROUP BY FLAVOR
    ORDER BY 3 DESC
) A
LIMIT 3
```

- FIRST_HALF 테이블과 JULY 테이블을 FLAVOR 컬럼을 키값으로 하여 JOIN 했다.
- JOIN한 테이블을 FLAVOR로 그룹화한다.
- SELECT 문에서 JULY 테이블의 TOTAL_ORDER 컬럼의 값을 집계 함수 `SUM`을 사용해 합치고, FIRST_HALF 테이블의 TOTAL_ORDER와 더한다.

```sql
-- UNION으로 합치기

SELECT FLAVOR
FROM (
    SELECT FLAVOR, SUM(TOTAL_ORDER) AS TOTAL_ORDER
    FROM (
        SELECT FLAVOR, TOTAL_ORDER
        FROM FIRST_HALF

        UNION

        SELECT FLAVOR, SUM(TOTAL_ORDER) AS TOTAL_ORDER
        FROM JULY
        GROUP BY FLAVOR
    ) AS CombinedFlavors
    GROUP BY FLAVOR
) AS FinalFlavors
ORDER BY TOTAL_ORDER DESC
LIMIT 3;
```

- 결합하려는 두 테이블의 SELECT 문의 열의 개수와 데이터 유형이 일치하므로 (FLAVOR - 문자열,TOTAL_ORDER/SUM(TOTAL_ORDER) - 숫자) UNION을 사용할 수 있다.

### UNION과 JOIN의 차이는?

**UNION**

- **`UNION`** 연산자는 두 개 이상의 SELECT 문의 결과를 합칠 때 사용된다.
- **`UNION`**을 사용하면 서로 다른 테이블이나 쿼리의 결과를 하나의 결과 집합으로 합칠 수 있다.
- **`UNION`**을 사용하면 각 SELECT 문의 결과 집합이 합쳐져 중복 행은 제거된다.
  - **`UNION ALL`**은 사용하면 중복된 행을 제거하지 않고 모든 행을 결과에 포함시킨다. 반면에 **`UNION`**만 사용할 경우 중복된 행은 하나만 결과에 포함시킨다.
- 각 SELECT 문에서 선택된 열의 개수와 데이터 유형이 일치해야 한다.
- **`UNION`**은 수직으로 데이터를 합친다.

**JOIN**

- **`JOIN`**은 두 개 이상의 테이블에서 특정 열을 기준으로 데이터를 결합한다.
- **`JOIN`**은 두 테이블 간에 공통된 열(일반적으로 외래 키)을 사용하여 행을 결합한다.
- **`JOIN`**은 특정 조인 조건에 따라 연결된 열을 기반으로 행을 결합한다.
- **`JOIN`**을 사용하면 두 테이블 간의 관계를 활용하여 특정 조건에 따라 데이터를 결합할 수 있다.
- **`JOIN`**은 수평으로 데이터를 합친다.
