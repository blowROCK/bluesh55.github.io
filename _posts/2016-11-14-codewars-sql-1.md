---
layout: post
title: "[SQL] SQL Basics: Simple table totaling"
date: 2016-11-14
tags: [codewars]
---

첫 번째 문제는 SQL 문제다. ORM만 사용하다가 답답함을 많이 느껴서 좀 더 자유로워지고자 SQL 공부를 하고 있다.
첫 문제로 난이도도 적당하고 꽤 재밌는  문제가 걸렸다.
문제는 [이곳](https://www.codewars.com/kata/sql-basics-simple-table-totaling)에서 확인할 수 있다.
그럼 문제를 확인해보자.

![](/public/img/blog/codewars/1/1.png)


**문제**: 유저 이름,포인트, 클랜명이 저장된 People 테이블이 주어지는데,
각 클랜의 랭크, 총점, 인원 수, 클랜명을 출력하는 쿼리를 작성해야 한다.
만약 클랜명이 없다면 클랜명 대신 '[no clan specified]' 문자열을 출력해야 한다.

랭크, 총점, 인원 수를 구해야하기 때문에 People 테이블의 컬럼을 그대로 출력하는게 아니라
People 레코드 데이터를 활용한 연산이 필요하다는 것을 알 수 있다.
하지만 각각의 People 레코드를 따로 연산하는 것이 아니라 클랜을 기준으로 해야하기 때문에
`GROUP BY`를 사용해야 한다는 것도 알 수 있다.
따라서 기본 코드를 작성해보면 다음과 같다.

```sql
SELECT rank, total_points, total_people, clan
FROM people
GROUP BY clan;
```

인원 수를 먼저 구해보자. `GROUP BY clan`으로 묶었으니 인원 수는 간단히 전체 개수를 카운팅하면 된다.

```sql
SELECT 
	rank,
	total_points,
	COUNT(*) as total_people,
	clan
FROM people
GROUP BY clan;
```

다음은 클랜원의 점수를 모두 합친 총점을 구해보자. 단순히 더하기만 하면 되니 SUM 함수를 사용한다.

```sql
SELECT 
	rank,
	SUM(points) as total_point,
	COUNT(*) as total_people,
	clan
FROM people
GROUP BY clan;
```

SQL에는 순위를 매기는 함수가 있다. `RANK(), DENSE_RANK(), ROW_NUMBER()` 이렇게 3개인데
랭크 함수에 관한건 [여기](https://sites.google.com/site/smcgbu/home/gongbu-iyagi/rankdenserankrownumbersun-wileulbanhwanhaneunhamsu)
를 참조하면 된다.
랭크는 총점을 기준으로 해야하는데 위에서 이미 총점을 구했으니 바로 작성할 수 있다.

```sql
SELECT 
	RANK() OVER (ORDER BY SUM(points) DESC) as rank,
	SUM(points) as total_point,
	COUNT(*) as total_people,
	clan
FROM people
GROUP BY clan;
```

이제 마지막으로 클랜명을 출력해야하는데 한 가지 조건이 있었다.
클랜명이 없으면 디폴트 문자열을 출력해야하는데 이걸 구현하는 방법은 많지만 나는 COALESCE를 사용했다.

```sql
SELECT 
	RANK() OVER (ORDER BY SUM(points) DESC) as rank,
	SUM(points) as total_point,
	COUNT(*) as total_people,
	COALESCE(NULLIF(clan, ''), '[no clan specified]')
FROM people
GROUP BY clan;
```

1. NULLIF는 두 파라미터의 값이 같으면 NULL을 리턴한다.
2. COALESCE는 파라미터 중에서 NULL이 아닌 첫 번째 값을 리턴하는 함수이다.

두 가지 함수를 조합해서 클랜명이 비어있으면 특정 문자열을 출력하도록 작성했다.

테스트 코드를 실행시키면 제대로 출력되는걸 볼 수 있다 ^.^

![](/public/img/blog/codewars/1/2.png)
