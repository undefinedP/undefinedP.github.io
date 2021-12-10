---
title: '[PostgreSQL]COALESCE 함수'
date: 2021-12-07
category: 'sql'
draft: false
---

### 문법

COALESCE(컬럼, 대체값1...대체값n)

👉 컬럼이 null인 경우에 대체값으로 반환

대체값이 null이 아닌 값이 나올 때까지 대체값을 불러온다.

```sql
COALESCE(name, null, null, null, 2)
-> return 2
```

### 활용

```sql
SELECT price, seat FROM events;
```

| price  | seat |
| ------ | ---- |
| 100000 | 50   |
| 55000  | null |
| 80000  | 279  |

여기서 `price`와 `seat`를 곱한 값을 `total`이라고 할 때, null이 있으면 값도 null이 나오기 때문에 저 값을 다른 값으로 치환해야한다.

```sql
SELECT price, seat,
  price * COALESCE(seat, 0) as total
FROM seat;
```

| price  | seat | total    |
| ------ | ---- | -------- |
| 100000 | 50   | 5000000  |
| 55000  | null | 0        |
| 80000  | 279  | 22320000 |

이런식으로 테이블이 나오는 것을 확인할 수 있다.

### 주의사항

COALESCE 함수 내부에 들어가는 parameter의 값은 항상 **똑같은 데이터 타입**이어야한다. 위에 그려진 표는 parameter이 전부 integer이어서 괜찮았지만, string과 integer을 연산할 수 없기 때문에 string을 integer로 바꿔주는 작업이 필요하다.

ref. https://augustines.tistory.com/64
