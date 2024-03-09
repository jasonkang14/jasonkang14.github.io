---
title: "Lazy Loading을 활용한 Web Vital개선"
date: "2023-04-09T20:35:37.121Z"
template: "post"
draft: false
slug: "/react/lazy-loading-to-improve-web-vitals"
category: "React"
tags:
  - "React"

description: "사용자 경험 개선을 위한 프론트엔드 개발자의 노력"
---

2월에 새로운 회사로 이직했다. 입사하고 파악한 시스템은 아래 순서로 이루어졌다.

1. admin용 웹페이지에서 처리한 데이터를
2. 서버로 전송하고
3. 모바일 앱에서 서버에 있는 데이터를 불러온다.

처음에는 분명히 Flutter로 모바일 앱 개발을 담당하기로 했는데, 입사 전에 누군가 작성해둔 admin용 웹페이지가 너무 느리다는 피드백을 갑자기 나에게(?) 하기 시작했다. 해당 페이지를 작업한 개발자는 미국에 있어서 시차로인해 실시간 소통이 어렵다는 기획자 분들의 의견에 따라 일단 한 번 들여다보기로 했다. 

웹사이트가 느렸던 가장 큰 원인은 약 200개의 이미지와 영상을 동시에 S3 bucket으로 요청해서 가져오고 있었기 때문이다. 이미지 200개, 영상 200개 총 400개정도의 request를 페이지가 렌더되자마자 하는 것이다. 페이지네이션도 없고 그냥 일단 다 불러온다. 얼마나 느린가 궁금해서 Performance tab에서 측정해봤다. 정확하진 않겠지만 **9초 이상** 소요되는 Long Task가 있었다. 

![long-task](https://i.imgur.com/ZhsXuqV.png)

보안상 Performance Tab에서 보여주는 스크린샷을 올릴 수는 없는데, 이 long task가 돌아가는 와중에도, 화면에는 아직 이미지와 영상이 나오지 않았다. Request는 보냈는데, 아직 불러오지도 못한 상황. 그리고 뒤에 Image Decoding이라는게 찍히는데, 이 부분이 약 4000ms, **4초가량** 소요됐다. 대충 계산해도 사용자가 약 **13초** 가량을 빈 화면을 보고있는 것이다. 

Lighthouse를 사용해서 퍼포먼스를 측정했더니 [LCP(Largest Contentful Paint)](https://web.dev/lcp/)가 2.7초로 측정됐다 

![lighthouse-before](https://i.imgur.com/56Kx51F.png)

GOOD은 2.5초이고, 4.0초가 넘어가면 NEEDS IMPROVEMENT라고 한다. Performance Tab에서 약 13초였는데, Lighthouse는 2.7초라고 하니 정확성에 의심이 되긴 하지만... 일단 개선이 필요한 것은 확실한 상황이다. 

처음에는 pagination만 생각하고 서버코드를 수정하려고 했다. 서버는 AWS Serverless로 배포되어있었다. 지금은 익숙해져서 코드도 수정하고 배포도 하지만, 당시에는 처음 접한 툴을 괜히 수정했다가 에러가 나면 안된다는 걱정에 프론트에서 자체적으로 해결하기로 했다. 그래서 이미지와 영상의 S3주소를 400개를 불러오는 건 어쩔 수 없고, 화면에 보이는 부분들만 S3 bucket으로 요청을 보내는 식으로 수정하기로 했다. 이를 위해 요즘 개발할 때 가장 좋은 친구(?)인 [ChatGPT](https://openai.com/blog/chatgpt)의 도움을 받았다.

![chat-gpt-custom-hook](https://i.imgur.com/KsiK9ok.png)

ChatGPT가 [Intersection Observer API](https://developer.mozilla.org/en-US/docs/Web/API/Intersection_Observer_API)를 사용해서 제공한 custom hook을 사용해서 이미지 편집기와 영상 편집기가 화면에 보이면 S3 bucket으로 요청을 보내도록 수정했다. 

```typescript
import { useState, useEffect, RefObject } from 'react';

function useVisibility<T extends HTMLElement>(ref: RefObject<T>): boolean {
  const [isVisible, setIsVisible] = useState<boolean>(false);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        setIsVisible(entry.isIntersecting);
      },
      {
        root: null,
        rootMargin: '0px',
        threshold: 0.1, // Adjust this value to control when the component should be considered visible.
      }
    );

    if (ref.current) {
      observer.observe(ref.current);
    }

    return () => {
      if (ref.current) {
        observer.unobserve(ref.current);
      }
    };
  }, [ref]);

  return isVisible;
}

export default useVisibility;
```

처음에는 자바스크립트 예제를 줘서 타입스크립트 예제로 바꿔달라고 요청했다. 

lazy loading 후 Lighthouse와 Performacne tab을 사용해서 performance를 확인했다. 
일단 가장 긴 long task의 길이가 9000ms에서 117.66ms로 약 **98.7%** 감소했다.

![performance-tab-after](https://i.imgur.com/KHj8Rpl.png)

어떤 task가 특히 오래걸린 [long task](https://web.dev/optimize-long-tasks/?utm_source=devtools)인지 지적을 해주는데, 표현되는 부분이 없었다. rendering은 꽤나 개선이 됐으니, 자바스크립트 리팩토링을 통해 scripting을 줄일 수 있는 방법을 고민해봐야겠다. 

이번에는 Lighthouse 측정 결과이다.

![lighthouse-after](https://i.imgur.com/JEVeY3m.png)

주황색이 살짝 아쉽긴하지만, 그래도 2.5초 미만이면 GOOD이라고 하니, 상당히 개선됐다. 급한불은 껐지만 추가로 개선할 부분들이 있으니, 더 개선되면 추가로 포스팅 해보도록 하겠다. 