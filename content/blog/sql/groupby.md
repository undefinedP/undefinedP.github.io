---
title: '[PostgreSQL]column must appear in the GROUP BY clause or be used in an aggregate function'
date: 2022-03-13
category: 'sql'
draft: false
---

![error](./images/groupby_error.png)
슉.슈슉.내.내한테.왜.이러는.데.

---

음~^^ 그럴 줄 알았습니다~^^ 라고 말할 수 있을 정도로 정말 정말 많이 보는 에러이다. 그런데 이 에러 도대체 왜 발생하는걸까?!

## GROUP BY

GROUP BY는 같은 값을 가진 행끼리 하나의 그룹으로 뭉춰주는 역할을 한다. 아래의 예시를 보면 더 빠르게 이해가 갈 것이다.

![group by](https://media.vlpt.us/images/dntjd7701/post/c759d336-1c98-40d3-bddf-83d08efd2506/image.png)

이는 Employee라는 테이블에서 DeptID를 기준으로 그룹핑해서 Salary의 평균을 구한걸 보여주고 있다.

위처럼 보통 GROUP BY절은 집계 함수와 같이 사용되고는 하는데, 집계 함수는 여러 행의 값을 더하거나, 평균 값을 내거나, 개수를 세는 등 여러 개의 데이터에 관한 계산을 한다.

주로 사용하는 집계 함수에는 `COUNT()`, `AVG()`, `MIN()`, `MAX()`, `SUM()` 등이 있다.

## Coulum must apper in the GROUP BY clause ...

아래와 같은 테이블이 있다고 하자.

| id  | name | location | cvs     | purchases |
| --- | ---- | -------- | ------- | --------- |
| 1   | KIM  | SEOUL    | GS25    | 20000     |
| 2   | LEE  | SEOUL    | Emart24 | 300       |
| 3   | PARK | BUSAN    | GS25    | 4500      |
| 4   | CHOI | DAEGU    | CU      | 90000     |
| 5   | KANG | BUSAN    | Emart24 | 34000     |
| 6   | YOON | DAEGU    | CU      | 2400      |

우리는 특정 편의점에서 발생한 구매액의 총액을 알고싶을 때, 이런식으로 쿼리를 쓸 수 있을 것이다.

```sql
SELECT location, cvs, SUM(purchases) AS sum
FROM table
GROUP BY cvs;
```

그런데 이런 쿼리로 실행하면 에러가 날것이다.

```
ERROR: column 'table.location' must appear in the
GROUP BY clause or be used in an aggregate function
```

이게 머..머선 소리고..

## 그래서 뭘 잘못했을까?

위의 쿼리를 잘 보자. Select를 해오는 컬럼은 총 세 가지이다. location, cvs, average. 그런데 group by 절에서는 cvs를 기준으로 묶는다고 적혀있고, 값은 집계함수인 SUM()함수를 사용해서 묶어주고 있다. 그런데 location은? 아무것도 없다! 그러니까 데이터베이스는 혼란이 온 것이다. 아니...뭘로...데이터를 그룹핑해야할지 모르겠어용...하면서 반환한 것이 위의 에러이다.

## 해결방법

### 1. GROUP BY 절에 추가

```sql
SELECT location, cvs, SUM(purchases) AS sum
FROM table
GROUP BY cvs, location
```

이런식으로 location까지 추가를 하게 되면, 편의점 별로, 또 위치별로 그룹핑이 될 것이다.

| location | cvs     | sum   |
| -------- | ------- | ----- |
| SEOUL    | GS25    | 20000 |
| SEOUL    | Emart24 | 300   |
| BUSAN    | GS25    | 4500  |
| BUSAN    | Emart24 | 34000 |
| DAEGU    | CU      | 92400 |

### 2. SELECT문에서 지우기

만약에 location이 필요가 없다면 없애는것도 하나의 방법이 된다. 그냥 편의점별로 총 액을 구하고싶다면, location은 있어봤자 필요없는 정보가 될 뿐이다.

```sql
SELECT cvs, SUM(purchases) AS sum
FROM table
GROUP BY cvs
```

| cvs     | sum   |
| ------- | ----- |
| GS25    | 24500 |
| Emart24 | 34300 |
| CU      | 92400 |

### 3. 집계함수를 사용

에러에서도 나오듯이 집계함수를 사용할 수도 있다. COUNT, SUM, AVG 등등..내가 필요한 함수를 잘 활용해서 사용한다면 GROUP BY에서 에러가 발생하지 않을 것이다.

아니면 GROUP BY절을 사용하지 않고, [window function](https://www.postgresql.org/docs/9.1/tutorial-window.html)을 사용하는 것도 하나의 방법이 된다.
