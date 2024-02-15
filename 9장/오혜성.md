# 모던 리액트 개발 도구로 개발 및 배포 환경 구축하기

## Next.js로 리액트 개발 환경 구축하기

* tsconfig의 `$schema`
  + 해당 JSON 파일이 무엇을 의미하는지, 어떤 키와 값이 들어갈 수 있는지 알려주는 도구 (자동완 성이 가능해짐)
  + schemaStore에서 제공해줌

* tsconfig
  + target - 타입스크립트가 변환을 목표로 하는 언어의 버전
  + lib - 타입스크립트가 참조하는 라이브러리의 목록
    - Promise, Map과 같은 문법
    - dom 요소 등이 포함
  + skipLibCheck - 라이브러리에서 제공하는 d.ts에 대한 검사 여부를 결정
    - 전체적인 프로젝트의 컴파일 시간이 길어져 일반적으로 꺼놓는 경우가 많음
  + strict - 엄격 모드를 제어
    - useUnknownInCatchVariables - catch문에서 error의 타입에 unknown을 할당함
  + noEmit - 컴파일 하지 않고 타입만 체크하게 됨
    - Next.js의 경우 swc가 타입스크립트 파일을 컴파일하므로 굳이 타입스크립트가 자바스크립트로 컴파일할 필요가 없다
  + jsx - 파일 내부에 있는 JSX를 어떻게 컴파일할지 설정
    - swc에서 JSX를 변환하기 때문에 변환하지 않는 옵션인 `preserve`를 사용할 수 있음
  + incremental - 마지막 컴파일 정보를 파일 형태로 저장함
    - 컴파일이 더 빨라짐
  

### create-*-app

* 선호하는 프로젝트 세팅을 cli 패키지로 만들어 배포할 수 있음
  + 조직 내에 마이크로서비스를 지향하고, 앞으로 생성해야 할 프로젝트가 많다면 검토해 볼 만한 방법

* https://blog.logrocket.com/creating-a-cli-tool-with-node-js/

> 🤘
> 사이드 프로젝트에서 만들어 본 적이 있는데요
> https://github.com/hyesungoh/create-comet-land
>
> create-로 시작하는 패키지라면
> yarn create foo-abr
> 처럼 create 이후에 띄어쓰기를 알아서 해준다더라구용

## 깃허브 100% 활용하기

* 깃허브 액션의 본래 목적은 깃허브 저장소를 기반으로 깃허브에서 발생하는 다양한 이벤트를 트리거 삼아 다양한 작업을 할 수 있게 도와주는 것

* pnpm/action-setup, borales/actions-yarn을 사용해 각 패키지 매니저를 사용할 수 있음
  + https://github.com/pnpm/action-setup
  + https://github.com/Borales/actions-yarn

* 깃허브에서 제공하는 기본 액션
  + actions/dependency-review-action
    - 의존성이 변경됐을 때 실행되는 액션으로 보안 또는 라이선스에 문제가 있다면 알려줌
    - https://github.com/actions/dependency-review-action
  + github/codeql-action
    - 정적 코드 분석 솔루션
    > 🤘 이전에는 LGTM bot이라는 이름의 서비스로 제공되고 있었는데, 인수 이후에 이렇게 이름이 바뀌었어요
    > 문법적으로 놓친 부분을 찾아줘서 좋아하곤 합니다

* 그 외 액션
  + calibreapp/image-actions
    - 이미지 파일을 무손실로 압축해 다시 커밋해 줌
    - https://github.com/calibreapp/image-actions
  + lirantal/is-website-vulnerable
    - 웹사이트에 라이브러리 취약점이 존재하는지 확인
    - https://github.com/lirantal/is-website-vulnerable
  + Lighthouse CI
    - https://github.com/GoogleChrome/lighthouse-ci
  

* 워크 플로우 성공 시 다른 워크 플로우 트리거

```yml
on:
  workflow_run:
    workflows: ["deploy"]
    types:
      - completed
```

* 시멘틱 버저닝
  + API 스펙이 변경된다면 메이저 버전을 올려야 함
    - 메이저 버전을 올리는 것이 껄끄럽지만 버그 수정을 해야하는 경우에는
    해당 API를 deprecated 처리하고 새로운 API를 만들어 마이너 버전을 올리는 것이 좋음

```
foo@1.0.0 - 해당 버전에만 의존
foo@^1.0.0 - 1.0.0 이상 2.0.0 미만
foo@~1.0.0 - 1.0.0 이상 1.1.0 미만
```

* 개발자들 간의 약속일 뿐, 해당 API의 버전이 시멘틱 버전에 맞춰 구현돼 있는지는 알 수 없음

* dependencies와 devDependencies를 구분하는 것에 의문을 제기
  + 실제 서비스에 배포해야 하는 라이브러리인지 결정하는 것은 번들러
    - 즉 두 차이가 최종 결과물에 전혀 영향을 미치지 않음
  + 과거에 유효했던 이유는 실제 프로젝트를 실행할 때 `npm install --only-production`으로 필요한 패키지만 빠르게 설치했기 때문
    - 현재는 타입스크립트와 같은 패키지로 인해 빌드조차 할 수 없게 됨
  + 하지만 구분이 완전히 무의미하진 않음
    - npm 패키지라면 dependencies의 패키지들만 최종 결과물에 포함되게 해야 함
  
* `npm ls foo-package`
  + 해당 패키지가 어떤 패키지에 의해 설치됐는지 확인할 수 있음

* 취약점 보고의 문제
  + 취약점이 존재한다고 해도, 실제로는 취약점이 사용되고 있지 않을 수 있음
  + 혼란과 불필요하게 취약점을 확인하는데 시간을 쏟아야 하므로 진짜 취약점을 보는 데 방해가 된다고 주장
  + 리액트 팀은 취약점이 있으나 문제가 없다고 판단되면 대응하지 않겠다고 선언한 바 있음
  + 이런 문제를 해결하기 위해 몇 가지 제안이 npm에 반영되고 있는 중

* 패키지 내부에 선언된 의존성을 강제로 올리는 방법
  + package.json에 overrides를 추가
  

```json
{
  "overrides": {
    "foo-package": "^1.0.0"
  }
}
```

* 의존성 관련 이슈를 방지하는 가장 좋은 방법은 의존성을 최소한으로 유지하는 것
  + 내재화할 수 있는 모듈은 내재화해 의존성을 최소한으로 유지하는 것이 좋다
  + 패키지를 선택할 때는 얼마나 많은 사용자가 존재하고, 얼마나 활발하게 유지보수되는지 살펴봐야 함

> 🤘 
> https://npmtrends.com/
> https://bundlephobia.com/

## 배포

* vercel의 serverless function
  + Next.js에서 제공하는 api 라우트가 서버리스 함수로 구분되어 접근 로그나 오류 등을 확인할 수 있음

## 도커라이즈하기

* 도커 이미지 상태로 애플리케이션을 준비해 둔다면 이미지를 실행할 수 있는 최소한의 환경에서 어디서든 배포할 수 있음
  + 즉 SaaS와 같이 특정 서비스에 종속적이지 않은 상태로 만들어 좀 더 유연하게 관리할 수 있음

### 도커란

도커란 개발자가 모던 애플리케이션을 구축, 공유, 실행하는 것을 도와줄 수 있도록 설계된 플랫폼이다. 도커는 지루한 설정 과정을 대신해 주므로 코드를 작성하는 일에만 집중할 수 있다.

- 도커 빌드 시 이미지 처리
  - `public` 폴더와 `.next/static` 폴더를 CDN으로 별도로 제공하는 방법

> 🤘
> 여러분들은 어떻게 배포되고 있나요??
> 저희는 AWS ECS, ECR를 이용해 도커 이미지를 관리하고 있는데
> 따로 롤백 시스템을 만들어서 쓰시는지??