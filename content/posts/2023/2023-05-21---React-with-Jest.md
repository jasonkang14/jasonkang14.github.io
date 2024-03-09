---
title: "Jest로 하는 React 테스트코드"
date: "2023-05-21T20:35:37.121Z"
template: "post"
draft: true
slug: "/react/unit-test-with-jest"
category: "React"
tags:
  - "React"

description: "로그인 화면에 테스트코드를 작성해보자"
---

간단한 사이드 프로젝트를 진행중입니다. [vite](https://vitejs.dev/)와 [styled-component](https://styled-components.com/)를 사용중인데요. 개발이 어느정도 진행되어서 테스트코드를 붙여보려고 합니다.

TDD가 근본인것으로 여겨지고 있지만, 테스트코드 작성은 쉽지 않습니다. 특히 회사에서 처음 프로토타입을 만들 때는 빠르게 사용자 반응을 봐야하기 때문에 더더욱 테스트코드 작성이 어렵다고 생각합니다. 물론 장기적으로 봤을 때는 테스트코드를 작성하는 것이 생산성에 도움이 되지만, 사용자 반응을 보고 서비스 자체를 하지 않을 수도 있는데 프로토타입에 테스트코드를 작성하는 것은 비효율적이라는 생각입니다.

따라서 회사에서 프로토타입을 만들 때, 저는

1. 일단은 테스트 코드 없이 작성하고,
2. 해당 프로젝트를 진행하기로 하면 **리팩토링 전에** 테스트코드를 작성합니다.

리팩토링할 때 테스트코드가 없다면, 물론 배포 전에 QA를 진행하기 때문에 서비스 자체에는 크게 문제 없지만, 리팩토링 효율이 떨어지기 때문입니다.

그럼 본격적으로(?) 테스트 코드를 보시겠습니다. 간단하게 로그인 화면부터 테스트코드를 작성하려고 합니다. 화면부터 보시죠!

![login-screen](https://i.imgur.com/IsHBaGs.png)

이메일과 비밀번호를 입력하고 로그인하는 간단한 플로우 입니다. 여기서 확인할 항목은 아래 3가지 입니다.

1. 화면이 처음 렌더될 때 인풋2개(이메일, 비밀번호) 와 버튼1개(로그인 버튼)가 존재하는지
2. 인풋을 클릭해서 focus 됐을 때 인풋의 `color`와 `border-bottom-color`가 변하는지
3. 인풋이 입력되면 버튼의 `background-color`가 변하는지

위 3가지를 테스트하기 위해 우선 필요한 패키지들을 설치합니다

```bash
yarn add --dev jest ts-jest jest-environment-jsdom @testing-library/react @testing-library/dom @testing-library/jest-dom
```

생각보다 많습니다. 유닛테스트를 위한 [jest](https://jestjs.io/)와 리액트 테스트를 위한 [react-testing-library](https://testing-library.com/docs/react-testing-library/intro/)는 많이 들어보셨을건데, 다른 것들은 생소할 수 있어서(저는 생소했습니다) 설명하도록 하겠습니다.

[ts-jest](https://www.npmjs.com/package/ts-jest)는 jest가 타입스크립트를 사용할 수 있게 해주는 역할입니다. 타입스크립트로 작성된 코드를 자바스크립트로 변경해서 jest가 실행할 수 있도록 합니다.

[jest-environment-jsdom](https://www.npmjs.com/package/jest-environment-jsdom)은 JSDOM library를 기반으로한 jest 환경입니다. 브라우저 기반코드를 Node.js환경에서 실행하고, 테스트를 할 수 있게 하는 역할입니다. DOM, window, document등의 객체를 제공해서 jest가 테스트를 할 수 있도록 해줍니다.

jest가 위에서 언급한 두 패키지를 사용할 수 있도록, `package.json`에 아래와 같이 설정해줍니다.

```json
"jest": {
  "preset": "ts-jest",
  "testEnvironment": "jest-environment-jsdom"
}
```

그리고 `yarn run test` 또는 `npm run test` 등의 명령어로 테스트를 진행하기 위해서 `package.json`에 추가로 아래와 같이 설정해줍니다

```json
"scripts": {
  "dev": "vite",
  "build": "tsc && vite build",
  "lint": "eslint src --ext ts,tsx --report-unused-disable-directives --max-warnings 0",
  "preview": "vite preview",
  "test": "jest --watchAll"
},
```

`--watchAll`은 파일이 수정될 때 `jest` 명령어를 새로고침해서, `yarn run test`를 계속해서 입력하지 않도록 해주는 역할입니다.

이제 테스트코드를 작성해봅니다. 로그인 컴포넌트가 `Login.tsx`이니, 테스트 파일은 `Login.test.tsx`로 생성합니다.

1. 화면이 처음 렌더될 때 인풋2개(이메일, 비밀번호) 와 버튼1개(로그인 버튼)가 존재하는지

먼저 확인해보겠습니다.

```tsx
import "@testing-library/jest-dom";

import { render } from "@testing-library/react";
import { screen } from "@testing-library/dom";
import { ThemeProvider } from "styled-components";

import Login from "./Login";
import theme from "../theme";

describe("Login component", () => {
  it("renders two inputs and a button", () => {
    render(
      <ThemeProvider theme={theme}>
        <Login />
      </ThemeProvider>
    );

    const emailInput = screen.getByLabelText("이메일");
    const passwordInput = screen.getByLabelText("비밀번호");
    const loginButton = screen.getByText("로그인");

    expect(emailInput).toBeInTheDocument();
    expect(passwordInput).toBeInTheDocument();
    expect(loginButton).toBeInTheDocument();
  });
});
```

위에서부터 차례대로 보자면, `describe`와 `it`은 모두 테스트 케이스를 설명하는데, 하나의 `describe`안에 여러개의 `it`이 들어갑니다. it이 특정 테스트 케이스의 상세 설명이라면, 각각의 테스트케이스들이 모여서 로그인 컴포넌트를 묘사한다(describe)라고 이해하시면 됩니다.

styled-component를 사용중이기 때문에, `Login.tsx`에서 접근하는 `theme`을 사용하기 위해서는 테스트코드에서도 `ThemeProvider`를 사용해서 해당 컴포넌트를 감싸줘야합니다. 만약 styled-component를 사용하지 않는다면 필요없는 작업입니다. 나머지는 jest에서 변수명을 잘 지어줘서 직관적인데요, text를 사용해서 해당 jsx 를 가져오고, 각 항목들이 document내에 있는지를 테스트합니다.

여기서 주목해야할 차이점은 `getByLabelText()`와 `getByText()`입니다. `<input>`의 경우에는 `<label>`에 작성된 텍스트로 가져오기 때문에 `getByLabelText()`라는 method를 사용하고, 버튼에는 별도의 레이블이 없기 때문에 `getByText()`를 사용합니다.

이제

2. 인풋을 클릭해서 focus 됐을 때 인풋의 `color`와 `border-bottom-color`가 변하는지

확인해봅니다.

```tsx
import "@testing-library/jest-dom";

import { render } from "@testing-library/react";
import { fireEvent, screen } from "@testing-library/dom";
import { ThemeProvider } from "styled-components";

import Login from "./Login";
import theme from "../theme";

describe("Login component", () => {
  beforeEach(() => {
    render(
      <ThemeProvider theme={theme}>
        <Login />
      </ThemeProvider>
    );
  });

  it("renders two inputs and a button", () => {
    const emailInput = screen.getByLabelText("이메일");
    const passwordInput = screen.getByLabelText("비밀번호");
    const loginButton = screen.getByText("로그인");

    expect(emailInput).toBeInTheDocument();
    expect(passwordInput).toBeInTheDocument();
    expect(loginButton).toBeInTheDocument();
  });

  it("should change the color and the border color of the input when the input is focused", async () => {
    const emailInput = screen.getByLabelText("이메일");
    emailInput.style.borderBottomColor = theme.colors.mono3;
    fireEvent.click(emailInput);
    emailInput.style.borderBottomColor = theme.colors.secondary;
    emailInput.style.color = theme.colors.secondary;
  });
});
```

여기서는 `fireEvent`라는 것을 사용했는데요, 사용자의 click event를 mocking하는 개념이라고 보시면 됩니다.

이제 마지막으로

3. 인풋이 입력되면 버튼의 `background-color`가 변하는지

테스트 해보겠습니다.

```tsx
import "@testing-library/jest-dom";

import { render } from "@testing-library/react";
import { fireEvent, screen } from "@testing-library/dom";
import { ThemeProvider } from "styled-components";

import Login from "./Login";
import theme from "../theme";

describe("Login component", () => {
  beforeEach(() => {
    render(
      <ThemeProvider theme={theme}>
        <Login />
      </ThemeProvider>
    );
  });

  it("renders two inputs and a button", () => {
    const emailInput = screen.getByLabelText("이메일");
    const passwordInput = screen.getByLabelText("비밀번호");
    const loginButton = screen.getByText("로그인");

    expect(emailInput).toBeInTheDocument();
    expect(passwordInput).toBeInTheDocument();
    expect(loginButton).toBeInTheDocument();
  });

  it("should change the color and the border color of the input when the input is focused", async () => {
    const emailInput = screen.getByLabelText("이메일");

    emailInput.style.borderBottomColor = theme.colors.mono3; // mono3는 스타일 theme에 선언된 값입니다
    fireEvent.click(emailInput);
    emailInput.style.borderBottomColor = theme.colors.secondary;
    emailInput.style.color = theme.colors.secondary;
  });

  it("should change the background color of the button when email and password have inputs", () => {
    const emailInput = screen.getByLabelText("이메일");
    const passwordInput = screen.getByLabelText("비밀번호");
    const loginButton = screen.getByText("로그인");

    loginButton.style.backgroundColor = theme.colors.mono1;
    fireEvent.change(emailInput, {
      target: { value: "jasonkang14@gmail.com" },
    });
    fireEvent.change(passwordInput, { target: { value: "password" } });
    loginButton.style.backgroundColor = theme.colors.primary;
  });
});
```

`fireEvent`를 사용해서 input들에 `onChange` 이벤트를 발생시켰습니다. 인풋에 입력된 텍스트가 없는 경우에는 회색 컬러를 유지하지만, 인풋에 무언가 입력되면 버튼의 컬러가 변경됩니다.

이제 `yarn run test`를 통해 테스트 코드 동작 결과를 확인하면,

![test-success](https://i.imgur.com/KWywRwC.png)

테스트가 성공적으로 돌아가는 것을 확인할 수 있습니다.

저는 지금 글또라는 개발자 글쓰기 커뮤니티에서 활동하고 있는데, 작년에했던 글또콘에서 테스트코드 관련된 세션이 있었습니다. 발표자였던 현구님의 팀에서는 테스트코드 항목, 여기서 `it`의 설명 구분에는 한국어를 사용한다고 합니다. 한국어로 수정하면 아래와 같이 작성할 수 있습니다.

```tsx
it('이메일 인풋, 패스워드 인풋, 로그인 버튼이 화면에 그려진다', () => {
  ...
});

it('인풋이 포커스 되면 텍스트 컬러와 보더 컬러가 밝은 색으로 변한다', async () => {
  ...
});

it('인풋에 값이 입력되면 버튼이 활성화된다', () => {
  ...
});
```

![test-in-korean](https://i.imgur.com/tfcKjED.png)

테스트도 정상적으로 작동하는 것을 볼 수 있습니다. 기획에서 테스트케이스를 잘 작성해서 넘겨줬고, 개발자가 해당 테스트케이스에 대해 테스트코드를 작성한다면, 한국어로 테스트 항목을 작성하는 것이 소통에도 큰 도움이 될 것 같습니다.

다음 포스트에서는 http request에 대한 테스트코드를 작성해보도록 하겠습니다.
