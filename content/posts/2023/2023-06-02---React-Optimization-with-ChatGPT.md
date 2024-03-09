---
title: "리액트 빌드 최적화 (feat. ChatGPT)"
date: "2023-06-02T20:35:37.121Z"
template: "post"
draft: false
slug: "/react/optimzation-with-chat-gpt"
category: "React"
tags:
  - "React"

description: "ChatGPT와 함께하는 빌드 최적화"
---

회사에서 [lazy loading을 통해서 web vital을 개선](/react/lazy-loading-to-improve-web-vitals)했었는데, 조금 더 최적화를 진행해보고 싶었습니다. 최적화가 필요하겠다고 생각했던 이유는 Amplify에서 측정해준 빌드시간이 9분을 넘기는 경우가 있어서인데요. 

![amplify-long-build-time](https://i.imgur.com/k9q9kiY.png)

물론 5분대도 있고, 가끔은 3-4분대도 측정되긴 하는데, 9분 38초는 너무 길어보였습니다. 만약 hotfix가 필요한 상황에서 빌드시간이 9분이 넘게 소요된다면 사용자는 계속해서 불편함을 겪을 수밖에 없습니다. 렌더링에도 문제가 있을 것 같아서 lighthouse 점수를 확인했는데 Performance가 85점밖에 되지 않았고 

![lighthouse-score-before](https://i.imgur.com/XP6Pjjh.png)

이전에 개선했던 LCP(Largest Contentful Paint)도 3초대로 성능이 악화된 것을 확인했습니다.

![web-vitals-before](https://i.imgur.com/IOIpmfT.png)

최적화를 통해서

1. 빌드 시간을 줄이고
2. Web Vital을 개선해보기로 했습니다. 

방법들을 찾아보던 중에, 요즘 개발에서 가장 큰 도움을 받고있는 ChatGPT를 활용했습니다. ChatGPT가 제안한 첫번째 방법은 Code Splitting 입니다.

![chatgpt-code-splitting](https://i.imgur.com/Eok3Z0r.png)

Code Splitting은 Webpack과 같은 번들러가 전체 앱을 여러개의 작은 번들로 나누는 것을 의미합니다. 사용자가 페이지에 접근할 때, 모든 코드를 로드하는 것이 아니라, 필요한 부분만 로드에서 성능을 개선하는 방법입니다. Code Splitting을 적용하지 않은 시점에서는, 모든것들을 하나의 자바스크립트 번들 파일에서 불러옵니다. 따라서 해당 파일의 용량이 크고, 지금 필요하지 않은 부분들도 불러오기 때문에 성능이 저하됩니다. 

![bundle-size-before-code-splitting](https://i.imgur.com/MJqnVs6.png)

지금은 `main.3303bc83.js`라는 파일의 **크기는 230kB**이고, 해당 파일을 불러오는데 **146ms가 소요**됐습니다.

리액트에서는 [lazy](https://react.dev/reference/react/lazy)와 [Suspense](https://react.dev/reference/react/Suspense)를 사용해서 이를 구현할 수 있습니다.

```typescript
// 기존에 작성되었던 Import문을 
import ChildComponent from 'components/ChildComponent'; 

import { lazy } from 'react';
// lazy를 사용하도록 수정합니다
const ChildComponent = lazy(() => import('/components/ChildComponent')); 
```

그리고 Suspense를 사용해서 Component가 보여지는 부분을 아래처럼 감싸줍니다. 

```jsx

export default function ParentComponent () {
  return (
    <Suspense fallback={<LoadingComponent />}>
      <ChildComponent />
    </Suspense>
  )
}

```

`<ChildComponent />` 가 로딩되는 동안 fallback으로 설정된 `<LoadingComponent />`를 보여주는 방식입니다. fallback을 별도로 지정해주지 않으면 컴포넌트가 로딩되지 않아 에러가 발생하고, 새로고침을 해줘야하는 문제가 있으니 꼭 fallback을 지정해주어야 합니다. 

처음 언급한 것처럼, Code Splitting을 통해 앱을 여러개의 작은 번들로 나눌 수 있습니다. 추가로 별도의 webpack설정을 하지 않은 이유는, `create-react-app`에서 code splitting을 지원하도록 설정 되어있기 때문입니다.

그리고 한가지 작업을 추가로 진행했는데요, 

![remove-unused-code-and-libraries](https://i.imgur.com/iOinnU5.png)

사용하지 않는 코드와 라이브러리를 제거하라는 것이었습니다. Code Splitting 후 Lighthouse로 확인해보니, 점수는 개선되었지만 `Reduce unused JavaScript`를 권장하고 있습니다.

![reduce-unused-javascript](https://i.imgur.com/Uwn83o4.png)

일단 레거시 코드라서 어떤 라이브러리가 사용되지 않는지 파악이 어려운 상황이었는데, 2021년에 토스 SLASH에서 이한님이 발표하신 [JavaScript Bundle Diet](https://toss.im/slash-21/sessions/3-2)이라는 세션을 찾았습니다. 여기서 `npm dedupe`라는 명령어를 사용해서 중복된 패키지 종속성을 줄일 수 있다는 것을 배웠습니다.

명령어를 실행하면 `package-lock.json`에서 중복된 디펜던시가 삭제된 것을 확인할 수 있습니다.

![npm-dedupe-success](https://i.imgur.com/vEt90tK.png)

이해는 되지 않지만, `index.html`에서 script 태그를 사용해서 폰트를 불러오는 코드가 있었습니다. 

![unused-fonts](https://i.imgur.com/wEnDMbM.png)

그래서 해당 코드도 추가로 제거했습니다. 폰트는 `assets`등의 디렉토리에 넣어서 관리하시는 분들이 많은데요, 저는 개인적으로 폰트를 [fontsource](https://fontsource.org/docs/api/introduction)를 사용해서 npm package를 활용해서 설치하는 것을 선호합니다. 실제로 폰트를 직접 asset에 넣는것과 비교했을 때, node_modules안에 설치되는 폰트의 용량이 조금 더 작습니다. 팀마다 회사마다 방식이 다르겠지만 훨씬 관리하기 편한 것 같습니다. 

이제 작아진 번들 사이즈를 확인해보겠습니다.

![bundle-size-after](https://i.imgur.com/fFMsgVG.png)

우선 파일이 여러개의 작은 번들(`chunk.js`)로 나누어졌고, 모든 자바스크립트 파일의 용량을 더해도 1.4kB로, 기존의 230kB와 비교했을 때 약 **94%** 감소한 것을 확인할 수 있습니다. 하지만 여기서 주의할 점은 전체적인 번들 사이즈가 94%감소했다는 것이 아니라, 초기에 로딩되는 사이즈가 감소했다는 것입니다. 저는 Amplify를 사용해서 CI/CD를 구축했는데요, 첨부한 스크린샷을 보시면 전체 번들 용량을 전부 더하면 기존의 230Kb와 유사한 수치에 도달합니다.

![amplify-bundle-sizes](https://i.imgur.com/ARZIxhE.png)

해당 스크린샷을 보시면 랜덤한 문자열들을 파일 이름에서 확인할 수 있습니다. 저는 저 값들이 그냥 랜덤하게 생성되고 아무런 의미가 없다고 생각했습니다. 하지만 최근 인터뷰에서 저 값들이 무엇을 나타내는지 질문을 받았는데 답변하지 못해서 찾아보게 되었습니다. 

해당 값들은 hash 또는 fingerprint라고 입니다. 웹 브라우저는 자바스크립트 파일과 같은 static 파일들을 cache 해서 해당 사이트에 재접속하는 경우 cache 되어있는 파일을 사용해서 사이트를 더 빠르게 로딩한다는 장점이 있습니다. 

하지만 만약 사이트를 업데이트하거나, 자바스크립트 파일을 수정하고 다시 배포를 한다면, 사용자의 브라우저가 기존에 cache한 파일들로 인해 업데이트 사항이 사용자의 화면에 바로 반영되지 않는 문제가 발생할 수 있습니다. 따라서 해당 hash값들을 비교해서, 새로운 자바스크립트 파일을 다운받을지 말지 결정하는 버전관리의 역할을 한다고 보면 됩니다.

추가로 Lighthouse 점수도 확인해보았는데요, 

![lighthouse-overall](https://i.imgur.com/WKBRc9G.png)

lighthouse 점수가 100점으로 올랐고, LCP를 비롯한 다른 Web Vital도 개선된 것을 확인할 수 있었습니다.

![lighthouse-after](https://i.imgur.com/wOsrlsH.png)

특히 LCP(Largest Contentful Paint)가 3.3초에서 1.7초, FCP(First Contentful Paint)가 2.3초에서 1.2초로 모두 거의 **반으로 줄었고**, Total Blocking Time도 120ms에서 10ms로 약 **93%** 개선되었습니다. Performance는 100점이 되었지만, Accessibility에서 아직 부족한 부분이 있는데, 이는 추후에 기회가되면 마저 개선해보겠습니다.