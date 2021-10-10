---
title: '🪄GitHub Actions를 이용해서 CI/CD를 구축해보자!'
date: 2021-10-07
category: 'DevOps'
draft: false
---

![GitHub Actions](https://media.vlpt.us/images/lingodingo/post/6faf3011-da9b-431b-980a-7324f1ffbc11/android-github-actions-setup-image-35b6a79fea4a7289acb6796cd4ad05b4.png)

블로그를 새로 파서 글을 작성을 하면서 아주 치명적으로 귀찮은 문제가 하나 생겼다. 바로
`deploy`!! 포스팅을 하나 쓸 때마다 일일히 배포를 하는 스크립트를 치는 것이 너무 귀찮았던 나는 자동화의 간절함을 느낀다. 구글에서 열심히 서치를 한 결과 `Jenkins`와 `GitHub Actions`이라는 툴을 발견했고 ... 그러다가 결국 `GitHub Actions`라는 방법을 채택하기로 결심을 하는데 ...더보기

## 잠깐만 그런데 CI/CD가 뭔데?

CI/CD는 어플리케이션 개발 단계를 `자동화`시켜서 짧은 주기로 고객에게 제공을 할 수 있게끔 하는 방법이다.

CI : Continuous Integration(지속적 통합)

CD : Continuous Delivery(지속적 제공)/ Continuous Deployment(지속적 배포)

의 줄임말이지만...무슨 말인지 모르겠다 더 알아보자

### **CI란?**

한 줄로 요약하면 **통합을 위한 단계(빌드, 테스트, 머지)를 자동화하는 것**이다

CI를 성공적으로 적용한다면 애플리케이션에 대한 새로운 코드 변경 사항이 정기적으로 빌드 및 테스트되서 공유 레파지토리에 통합이 된다. 여러 명의 개발자가 동시에 어플리케이션 개발과 관련된 코드 작업을 할 경우에 충돌하는 문제를 해결할 수 있다.

### CI의 장점

- 개발의 편의성이 증가
  - 디버깅이 쉬워지기 때문
- 변경된 코드에 대한 즉각적 피드백과 검증 가능
- 좋은 코드 퀄리티 유지 가능
  - 테스트를 통과하는 코드만 공유 레포지터리에 올라갈 수 있기 때문
- 소스코드의 통합과 검증에 들어가는 시간이 단축된다

### **CD란?**

![제공과 배포의 차이](https://blog.kakaocdn.net/dn/eeSLmu/btqI9pXqCN8/iIopSPh3KSK1SwhRjkWPf1/img.png)

**지속적 제공**

- CI를 거친 이후에 배포를 할 준비를 하고, 이 과정에서 배포가 될 것이 아무런 문제가 없는지 개발자/검증 팀이 검증을 한 뒤에 _수동으로_ 배포를 한다

**지속적 배포**

- 배포를 할 준비가 되었다면 자동적으로 사용자에게까지 배포가 될 수 있도록 자동화를 시킬 수 있고, 그 일련의 과정을 **지속적 배포**라고 말한다

### CD의 장점

- 개발부터 배포까지 과정이 번거롭지 않고 간소화되기 때문에 사용자 피드백을 빠르게 반영할 수 있다
  - 즉, **장애 대응이 빨라진다**

### CI/CD의 흐름

- CI

  1. 개발자는 자신이 개발한 소프트웨어의 소스코드를 공통된 버전 관리 시스템(ex.GitHub)에 저장한다.
  2. 소스코드상에서 변동이 생기면 버전 관리 시스템에서는 CI 툴로 코드의 변경을 알린다
  3. CI툴에서는 변경된 코드를 빌드, 테스트, 머지를 한다.

- CD
  1. CI를 통해 소스코드를 검증한다
  2. 검증된 소프트웨어를 실제 프로덕션 환경으로 배포한다.

## ¿¿GitHub Actions??

- GitHub에서 제공하는 워크플로우를 자동화하도록 도와주는 도구이다.

- Github에서 여러가지 workflow 템플릿을 추천해주는데, 이걸 사용함으로서 테스트/빌드/배포 등 다양한 작업들을 자동화하여 뚝딱 처리할 수 있다.

### GitHub Actions을 선택한 이유

1. Jenkins처럼 설치 후 따로 서버를 팔 필요가 없다
2. 기왕이면 GitHub 하나로 자동화를 관리하는 것이 편하다고 판단했다.

## GitHub Actions의 구성

**Workflow(워크 플로우)**

- 저장소에 추가하는 자동화된 프로세스. 하나 이상의 `job`으로 이루어져 있으며, 이벤트에 의해 실행된다.
- Workflow 파일은 `YAML`으로 작성되고, `.github/workflows`폴더 아래에 저장된다.

**Event(이벤트)**

- 워크 플로우를 실행하는 특정 활동이나 규칙.
- 커밋의 push/pull request가 생성되었을 때 뿐만 아니라, GitHub 외부에서 발생하는 활동으로도 이벤트를 발생시킬 수도 있다.

ex)

```yaml
# push나 PR이 발생할 때 워크플로우 실행
on: [push, pull_request]
```

게다가 cron 문법으로 `schedule` 이벤트를 설정할 수 있다!

ex)

```yaml
# 월-금 오전 9시
on:
  schedule:
    - cron: '0 9 * * 1-5'
```

**Job(작업)**

- 워크플로우의 기본 단위. 여러 Step으로 구성되어 있다.
- 기본적으로는 여러 작업을 병렬적으로 실행하지만, 순차적으로 실행하도록 설정할 수도 있다.

**Step(스텝)**

- Job안에서 순차적으로 실행되는 프로세스의 단위. 여기서 명령을 내리거나, action을 실행할 수 있다.
- 한 job의 각 스텝들은 동일한 러너에서 실행되기 때문에, 해당 작업의 액션들은 서로 데이터를 공유하게 된다.

**Runner(러너)**

- `GitHub Actions Runner Application`이 설치된 서버.
- GitHub에서 호스팅하는 러너를 사용할 수도 있고, 직접 호스팅을 할 수도 있다.
- 워크플로우가 실행될 인스턴스이다.

## 배포 자동화 가보자고.

### Access Token 생성

GitHub Actions를 사용하기 위해서는 `Access Token`이 필요하다. 레포지토리에 직접적으로 접근을 해야하기 때문!!

[여기](https://github.com/settings/tokens)에서 발급받을 수 있다.

`Generate New Token` 버튼을 클릭해서 Note의 부분에 토큰 이름을 적고, Select scopes의 Repo부분을 전부 체크해준다.
![newtoken](./images/new_token.png)

`Generate Token` 버튼을 누르면 토큰이 생성된다! 그런데 토큰은 다음에 다시 확인할 수 없으니까 꼬옥..복사해서 어디 적어놔야한다..

### Access Token 할당

위에서 발행한 토큰을 레포지토리에 할당을 해주자.

레포지토리의 설정에 들어가 `Secrets`탭에 있는 `New repository secret` 버튼을 클릭해서 Name에는 토큰 이름(위의 토큰 이름과 달라도 됨!), Value에는 위에서 발행한 토큰 값을 넣어주고 추가를 하면 끝!

### Workflow 생성

워크플로우를 실행시키기 위해서는 먼저 `.github/workflows` 밑에 `main.yml` 파일을 추가해준다.

```yaml
# GitHub Action의 workflow 이름
name: Gatsby Publish

# gh-pages라는 브랜치에 push를 할 때마다 jobs를 실행
on:
  push:
    branches:
      - gh-pages
jobs:
  build_gatsby:
    name: build
    # 실행 환경이 ubuntu-latest
    runs-on: ubuntu-latest
    steps:
      # checkout을 해주는 액션 소스코드를 사용
      - uses: actions/checkout@v2

      - name: packages install🛸
        run: yarn install

      - name: gatsby build🚀
        run: yarn build
        env:
          # GH_API_KEY에 secret을 생성할 때 설정한 이름으로 넣어야 한다
          GH_API_KEY: ${{secrets.TOKEN_KEY}}

      - name: deploy🛰
        uses: maxheld83/ghpages@v0.2.1
        env:
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
          # 여기도 마찬가지! TOKEN_KEY 대신 자신이 설정한 이름을 넣어주면 된다
          GH_PAT: ${{secrets.TOKEN_KEY}}
          BUILD_DIR: 'public/'
```

위 코드를 작성한 뒤에 브랜치에 푸시를 해주면, repository의 Actions탭에 워크플로우가 추가된 것을 확인 할 수 있다!

### 동작확인
