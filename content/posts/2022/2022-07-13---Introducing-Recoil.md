---
title: Context API -> Recoil
date: "2022-07-13T02:21:37.121Z"
template: "post"
draft: false
slug: "/react/introducing-recoil"
category: "React"
tags:
  - "React"

description: "Context API를 들어내고 Recoil을 사용해본다"
---

# TL;DR

### Recoil의 사용으로 불필요한 rendering이 줄어들어 효율이 개선되었다.

[이전 포스트](https://jasonkang14.github.io/react/introducing-react-query)에서 `react-query` 도입의 타당성은 찾았고, 이제 Recoil을 사용하면 좋은 이유에 대해 찾아본다. 

[recoil 공식문서](https://recoiljs.org/ko/docs/introduction/motivation/)를 보면 리액트의 한계에 대해 설명하면서 왜 recoil을 개발했는지 잘 설명되어 있다. 

그럼 메타에서 어떻게 리액트스러운 상태관리 툴을 만들었는지 적용해보도록 한다. 

Recoil에는 `atom`과 `selector`가 있다. 

`atom`이 기존의 Context API reducer의 state와 유사한 개념이라고 보면 된다. 차이점은 Context API의 경우, context에 특정 값에 의존한다면, 의존하지 않는 값이 변할때도 렌더링이 일어난다. 하지만 `atom`의 경우에는 해당 값에 의존하는 경우에만 렌더링이 일어나기 때문에 훨씬 효율적이다. 

`select`는 `atom`에 있는 값을 변경할 때 사용된다. 지금 느끼기에는 component에서 할 작업들을 `select`에서 해주는 정도로만 느껴진다. 예제 코드를 보고 조금 더 잘 이해해보도록 하자. 우선 Context API를 사용해서 작성된 코드를 본다. 

```typescript
// provider.tsx

export default function Provider() {
  const handleRemoteUser = (remoteUsers) => {
    dispatch({
      type: 'SET_REMOTE_USER',
      paylaod: remoteUsers
    })
  }
}

// MediaContent.tsx
export default function MediaContent() {
  const { remoteUsers } = useCall();
  // custom hook syntax를위해 useContext(CallContext) 를 useCall로 export()한다.

  const rtcUsers = remoteUsers.filter((user) => user.uid !== 'screen');

  return (
    <>
      {rtcUsers.map((rtcUser) => <>{rtcUser}</>)}
    </>
  )
}
```

비대면 진료 프로젝트이기 때문에 화상연결이 필수인데, 새로운 사용자가 연결될 때마다 렌더링이 일어난다. 하지만 여기의 문제는 `remoteUsers`라는 값은 화상연결된 사용자를 보여주는 component만 다시 렌더하면 되는데, `CallContext`를 사용하는 다른 component들에서도 새로운 사용자가 나타날 때마다 렌더된다는 문제가 있다. 또한 화면공유를 위해 사용되는 RTCPeerConnection을 예외처리 하기 위해 `filter`라는 연산을 하게된다. 가장 확실한 것은 시간을 측정하는 것이니 Profiler를 사용해서 rendering 시간을 측정하도록 한다.

![context-api](https://i.imgur.com/8oPGB2L.png)

Context API를 사용하면 화상연결 Component를 렌더링하는데 약 **11.3ms**가 소요된다. 스크린샷에 나오지는 않았지만 Context가 업데이트 되면서, Context.Provider에서도 추가로 **22.7ms**가 소요되었다. 사용자가 한 명 입장하고, 그 영상을 렌더링하는데 **34ms**가 소요된다. remote user의 video component만 렌더되는데는 **5.3ms** 소요됐다

이제 recoil의 `atom`을 사용해서 `remoteUsers`라는 값을 관리해보도록 한다. 기존 provider.tsx에서 변수를 reducer를 사용하지 않고 `atom`을 사용해서 업데이트한다.

```typescript
// provider.tsx
export const remoteUserState = atom({
  key: 'remoteUserState',
  default: null,
});

export default function Provider() {
  const setRemoteUsers = useSetRecoilState(remoteUserState);

  const handleRemoteUser = (remoteUsers) => {
    setRemoteUsers(remoteUsers);
  }
}

// MediaContent.tsx
export default function MediaContent() {
  const [remoteUsers, setRemoteUsers] = useRecoilState(remoteUserState);

  const rtcUsers = remoteUsers.filter((user) => user.uid !== 'screen');

  return (
    <>
      {rtcUsers.map((rtcUser) => <>{rtcUser}</>)}
    </>
  )
}
```

![recoil](https://i.imgur.com/FXPKInC.png)

사용자 한 명이 늘어나고, 그 영상을 렌더링하는데 **6.8ms** 소요된다. remote user의 video component만 렌더되는데는 **4.7ms** 소요됐다. 스크린샷을 비교하면 rendering이 일어난 최상단의 component가 다른 것을 확인할 수 있다. Recoil에서 제공하는 `atom`을 사용하면 MediaContent.tsx에서만 렌더링이 일어나지만, Context API를 사용하면 MediaContent.tsx 컴포넌트의 부모 component와 해당 Context의 값에 의존하는 다른 component들에서 렌더링이 일어나기 때문이다. 

단순히 시간만 측정한다고 해도, Context API를 들어내면 렌더링 시간이 **27.2ms** 감소하고, 비율로 측정하면 약 **80%** 개선된다. 앞으로 Context API는 사용하지 말아야겠다. 