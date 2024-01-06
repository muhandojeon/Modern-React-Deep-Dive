# 리액트 개발 도구로 디버깅하기

## 컴포넌트 탭

* 현재 리액트 애플리케이션의 컴포넌트 트리 및 props와 내부 hooks 등 다양한 정보를 확인할 수 있다.

```tsx
// ./DefaultExportComponent.tsx
export default () => <>default export</>;

// ./App.tsx
import DefaultExportComponent from './DefaultExportComponent';

const App = () => {
  return (
    <div>
      <DefaultExportComponent />
    </div>
  );
};
```

* 실제로 default export로 내보낸 함수의 명칭은 추론할 수 없다.
  + 따라서 `_default`로 표시된다

* memo를 사용해 익명 함수로 만든 컴포넌트를 감싼 경우도 함수명을 추론하지 못해 `Anonymous`로 표시된다.
  + 개발자 도구에 memo 라벨이 붙어, memo로 감싸진 컴포넌트임을 알 수 있다.

* 리액트 16.9 버전 이후부터는 익명 컴포넌트에 `_c3`,           `_c5`와 같이 컴포넌트를 추정할 수 있는 힌트를 제공한다.

```tsx
const MemoizedComponent = memo(function Foo() {});

const HOCComponent = withHOC(function Bar() {});
```

* 그렇기 때문에 위 예제처럼 기명 함수로 변경하는 것이 디버깅에 좋다.
  + 만약 함수를 기명 함수로 바꾸기 어렵다면 함수에 `displayName` 속성을 추가하는 방법도 있다.

```tsx
const MemoizedComponent = memo(() => {});
MemoizedComponent.displayName = 'MemoizedComponent';
```

* 고차 컴포넌트의 경우 displayName 설정의 우선 순위를 하위 컴포넌트 > HOC 컴포넌트 순으로 제공하면 디버깅에 도움이 될 수 있다.

### 컴포넌트 도구

* 눈 아이콘을 눌러 HTML 어디에서 렌더링됐는지 알 수 있다.
* 벌레 아이콘을 눌러 해당 컴포넌트의 정보를 console에 기록할 수 있다.
* 소스코드 아이콘을 눌러 해당 컴포넌트의 소스코드를 확인할 수 있다.
  + 난독화되어 있지만,  `{}` 버튼을 눌러 읽기 쉬운 형태로 변경할 수 있다.

### 컴포넌트 hooks

* useState는 `State`와 같이 use가 생략된 이름으로 나타난다.
  + 사용자 정의 훅의 경우도 마찬가지로 `use`가 생략된 이름으로 나타난다. `useCounter -> Counter`

### rendered by

* 개발 모드에서 해당 컴포넌트를 렌더링한 부모 컴포넌트까지 확인할 수 있다.

## 프로파일러 탭

* 리액트가 렌더링하는 과정에서 발생하는 상황을 확인하기 위한 도구

* `Hide logs during second render in Strict Mode`
  + 리액트 애플리케이션이 엄격 모드에서 실행되는 경우, 원활한 디버깅을 위해 useEffect 등이 두 번씩 작동하는 의도적인 작동이 숨겨져 있다.
  + 해동 옵션을 통해 useEffect 안에 있는 console.log를 한 번 출력되도록 할 수 있다.

<!-- 추가한 내용 -->

```
어떤 라이브러리의 경우, 개발 모드에서 effect가 두 번 실행되지 않도록 하는 effect 훅을 제공하던데 .... 출처는 기억나지 않습니다 :C
```

### Flamegraph

* 렌더 커밋별로 어떠한 작업이 일어났는지 나타낸다

* 렌더링되지 않는 컴포넌트는 회색으로 표시되며 `Did not render`라는 메시지가 표시된다.
  + 이를 활용해 메모이제이션이 작동하고 있는지, 렌더링이 의도한 대로 제한적으로 발생하고 있는지 확인할 수 있다.

### Ranked

* 해당 커밋에서 렌더링하는 데 오랜 시간이 걸린 컴포넌트를 순서대로 나열한 그래프

* Flamegraph와 다르게 모든 컴포넌트를 보여주는 것이 아니라, 렌더링이 발생한 컴포넌트만 보여준다.

### 타임라인

* 시간이 지남에 따라 컴포넌트에 어떤 일이 일어났는지를 확인할 수 있다.

* 프로파일링 기간 동안 무슨 일이 있었는지, 무엇이 렌더링됐고, 또 어느 시점에 렌더링됐는지, 리액트의 유휴 시간은 어느 정도였는지 등을 확인할 수 있다.

## 사용 예

* 불필요한 setState 감지
* state 갱신에 따라 불필요한 리렌더링 요소 포함 점검
  + state 변경을 최소 컴포넌트 단위로 분리하는 게 좋다는 것의 증명
* 컴포넌트 memoization

## 정리

* 실제 프로덕트는 매우 크고 복잡한 구조를 가지고 있어 디버깅이 만만치 않다.
  + 그렇기에 더 복잡해지기 전에 틈틈이 리액트 개발 도구를 활용해 의도한 대로 동작하는지 확인하면 좋다.

* 꾸준히 리액트 개발 도구를 가까이에 두고 디버깅을 손에 익히고 사전에 버그를 방지하고 꾸준히 성능을 개선하려고 노력한다면 훌륭한 리액트 애플리케이션을 개발하는 데 많은 도움을 얻을 수 있다.
  
