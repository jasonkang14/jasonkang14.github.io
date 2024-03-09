---
title: getServerSideProps로 Next.js 효율 개선
date: "2022-09-02T21:41:37.121Z"
template: "post"
draft: false
slug: "/nextjs/check-the-performance-of-getserversideprops"
category: "Nextjs"
tags:
  - "Nextjs"

description: "getServerSideProps에서 fetch는 client side의 fetch 보다 빠르다"
---

개인 학습 목적도 있지만, 이 포스트는 팀원들에게 [getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching/get-server-side-props)를 많이 사용해야 하는 근거를 제공하기 위해 작성한다. 

서버사이드 렌더링의 장점은 렌더링 속도가 빠르다는 것이다. 이론상으로는 서버 <-> 서버 사이에 통신이 이루어지기 때문에 브라우저 <-> 서버 간의 통신보다 빠르다. 그리고 JavaScript 코드를 다운로드 하기 전에 HTML을 먼저 보여주기 때문이다. 그리고 그것이 팩트인지[React Profiler로 시간을 측정](https://jasonkang14.github.io/nextjs/is-server-side-rendering-really-faster)해서 확인했다. 

기존에 리액트로 이루어진 코드(Client Side Rendering)을 Next.js(Server Side Rendering)로 전환했기 때문에, 기존에 모든 코드의 fetch는 클라이언트 사이드에서 일어나고 있었다. 따라서 이것을 모두 `getServerSideProps`로 옮겨달라고 제안하기에는 뭔가 근거가 부족했다. 특히 서비스는 이미 전혀 무리없이 잘 굴러가고 있기 때문에. 그래서 렌더링 효율을 측정해보기로 했다. 

우선 기존 코드의 렌더링 시간이다. Profiler를 `pages` directory의 컴포넌트를 감싸도록 설정했다.

```jsx
// index.tsx

export default function Page() {
  return (
    <Profiler id="page" onRender={() => {}}>
      <Component />
    </Profiler>
  );
}
```

렌더링 시간을 확인해본다. 
![client-side-fetch](https://i.imgur.com/7dbl5rV.png)
해당 컴포넌트를 렌더링 하는데 **67.5ms** 소요됐다. 

이제 `getServerSideProps`를 사용해서 데이터를 불러오고, react-query를 통해 이 데이터를 Hydrate해주기로 한다. 위에 `index.tsx` 하단에 아래 코드를 추가한다. 

```typescript
export async function getServerSideProps({ req }: { req: NextApiRequest; res: NextApiResponse }) {
  const queryClient = new QueryClient();

  const fetchSummary = async () => {
    await Axios.get(FETCH_API_ADDRESS, {
      headers: {
        Authorization: token,
      },
    });
  };

  await queryClient.prefetchQuery(['Page', 'state'], fetchSummary);

  return { props: { token, dehydratedState: dehydrate(queryClient) } };
}

```

이제 시간을 측정한다
![server-side-fetch](https://i.imgur.com/YhvKmb3.png)

해당 컴포넌트를 렌더링 하는데 **47.1ms**소요됐다. 

단순히 수치로 계산하면 20ms, 비율로 따지면 **33%** 더 효율적이다. 이제 최초 렌더링에 필요한 데이터는 모두 `getServerSideProps`에서 fetch하기로 한다.

