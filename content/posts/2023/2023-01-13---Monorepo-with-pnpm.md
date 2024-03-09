---
title: pnpm과 함께하는 Frontend 모노레포 세팅
date: "2023-01-13T14:35:37.121Z"
template: "post"
draft: false
slug: "/react/monorepo-with-pnpm"
category: "React"
tags:
  - "React"
  - "Monorepo"

description: "효율적인 프로젝트 관리를 위한 모노레포 세팅"
---

사내 프런트엔드 리팩토링을 하면서 모노레포를 도입하기로 했다. 다양한 프런트엔드 프로젝트가 있는데, 그 중 관리자용 대시보드를 
1. 사용자의 권한에 따라
2. 사용자의 담당 프로젝트에 따라
3. 사용자의 소속에 따라 

나누어서 보여줘야 했다. 
기존 프로젝트는 authorization을 기반으로 한 조건문을 활용해서 접근 권한을 나누었는데, 
리팩토링 하면서 별도의 도메인으로 각각의 기능을 나누어 보기로 했다. 
스타일(HTML, CSS)은 똑같은데, 기능(JS)코드만 달라지니, 모노레포를 사용해서 공통되는 스타일과 함수들을 `common`이라는 package에 모아두고, 다르게 구현해야할 내용들만 각각의 프로젝트 디렉토리에 포함시키기로 했다. 

모노레포의 장점은 Naver D2에서 공개한 [모던 프론트엔드 프로젝트 구성 기법 - 모노레포편](https://d2.naver.com/helloworld/0923884)에 매우 자세히 나와있다. 

많은 자바스크립트 패키지 관리자 중 [npm](https://www.npmjs.com/)과 [yarn](https://classic.yarnpkg.com/lang/en/) 대신 [pnpm](https://pnpm.io/ko/)을 선택한 이유는 성능 향상과 디스크 효율성을 극대화하기 위함이다. [자바스크립트 패키지의 역사](https://yceffort.kr/2022/05/npm-vs-yarn-vs-pnpm)는 링크로 걸어둔 블로그에 매우 상세히 설명되어있다. 

그리고 작년에 우하한형제들에서 [우아콘](https://woowacon.com/ko/2022/detailVideo/26)에서 발표한 세션에서도 pnpm의 장점을 매우 자세히 설명하고 있기 때문에, 이번에는 pnpm을 사용해보기로 했다. 

npm을 설치하지 않았다면 설치한다
```bash
npm install -g pnpm
```

그리고 디렉토리를 생성하고, 해당 디렉토리 내에서 `pnpm init`을 해준다. init을 하면 해당 디렉토리에 `package.json`을 생성한다. 생성되는 `package.json`을 아래와 같이 수정해서 
1. 우리는 모노레포를 사용할 것이고 (`workspaces`)
2. pnpm을 활용할 것(`packageManager`) 을
설정한다

```json
// package.json

{
  "name": "web-client",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "private": true,
  "scripts": {
    "preinstall": "pnpm dlx only-allow pnpm"
  },
  "engines": {"node": ">16.14.2", "pnpm": ">=7.9.0"},
  "workspaces": ["packages/*"],
  "keywords": [],
  "author": "",
  "license": "ISC",
  "packageManager": "pnpm@7.9.0", 
  "devDependencies": {
    "pnpm": "^7.9.0"
  }
}
```

다른 모노레포 툴과 다르게 json파일이 아니라 `pnpm-workspace.yaml`이라는 파일을 생성하고 아래와 같이 선언한다

```yml
packages:
  - 'packages/*'
```

모노레포에서는 각각의 프로젝트가 자바스크립트 패키지처럼 사용돼서 `packages`라는 설정을 해주는 것으로 보인다. 아래 설명은 `packages`라는 디렉토리 하단에 있는 것들이 이 모노레포의 프로젝트가 된다는 뜻이다. 이제 `package.json`의 설정대로 패키지들을 설치한다. 

```bash
pnpm install
```

`packages` 디렉토리 내에서 공용으로 사용될 `common`이라는 디렉토리를 생성하고, [vite](https://vitejs.dev/)를 사용해서 리액트 프로젝트를 하나 추가한다. 

![pnpm-create-vite]([Imgur](https://i.imgur.com/VtTwvF5.png))

[create-react-app](https://create-react-app.dev/)보다 훨씬 빠르고, 나중에 패키지 설치할 때 스크린샷을 추가하겠지만 컬러가 더 예쁘다

현재 프로젝트의 구조는 아래와 같다 
```
.
├── packages/
│   ├── admin-dashboard
│   └── common
├── package.json
├── pnpm-lock.yaml
└── pnpm-workspace.yaml
```

`admin-dashboard`에는 `package.json`이 있지만, `common`에는 없기때문에 추가해준다. 
```json
{
    "name": "@web-client/common",
    "private": true,
    "version": "1.0.0",
    "type": "module",
    "main": "./src/index.tsx",
    "dependencies": {
      "react": "^18.2.0"
    },
    "devDependencies": {
      "@types/react": "^18.0.26",
      "typescript": "^4.9.3"
    }
}
```

jsx component를 선언할 수도 있기 때문에 dependencies에 react를 추가했다. vite로 생성된 admin-dashboard에 있는 것과 맞춰서 직접 작성했는데, 만약 전체 프로젝트에 필요한 것이 아니라 특정 프로젝트에서만 어떤 패키지가 필요하다면 아래 명령어로 추가할 수 있다. 

```bash
pnpm add --filter <MONOREPO_PACHAGE_NAME> <NODE_PACKAGE_NAME>
```

`common` 디렉토리에서 jsx 컴포넌트 또는 자바스크립트 함수를 선언하고, `src/index.ts`에서 export하면 `node_modules`에 설치된 패키지처럼 사용할 수 있다. 

간단하게 버튼을 하나 만들어본다
```tsx
// packages/common/src/components/Button.tsx

import React from 'react';
import type { FC, PropsWithChildren } from 'react';

interface Props extends PropsWithChildren {}

const Button: FC<Props> = ({children}) => {
    return <button>{children}</button>;
};

export default Button;
```

이제 이 버튼을 `src/index.tsx`에서 export한다. 
```typescript
// packages/common/src/index.tsx

export {default as Button} from './components/Button';
```

이제 `admin-dashboard`에서 Button을 사용하기 위해 `@web-client/common`을 dependency로 추가한다. 

```json
// packages/admin-dashboard/package.json

{
  "name": "admin-dashboard",
  "private": true,
  "version": "0.0.0",
  "type": "module",
  "scripts": {
    "start": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "@voiceemr/common": "1.0.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0"
  },
  "devDependencies": {
    "@types/react": "^18.0.26",
    "@types/react-dom": "^18.0.9",
    "@vitejs/plugin-react": "^3.0.0",
    "typescript": "^4.9.3",
    "vite": "^4.0.0"
  }
}
```

`vite`로 프로젝트를 생성하면 `scripts`의 첫 명령어가 `dev`인데, 나중에 `pnpm start`로 모든 프로젝트들을 시작하기 위해 `start`로 수정한다. 

```bash
pnpm install
```

위 명령어를 사용해서 패키지를 설치하면, `packages/common`이 `packages/common/package.json`에 선언한 `@web-client/common`이라는 이름으로 `packages/admin-dashboard`의 `node_modules`에 추가된다

![web-client-comon](https://i.imgur.com/MilSMuJ.png)

이제 import해서 코드를 구현하고 
```tsx
// packages/admin-dashabord/src/App.tsx

import reactLogo from './assets/react.svg'
import './App.css'
import { Button } from '@voiceemr/common'

function App() {
  return (
    <div className="App">
      <div>
        <a href="https://vitejs.dev" target="_blank">
          <img src="/vite.svg" className="logo" alt="Vite logo" />
        </a>
        <a href="https://reactjs.org" target="_blank">
          <img src={reactLogo} className="logo react" alt="React logo" />
        </a>
      </div>
      <h1>Vite + React</h1>
      <div className="card">
        <Button>common에서 가져왔어요</Button>
      </div>  
    </div>
  )
}

export default App
```

```bash
pnpm start
```

화면을 확인한다 

![vite-in-action](https://i.imgur.com/5bLZL8i.png)

잘 올라간 것을 확인할 수 있다. 

초기 세팅만 완료된거라 추후 업데이트 되면 새로운 포스팅에서 공유해보겠다. 