---
title: Azure에서 React.js배포
date: "2022-12-16T19:35:37.121Z"
template: "post"
draft: false
slug: "/azure/static-web-app-to-deploy-react"
category: "Azure"
tags:
  - "Azure"
  - "React"

description: "Azure Static Web App에서 React.js 프로젝트를 배포하는 방법"
---

리액트 프로젝트를 배포하는 방법은 다양하다. 서버에 빌드 시 생성된 디렉토리를 scp로 업로드해서 경로만 지정해줘도 되는데, 이런 경우에는

1. 서버를 직접 관리해줘야하는 번거로움이 있고,
2. 발급받은 도메인에 SSL인증서를 적용하고 주기적으로 갱신해줘야한다

따라서 [React 공식 홈페이지에서 제안하는 다양한 방법들](https://create-react-app.dev/docs/deployment/)중에 회사에서 사용하는 Azure에서 배포를 해보기로 한다.

영어로는 Static Web App인데 우리말로는 [정적웹앱](https://learn.microsoft.com/ko-kr/azure/static-web-apps/?WT.mc_id=build2020_swa-docs-cxa)이다.

![azure-static-web-app](https://i.imgur.com/v5lAsOV.png)

클릭한다

그리고 resource group을 선택하고, 이름을 부여하고, Github인증을 통해 repo와 브랜치를 선택하면 Azure Static Web App과 Github repo를 연동할 수 있다. 아래 스크린샷을 보면 심지어 무료로 호스팅이 가능하고, 3개까지 staging environment를 제공한다. 매우 좋다.

![github-auth](https://i.imgur.com/mh1iqos.png)

그리고 React를 선택하면, 타겟 빌드 디렉토리를 선택하고. 다음을 누르면 배포까지 바로 이루어진다.

![choose-react](https://i.imgur.com/Hj7ie0V.png)

Azure가 기본으로 도메인을 제공하고 SSL 인증서도 적용해준다. 매우 편하다. Static Web App과 연동이 끝나면 Azure가 자동으로 `.github/workflows` 디렉토리에 `yml`파일을 생성해준다.

환경변수를 추가한다면 `jobs` 섹션 위에 Github Secret을 사용해서 아래처럼 추가하면 된다

```yml
env:
  REACT_APP_API_ADDRESS: ${{ secrets.DEV_API_ADDRESS }}
```

그리고 기본적으로 정적이기 때문에, 최초 진입시에는 React.js에서 설정한 route들을 잘 찾아가지만, 새로고침하는 경우 404에러가 발생한다.
따라서 Azure에서 권장하는대로 `routes.json` 파일을 프로젝트 `public` 디렉토리에 생성하고, 아래와 같은 내용을 추가해줘야한다.

```json
{
  "routes": [
    {
      "route": "/*",
      "serve": "/index.html",
      "statusCode": 200
    }
  ]
}
```

경로에 상관없이 모두 `index.html`을 가리키라는 뜻이다. 해당 index.html에 React의 SPA가 구축되기 때문에 새로고침 하더라도 경로를 잘 찾아갈 수 있다.
