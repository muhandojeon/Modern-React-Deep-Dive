# 좋은 리액트 코드 작성을 위한 환경 구축하기

## ESLint

### ESLint는 어떻게 코드를 분석할까?

자바스크립트 코드를 문자열로 읽고, 이를 AST(Abstract Syntax Tree)라는 구조화된 트리로 변환한다. 

이 트리를 기준으로 각종 규칙과 대조하여 위반된 코드를 알리거나 수정한다.

* 기본 파서는 [espree](https://github.com/eslint/espree)

### Plugin과 Config

* 플러그인은 규칙을 모아놓은 패키지
* 컨피그는 플러그인을 한데 묶어 환벽하세 한 세트로 제공하는 패키지

* [@titicaca/triple-config-kit](https://github.com/titicacadev/triple-config-kit)
  + 트리플에서 만든 컨피그
  + airbnb 기반으로 작성하지 않음
  + 테스트 코드 존재
  + 프리티어와 스타일린트도 있음

* eslint-config-next
  + 컴포넌트에서 반환하는 JSX 구문 및 HTML 코드 또한 정적 분석 대상으로 분류해 제공

> 🤘
> https://github.com/vercel/next.js/blob/canary/packages/eslint-plugin-next/src/utils/node-attributes.ts
> 이런 유틸을 통해서 제공하더라구요
>
> 머윤님이랑 함께한 영감탱에서 저 유틸을 참고해 만든 eslint rule이 있는데 이것도 공유드려용
> https://github.com/depromeet/ygtang-client/blob/develop/eslint/lib/rules/internal-router-passhref.js

```
리액트 17 이후부터는 JSX 런타임 덕분에 import React문이 필요 없어짐
이에 따라 import React를 삭제하면?

- 빌드시에는 번들링 크기가 똑같음
  - 트리쉐이킹되기 때문에

- 없애면 트리쉐이킹에 드는 시간을 줄여 빌드 속도에 이점이 있음
```

## eslint rule 만들기

* yo와 generate-eslint를 활용해 환경 구성을 빠르게 할 수 있음
* https://github.com/yeoman/yo
* https://github.com/eslint/generator-eslint

```
react/no-exhaustive-deps rule을 위해 1800줄의 코드가 작성돼 있음 ㄷㄷ
```

> 🤘
> 제가 좋아하는 린트 환경을 구성해둔 패키지가 있는데..
> 이것도 링크를 남겨봅니다
> https://github.com/hyesungoh/eslint-plugin

## 테스트

* 프런트엔드의 테스트 코드는 블랙박스 형태로 이루어짐
  + 코드가 어떻게 됐든 의도한 대로 작동하는지를 확인하는데 좀 더 초점이 맞춰져 있음
  

### React Testing Library

* jsdom을 기반으로 하고 있음
  + jsdom은 순수하게 자바스크립트로 작성되어 있음
  + HTML이 없는 자바스크립트만 존재하는 환경에서 HTML과 DOM을 사용할 수 있도록 해줌

* 좋은 테스트 코드는 다양한 테스트 코드가 작성되고 통과하는 것뿐만 아니라 어떤 테스트가 무엇을 테스트하는지 일목요연하게 보여주는 것도 중요함

* Jest가 유지보수되고 있지 않았다가 메타에서 오픈소스 재단으로 관리 주체가 바뀜

```
jest 사용 시 api를 임포트 안해도 되는 이유

- jest cli에서 전역 스코프에 기본적으로 넣어주기 때문
```

> 🤘
> 저는 최근에 jest 대신 vitest를 선호하긴해요
> jest api와 동일해서 새롭게 배울 필요도 없고, 
> jest에 비해 설정이 쉽고, 실행도 빠르다는 점이 좋게 느껴졌어용
> https://github.com/vitest-dev/vitest

### 테스트 코드 작성하기

* getBy, findBy, queryBy
  + findBy: 비동기로 찾음 (기본 1000ms 타임아웃)

* 🤘 `toBeVisible()` vs `toBeInTheDocument()`
  + 책의 테스트 코드에서는 toBeVisible을 사용하는데 차이점이 궁금하더라구요
  + https://stackoverflow.com/questions/69852702/what-is-the-difference-between-tobeinthedocument-and-tobevisible
  + 후자는 단순히 DOM에 있는지만 확인한다면
  + 전자는 style을 통해 보이는지 여부까지 확인한다고 합니다

* 데이터셋
  + `data-testid`와 같이 테스트 환경이 아닌, 개발시에도 유용하게 사용할 수 있음
    - 이벤트 버블링 활용 등
  + 🤘 next.config에서 특정 프로퍼티를 지워줄 수 있음
    - https://github.com/depromeet/na-lab-client/blob/0f4fcc286e1b3c8016516a1e056912bd6bfd0398/next.config.js#L21-L25

* 동적 컴포넌트 테스트 예제 코드의 셋업 함수 예제

```tsx
const setup = () => {
  const screen = render(<DynamicComponent />);
  const input = screen.getByPlaceholderText('이름을 입력해주세요') as HTMLInputElement;
  
  return {
    input,
    screen,
  };
}
```

* `userEvent` vs `fireEvent`
  + `userEvent`는 실제 사용자가 사용하는 것처럼 테스트를 진행할 수 있음
    - ex. userEvent.click = fireEvent.mouseOver > fireEvent.mouseMove > fireEvent.mouseDown > fireEvent.mouseUp > fireEvent.click
  + 특별히 사용자의 이벤트를 흉내 내야 할 때만 userEvent를 사용하면 됨, 대부분의 경우 fireEvent로 충분

* MSW
  + 네트워크 요청을 모킹
  + https://github.com/mswjs/msw

* @testing-library/react-hooks
  + 리액트 18부터는 `@testling-libary/react`에 통합됨
  + renderHook 내부에서 컴포넌트를 생성하고 전달받은 훅을 실행하는 방식으로 실행됨

* rerender with new props

```tsx
const { rerender } = renderHook(({foo}) => useFoo(foo), {initialProps: {foo: 1}}) ;

rerender({foo: 2});
```

### 테스트를 작성하기에 앞서 고려해야 할 점

* 테스트 커버리지를 맹신하지 말기
  + 얼마나 많은 코드가 테스트되고 있는지일 뿐, 잘되고 있는 것은 아닐 수 있음

* 서버 코드와 다르게 프런트엔드 코드는 사용자의 입력이 매우 자유롭기 때문에 모든 상황을 커버해 테스트를 작성하기란 불가능함
  + 실무에서는 테스트 코드를 작성하고 운영할만큼 여유로운 상황이 별로 없음
  + 때로는 QA에 의존해 개발을 빠르게 진행해야 할 수도 있고, 이후에 또 개발해야 할 기능이 산적해 있을 수 있음
    - 따라서 테스트 코드를 작성하기 전에 생각해봐야 할 최우선 과제는 `애플리케이션에서 가장 취약하거나 중요한 부분을 파악`하는 것

* 테스트 코드는 소프트웨어 품질에 대한 확신을 얻기 위해 작성하는 것

* 🤘 E2E test - [playwright](https://playwright.dev/)

## 정리

테스트할 수 있는 방법은 여러 가지 있지만

테스트가 이뤄야 할 목표는 애플리케이션이 비즈니스 요구사항을 충족하는지 확인하는 것 한 가지 뿐이다.
