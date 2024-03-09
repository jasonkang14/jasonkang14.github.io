---
title: React 효율 개선을 위한 Fiber Reconciler
date: "2023-03-12T20:35:37.121Z"
template: "post"
draft: false
slug: "/react/what-is-fiber-architecture"
category: "React"
tags:
  - "React"

description: "Fiber Reconciler는 React 렌더링을 어떻게 개선했는가"
---

최근 리액트 렌더링 효율에 관심을 갖게되면서, 소스코드를 조금씩 파악하고 있다. 소스코드를 보다보니  `FiberNode`, `FiberRootNode`에 대해 먼저 이해할 필요성을 느꼈다. rendering, re-rendering이라는 용어를 업데이트 라는 단어와 혼용해서 사용할 예정이니, 부디 혼돈이 없기를

`Fiber`라는 개념은 React 16에서 처음 등장했다. 리액트에서 상태가 업데이트 되면 `Fiber Reconciler`를 사용해서 변화를 탐지하고, 화면을 렌더링한다. 코딩은 단어 뜻이 중요하니 하나씩 나눠서 살펴보자면

1. Fiber는 근육의 섬유(인체 도감? 같은거 보면 보이는 얇은 줄들)를 나타내고
2. Reconciler는 fiber들이 어떤 순서로 렌더링 될지를 중재하는 역할이다.

중재라는 개념이 나중에 중요하다. 

|![muscle-fiber](https://www.ideafit.com/wp-content/uploads/2022/04/Muscle-Fiber-Types-903x602.jpg)|
|:--:|
|React fiber는 브라우저 화면을 움직인다|

중재자라고 설명한 Reconciler의 [Reconciliation](https://reactjs.org/docs/reconciliation.html)은 React에서 중요한 개념인데, DOMTree에서 변경된 부분만 새롭게 렌더링 하기 위해 사용되는 기법이다. React뿐 아니라 React Native에서도 효율적인 업데이트(re-rendering)를 위해 사용된다고 한다. 

`Fiber Reconciler`가 왜 효율적인지 알아보기 전에, 기존의 `Stack Reconciler`를 이해해야한다. 개발을 시작한게 React 16출시 이후라서, 개인적으로 그 이전의 Reconciler의 한계를 경험해보지는 못했다. 소스코드를 찾기 어려워서 여기저기 검색한 결과에 따르면, 

`Stack Reconciler`는 아래의 과정을 통해 동기적으로 리렌더링을 처리했다. 

1. DOMTree의 top부터 시작해서 재귀 형식으로 **모든 component에 `.render()`를 호출** 하고
2. VirtualDOM에 변경된 사항을 확인하고, 
3. 업데이트가 필요한 component를 파악하고
4. 해당 component와 모든 자식 component들도 업데이트가 필요한지 파악하고
5. component 하나씩 업데이트를 시작한다. 

아래 코드를 업데이트한다고 생각해보자. 

```javascript
function Test() {
  return (
    <Parent>
      <ChildA />
      <ChildB />
      <ChildC />
    </Parent>
  );
}
```

그림으로 보자면 이런 느낌이다 

|![stack-reconciler-diagram](https://i.imgur.com/z6YEMYG.png)|
|:--:|
|**Stack이라는 이름답게 LIFO (Last-In-First-Out) 식으로 업데이트 된다**|

**JavaScript Call Stack**과 비슷한 원리이다. 동기적으로 화면을 업데이트한다. 

합리적으로 보이지만 

- 많은 component들을 동시에 업데이트 하거나, 
- 특정 업데이트가 오래 걸리는 연산을 포함하는 경우 

동기적으로 하나씩 업데이트를 처리한다는 특성상 사용자 입장에서 화면이 멈춘것 처럼 보일 수 있다. 

`Stack Reconciler`는 동기적으로(synchronous) 컴포넌트 하나씩 화면을 업데이트(re-render)하는데 문제가 있었으니, `Fiber Reconciler`는 [비동기 렌더링](https://reactjs.org/blog/2017/09/26/react-v16.0.html#new-core-architecture)(asynchronous)을 지원한다. 

Team React 소속인 Andrew Clark에 따르면 [react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)의 목표는 `incremental rendering` 기법을 사용해서 `Fiber Reconciler`가 지정한 우선순위 대로 렌더링을 일어나게 하는 것이다. 우선순위대로, 비동기적으로. 

state 변경으로 업데이트(re-rendering)가 필요한 상황이되면, 어떻게 업데이트할지 결정해야하는데 Andrew Clark은 하나의 업데이트를 `work`라고 부른다. 그리고 어떤 `work`를 실행해야할지 결정하는 과정을 `scheduling`이라고 표현한다.

`Stack Reconciler`는 업데이트할 component를 찾으면 stack에 넣고 하나씩 업데이트를 했다면, `Fiber Reconciler`는 업데이트할 component들의 우선순위를 파악해서 점수를 준다. 우선순위는 [사전에 작성해둔 카테고리](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberLane.js)에 따라 부여된다. 그러기 위해서는 4가지 일을 할 수 있어야 한다. 

1. `work`를 중지하고, 필요 시 다시 시작할 수 있어야 한다. 
2. 다른 종류의 `work`들에게 우선순위를 부여할 수 있어야 한다. 
3. 이미 완료된 `work`를 재사용 할 수 있어야 한다. 
4. `work`가 더이상 필요 없게 되면 버릴 수 있어야 한다. 

예를들면 우선순위 3점짜리 `work`를 하고있는데 갑자기 20점짜리 `work`가 들어오면, 3점짜리를 중지하고, 20점짜리를 실행하고, 다시 3점짜리를 시작한다. 이를 위해 `work`를 조금 더 세밀하게 쪼갤 필요를 느꼈고, 이 쪼개짐(?)이 `fiber`이다. fiber들의 변동사항들이 모여서 work를 만드는 것이다. 

`Fiber`는 component와 component의 input / output정보를 갖고있는 JavaScript 객체이다. [react-reconciler의 소스코드를 보면](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiber.js#L134) 실제로 자바스크립트 객체로 구현되어있다. 이 객체는, 담당하는(?) React Element의 위치를 DOM Tree내에서 표현한다. 아래와 같은 React Component가 있다고 하자

```jsx
<ComponentA>
  <ComponentB>
    <ComponentC>
      <ComponentD />
    </ComponentC>
  </ComponentB>
  <ComponentE>
    <ComponentF />
    <ComponentG />
  </ComponentE>
</ComponentA>
```

위에 나타난 React Component를 Fiber를 활용한 DOM Tree로 변경하면 아래와 같은 그림이 된다.

![dom-tree-with-fiber-node](https://i.imgur.com/7TxWF9y.png)

`Fiber Reconciler`는 queue를 사용해서 화면을 업데이트한다. update queue의 소스코드는 [여기서](https://github.com/facebook/react/blob/main/packages/react-reconciler/src/ReactFiberClassUpdateQueue.js)확인할 수 있다. 

queue는 두가지 종류가 있는데, 하나는 `current queue`, 다른 하나는 `work-in-progress queue` 이다. `current queue`는 이름에서 유추할 수 있듯 현재 화면에 보이는 상태를 나타내고, `work-in-progress queue`는 화면을 업데이트하기 위해 fiber node를 처리하는 queue이다. `Fiber`가 JavaScript 객체이기 때문에, `work-in-progress queue`의 fiber들은 commit되기 전까지 비동기적으로 자유롭게 mutate가 가능하다. 그림으로 보자면 이런 느낌이다. 

![two-fiber-queues](https://i.imgur.com/dpEIsho.png)

초록색으로 표시된 node들이 업데이트가 필요한 React Component이다. `Fiber Reconciler`가 업데이트를 감지하면, 변동사항이 바로 화면에 적용되는 것이 아니다. 현재 화면을 나타내는 `cureent queue`를 복제해서 `work-in-progress queue`를 생성하고, 해당 `work-in-progress queue`에 변동사항을 반영한다. `work-in-progress queue`의 node들의 mutation이 완료되면, `commit`되어 해당 `work-in-progress queue`가 새로운 `current queue`가 된다. 

다음 포스트에서는 소스코드를 조금 더 파고들어서 

1. `App.tsx`에서 `.createRoot()`을 호출하고 `.render()`를 호출하면 어떤 일이 일어나는지 
2. `setState`가 호출되면 `work-in-progress queue`가 어떻게 동작하는지에 대해 알아보도록 하겠다. 