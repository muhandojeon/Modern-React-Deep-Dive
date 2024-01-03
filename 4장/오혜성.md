# 서버 사이드 렌더링

## 서버 사이드 렌더링이란?

### SPA

렌더링과 라우팅에 필요한 대부분의 기능을 서버가 아닌 브라우저의 자바스크립트에 의존하는 방식

* `history.pushState`,             `history.replaceState`로 페이지 전환이 이루어짐

* 최초에 로딩해야 할 자바스크립트 리소스가 커지는 단점
* 한번 로딩된 이후에는 서버를 거쳐 필요한 리소스를 받아올 일이 적어져 UXUI가 좋음

* SPA 중 가장 완성도가 높다고 손꼽히는 것은 Gmail

### JAM 스택

JavaScript, API, Markup

* 과거 PHP, JSP 기반 웹 애플리케이션에 비해 더 간편한 개발 경험 제공
  
* 평균 자바스크립트 리소스 크기가 모바일, 데스크톱에서 모두 꾸준히 증가하고 있으며 12년 대비 100배 이상 증가함
* 스크립트가 페이지당 소비하는 CPU 시간도 300% 증가함

* 사용자 기기와 인터넷 속도 등 웹 전반을 이룬느 환경이 크게 개선됐음에도
  + 실제 사용자들이 느끼는 웹 애플리케이션의 로딩 속도는 크게 차이가 없거나 오히려 더 느리다
  + 그렇기 때문에 성능을 역행하는 추세에 책임감을 가질 필요가 있다

### 서버 사이드 렌더링 장점

* 최초 페이지 진입이 비교적 빠르다
  + 최초 페이지 진입 시 유의미한 정보가 그려지는 시간(First Contentful Paint)이 더 빨라질 수 있다
    - 일반적으로 서버에서 HTTP 요청을 수행하는 것이 더 빠르고
    - HTML을 그리는 작업도 서버에서 해당 HTML을 문자열로 미리 그려서 내려주는 것이 클라이언트에서 기존 HTML에 삽입한느 것보다 더 빠르기 때문

* 검색 엔진과 SNS 공유 등 메타데이터 제공이 쉽다

<!-- 추가한 내용 -->

```
https://developers.google.com/search/docs/crawling-indexing/javascript/javascript-seo-basics?hl=ko
구글의 서치 엔진은 자바스크립트를 부분적으로 실행함
```

* 누적 레이아웃 이동이 적다
  + 누적 레이아웃 이동이란 사용자에게 페이지를 보여준 이후에 뒤늦게 어떤 HTML 정보가 추가되거나 삭제되어 마치 화면이 덜컥 거리는 것과 같은 부정적인 사용자 경험을 말함
  + 리액트 18의 스트림으로 해결할 수도 있다

* 사용자 디바이스 성능에 비교적 자유롭다
* 보안에 좀 더 안전하다

### 서버 사이드 렌더링 단점

* 소스코드를 작성할 때 항상 서버를 고려해야 한다
* 적절한 서버가 구축돼 있어야 한다

<!-- 추가한 내용 -->

```
여러분들의 웹 서버는 어떻게 구성돼 있는지 궁금합니다요
```

* 서비스 지연에 따른 문제
  + TTFB(time to first byte)가 느릴 시 어떤 정보도 제공할 수 없음

### 결국엔

서버 사이드 렌더링 역시 만능이 아니다

* 가장 뛰어난 SPA는 가장 뛰어난 멀티 페이지 애플리케이션보다 낫다
* 평균적인 SPA는 평균적인 멀티 페이지 애플리케이션보다 느리다
  + 최근에는 멀티 페이지 애플리케이션에서 발생하는 라우팅으로 인한 문제를 해결하기 위해 다양한 브라우저 API가 추가되고 있음
    - 페인트 홀딩: 같은 출처에서 라우팅이 일어날 경우 화면을 잠깐 하얗게 띄우는 대신 이전 페이지 모습을 보여주는 기법
    - back forward cache: 브라우저 앞으로 가기, 뒤로가기 실행 시 캐시된 페이지를 보여주는 기법
    - Shared Element Transition: 라우팅 시 두 페이지에 동일 요소가 있다면 콘텍스트를 유지해 부드럽게 전환되게 하는 기법

### 현대의 서버 사이드 렌더링

두 가지 장점을 모두 취한 방식으로 작동함

* 최초에는 SSR로
* 이후 라우팅은 자바스크립트를 바탕으로 SPA처럼

## 서버 사이드 렌더링을 위한 리액트 API

### renderToString

리액트 컴포넌트를 렌더링해 HTML 문자열로 반환하는 함수

* 훅과 이벤트 핸들러는 결과물에 포함되지 않는다
  + 이는 의도된 것으로 컴포넌트를 빠르게 브라우저가 렌더링할 수 있는 HTML을 제공하는 데 목적이 있는 함수기 때문
* `data-reactroot`는 리액트 컴포넌트의 루트 엘리먼트가 무엇인지 식별하는 역할을 함
  + hydrate 함수에서 루트를 식별하는 기준점

### renderToStaticMarkup

renderToString처럼 HTML 문자열을 만들지만, `data-reactroot` 와 같은 리액트에서만 사용하는 추가적인 DOM 속성을 만들지 않음

* hydrate를 수행하지 않는다는 가정하에 순수한 HTML만 반환

### renderToNodeStream

결과물은 renderToString과 완전히 동일하지만 몇가지 차이점이 있음

* 브라우저에서 사용하는 것이 완전히 불가능
  + 결과물이 Node.js의 ReadableStream
    - ReadableStream은 utf-8로 인코딩된 바이트 스트림
      - 스트림은 큰 데이터를 다룰 때 데이터를 청크로 분할해 조금씩 가져오는 방식을 의미
      - ReadableStream은 브라우저에서도 사용할 수 있는 객체지만 만드는 과정이 브라우저에서 불가능하게 구현되어 있음
      

* 크기가 큰 문자열을 한 번에 메모리에 올려두고 응답을 수행하면 서버에 부담이 될 수 있음
  + 청크 단위로 분리하면 순차적으로 처리할 수 있는 장점이 있음

* 대부분 널리 알려진 리액트 SSR 프레임워크는 모두 renderToNodeStream을 사용함

### renderToStaticNodeStream

* renderToNodeStream과 동일하지만 `data-reactroot`와 같은 리액트에서만 사용하는 추가적인 DOM 속성을 만들지 않음

### hydrate

renderToString, renderToNodeStream으로 생성된 HTML에 자바스크립트 핸들러나 이벤트를 붙이는 역할

* render와의 차이점은 기본적으로 이미 렌더링된 HTLM이 있다는 가정하에 수행되는 것
  + 그리고 렌더링된 HTML 기준으로 이벤트를 붙이는 작업만 실행한다는 것

* 불일치가 발생하면 hydrate가 렌더링한 기준으로 웹페이지를 그리게 됨

* 제한적인 경우에 `suppressHydrationWarning` 속성을 추가해 경고를 끌 수 있음

## Next.js

* PHP에 영감을 받아 만들어짐
  + 실제로 PHP 대용품으로 사용되기 위해 만들어졌다고 언급한 적 있음

* 메타 팀에서 리액트 기반 SSR을 위해 고려했던 프로젝트로 `react-page`가 있음

* 설정 파일을 읽는 소스 코드

<!-- 추가한 내용 -->

```
https://github.com/vercel/next.js/blob/canary/packages/next/src/server/config-schema.ts

내부적으로 zod를 사용하네용
```

* Next.js에는 `next/document` head와 `next/head`가 있음
  
* `pages/_error.tsx`
  + 클라이언트에서 발생하는 에러와 서버에서 발생하는 500 에러를 처리
  + 개발 모드에서는 방문할 수 없음
    - 확인하려면 프로덕션으로 빌드해서 확인해야 함

* `pages/500.tsx`
  + 서버에서 발생하는 에러를 핸들링
  + _error와 500 둘 다 있다면 500이 우선적으로 실행됨

* `pages/foo/[bar].tsx`
  + 만약 `pages/foo/lorem.tsx`와 같이 이미 정의된 주소가 있다면 이것을 우선시함

* `pages/foo/[...props].tsx`
  + /foo를 제외한 foo 하위의 모든 주소가 여기로 옴
    - `/foo/bar`, `foo/bar/baz` 등

* getServerSideProps가 없다면 해당 페이지를 서버에서 실행하지 않아도 되는 페이지로 처리
  + typeof window를 모두 object로 바꾸고 빌드 시점에 트리쉐이킹 해버림

* `__NEXT_DATA__`
  + getServerSideProps의 정보인 props뿐만 아니라 현재 페이지 정보, query 등 Next.js 구동에 필요한 다양한 정보가 담겨 있음
  + 왜 스크립트 형태로 삽입돼 있을까?
    - 서버에서 fetch한 정보를 중복해서 클라이언트에서 다시 fetch하지 않도록 하기 위함
      - Next.js는 이 정보를 window 객체에도 저장해 둠

* 일반적인 리액트 JSX와 다르게 getServerSideProps의 props로 내려줄 수 있는 값은 JSON으로 제공할 수 있는 값으로 제한되어 있음
  + JSON.stringify로 직렬화할 수 있는 값만 제공해야 함
  + 값에 대해 가공이 필요하다면 실제 페이지나 컴포넌트에서 하는 것이 옳음

* `getInitialProps`
  + 라우팅에 따라서 서버와 클라이언트 모두에서 실행 가능함
  + context.req에 isServer 값이 있음

* `context.query`
  + pathname에 있는 값도 포함되고, 같은 이름으로 쿼리 문자열이 있다면 pathname에 있는 값이 우선시됨

* 프로덕션 모드로 빌드 시 `<style />` 내부가 비어있으나, 스타일은 적용돼 있음. 왜?
  + styled-components가 개발 모드와 다르게 프로덕션에서는 `SPEEDY_MODE`라고 하는 설정을 사용하기 때문
  + HTML에 스타일을 적용하지 않고, 자바스크립트를 활용해 CSSOM 트리에 직접 스타일을 넣음
  + 이는 기존 스타일링 방식보다 훨씬 빠름

* `next/link` 라우팅은 클라이언트 렌더링처럼 동작함
  + 라우팅 하는 페이지에 getServerSideProps 같은 서버 관련 로직이 있을 떄
    - 해당 페이지의 getServerSideProps의 실행 결과의 json 파일만을 요청해서 가져옴

* `next.config.js`
  + basePath: `localhost:3000` > `localhost:3000/foo`
  + poweredByHeader: 응답 헤더의 `X-Power-by`를 제공하지 않을 수 있음
  + assetPrefix: 빌드된 결과물을 동일한 호스트가 아닌 다른 CDN 등에 업로드하고자 할 때 CDN 주소를 명시

* 리액트 개발 팀이나 구글 Core Web Vitals 팀에서도 Next.js 개발진과 긴밀하게 협업 중에 있음

<!-- 추가한 내용 -->

## 추가적으로 읽어보면 재미있을 글들

* remix, react/testing-lib 개발자 Kent C. Dodds의 '내가 Next.js를 사용하지 않는 이유' 
  + https://www.epicweb.dev/why-i-wont-use-nextjs

* 이에 대한 답가?로 Vercel의 Lee Robinson의 '내가 Next.js를 사용하는 이유'
  + https://leerob.io/blog/using-nextjs
