---
title: Reactjs 프로젝트 Nextjs로 이관
date: "2022-07-28T01:41:37.121Z"
template: "post"
draft: false
slug: "/nextjs/migrating-from-react"
category: "Nextjs"
tags:
  - "Nextjs"

description: "React로 이루어진 프로젝트를 Next.js로 이관한다"
---

회사에서 새로운 프로젝트를 하는데, 기획에 대시보드가 생겼다. 
대시보드는 특성상 그래프가 많이 들어가서 서버에서 불러온 데이터를 화면에서 보여주는데 시간이 많이 걸릴 것 같았다.
[마이리얼트립 블로그](https://medium.com/myrealtrip-product/%EC%83%81%ED%99%A9%EC%97%90-%EB%A7%9E%EB%8A%94-%EB%A1%9C%EB%94%A9-%EC%95%A0%EB%8B%88%EB%A9%94%EC%9D%B4%EC%85%98-%EC%A0%81%EC%9A%A9%ED%95%98%EA%B8%B0-2018af51c197)에서 본 Skeleton UI를 적용할 까 고민하다가, 서버사이드 렌더링이 해결책이 될 수 있을 것 같았다. 

Next.js는 사이드 프로젝트 목적으로 공부중이기도 하고, 사실 지금 private repo에 완성된 프로젝트들도 있어서, 생각보다 쉽게 Next.js로 전환할 수 있을 것 같았다. 
지금 React로 이루어진 웹 클라이언트의 구조는 아래와 같다 
```tree
.
├── README.md
├── package-lock.json
├── package.json
├── public
├── src
│   ├── App.tsx
│   ├── hooks
│   ├── index.tsx
│   ├── libs
│   │   └── axios
│   ├── pages
│   │   ├── Login
│   │   │   ├── index.tsx
│   │   │   └── queries
│   │   │       └── useLoginQuery.ts
│   │   └── recoil
│   │       └── auth
│   │           ├── atom.ts
│   │           └── index.ts
│   ├── react-app-env.d.ts
│   ├── setupTests.ts
│   ├── styles
│   ├── types
│   └── utils
└── tsconfig.json
```

상세한 파일 컴포넌트 명들은 생략했다. 
디렉토리 구조 상 React.js와 Next.js의 차이는, Next.js에는 `pages`라는 디렉토리가 `src`의 밖에 위치하며, `pages` 디렉토리 내의 sub-directory들의 이름으로 routing이 이루어진다는 것이다. 

따라서 지금 React.js 프로젝트에서 `App.tsx` 내에서 [react-router](https://v5.reactrouter.com/web/guides/quick-start)로 import해서 사용하는 `src/pages` 내의 컴포넌트들을 `pages/login`과 같은 디렉토리에서 import해서 사용해주면 필요한 패키지가 모두 설치되어있다는 전제하에 정상적으로 작동해야한다. 

일단 create-next-app을 사용해서 프로젝트를 init한다
```bash
npx create-next-app PROJECT_NAME --typescript
```

자바스크립트 사용 프로젝트는 항상 타입스크립트를 사용한다. 블로그에서 여러번 언급했지만, University College London과 마이크로소프트가 발표한 [논문](https://earlbarr.com/publications/typestudy.pdf)에 따르면, 자바스크립트 기준으로 볼 때 타입스크립트를 사용하면 버그가 15%정도 줄어든다고 한다. 회원가입(CREATE) 기준으로 작성한다. 놓치는 버그들을 잡는데 유리하고, 타입스크립트를 사용하면(any를 남발하지 않는다는 전제하에) 프로젝트 규모가 커졌을 때 용이하다. any를 남발하지 않기위해 `tsconfig.json`에서 항상 `"noImplicitAny": true`로 설정하고 프로젝트를 진행한다. 

처음 생성된 Next.js 프로젝트의 디렉토리 구조는 아래와 같다. 
```tree
.
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── pages
│   ├── _app.tsx
│   ├── api
│   │   └── hello.ts
│   └── index.tsx
├── styles
│   ├── Home.module.css
│   └── globals.css
├── tsconfig.json
└── yarn.lock
```

이제 React.js 프로젝트의 `src` 디렉토리를 옮긴다. 스타일은 React.js프로젝트에 적용되어있던 것을 사용할 것이기 때문에 `styles` 디렉토리는 삭제한다. 그럼 아래와 같이 구조가 변경된다

```tree
.
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── pages
│   ├── _app.tsx
│   ├── api
│   │   └── hello.ts
│   └── index.tsx
├── src
│   ├── App.tsx
│   ├── hooks
│   ├── index.tsx
│   ├── libs
│   │   └── axios
│   ├── pages
│   │   ├── Login
│   │   │   ├── index.tsx
│   │   │   └── queries
│   │   │       └── useLoginQuery.ts
│   │   └── recoil
│   │       └── auth
│   │           ├── atom.ts
│   │           └── index.ts
│   ├── react-app-env.d.ts
│   ├── setupTests.ts
│   ├── styles
│   ├── types
│   └── utils
├── tsconfig.json
└── yarn.lock
```

next에 필요없는 파일들을 삭제하면 구조는 아래와 같이 바뀐다. 


```tree
.
├── README.md
├── next-env.d.ts
├── next.config.js
├── package.json
├── pages
│   └── login
│       └── index.tsx
│   ├── _app.tsx
│   ├── api
│   │   └── hello.ts
│   └── index.tsx
├── src
│   ├── hooks
│   ├── index.tsx
│   ├── libs
│   │   └── axios
│   ├── pages
│   │   ├── Login
│   │   │   ├── index.tsx
│   │   │   └── queries
│   │   │       └── useLoginQuery.ts
│   │   └── recoil
│   │       └── auth
│   │           ├── atom.ts
│   │           └── index.ts
│   ├── styles
│   ├── types
│   └── utils
├── tsconfig.json
└── yarn.lock
```

이제 `pages` 디렉토리 아래에 `login` 디렉토리를 만들고, 해당 디렉토리 내의 `index.tsx`에서 `src/pages/Login`의 component를 import한다. 

```typescript
import Login from "pages/Login";

export default function LoginPage() {
  return <Login />;
}
```

절대경로를 사용한 것을 볼 수 있는데. 개인적으로 절대경로를 선호한다. 리팩토링 하거나 프로젝트 구조를 변경할 때 절대경로를 사용하면 에러가 덜 나기 때문이다. TypeScript에서 절대경로를 사용하려면 `tsconfig.json`에서 `baseUrl: "./src"`로 해주면된다. 

이제 실행해서 확인한다 
```bash
npm run dev
```

아래와 같이 잘 보이는 것을 확인할 수 있다. 
![login-with-nextjs](https://i.imgur.com/w4Be4er.png)