---
title: monorepo에 tailwindcss 세팅
date: "2023-01-23T14:35:37.121Z"
template: "post"
draft: false
slug: "/react/monorepo-with-tailwind"
category: "React"
tags:
  - "React"
  - "Monorepo"
  - "Tailwind"

description: "손쉬운 css 적용을 위한 tailwind 설정"
---

한동안 CSS-in-JS가 유행이어셔 [styled-components](https://styled-components.com/)를 활용했다. 하지만 [Next.js](https://nextjs.org/)로 프로젝트를 진행해보니 런타임에 동작하는 styled-components 특성상 서버사이드 렌더링이 일어날 때 어떤 스타일을 적용할 수 없는지 파악이 되지 않아

1. 스타일이 적용되지 않거나
2. 새로고침을 해야지 스타일이 적용되는

문제가 있었다. 따라서 추가로 패키지를 설치하거나, document 설정을 해줬어야 했는데. 타입스크립트를 사용하는 경우 이 document 설정이 상당히 귀찮았다.

그래서 다시 sass나 scss를 쓰려고 찾아보던 중 요즘 [tailwindcss](https://tailwindcss.com/)가 인기라서 설정을 해보기로 했다.

공식홈페이지에 나온대로 따라하면 아래와 같은 과정을 거치면 된다

```npm
pnpm add -D -w tailwindcss postcss autoprefixer
npx tailwindcss init -p
```

`-w` 명령어를 제외하고 설치를 시도하면 `root에 설치되는데 진심이냐는 경고메세지`가 나온다. 모노레포를 사용하면 root에 패키지를 설치하는 것 보다 각 워크스페이스의 빌드 환경에 맞게 필요한 NPM 패키지를 따로 설치하는 것이 일반적이기 때문이다. common에 공용된 style을 선언하고 import해서 사용할 예정이기 때문에 project root에 tailwind 관련 패키지들을 설치했다. 패키지를 따로 설치하려면 아래 명령어를 사용하면 된다.

```npm
pnpm add --filter <WORKSPACE_NAME> <NPM_PACKAGE>
```

[PostCSS](https://postcss.org/)는 ChatGPT에 따르면 TailwindCSS와 같이 사용했을 때 최적화나 각종 설정을 편하게 해준다고 한다.

위 명령어를 실행하면 `tailwind.config.js`, `postcss.config.js` 라는 파일이 만들어진다.

`tailwind.config.js`에 스타일들이 존재하는 경로를 지정해주고

```javascript
// tailwind.config.js

/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ["./index.html", "./src/**/*.{js,jsx,ts,tsx}"],
  theme: {
    extend: {},
  },
  plugins: [],
};
```

추가로 `index.css`에 아래 tailwind 설정들을 추가한다

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

프로젝트를 구동하면 아래와 같은 에러가 발생한다.

![js-cjs-error](https://i.imgur.com/ad4tWQN.png)

`package.json`에 type이 `module`로 되어있기 때문에 설정파일에 있는 `module.exports`를 사용하려면 extension을 `.cjs`로 변경해줘야한다. extension을 변경해주고 구동하면 잘 돌아간다.

![vite-run-success](https://i.imgur.com/ZYD2O1J.png)

이제 common에 선언한 패키지를 import해서 사용해본다

```jsx
// admin-dashboard/App.tsx

import reactLogo from "./assets/react.svg";
import "./App.css";
import { Button } from "@app/common";

function App() {
  return (
    <div className="App">
      <div className="flex justify-center">
        <a href="https://vitejs.dev" rel="noreferrer" target="_blank">
          <img alt="Vite logo" className="logo" src="/vite.svg" />
        </a>
        <a href="https://reactjs.org" rel="noreferrer" target="_blank">
          <img alt="React logo" className="logo react" src={reactLogo} />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <Button>Common에서 가져왔어요</Button>
      </div>
    </div>
  );
}

export default App;
```

매우 잘 나온다
![vite-whatup](https://i.imgur.com/92WDrsG.png)

```jsx
<div className="flex justify-center">
```

여기있는 className들이 tailwindcss로 적용한 스타일이다. css로 변환하면 아래와 같은 설정이다

```css
 {
  display: flex;
  justify-content: center;
}
```

이제 common에서 tailwind를 사용해서 스타일을 수정해준다

```jsx
// common/components/Button.tsx

import type { PropsWithChildren, ReactElement } from "react";

interface Props extends PropsWithChildren {}

export default function Button({
  children,
}: Props): ReactElement<HTMLButtonElement> {
  return (
    <button className="underline font-bold text-3xl bg-slate-50" type="button">
      {children}
    </button>
  );
}
```

아주 잘 적용된 것을 볼 수 있다.

![tailwind-success](https://i.imgur.com/Bua81wr.png)
