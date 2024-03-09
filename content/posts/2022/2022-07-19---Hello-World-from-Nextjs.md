---
title: Hello World from Next.js
date: "2022-07-19T11:21:37.121Z"
template: "post"
draft: false
slug: "/nextjs/hello-world"
category: "Nextjs"
tags:
  - "Nextjs"

description: "Next.js로 Hello World!"
---

[서버에서 회원가입 api](https://jasonkang14.github.io/fastapi/signup-view)를 만들었으니, 이제 클라이언트를 만들어본다. 

모바일 앱으로 하려다가, 개발자 등록 비용도 지불해야하고, 승인하는 절차도 오래걸리니 웹으로 일단 만들어 보려고 한다. 하는김에 [SEO](https://developers.google.com/search/docs/beginner/seo-starter-guide)에 유리하도록 Server-Side Rendering을 하기 위해 [Next.js](https://nextjs.org/)를 선택했다. Next.js를 선택한 또 다른 이유는 배포가 매우 쉽기 때문인데, Vercel에 깃헙 레포만 등록해주면 자동으로 CI/CD 까지 다 해준다. 

이제 본격적으로 시키는 대로 init을 해본다. 언어는 TypeScript를 사용하려고 하는데, 그 이유는 University College London과 마이크로소프트가 발표한 [논문](https://earlbarr.com/publications/typestudy.pdf)에 따르면, 타입스크립트를 사용하면 버그가 15%정도 줄어든다고 한다. 성격상 놓치는 부분이 많을거라서, 언어의 힘을 빌려본다. 

```bash
npx create-next-app mukkang_client --typescript
```

위 명령어를 실행하면 아래와 같은 구조의 tree가 생성된다.
```
.
├── README.md
├── next-env.d.ts
├── next.config.js
├── node_modules
├── package-lock.json
├── package.json
├── pages
│   ├── _app.tsx
│   ├── api
│   │   └── hello.ts
│   └── index.tsx
├── public
│   ├── favicon.ico
│   └── vercel.svg
├── styles
│   ├── Home.module.css
│   └── globals.css
└── tsconfig.json
```

예전에 사이드 프로젝트를 하기 위해서 받아놓은 디자인이 있어서, 그 디자인을 활용하려고 한다. 같이 진행하기로 했던 친구한테 허락도 받았다. 스타일은... 뭐 이것저것 써보려다가 그냥 styled-component로 간다. 패키지를 설치하고 디자이너분이 주셨던 컬러와 폰트 크기들을 미리 세팅한다. 

```bash
npm install --save styled-components && npm install --save-dev @types/styled-components
```

폰트는 [fontsource](https://fontsource.org/fonts/noto-sans-kr)에서 다운받았다. 직접 .otf와 같은 파일을 다운받아서 적용하는 것보다 용량이 많이 세이브 된다. 
![fontsource vs localfile](https://i.imgur.com/bYp4Gtr.png)
회사에 적용하려고 스크린샷 찍어둔게 있었다. 

```bash
npm install --save @fontsource/noto-sans-kr
```

이미 있는 `styles` 디렉토리에 `fonts.ts`를 추가하고, `globals.css`에 컬러들을 추가한다.
```typescript
// fonts.ts

import { css } from "styled-components";

const Bold = css`
  font-weight: bold;
`;

const Regular = css`
  font-weight: regular;
`;

export const H1 = css`
  ${Bold}
  font-size: 30px;
`;
```

```css
/* global.css */

html,
body {
  padding: 0;
  margin: 0;
  font-family: "Noto Sans KR", sans-serif;
}

* {
  box-sizing: border-box;
}

:root {
  --primary: #1D2745;
  --sub-100: #1de5d4;
  --sub-200: #f52c50;
  --white: #ffffff;
  --mono-100: #f1f1f1;
  --mono-200: #bebebe;
  --error: #d01e1e;
}
```

`npm run dev`를 통해 화면이 잘 나오는 것을 확인할 수 있다. 절대경로 사용을 선호하기 때문에 `tsconfig.json`에서 `baseUrl`을 설정해주고 싶은데, 아직은 어떤식으로 디렉토리 구조를 설정할지 결정하지 못했기 때문에 일단은 여기까지만 하도록 한다. 