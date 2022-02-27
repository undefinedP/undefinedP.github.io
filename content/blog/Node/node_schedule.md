---
title: 'node-schedule을 이용해서 특정시간에 이벤트를 발생시키기'
date: 2022-02-27
category: 'node.js'
draft: false
---

[node-schedule](https://github.com/node-schedule/node-schedule)은 `cron`형식을 기반으로 rule을 설정해서, 해당하는 시간에 함수를 실행하는 방식으로 구동된다.

## Cron이란?

컴퓨터 운영체제에서 시간 기반으로 job-scheule을 설정할 수 있는 프로세스를 말한다. 여기서 스케줄링 파라미터로 사용되는 표현식을 `Cron 표현식`이라고 한다.

```
*    *    *    *    *    *
┬    ┬    ┬    ┬    ┬    ┬
│    │    │    │    │    │
│    │    │    │    │    └ day of week (0 - 7) (0 or 7 is Sun)
│    │    │    │    └───── month (1 - 12)
│    │    │    └────────── day of month (1 - 31)
│    │    └─────────────── hour (0 - 23)
│    └──────────────────── minute (0 - 59)
└───────────────────────── second (0 - 59, OPTIONAL)
```

## 사용하기전에 잠깐!

[[공식문서]](https://github.com/node-schedule/node-schedule#overview)

- Node Schedule은 스크립트가 동작하고 있을 때에만 실행된다!! 만약에 실행이 되지 않는다면 예약되어있는 스케쥴이 전부 사라진다고 한다.
  만약에 스크립트가 동작하고 있지 않을 때에도 스케쥴을 걸고싶다면, **cron**을 사용하라고 공식 문서에 명명되어있으니 참고하자!
- 만약에 특정 시간이 아닌 `특정 간격`마다(ex. 10분간격으로 함수를 호출) Job을 수행시키고 싶다면 **tode-scheduler**을 이용하는게 더 좋다고 한다.

## 설치

```bash
npm install node-schedule
```

```bash
yarn add node-schedule
```

## 사용법

### Cron-syle Scheduling

```js
import schedule from 'node-schedule'

// UTC 06
const job = schedule.scheduleJob('00 06 * * *', function () {
  //실행
  Job()
})
```

위의 코드는 cron형식으로 매일 UTC 06시(한국시간으로는 15시)에 Job을 수행하는 코드이다.

여기서 있는 \*은 all을 의미하는거기때문에 무지성으로 넣다간..생각과는 다른 결과가 나올 수도 있으니 주의하자.

### Data-based Scheduling

```js
import schedule from 'node-schedule'
const date = new Date(2023, 0, 27, 0, 0, 0)

const job = schedule.scheduleJob(date, function () {
  console.log('happy birthday')
})
```

만약에 23년 1월 27일 00시 00분 00초같이 매우 특정한 시간에 스케쥴을 작동시키고 싶다면 data-based style을 사용하는것도 좋은 방법이다.

### Recurrence Rule (재귀식 규칙 설정)

```js
import schedule from 'node-schedule'

const rule = new schedule.RecurrenceRule()

rule.dayOfWeek = [0, new schedule.Range(2, 4)]
rule.hour = 9
rule.minute = 0
rule.tz = 'Asia/Seoul'

schedule.scheduleJob(rule, function () {
  Job()
})
```

위의 코드는 일요일, 화요일, 수요일, 목요일 9시에 Job을 실행시키는 코드이다.

RecurrenceRule에는 위처럼 배열로 범위를 설정할 수 있고, 타임존도 적용을 시킬 수 있다. 사용 가능한 타임존의 기준은 [이 문서](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)를 따른다.

rule에 설정할 수 있는 값들은 아래와 같다.

- second (0-59)
- minute (0-59)
- hour (0-23)
- date (1-31)
- month (0-11)
- year
- dayOfWeek (0-6) Starting with Sunday
- tz

### Object Literal Syntax

```js
const job = schedule.scheduleJob(
  { hour: 14, minute: 30, dayOfWeek: 0 },
  function () {
    console.log('this works every Sunday at 14:30')
  }
)
```

non-cron방식으로 Object Literal 방식을 사용할 수도 있다. 규칙은 동일하게 위의 rule을 따른다.

### 시작시간과 종료시간 설정

```js
const startTime = new Date(Date.now() + 5000)
const endTime = new Date(startTime.getTime() + 5000)

const job = schedule.scheduleJob(
  { start: startTime, end: endTime, rule: '*/1 * * * * *' },
  function () {
    console.log('start and end')
  }
)
```

이런식으로 시작시간과 종료시간을 설정할 수도 있다. 위의 코드는 1분마다 5초후에 `console.log()`를 찍고, 10초 후에 멈춘다.

## 시간 설정 잘 했는데 제 시간에 얘가 작동을 안해요!

그럴 수 있다...당연함...로컬 시간과 서버 시간 기준이 다를 수 있기 때문에 서버시간에 맞춰서 작업을 하거나, 아니면 Reccurence Rule을 사용해서 타임존을 명시를 해주면 해결된다!

> 우리 회사같은 경우에는 서버 시간이 미국 시간으로 맞춰져있었나...하여튼 그래서 처음에는 스케줄이 계속 안걸려서 내가 뭔가 이상한건가 버그인건가 고민했었는데 그냥 타임존이 안맞았던거였다 얼척X 다들 타임존 확인을 잘하자...!

---

상황에 맞게 node-schedule을 사용하면 굉장히 사람이 편해진다... 만약에 특정 시간에 어떠한 작업을 반복해야하는 로직을 짜야한다면 적극적으로 도입해보는걸 추천한다! 정말최고에
