---
title: ElectronJs에 React 연결하기
date: "2022-05-01T23:34:37.121Z"
template: "post"
draft: false
slug: "/electron/connecting-electron-with-react"
category: "Electron"
tags:
  - "Electron"

description: "ElectronJS에 React를 연동해서 개발한 경험을 공유합니다."
---

이건 회사 프로젝트 건은 아니고, 진행하는 국책과제중에 TTA라는 기관에서 인증을 받아야 하는 절차가 있었다. 병원과 EMR연동이 실제로 이루어졌는지 확인하는 테스트였는데, API만 제공하니 비용이 너무 높게 책정돼서 자동화 프로그램을 같이 제공했어야 했다. 

웹에서 간단하게 띄워서 제공해도 되지만, 데스크탑 어플리케이션을 만들어서 제공하면 있어보일 것 같다는 생각이 들어 [electronjs](https://www.electronjs.org/)를 사용해보기로 했다. 그리고 기관에서 윈도우 PC를 사용할 가능성이 높아서 윈도우에서 개발했다. 입사했을 때 [Kivy](https://kivy.org/)를 사용해서 윈도우 프로그램을 만들었어서 윈도우 장비를 지급받았는데, iOS개발한다고 한동안 묵혀두다가 오랜만에 다시 꺼냈다. 사양은 괜찮은 컴퓨터인데 놀리는게 살짝 아깝긴하다. 

[튜토리얼](https://www.electronjs.org/docs/latest/tutorial/tutorial-first-app)을 잘 따라하면 Hello World 까지는 어렵지 않다. 

문제는 EMR연동을 확인해야 하기 때문에 서버로 HTTP request를 보내야하는데, Electron에서 제공하는 [ClientRequest](https://www.electronjs.org/docs/latest/api/client-request)를 사용해서는 우리 서버로 요청을 보낼 수 없었다. 그래서 찾아보니 React를 연동할 수 있는 방법이 있어서 React와 연동하고 Fetch API를 사용하기로 했다. 

ElectronJS은 npm init 후 직접 `package.json`을 수정하기 때문에, [create-react-app](https://create-react-app.dev/)을 통해 리액트 환경을 설정하고, 여기에 electron을 얹어봤다. 

```bash
npx create-react-app tta
```

회사 프로젝트라면 타입스크립트를 썼겠지만, 최대한 공수를 줄이기 위해 자바스크립트로 진행한다. 먼저 electron을 devDependencies에 추가한다. 

```bash
npm install --save-dev electron
```

그리고 `public` 디렉토리 아래에 `electron.js` 파일을 생성한다. 

```javascript
// electron.js
const { app, BrowserWindow } = require('electron');

app.whenReady().then(() => {
  const win = new BrowserWindow();
  win.loadURL('http://localhost:3000'); // 리액트의 default port
})

app.on('window-all-closed', () => {
    app.quit();
})
```

electron 튜토리얼에 나온 것 처럼 `package.json`의 script를 변경한다.

```json
// package.json
{
  "name": "PuzzleAI TTA TC2",
  "main": "public/electron.js",
  "homepage": "./",
  "scripts": {
    "react:start": "react-scripts start",
    "electron:start": "electron ."
  },
  ...
}
```

설정이 완료 되었으니, react를 먼저 올려서 http://localhost:3000을 먼저 띄우고, electron을 실행하면 electron이 실행한 브라우저가 localhost:3000을 가리키게 된다. 예를들면 아래와 같은 명령어를 사용한다 

```bash
npm run react:start && npm run electron:start
```

아쉽게도 이 방법은 작동하지 않는다. react 프로젝트를 생각해보면, 수정사항을 저장할 때 마다 자동으로 화면에 반영된다. 이는 react 프로젝트는 `react-scripts start`로 구동된 후에도 계속적으로 watch 상태이기 때문이다 (next.js도 그렇던데 대부분 npm 개발모드가 그런듯). 그래서 npm package중 [concurrently](https://www.npmjs.com/package/concurrently)를 사용해야한다.

이제 `package.json`의 script를 다시 수정한다

```json
// package.json
{
  "scripts" : {
    "react:start": "react-scripts start",
    "electron:start": "electron .",
    "start": "concurrently \"npm run react:start\" \"npm run electron:start\""
  }
}
```

하지만 이것도 작동하지 않는다. concurrently라는 단어 뜻에 이유가 숨어있는데, react를 올리고나서 await한 후에 electron을 올리는게 아니라 동시에 올라가기 때문이다. 따라서 화면은 비어있다. npm package중 [wait-on](https://www.npmjs.com/package/wait-on)을 사용해서 electron이 http://localhost:3000을 기다렸다가 실행되도록 `package.json`을 수정한다.

```json
// package.json
{
  "scripts" : {
    "react:start": "react-scripts start",
    "electron:start": "wait-on http://localhost:3000 && electron .",
    "start": "concurrently \"npm run react:start\" \"npm run electron:start\""
  }
}
```

`npm run start`를 하면 electron에서 http://localhost:3000을 참조하는 것을 볼 수 있다. 하지만 브라우저가 먼저 보이고, 같은 화면을 electron에서 보게 되는데, 브라우저가 열리는 것을 막기 위해 `BROWSER=none`을 `.env`에 추가한다. `npm run start`를 하면 브라우저는 열리지 않는 것을 확인할 수 있다.

이제 배포준비를 한다. 배포는 [electron-builder](https://www.npmjs.com/package/electron-builder)라는 npm package를 사용하는 것이 일반적이다. 설치 후 package.json에 설정을 추가한다. 윈도우로 배포할 것이기 때문에 윈도우 설정만 해주면 된다.

```json
// package.json
{
  "scripts" : {
    "react:start": "react-scripts start",
    "electron:start": "wait-on http://localhost:3000 && electron .",
    "start": "concurrently \"npm run react:start\" \"npm run electron:start\"",
    "react:build": "react-scripts build",
    "build:win": "npm run react:build && electron-builder --win --publish=always"
  },
  "build": {
    "win": {
      "target": "nsis",
      "icon": "public/logo.ico",
      "publish": [
        "github"
      ]
    },
    "nsis": {
      "oneClick": true,
      "installerIcon": "public/logo.ico",
      "shorcutName": "PuzzleAI",
      "artifactName": "PuzzleAI_${version}.exe"
    }, 
    "files": [
      "build/**/*",
      "node_modules/**/*"
    ],
    "directories": {
      "buildResources": "assets"
    }
  }
}
```

`npm run build:win`을 하게 되면 react 프로젝트를 빌드하는 것 마냥 `build`라는 디렉토리가 생성되고 그 안에 `index.html`이 생성된다. 하지만 배포 시에는 http://localhost:3000이 아니라 빌드에 생성된 index.html을 지시해야한다. 따라서 electron.js파일을 수정한다.

```javascript
// electron.js
const { app, BrowserWindow } = require('electron');
const path = require('path');

app.whenReady().then(() => {
  const win = new BrowserWindow();
  win.loadFile(`file://${path.join(__dirname, '../build/index.html')}`)
})

app.on('window-all-closed', () => {
    app.quit()
})
```

이렇게 되면 끝인 것 같지만, 인증서가 없다는 에러가 발생한다. 예전에 kivy로 개발할 때는 [PyInstaller](https://pyinstaller.org/en/stable/)로 만든 설치파일에 애니서트에서 구입한 인증서를 씌워줬었는데, `package.json` 설정에서 보이듯이 `github`을 publisher로 정했기 때문에, 깃헙의 [personal token](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/creating-a-personal-access-token)을 사용하면 인증서를 대신할 수 있다. 발급받은 토큰을 환경변수로 저장하면 된다 

```json
// .env
BROWSER=none
GH_TOKEN=GITHUB_SECRET_TOKEN
```

이제 `npm run build:win`을 하면 설치파일이 생성되고, 그 설치파일을 실행하면 electron 으로 빌드된 파일을 데스크탑 어플리케이션으로 실행할 수 있다. 매우 간단한 호출만 하면 돼서 전역변수 툴들은 사용하지 않고 fetch api를 사용해서 로컬에서 모든 것을 처리했다. 인증은 잘 통과했고 과제는 성공적으로 잘 마무리 되었다!