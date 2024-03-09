---
title: Server Side Rendering은 정말 빠르게 render 되는가
date: "2022-08-05T21:41:37.121Z"
template: "post"
draft: false
slug: "/nextjs/is-server-side-rendering-really-faster"
category: "Nextjs"
tags:
  - "Nextjs"

description: "Server Side Rendering과 Client Side Rendering의 rendering 시간 비교"
---

개인 학습 목적도 있지만, 사실 이 포스트는 팀원들과 공유하기 위한 목적이 더 크다. 이정도면 회사에서 기술블로그를 운영하는 것도 나쁘지 않을 것 같은데 왠지 일이 커질 것 같아서 망설이는 중이다.  

개인적으로 새로운 툴이나 라이브러리를 도입할 때마다 [Profiler](https://reactjs.org/docs/profiler.html)로 시간을 비교하는 것을 좋아한다. 설득이 쉽기 때문이다. 단순히 "이렇게 하는게 더 좋대요~" 보다는 "이렇게 하는게 OO ms 더 빠릅니다"가 더 개발자스럽다고 해야하나. 실제로 상당히 많은 포스트들을 작성했다. 

1. [함수평 프로그래밍이 렌더링 효율에 미치는 영향](https://jasonkang14.github.io/react/functional-programming-with-react-part-three)
2. [React Query와 Context API의 성능 비교](https://jasonkang14.github.io/react/introducing-react-query)
3. [Recoil과 Context API의 성능 비교](https://jasonkang14.github.io/react/introducing-recoil)

React로 작성하던 프로젝트를 Next로 이관한 가장 큰 이유인 대시보드를 위해 사용되는 데이터를 서버사이드와 클라이언트 사이드에서 각각 호출하고 Profiler를 사용해서 렌더링 시간을 비교해보려고 한다. 

dashboard 로 routing될 때 `Layout`이라는 컴포넌트가 바뀌기 때문에, 해당 컴포넌트의 렌더링 시간을 Profiler로 측정해본다.

클라이언트 사이드 먼저 확인한다. 지금은 [react-query]를 사용하고 있다. 리액트 도입할 때 사용하기로 결정한 라이브러리라서 클라이언트 리퀘스트는 react-query를 사용하고 있다. 

```typescript 
import { useQuery } from '@tanstack/react-query';

const fetchDashboardData = async () => {
    const res = await getDashboardInfo()

    return res.data;
};

export const useDashboardQuery = () =>
    useQuery<IDashboardInfo>(['Dashboard', 'dashboardData'] () => fetchDashboardData(), {
    onError: (error) => {
        console.error('get team info error', error);
    },
});

export default function Dashboard() {
    const { data } = useDashboardQuery();
}
```
Layout이 render되는데 약 **29.7ms** 소요된다.

![react-query-layout](https://i.imgur.com/WBkutcw.png)

그리고 layout이 렌더된 이후에 데이터를 추가로 불러와야 하기 때문에, 시간이 추가로 소요된다. 

![react-query-data](https://i.imgur.com/8sKeSir.png)

두 소요 시간을 더하면 약 **67.2ms** 가 소요된다.

그리고 Next.js 공식문서에서는 클라이언트 사이드에서 서버로 데이터 요청을 할 때 [SWR](https://nextjs.org/docs/basic-features/data-fetching/client-side#client-side-data-fetching-with-swr)을 권장한다. 따라서 SWR도 사용해서 시간을 측정해보도록 한다. 

```typescript
import useSWR from 'swr';

const fetcher = async (...args: any[]) => {
  const res = await getDashboardInfo()

  return res.data;
};


```

![swr-layout](https://i.imgur.com/oI0sidh.png)

layout을 렌더할 때 약 **25.8ms** 소요된다. 그리고 dashboard가 다시 한 번 업데이트되는데, 이 부분이 데이터를 불러오는 영역이라고 판단된다. 

![swr-data-fetching](https://i.imgur.com/jIFkV6O.png)
여기서 약 **53.4ms** 소요된다. 따라서 총 **79.2ms** 소요된다. 
>> 여러번 반복해서 측정해도 디테일한 시간은 달라지지만 <br/>
react-query가 swr보다 빠른 것을 볼 수 있었다

공식적으로 추천하는 라이브러리보다 다른게 빠르다니 상당히 충격적이다.

이제 서버사이드를 확인한다.
```typescript
export async function getServerSideProps() {
    const res = await getDashboardInfo()

    return {
        props: {
            dashboardInfo: res.data
        }
    }
}

export default function DashboardPage({dashboardInfo}) {
    return (
        <Dashboard dashboardInfo={dashboardInfo} />
    )
}
```

 먼저 서버사이드 렌더링의 경우 아래와 같다 
![server-side-rendring](https://i.imgur.com/Za5Zwoz.png)

레이아웃 컴포넌트 렌더링 시간만 비교해도 **24.1ms**로 클라이언트 사이드 렌더링보다 짧게 소요된다. 사실 이건 로그인 이후에 넘어가는 과정이라 큰 의미는 없다. 실제로 layout의 렌더링시간만 비교했을 때는 클라이언트 사이드 렌더링과 크게 차이가 나지 않는다. 

> 하지만 서버사이드 렌더링의 경우 <br/> 이미 데이터를 가져온 상태로 렌더가 이루어지기 때문에 <br/> react-query나 swr의 경우와 다르게<br/> 데이터를 불러오면서 발생하는 추가 렌더링이 일어나지 않는다. 

따라서 길게는 약 **60ms**의 렌더링 시간을 절약할 수 있다. 지금 하나 걸리는 점은 로그인에서 넘어갈 때 약간의 버퍼링이 있다. 24ms도 엄청 긴 시간이기 때문이다. 다음 스프린트에서는 저 시간을 어떻게 줄일지 고민할 예정인데, 결론에 도달하면 공유해보도록 하겠다. 