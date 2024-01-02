## 혜성

- spa중 가장 완성도가 높은 것은 지메일
- 기술 발전에 따라 성능이 좋아졌을 줄 알았는데, 오히려 리소스의 증가로 성능은 안좋아졌다
- 많이들 SEO에 유리한 것을 SSR의 장점으로 꼽는데, 구글 크롤링봇은 js도 잘 크롤링 해온다.
- SSR의 단점은 서버를 구성해야한다는 것인데, vercel 외에 다른 수단을 사용하는지 궁금하다.
- nextjs는 SSR CSR을 섞어서 동작한다.
- nextjs는 PHP에 영감을 받아 만들어짐.
  - zod라는 라이브러리를 사용하고 있다.
- styled-components는 개발모드와 프로덕션에서 다르게 동작한다.
  - 프로덕션에서는 HTML에 스타일을 적용하지 않고, 자바스크립트를 활용해 CSSOM 트리에 직접 스타일을 넣음
- https://www.epicweb.dev/why-i-wont-use-nextjs

## 창완

- SSR의 장점 중에서 사용자 기기별 편차가 적다는게 기억에 남음
- nextjs를 vercel이 아닌 다른 수단으로 배포하면 손이 많이 가고 인프라 지식이 필요한게 단점
- 서버컴포넌트에 대한 이슈가 많아서 그것도 단점
- SSR이 CSR보다 더 오래걸릴때, 유저에게 더욱 안좋은 경험을 줄 수 있다.
- renderToString, renderToNodeStream을 자세히 알게됨
- nextjs의 빌드 결과물을 자세히 볼 수 있어서 좋았음.
- getServerSideProps를 쓰면 빌드 결과물이 자바스크립트. 번들 사이즈가 더 큼
- css-in-js는 context를 사용하는데, 이 때문에 서버 컴포넌트에서 사용이 불가능하다.

## 대윤

- SSR의 단점 중에서 적절한 서버가 필요하는 내용이 있는데, 회사에서 CRUD를 next server에 구현해뒀던 것을 분리했다.
- renderToNodeStream에서 큰 데이터를 청크 단위로 분할해서 가져오는게 신기했다.
- 하이드레이션 에러가 발생시 suppressHydrationWaring으로 해결할 수 있다.
- nextjs 13.4 부터 reactStrictMode가 true 기본

## 오연

- SSR, CSR, SPA등의 개념은 면접 단골
- JAM 스택
- CSR은 사용자 기기의 성능에 영향을 받지만 SSR은 서버에서 제공하기 때문에 비교적 안정적인 렌더링이 가능
- 최초 렌더링에 지연이 발생된다면 서버에서 사용자에게 보여줄 페이지에 대한 렌더링 작업이 끝나기 까지 사용자에게 그 어떤 정보도 제공할 수 없다 → dynamic import 나 Suspense 같은 것들로 개선은 가능
- hydrate에 대해 제대로 정의할 수 있었음
- getServerSideProps에서 에러가 발생하면 500.tsx로 리다이렉트
- 경로에 없는 라우팅을 했을 경우 404.tsx 페이지로 리다이렉트
