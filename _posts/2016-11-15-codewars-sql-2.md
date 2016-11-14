---
layout: post
title: "[SQL] SQL Basics: Simple UNION ALL"
date: 2016-11-15
tags: [codewars]
---

두 번째 문제도 SQL 문제이다. SQL 문제가 은근 푸는 맛이 있는 것 같다.
이번 문제는 UNION 사용법만 알고 있으면 풀기 쉬운 문제였다.
문제는 [여기](https://www.codewars.com/kata/sql-basics-simple-union-all)서 풀어볼 수 있다.

![](/public/img/blog/codewars/2/1.png)

**문제**: 컬럼의 구조가 같은 `ussales`와 `eusales` 테이블이 존재한다.
테이블 스키마는 다음과 같다.

|id|name                |price|card_name  |card_number        |transaction_date|
|--|--------------------|-----|-----------|-------------------|----------------|
|1 |Awesome Silk Shirt  |70.61|Savion Koch|1234-2121-1221-1211|2014-09-05      |
|2 |Gorgeous Bronze Bag |54.17|Moses Spink|1228-1221-1221-1431|2014-02-15      |
|.. |...|...|...|...|...|


위와 같은 2개의 테이블에서 price가 50.00보다 큰 레코드를 뽑아서 출력하면 된다.
출력할 땐 테이블의 6가지 컬럼 뿐만 아니라 `location` 컬럼에
`ussales` 테이블 레코드이면 US를, `eusales` 테이블 레코드면 EU를 출력해야 한다.


`ussales`와 `eusales` 두 테이블의 조회 결과를 하나로 합치려면 UNION을 사용해야 한다.
UNION의 기본 사용법은 다음과 같다.

```sql
SELECT
  column1,
  column2,
  column3
  FROM table1
UNION
SELECT
  column1,
  column2,
  column3
  FROM table2;
```

두 개의 SELECT문 사이에 UNION을 쓰기만 하면 된다. 단, 조회하려는 컬럼은 두 테이블이 같아야한다. 순서는 상관없다.

UNION 사용법도 알았으니 문제에 적용해보자.
location 컬럼과 전체 테이블 컬럼을 가져오는 와일드카드 `*`를 사용했다.
그리고 50.00보다 큰 price를 찾기 위해 WHERE 조건문을 추가했다.

```sql
SELECT
  'US' as location, *
  FROM ussales
  WHERE price > 50.00
UNION
SELECT
  'EU' as location, *
FROM eusales
WHERE price > 50.00
```

이렇게 하면 통과될 줄 알았는데 테스트 하나가 실패되었다.

![](/public/img/blog/codewars/2/2.png)

UNION은 2가지 종류가 있다. UNION과 UNION ALL인데 이번 문제에서는 UNION ALL을 사용해야 하는 것 같다.

```sql
SELECT
  'US' as location, *
  FROM ussales
  WHERE price > 50.00
UNION ALL
SELECT
  'EU' as location, *
FROM eusales
WHERE price > 50.00
```

UNION ALL을 사용하면 테스트가 통과하는 것을 볼 수 있다 ^.^

![](/public/img/blog/codewars/2/3.png)

이제 UNION과 UNION ALL의 차이점을 알아보자.
먼저 UNION은 UNION DISTINCT와 같다. DISTINCT에서 알 수 있듯이 중복 레코드를 제거해준다.
그럼 UNION ALL은? 중복 레코드를 제거하지 않고 결과를 전부 돌려준다.

UNION ALL

|id|name                |price|card_name  |card_number        |transaction_date|
|--|--------------------|-----|-----------|-------------------|----------------|
|1 |Awesome Silk Shirt  |70.61|Savion Koch|1234-2121-1221-1211|2014-09-05      |
|1 |Awesome Silk Shirt  |70.61|Savion Koch|1234-2121-1221-1211|2014-09-05      |

UNION

|id|name                |price|card_name  |card_number        |transaction_date|
|--|--------------------|-----|-----------|-------------------|----------------|
|1 |Awesome Silk Shirt  |70.61|Savion Koch|1234-2121-1221-1211|2014-09-05      |

두 개의 UNION이 별 차이 없어보이지만 중복 레코드를 제거하기 위해 적지 않은 비용이 든다고 한다.
UNION ALL이 UNION보다 훨씬 빠르니까 정말 중복 레코드를 없애야할 때만 UNION을 사용하고
아니라면 UNION ALL을 사용하도록 하자.


참고 링크

* [UNION과 UNION ALL](http://intomysql.blogspot.kr/2011/01/union-union-all.html)
