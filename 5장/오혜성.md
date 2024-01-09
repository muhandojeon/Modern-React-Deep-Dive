# 리액트와 상태 관리 라이브러리

## 상태 관리는 왜 필요한가?

웹 애플리케이션을 개발할 때 이야기하는 상태는 어떠한 의미를 지닌 값이며 애플리케이션의 시나리오에 따라 지속적으로 변경될 수 있는 값을 의미

## 리액트 상태 관리의 역사

### Flux 패턴의 등장

* 리액트에서 Context API가 선보인 것은 16.3
  + useContext를 선보인 것은 16.8

그 전까지는 리덕스가 나타나기 전까지 리액트 애플리케이션에서 딱히 이름을 널리 알린 상태 관리 라이브러리는 없었음

2014년 경 Flux를 소개됨
* 애플리케이션이 비대해지고 상태도 많아짐에 따라 어디서 어떤 일이 일어나서 이 상태가 변했는지 등을 추적하고 이해하기가 매우 어려운 상황이였음
* 페이스북 팀은 이러한 문제의 원인을 양방향 데이터 바인딩으로 봤음
  + 뷰(HTML)가 모델(자바스크립트)을 변경할 수 있으며, 반대도 가능한 것
  + 이는 코드의 양도 많이지고, 변경 시나리오가 복잡해질수록 관리가 어려워짐

* 페이스북 팀은 양방향이 아닌 단방향으로 데이터 흐름을 변경하는 것을 제안하고, 이것이 Flux 패턴의 시작

액션 > 디스패처 > 스토어 > 뷰 > 액션

### 시장 지배자 리덕스 등장

리덕스는 Flux 구조를 구현하기 위해 만들어진 라이브러리 중 하나였음

* Elm 아키텍처를 도입
  + Elm은 웹페이지를 선언적으로 작성하기 위한 언어
  + Elm은 Flux와 마찬가지로 데이터 흐름을 세 가지로 분류하고
    - 이를 단방향으로 강제해 웹 애플리케이션의 상태를 안정적으로 관리하고자 노력
    - 리덕스는 이 아키텍처의 영향을 받아 작성됐음

* 하고자 하는 일에 비해 보일러플레이트가 너무 많다는 비판의 목소리

### Context API와 useContext

* 리액트 16.3 이전에도 context가 존재는 했고, 이를 다루기 위해 getChildContext()가 제공되었음
  + 상위 컴포넌트가 렌더링되면 getChildContext가 호출됨과 동시에 shouldComponentUpdate가 항상 true를 반환해 불필요하게 렌더링이 일어난다는 점
  + 사용하기 위해 context를 인수로 받아야해 컴포넌트와 결합도가 높아지는 등의 단점이 존재했음

이러한 문제를 해결하기 위해 Context API가 등장

* Provider와 memo를 통해 단점을 보안했다고 생각

```
하나의 이슈에 대해 여러가지 해결책이 나온다는 것은 그만큼 이 분야가 건강하게 성장하고 있다는 증거이기도 하다
```

## 리액트 훅으로 시작하는 상태 관리

### useState와 useReducer

* 관리해야 하는 상태가 복잡하거나 상태를 변경할 수 있는 시나리오가 다양해진다면, 사용자 정의 훅을 통해 격리하여 제공할 수 있다는 장점이 더욱 크게 드러날 것

### 컴포넌트 밖에서 상태를 관리

* 리액트 외부에서 관리되는 값에 대한 변경을 추적하고, 이를 리렌더링까지 할 수 있는, 페이스북 팀에서 만든 `useSubscription`이 존재
  + 리액트 18에서는 useSubscription 훅이 useSyncExternalStore로 재작성돼 있음

```
- https://react-ko.dev/reference/react/useSyncExternalStore
- https://junghyeonsu.com/posts/react-use-sync-external-store/
```

### Recoil

* Recoil 팀에서는 리액트 18에서 제공되는 동시성 렌더링, 서버 컴포넌트, Streaming SSR 등이 지원되기 전까지는 1.0.0을 릴리스하지 않을 것이라고 밝힌 바 있음

* RecoilRoot
  + Recoil에서 생성되는 상태값을 저장하기 위한 스토어를 생성
  + Recoil 상태값은 RecoilRoot로 생성된 Context의 스토어에 저장됨
  + 스토어의 상태값에 접근할 수 있는 함수들이 있고, 이 함수를 활용해 상태값에 접근하거나 변경할 수 있음
  + 값의 변경이 발생하면 이를 참조하고 있는 하위 컴포넌트에 모두 알림

* useRecoilValue
  + 외부의 값을 구독해 렌더링을 강제로 일으킨다

* useSetRecoilState
  + 값이 변경되면 forceUpdate 같은 기법을 통해 리렌더링을 실행해 최신 atom 값을 가져오게 된다

* 특징
  + 메타 팀에서 주도적으로 개발하고 있기 때문에 다른 라이브러리보다 잘 지원할 것으로 기대됨

```
axios도 드디어 1.0.0을 릴리스
```

### Jotai

* atom
  + config라는 객체를 반환하고 여기에는
    - 초기값의 init, 가져오는 read, 설정하는 write만 존재
    - 즉 Jotai에서의 atom에 따로 상태를 저장하고 있지 않다

* useAtomValue
  + Recoil과는 다르게, 컴포넌트 루트 레벨에서 Context가 존재하지 않아도 됨
    - Context가 없다면 Provider가 없는 형태로 기본 스토어를 루트에 생성하고 이를 활용해 값을 저장하기 때문

```
코드를 보면 생각보다 어렵지 않은데,
이런 코드 조각으로 라이브러리의 차별점을 만들어낸 것이 굿끼제먹
```

  + store에 atom 객체 그 자체를 키로 활용해 값을 저장 `WeakMap`
    - 🤘 framer motion에서는 성능을 위해 해당 자료형을 사용

```
https://github.com/framer/motion/blob/4623edfe2c85228e112e9d4265ae9974e1892548/packages/framer-motion/src/render/dom/viewport/index.ts#L24

많이 사용해보셨을 인터섹션 옵저버... 관련 코드인데요
프레이머 모션에서도 제공하길래 어떻게 되어 있나 보니까 요 자료형을 사용하더라구용
```

* 특징
  + Jotai는 사용자가 키를 관리할 필요가 없음

### Zustand

* Zustand의 스토어는 스토어가 가지는 클로저를 기반으로 생성됨
  + 이 스토어의 상태가 변경되면 이 상태를 구독하고 있는 컴포넌트에 전파해 리렌더링을 알리는 방식

* setState는 partial과 replace로 나뉘어져 있음
  + partial은 state의 일부분만 변경하고 싶을 때
  + replace는 state를 완전히 교체하고 싶을 때
    - 이로써 state의 값이 객체일 때 필요에 따라 나눠서 사용할 수 있을 것으로 보인다

* 바닐라 코드는 아무것도 import 하고 있지 않아서, 프레임워크와는 독립적으로 구성돼 있음

* 전체적으로 코드가 매우 간결하다
  + 🤘 이런 이유 + 문제 정의가 뛰어나서 요 포인만드레스를 참 좋아함 🤘

* 🤘 useSyncExternalStoreWithSelector는 리액트 스펙?

```
https://github.com/facebook/react/blob/9fcaf88d58cfd942e2fdd303ae8291dbf4828969/packages/use-sync-external-store/src/useSyncExternalStoreWithSelector.js

그렇다고합니다~
```

* 🤘 그래서 얘네도 setState를 통해 강제 리렌더링하는건가?
```
https://github.com/search?q=repo%3Apmndrs%2Fzustand%20useState&type=code

아니네용
```


* 특징
  + 번들링 사이즈가 위 두 개에 비해 작음
