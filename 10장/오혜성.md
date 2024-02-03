# 리액트 17과 18의 변경 사항 살펴보기
* 리액트를 사용하고 있는 사이튿르은 일반적으로 16 버전을 사용하고 있음
  + 55.1%
  + 에어비앤비, 넷플릭스도 여전히 16 버전 사용중

## 리액트 17 버전 살펴보기

* 리액트 16과 다르게 새롭게 추가된 기능이 없으며, 호환성이 깨지는 변경 사항을 최소화 했음

* 점진적인 업그레이드를 지원하기 위한 리액트의 일부 컴포넌트 변경이 17의 주요 변경 사항 중 하나
  + 일종의 업데이트를 위한 업데이트

* 한 애플리케이션 내에서 여러 버전의 리액트가 존재할 수 있지만, 임시방편일 뿐 하나의 리액트 버전을 쓰는 것이 좋다고 언급

### 이벤트 위임 방식의 변경

* 리액트는 이벤트 핸들러를 각각의 DOM에 부착하는 것이 아닌, 이벤트 타입당 하나의 핸들러를 루트에 부착함
  + 16 버전까지는 모두 document에 부착되고 있음
  + 17 버전부터는 document가 아닌 리액트 컴포넌트 최상단 트리, 즉 루트 요소로 바뀜
  + 서로 다른 리액트 버전에서 발생할 수 있는 문제를 해결하기 위해 document에서 컴포넌트의 최상위로 변경함
    - 각 리액트 컴포넌트 트리 수준으로 격리되어 버블링으로 인한 혼선을 방지할 수 있음

### `import React from 'react'` 문장의 제거

* JSX 변환을 위해 해당 구문이 필요했음
* 17 버전부터는 바벨과 협력해 이러한 import 구문 없이도 JSX를 변화할 수 있게 됨

```js
// 16
var Component = React.createComponent();

// 17
var _jsxRuntime = require('react/jsx-runtime');
var Component = (0, _jsxRuntime.jsx)('div', {});
```

* 이처럼 require 구문도 같이 추가되어 import React를 작성하지 않아도 됨

* 기존 코드에서 import React를 한 번에 제거하기 위해 codemod 사용 가능

```
npx react-codemod update-react-imports
```

* React.createElement와 react/jsx-runtime의 차이점
  + 두 함수 모두 ReactElement를 반환하지만 react/jsx-runtime이 내부 로직이 간결함

### 이벤트 풀링 제거

* 리액트 16에서는 이벤트 풀링이라 불리는 기능이 있었음
  + 리액트에는 이벤트를 처리하기 위한 `SyntheticEvent`가 존재
    - 브라우저의 기본 이벤트를 한 번 더 감싼 이벤트 객체
  + 이벤트가 발생할 때마다 이 이벤트를 새로 만들어야 했고, 메모리 할당 작업이 일어날 수밖에 없었음
    - 메모리 누수를 위해 주기적으로 해제해야 했음

* 이벤트 풀링이란 SyntheticEvent 풀을 만들어서 이벤트가 발생할 때마다 가져오는 것을 의미

```
1. 이벤트 핸들러가 이벤트를 발생시킴
2. 합성 이벤트 풀에서 합성 이벤트 객체에 대한 참조를 가져옴
3. 이 이벤트 정보를 합성 이벤트 객체에 넣음
4. 이벤트 핸들러가 실행됨
5. 이벤트 객체가 초기화되고 다시 이벤트 풀로 돌아감
```

* 이로 인해 비동기 코드로 이벤트 핸들러에 접근하기 위해 `e.persist()` 같은 문법을 사용해야 했음
  + 해당 시점에서 이벤트 객체가 null이 될 수 있기 때문
* 별도 메모리 공간이 필요한 점, 모던 브라우저에서는 성능 향상에 크게 도움이 안된다는 점 때문에 이러한 풀링 개념을 삭제하게 됨
  + 모던 브라우저에선느 이벤트 처리에 대한 성능이 많이 개선됐기 때문에 의미가 퇴색

* 덕분에 이벤트 핸들러 내부에서 이벤트 객체에 접근할 때, 동기와 비동기 관계없이 일관적으로 코딩할 수 있게 됨

### useEffect 클린업 함수의 비동기 실행

* 16 버전까지는 동기적으로 처리됐음
  + 클린업 함수가 완료되기 전까지 다른 작업을 방해하므로 불필요한 성능 저하로 이어짐

* 17 버전부터는 컴포넌트의 커밋 단계가 완료될 때까지 지연된 후 실행 됨
  + 화면이 업데이트가 완전히 끝난 이후에 실행됨
  + 성능적 이점을 볼 수 있게 됨

#### 🤘 `import { Profiler } from 'react'`

* 성능 최적화를 측정하기 위해 사용됨

```jsx
import { Profiler } from 'react'

const callback = (
  id, // 프로파일링 식별자
  phase, // "mount" | "update"
  actualDuration, // 수행 시간
  baseDuration, // 기본 수행 시간
  startTime, // 시작 시간
  commitTime, // 커밋 시간
  interactions // 상호작용
) => {
  // 콜백 함수
}

<Profiler id="App" onRender={callback}>
  <App />
</Profiler>
```

### 컴포넌트의 undefined 반환

* 리액트 16, 17에서는 undefined를 반환하면 에러가 발생함
  + 의도치 않게 잘못된 반환으로 인한 실수를 방지하기 위해

* 16에서는 forwardRef, memo에서 undefined를 반환하는 경우에 에러가 발생하지 않는 버그
  + 17에서 정상적으로 에러를 반환하게 됨
  
* 18에서는 undefined를 반환하는 것이 허용됨
  

## 리액트 18 버전 살펴보기

* 가장 큰 변경점은 동시성 지원
  + 수년 전부터 고민했었고, 리액트 개발 팀 이외에 다양한 개발자들의 의견을 듣고자 별도의 워킹 그룹을 구축할 정도로 심혈을 쏟음

### 새로 추가된 훅 - useId

* 컴포넌트별로 유니크한 값을 생성하는 훅

* 하나의 컴포넌트가 여러 군데에 재사용되는 경우, 서버 사이드 렌더링 환경과 하이드레이션이 일어날 때 동일한 값을 가져야하는 점 때문에 쉽지 않았음

* 컴포넌트 내부의 고유한 값이며, 클라이언트와 서버에서의 불일치를 피할 수 있는 값을 useId로 생성할 수 있음

* useId로 생성하는 값은 `:`로 감싸져 있는데, CSS 선택자나 querySelector에서 작동하지 않도록 하기 위한 의도적 결과
  
* 사용된 알고리즘은 현재 트리에서의 자신의 위치와 부모의 트리를 이용한 것

### useTransition

* UI 변경을 가로막지 않고 상태를 업데이트할 수 있는 훅
  + 이를 활용해 상태 업데이트를 긴급하지 않은 것으로 간주해 무거운 렌더링 작업을 조금 미루어 사용자 경험을 개선할 수 있음

```jsx
const [isPending, startTransition] = useTransition();
const [tab, setTab] = useState()

function handleTabChange(newTab) {
  startTransition(() => {
    setTab(newTab)
  })
}
```

* 동시성을 다룰 수 있는 새로운 훅
  + 과거 리액트의 모든 렌더링은 동기적으로 작동해, 느린 렌더링 작업이 있을 경우 애플리케이션 전체적으로 영향을 끼침
  + 동시성을 지원해 느린 렌더링 과정에서 로딩 화면을 보여주거나, 지금 진행 중인 렌더링을 버리고 새로운 상태값으로 다시 렌더링하는 등의 작업을 할 수 있게 됨

* 훅을 사용할 수 없다면, startTransition을 바로 import 할 수도 있음

* 사용할 때 주의점
  + startTransition 내부는 반드시 상태를 업데이트하는 함수와 관련된 작업만 넘길 수 있음
    - 만약 props나 사용자 정의 훅에서 반환하는 값 등을 사용하고 싶다면 useDeferredValue 훅을 사용해야 함
  + startTransition으로 넘겨주는 상태 업데이트는 다른 모든 동기 상태 업데이트로 인해 실행이 지연될 수 있음
    - 타이핑으로 setState하는 경우에는 타이핑이 끝날 때까지 지연시킨 상태 업데이트가 일어나지 않음
  + startTransition으로 넘겨주는 함수는 동기 함수여야 함
    - setTimeout 같은 비동기 함수를 넣으면 제대로 작동하지 않는데, 작업을 지연시키는 작업과 비동기로 함수가 실행되는 작업 사이에 불일치가 일어나기 때문

### useDeferredValue

* 리렌더링이 급하지 않은 부분을 지연할 수 있게 도와주는 훅
  
* 디바운스와 비슷하지만, useDeferredValue만이 가진 장점이 몇 가지 있음
  + 디바운스는 고정된 지연 시간을 이용하지만, useDeferredValue는 첫 번째 렌더링이 완료된 이후에 지연된 렌더링을 수행함

* useTransition과의 차이점
  + useTransition은 state 값을 업데이트하는 함수를 감싸서 사용하는 반면
  + useDeferredValue는 state 값을 감싸서 사용함
  + 지연된 렌더링을 한다는 점에서는 동일함
  + 직접 상태를 업데이트할 수 있는 코드에 접근할 수 있다면 useTransition을, 그러나 props와 같이 값만 받아야 하는 상황이라면 useDeferredValue를 사용하는 것이 타당

### useSyncExternalStore

* 테어링은 하나의 state 값이 있음에도 서로 다른 값을 기준으로 렌더링되는 현상을 말함
  + 17에서는 이런 현상이 일어날 여지가 없음
  + 18에서는 렌더링을 일시 중지하거나, 뒤로 미루는 최적화가 가능해지면서 동시성 이슈가 발생할 수 있음
  
* 리액트에서 관리하는 state라면 동시성 문제를 해결하기 위해 처리를 해두었지만, 리액트 클러저 범위 밖의 외부 데이터 소스라면 문제가 생길 수 있음
  + document.body, DOM, window.innerWdith 등

* 이 문제를 해결하기 위한 훅이 `useSyncExternalStore`

* 첫 번째 인수는 콜백 함수를 받아 스토어에 등록하는 용도
  + 스토어에 있는 값이 변경되는 이 콜백이 호출되어야 함
*  두 번째 인수는 컴포넌트에 필요한 현재 스토어의 데이터를 반환하는 함수
  + 이전 값과 Object.is로 비교하고, 변경되었을 경우 컴포넌트를 리렌더링함
* 옵셔널인 세 번째 인수는 서버 사이드 렌더링 시에 초기 데이터를 가져오는 함수

* 리렌더링을 발생시키기 위해 useState나 useReducer를 호출하지 않는데, useSyncExternalStore 어딘가에 콜백을 등록하고, 이 콜백이 호출될 때마다 렌더링을 트리거하는 장치가 마련돼 있다는 것을 알 수 있음

```tsx
function subscribe(callback: (this: Window, ev: UIEvent) => void) {
  window.addEventListener('resize', callback)
  
  return () => {
    window.removeEventListener('resize', callback)
  }
}

function App() {
  const windowSize = useSyncExternalStore(
    subscribe,
    () => window.innerWidth,
    () => 0);
}
```

### useInsertionEffect

* CSS의 추가 및 수정은 브라우저에서 렌더링하는 작업 대부분을 다시 계산해야 하기 때문에 매우 무거운 작업임
  + 따라서 서버 사이드에서 스타일 코드를 삽입해야 하는데, 이 작업을 훅으로 처리하기 쉽지 않았음
  + 훅에서 이러한 작업을 할 수 있도록 도와주는 훅

* useEffect와 구조가 동일하지만 실행 시점에 차이가 있음
  + DOM이 실제로 변경되기 전에 동기적으로 실행됨
  + 이 훅 내부에 스타일을 삽입하는 코드를 넣음으로써 브라우저가 레이아웃을 계산하기 전에 실행될 수 있게끔 함

```tsx
function App() {
  useEffect(() => console.log('effect')); // 3
  useLayoutEffect(() => console.log('layout effect')); // 2
  useInsertionEffect(() => console.log('insertion effect')); // 1
}
```

* useLayoutEffect와 비교했을 때, 두 훅 모두 브라우저에 DOM이 렌더링되기 전에 실행된다는 공통점이 있지만
  + layoutEffect는 모든 DOM의 변경 작업이 다 끝난 이후에 실행
  + insertionEffect는 DOM 변경 작업 이전에 실행됨
  + 이를 통해 브라우저가 다시 스타일을 입혀 DOM을 재계산하지 않아도 된다는 점을 낳음

* useInsertionEffect는 실제 애플리케이션 코드를 작성할 때 사용될 일이 거의 없으므로, 가급적 사용하지 않는 것을 리액트 팀에서 권고함

### react-dom/client

* createRoot
  + 동시성 지원을 위해 루트 컴포넌트가 렌더링되는 곳에서 변경해야함
* hydrateRoot
  + 서버 사이드 렌더링 애플리케이션에서 하이드레이션 하기 위한 메서드

### react/dom/server

* renderToPipeableStream
  + 컴포넌트를 HTML로 렌더링하는 메서드
  + Suspense 서버 사이드 렌더링을 지원
  + 기존 renderToNodeStream은 무조건 렌더링을 순서대로 해야 했음

* renderToReadableStream
  + renderToPipeableStream은 Node.js 환경에서의 렌더링이라면, renderToReadableStream은 웹 스트림 기반으로 작동함
    - 클라우드플레어나 디노 같은 웹 스트림을 사용하는 모던 엣지 런타임 환경에서 사용되는 메서드

### 자동 배치

* 리액트는 여러 상태 업데이트를 하나의 리렌더링으로 묶어서 성능을 향상시킴

* 단 17 버전까지는 Promise, setTimeout 같은 비동기 이벤트에서는 자동 배치가 이뤄지지 않았음
* 18 버전부터는 비동기 이벤트에서도 자동 배치가 이뤄짐

* 자동 배치를 하고 싶지 않은 경우에는 flushSync를 사용하면 됨

### 엄격 모드에서 side effect 검사

* 다음 내용을 의도적으로 이중 호출함
  + useState, useMemo, useReducer에 전달되는 함수
  + 함수 컴포넌트의 body
  + 클래스 컴포넌트의 constructor, render, getDerivedStateFromProps, shouldComponentUpdate
  + 클래스 컴포넌트의 setState의 첫 번째 인수

* 함수형 프로그래밍 원칙에 따라 모든 컴포넌트는 항상 순수하다고 가정하기 때문
  + state, props, context가 변경되지 않으면 항상 동일한 JSX를 반환해야 함
  + 리액트는 이러한 내용을 사전에 개발자가 파악할 수 있도록 유도하는 것

* 콘솔 로그는 17 버전에서 의도적으로 두 번씩 기록되지 않게 함
  + 18에서는 두 번씩 기록하되, 두 번째 로그는 회색으로 표시함

### 리액트 18에서 추가된 엄격 모드

* 향후 리액트는 컴포넌트가 마운트 해제된 상태에서도 컴포넌트 내부의 상태값을 유지할 수 있는 기능을 제공할 예정이라 밝힘
  + 뒤로가기 후 돌아왔을 때 즉시 이전 상태를 유지해 표시할 준비하는 기능이 추가된느 것

* 이를 위해 컴포넌트가 최초에 마운트될 때 자동으로 모든 컴포넌트를 마운트 해제하고 두 번째 마운트에서 이전 상태를 복원하게 됨

* 때문에 useEffect가 두 번 작동함
  + 미래에 있을 업데이트에 대비하려면, 두 번의 useEffect 호출에도 자유로운 컴포넌트를 작성하는 것이 좋음

```tsx
let count = 0;

function App() {
  useEffect(() => {
    count += 1;
    console.log(`mount ${count}`);

    return () => console.log (`unmount ${count}`);
  }, []);
}

// react 17
// mount 1

// react 18
// mount 1
// unmount 1
// mount 2
```

### Suspense 기능 강화

* Suspense 내에 스로틀링이 추가됨
  다음 렌더링을 보여주기 전에 잠시 대기해 중첩된 Suspense fallback이 있다면 최대한 자연스럽게 보여주기 위해 노력함
* 마운트되기 전에 effect가 실행되는 문제 수정
* Suspense로 인해 컴포넌트가 보이거나 사라질 때도 effect가 정상적으로 실행됨

> 🤘 https://github.com/pmndrs/suspend-react

## 정리

리액트 18의 핵심은 동시성 렌더링

* 리액트 팀의 목적은 점진적으로 기능을 채택할 수 있게 하는 것
  + 추가적인 노력 없이도 동시성 렌더링이 작동하는 것을 목표로 하고 있음
  + 생태계 내에 있는 라이브러리에는 큰 변화를 가지고 옴
    - 동시성 렌더링을 지원하기 위해 useSyncExternalStore의 도움을 필수적으로 받아야 했음
