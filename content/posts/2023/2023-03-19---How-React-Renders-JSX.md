---
title: "createRoot()에 render()를 호출하면 어떤 일이 일어날까?"
date: "2023-03-19T20:35:37.121Z"
template: "post"
draft: false
slug: "/react/how-react-renders-jsx"
category: "React"
tags:
  - "React"

description: "React 소스코드 분석을 통해 알아보는 JSX 렌더링"
---

요즘 힙한(?) [vite](https://vitejs.dev/)로 React 프로젝트를 시작하면 `main.tsx`에 아래와 같은 코드가 나타난다. (사실 create-react-app으로 해도 마찬가지)

```typescript
// main.tsx

import {StrictMode} from 'react'
import ReactDOM from 'react-dom/client'
import App from './App'
import './index.css'

ReactDOM.createRoot(document.getElementById('root') as HTMLElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

`index.html`에 아래 코드가 들어있기 때문에, `<div id="root">`에 접근할 수 있다. 

```html
<!-- index.html -->

<script type="module" src="/src/main.tsx"></script>
```

오픈소스라 그런지 변수명을 아주 잘 지었기 때문에, `main.tsx`에서 이루고자 하는 것은

1. `ReactDOM`에 `<div id="root">`를 사용해서 `root`를 만들고
2. 그 root에 `<App />`을 파라미터로 해서 `.render()`라는 함수를 호출한다.

`ReactDOM`은 `react-dom/client`에서 default로 export된 값이기 때문에, `main.tsx`는 아래처럼 간소화(?) 될 수 있다. 

```typescript
// main.tsx

import {StrictMode} from 'react'
import {createRoot} from 'react-dom/client'
import App from './App'
import './index.css'

createRoot(document.getElementById('root') as HTMLElement).render(
  <StrictMode>
    <App />
  </StrictMode>,
)
```

`react-dom` 패키지의 소스코드를 분석해서, 이것이 어떻게 이루어지는지 알아보려고 한다. 그럼 `createRoot`부터 확인한다.

```javascript
// react-dom.development.js

function createRoot(container, options) {
  if (!isValidContainer(container)) {
    throw new Error('createRoot(...): Target container is not a DOM element.');
  }

  warnIfReactDOMContainerInDEV(container);
  ...
}
```

`isValidContainer()`를 호출해서, `container`가 `HTMLElement`인지 확인한다. 
`warnIfReactDOMContainerInDEV()`는 container를 추가로 검증하는 함수이다. 

```javascript
function warnIfReactDOMContainerInDEV(container) {
  {
    if (container.nodeType === ELEMENT_NODE && container.tagName && container.tagName.toUpperCase() === 'BODY') {
      error('createRoot(): Creating roots directly with document.body is ' + 'discouraged, since its children are often manipulated by third-party ' + 'scripts and browser extensions. This may lead to subtle ' + 'reconciliation issues. Try using a container element created ' + 'for your app.');
    }

    if (isContainerMarkedAsRoot(container)) {
      if (container._reactRootContainer) {
        error('You are calling ReactDOMClient.createRoot() on a container that was previously ' + 'passed to ReactDOM.render(). This is not supported.');
      } else {
        error('You are calling ReactDOMClient.createRoot() on a container that ' + 'has already been passed to createRoot() before. Instead, call ' + 'root.render() on the existing root instead if you want to update it.');
      }
    }
  }
}
```

첫번째 if문은 `<body>`를 container로 사용할 수 없다는 것이고, 아래 if문은 container가 이미 root로 등록된 tag의 child component인지를 확인하는 함수이다. 즉 또다른 tag를 `index.html`에 추가하지 않는다면 앞에서 언급한 `<div id="root">`만 root가 될 수 있다. 

`createRoot()`를 마저 살펴보자

```javascript
// react-dom.development.js

function createRoot(container, options) {
  ...
  var root = createContainer(container, ConcurrentRoot, null, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError);
  ...
}
```

container는 `<div id="root">`이고, options는 말 그대로 필수항목은 아닌데 `identifierPrefix`와 `onRecoverableError`를 받을 수 있다. 
`identifierPrefix`는 [useId](https://beta.reactjs.org/reference/react/useId)로 id를 생성할 때 앞에 prefix로 붙일 string을 지정하는 역할이다. 예를들면 그냥 `useId()`를 호출할 경우 `r0`이라는 값을 리턴하는데, `jason`과 같은 prefix를 넣어주면 `jasonr0`을 리턴한다. `onRecoverableError`는 렌더링하다가 에러가 발생하는 경우 호출하는 함수이다. console.error()가 디폴트인데, 만약 다르게 뭔가를 하고싶다면 함수를 보낼 수 있다. 하지만 굳이???

계속 보자면, `createContainer()`는 내부적으로 `createRiberRoot()`를 호출해서 `FiberRootNode`를 return한다.

Fiber라는 개념에 대해 생소할 수 있는데, Fiber에 관해서는 [이 포스트](https://jasonkang14.github.io/react/what-is-fiber-architecture)에서 조금 자세하게 다뤘다. 간단하게 설명하면 Fiber는 VirtualDOM Tree에서 각 React Element를 나타내는 node이다. 

`FiberRootNode`는 Fiber들의 root라고 보면 된다. `<div id="root">`가 VirtualDom Tree에서는 `FiberRootNode`이다. 그림으로 보자면 이런 느낌이다. 

![virtual-dom-tree](https://i.imgur.com/8vWRrP6.png)

이제 `FiberRootNode`를 살펴본다. 

```javascript
function FiberRootNode(containerInfo, tag, hydrate, identifierPrefix, onRecoverableError) {
  // attribute들 중에 `null`인 것들과 상수인 것들은 일단 생략했다. 
  this.tag = tag;
  this.containerInfo = containerInfo;
  this.eventTimes = createLaneMap(NoLanes);
  this.expirationTimes = createLaneMap(NoTimestamp);
  this.entanglements = createLaneMap(NoLanes);
  this.identifierPrefix = identifierPrefix;
  this.onRecoverableError = onRecoverableError;

  {
    this.memoizedUpdaters = new Set();
    var pendingUpdatersLaneMap = this.pendingUpdatersLaneMap = [];

    for (var _i = 0; _i < TotalLanes; _i++) {
      pendingUpdatersLaneMap.push(new Set());
    }
  }

  {
    switch (tag) {
      case ConcurrentRoot:
        this._debugRootType = hydrate ? 'hydrateRoot()' : 'createRoot()';
        break;

      case LegacyRoot:
        this._debugRootType = hydrate ? 'hydrate()' : 'render()';
        break;
    }
  }
}
```

모든 파라미터가 `createContainer()`에서 오는데, `hydrate`는 false라서 무시한다. `hydrate`는 Server-Side Rendering에서 사용되는데, 그건 기회가 되면 나중에 훑어보도록 해야겠다. 그럼 우리에게 남은 것은

1. containerInfo
2. tag
3. identifierPrefix
4. onRecoverableError

하나씩 확인해본다. `createContainer()`를 호출하는 곳과, `createContainer()`를 비교해보면

```javascript
var root = createContainer(container, ConcurrentRoot, null, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError);

function createContainer(containerInfo, tag, hydrationCallbacks, isStrictMode, concurrentUpdatesByDefaultOverride, identifierPrefix, onRecoverableError, transitionCallbacks)
```

1. `containerInfo === container`이다. 이 경우에는 `<div id="root">`이다. 
2. `tag`는 `ConcurrentRoot`인데 파일에 1로 선언된 상수이다. 리액트는 다양한 값들을 상수로 활용하는데, 아마 계속 찾아보면 보이지 않을까 싶다.
3. `identifierPrefix`와 `onRecoverableError`위에서 언급한 것처럼 `createRoot()`을 호출할 때 사용되는 optional parameter이다. 

`Concurrent`라는 개념에 대해 조금 짚고 넘어가자면, React 18에서 추가된 [렌더링 효율 개선 방안](https://react.dev/blog/2022/03/29/react-v18#what-is-concurrent-react)이다. 요약하면 컴포넌트의 비동기적 렌더링과 병렬로 처리되는 업데이트를 가능하게 하여, 렌더링 성능과 사용자 경험을 개선하는 데 큰 역할을 한다. React16에 추가된 Fiber의 활용을 극대화한 것이라고 볼 수 있다. 

`createLaneMap()`함수를 짚고 넘어가자면, V8엔진에서 메모리를 최적화 하기위한 방법이다. `createLaneMap()`은 숫자로 이루어진 array를 리턴하는데, JavaScript array에 값을 `push()`해서 array를 생성하는 함수이다. 

array는 `HOLEY_ELEMENTS`와 `PACKED_ELEMENTS`로 구분된다. 
array를 선언할 때, 아래처럼 비어있는 array를 먼저 선언하고 나중에 채우면 `HOLEY_ELEMENTS`가 된다고 한다.

```javascript
const array = new Array(3);
array[0] = 'a';
array[1] = 'b';
array[2] = 'c';
```

이와 다르게 처음부터 가득차게 채우면 `PACKED_ELEMENTS`가 된다.
```javascript
const array = ['a', 'b', 'c'];
```

`HOLEY_ELEMENTS`의 경우에는 garbage collected되더라도 메모리에 빈 공간을 남겨둬서 memory fragmentation을 야기할 수 있다. 만약에 값이 추가될 예정인지 아닌지 알 수 없다면, `new Array(3)`으로 선언하지 말고, empty array를 선언한 후에 `.push()`를 호출해야한다. 메모리를 효율적으로 사용하기 위한 리액트의 노력이라고 봐주면 되겠다. 

뒤에서 다시 언급하겠지만 `lane`은 업데이트 종류에 따라 배정된 상수이다. `laneMap`은 array인데, `TotalLanes`의 길이만큼 array를 생성해서 리턴한다. 그리고 특정 업데이트가 발생하면, 해당 업데이트의 종류를 `lane`으로 확인한 후에, `laneMap`에 해당 업데이트가 요청된 시간을 할당한다. 

다시 `FiberRootNode`를 생성하는 `createFiberRoot()`로 돌아간다. `createFiberRoot()`는 `FiberRootNode`에 값들을 할당한다.

```javascript
// react-dom.development.js

function createFiberRoot(MANY_PARAMETERS) {
  var root = new FiberRootNode(containerInfo, tag, hydrate, identifierPrefix, onRecoverableError);

  var uninitializedFiber = createHostRootFiber(tag, isStrictMode);
  root.current = uninitializedFiber;
  uninitializedFiber.stateNode = root;

  {
    var _initialState = {
      element: initialChildren // initialChildren == null,
      isDehydrated: hydrate, // hydrate == false
      cache: null,
      transitions: null,
      pendingSuspenseBoundaries: null
    };
    uninitializedFiber.memoizedState = _initialState;
  }

  initializeUpdateQueue(uninitializedFiber);
  return root;
}
```

`FiberRootNode`에 파라미터로 넘겨주는 `containerInfo`는 `<div id="root">`인데, `root.containerInfo`에 할당된다. 이건 뒤에 나오니까 기억해두도록 한다.

그리고 `root.current`는 기존에 `null`이었는데 `createHostRootFiber()`가 return하는 `HostRoot`가 할당된다. React Component가 parent-child관계를 갖는 것처럼 `current`에 할당해서 `HostRoot`를 `RootFiberNode`의 자식 node로 관리한다는 개념으로 이해하면 된다. `HostRoot`는 그냥 `FiberNode`중에 하나다. React에는 FiberNode의 종류들도 [숫자로 관리한다](https://github.com/facebook/react/blob/9e3b772b8cabbd8cadc7522ebe3dde3279e79d9e/packages/react-reconciler/src/ReactWorkTags.js). 코드를 보면 `HostRoot`은 3에 해당하고, `createHostRootFiber`는 `createFiber()`라는 함수에 해당 숫자를 넘겨줘서 `HostRoot`를 생성한다. 

위에서 나타낸 VirtualDOM Tree는 사실상 이런모양이다. 

![virtual-dom-tree](https://i.imgur.com/eKb5taU.png)

이제 `initializeUpdateQueue()`를 호출해서, `HostRoot`에 `updateQueue`를 할당한다. React에서 Fiber개념을 도입하면서, Re-rendering에 업데이트라는 단어를 혼용해서 한다. `updateQueue` state가 변경되면서 re-rendering이 일어날때, 화면 변화를 일으키는 queue라고 생각하면 된다. 

`updateQueue`는 아래와 같은 구조이다 

```javascript
var queue = {
  baseState: fiber.memoizedState,
  firstBaseUpdate: null,
  lastBaseUpdate: null,
  shared: {
    pending: null,
    interleaved: null,
    lanes: NoLanes
  },
  effects: null
};
```

`state`가 변경되면서 rendering이 어떻게 되는지에 대해도 포스트 하려고 하는데, `updateQueue`는 다음 포스트에서 다루겠다. 다시 `createRoot()`로 돌아가면,

```javascript
// react-dom.development.js

function createRoot(container, options) {
  ...

  markContainerAsRoot(root.current, container);
  var rootContainerElement = container.nodeType === COMMENT_NODE ? container.parentNode : container;
  listenToAllSupportedEvents(rootContainerElement);
  return new ReactDOMRoot(root);
}
```

`markContainerAsRoot()`를 같이 보자. 

```javascript
function markContainerAsRoot(hostRoot, node) {
  node[internalContainerInstanceKey] = hostRoot;
}
```

`markContainerAsRoot()`이 파라미터로 넘겨받은 `hostRoot`는 `HostRoot`이고, `node`는 `<div id="root">`이다. 그리고 `internalContainerInstanceKey`는 `container`를 나타내는 string이다. 

```javascript
var randomKey = Math.random().toString(36).slice(2);
var internalContainerInstanceKey = '__reactContainer$' + randomKey;
```

`<div id="root">`와 `HostRoot` 연결시키는 것이라고 볼 수 있다. 리액트에서 업데이트가 발생하면, 자식들부터 업데이트를 시작해서 부모 node들로 타고 올라오는데, `<div id="root">`와 연결된 `HostRoot`를 만나는 순간 이제 그만 확인하고 업데이트를 멈추라는 것을 알릴 수 있다. 

![div-root-with-container-instance-key](https://i.imgur.com/viuBWKI.png)

그리고 이제 `<div id="root">`에 모든 이벤트리스너를 붙인다. 이건 React 17에서 업데이트된 사항인데, 기존에는 모든 이벤트를 `body`에 붙였는데, React 17부터 `<div id="root">`에 붙인다. 

![root-node-listens-to-all-the-events](https://i.imgur.com/kR5Ic4j.png)

그렇게 이벤트리스너를 모두 `<div id="root">`에 붙이면, `ReactDOMRoot`를 리턴한다. 

```javascript
// react-dom.development.js

function ReactDOMRoot(internalRoot) {
  this._internalRoot = internalRoot;
}
```

`FiberRootNode`를 `this._internalRoot`에 할당하는 것으로 `createRoot()`는 마무리된다. 요악하면 아래와 같은 과정을 거친다. 

1. `RootFiberNode`생성
2. `RootFiberNode`의 직계 자식 node로 `HostRoot`생성
3. `HostRoot`를 `<div id="root">`와 연결
4. `<div id="root">`에 리액트에서 발생하는 모든 업데이트 할당 

이렇게 해서 `root`가 생성된다. 

해당 `ReactDOMRoot`가 `render()` method를 호출한다. 이제 `render()`를 하나씩 쪼개보자.

```javascript
// react-dom.development.js

ReactDOMRoot.prototype.render = function (children) {
  var root = this._internalRoot; 

  // root === FiberRootNode이기 때문에 에러는 발생하지 않는다. 
  if (root === null) {
    throw new Error('Cannot update an unmounted root.');
  }

  // `render()`를 호출할 때 arguments가 하나밖에 없기 때문에 `arguments[1] === undefined`라서 bypass한다.
  // 사실상 없어도 되는게 아닌가 생각한다
  {
    if (typeof arguments[1] === 'function') {
      error('render(...): does not support the second callback argument. ' + 'To execute a side effect after rendering, declare it in a component body with useEffect().');
    } else if (isValidContainer(arguments[1])) {
      error('You passed a container to the second argument of root.render(...). ' + "You don't need to pass it again since you already passed it to create the root.");
    } else if (typeof arguments[1] !== 'undefined') {
      error('You passed a second argument to root.render(...) but it only accepts ' + 'one argument.');
    }
  }
  var container = root.containerInfo;

  if (container.nodeType !== COMMENT_NODE) {
    var hostInstance = findHostInstanceWithNoPortals(root.current);

    if (hostInstance) {
      if (hostInstance.parentNode !== container) {
        error('render(...): It looks like the React-rendered content of the ' + 'root container was removed without using React. This is not ' + 'supported and will cause errors. Instead, call ' + "root.unmount() to empty a root's container.");
      }
    }
  }
  
  updateContainer(children, root, null, null);
};
```

`container`에 할당되는 `root.containerInfo`는 `<div id="root">`이다. `container.nodeType`은 `COMMENT_NODE`가 아니기 때문에 if문 안으로 들어가고, `root.current`는 `RootFiberNode`의 child node에 해당하는 `HostRoot`이다. 

`findHostInstanceWithNoPortals()`는 `host instance`를 찾는 함수인데, `host instance`는 리액트 컴포넌트의 output을 스크린에 렌더하는 역할을 담당한다. DOM node라고 보면 된다. `NoPortals`라는 조건이 붙으면서, portal이 아닌 것들 중에서 파라미터로 넘겨준 `fiber`의 direct parent component를 찾아낸다. 

`<div id="root">`는 parent가 없기때문에, `hostInstance`가 없어서 아래 if문은 bypass하고 `updateContainer()`를 호출한다. 

이제 `updateContainer()`를 살펴보자. 

```javascript
// react-dom.development.js
function updateContainer(element, container, parentComponent, callback) {
  {
    onScheduleRoot(container, element);
  }
  var current$1 = container.current;
  var eventTime = requestEventTime();
  var lane = requestUpdateLane(current$1);
  ...
}
```

여기서 `element`은 `<App />`이고 `container`는 `FiberRootNode`이다. 

```javascript
// react-dom.development.js
function updateContainer(element, container, parentComponent, callback) {
  {
    onScheduleRoot(container, element);
  }

  var current$1 = container.current;
  var eventTime = requestEventTime();
  var lane = requestUpdateLane(current$1);
  ...
}
```

`onScheduleRoot()`는 개발모드에서 React와 React Developer Tools, Redux DevTools와 같은 다양한 third-party 툴들을 React와 연결시키는 역할이다. 이를 통해 디버깅이 더 수월해진다는 장점이 있다. production mode에는 없으므로 렌더링과 직접적인 관련이 없으니 생략하고 지나가겠다. 

`container.current`는 `HostRoot`이고, `requestEventTime()`은 현재 timestamp를 리턴한다. `requestedUpdateLane`은 업데이트 우선순위를 리턴한다. 

`lane`은 React18에서 도입된 개념인데, 단어만 보자면 자동차 도로에서 차선을 나타낸다. 1차선의 차량이 가장 빠르고, 뒤로 갈수록 점점 느려지는 것을 생각해보면, 업데이트의 중요도에 따라서 `lane`을 배정한다고 이해하면 된다. `lane`은 미리 지정된 상수인데, 우선순위는 다음에 기회가 되면 자세히 알아봐야겠다.

리액트 렌더링에는 `reconciler`가 활용된다. `reconciler`는 우리말로 하면 중재자 정도인데, 컴포넌트에서 업데이트가 일어날 때 업데이트의 우선순위를 판단해서, 우선순위대로 업데이트 사항을 반영한다. 여기서는 `DefaultEventPriority`라는 상수가 리턴된다 (숫자 16). 찾아보니 숫자의 크기가 업데이트 순서와는 관계가 없는 것 같은데, 이건 나중에 state 업데이트와 연관된 렌더링에 대해 찾아볼 때 조금 더 알아보겠다. 

계속 이어서 보자면 

```javascript
// react-dom.development.js
function updateContainer(element, container, parentComponent, callback) {
  ...
  {
    markRenderScheduled(lane);
  }

  var context = getContextForSubtree(parentComponent);

  if (container.context === null) {
    container.context = context;
  } else {
    container.pendingContext = context;
  }

  {
    if (isRendering && current !== null && !didWarnAboutNestedUpdates) {
      didWarnAboutNestedUpdates = true;

      error('Render methods should be a pure function of props and state; ' + 'triggering nested component updates from render is not allowed. ' + 'If necessary, trigger nested updates in componentDidUpdate.\n\n' + 'Check the render method of %s.', getComponentNameFromFiber(current) || 'Unknown');
    }
  }

  ...
}
```

`markRenderScheduled()`는 `injectedProfilingHooks`가 있다면, 렌더링 계획에 대해 업데이트를 하는데, 여기서는 null이라서 아무런 일도 일어나지 않는다. [React Profiler](https://reactjs.org/docs/profiler.html)를 사용했을 때 사용된다. 지금은 `Profiler`가 없기때문에 아무런 효과가 없다. 

`getContextForSubtree()`는 `context` 정보를 가져오는 역할이다. `useContext` 훅도 내부적으로는 `getContextForSubtree()`를 호출해서 context정보를 가져온다고 보면된다. 여기서는 `parentComponent`가 null이라 `{}`를 리턴하지만, `parentComponent`가 있다면 그 정보를 활용해서 context정보를 리턴한다. `container.context`는 null이기 때문에, `{}`를 할당한다. 

이후 if문은 바이패스하는데, 저 에러문구는 본적이 있다. `render()`에서--함수형 컴포넌트를 사용한다면 return에서--사이드 이펙트가 발생하면 보여주는 메세지이다. 예를들면 `render()`안에서 state를 업데이트 하거나, prop으로 넘겨주는 값을 수정하는 것이다. 라이브러리를 잘못 사용할 때도 종종 발생하는 것 같다. [error from react-toastify](https://github.com/fkhadra/react-toastify/issues/406#issuecomment-564630604)

계속 가보자면...

```javascript
// react-dom.development.js

function updateContainer(element, container, parentComponent, callback) {
  ...
  
  var update = createUpdate(eventTime, lane); 
  
  update.payload = {
    element: element
  };
  callback = callback === undefined ? null : callback;

  if (callback !== null) {
    {
      if (typeof callback !== 'function') {
        error('render(...): Expected the last optional `callback` argument to be a ' + 'function. Instead received: %s.', callback);
      }
    }

    update.callback = callback;
  }

  var root = enqueueUpdate(current$1, update, lane);

  if (root !== null) {
    scheduleUpdateOnFiber(root, current$1, lane, eventTime);
    entangleTransitions(root, current$1, lane);
  }

  return lane;
  ...
}
```

`createUpdate()`는 `update`라는 객체를 리턴한다. 아래 모양이다.

```javascript
var update = {
  eventTime: eventTime,
  lane: lane,
  tag: UpdateState,
  payload: null,
  callback: null,
  next: null
};
```

update에 사용되는 queue가 linked list이기 때문에 `next`라는 key를 가지고 있는 것 같다. 해당 객체가 `enqueueUpdate`에 넘겨져서 렌더링 queue에 들어가기 때문에 pointer로 활용된다. `update.payload`에 할당되는 `element`는 `main.tsx`에 선언된 `<App />`이다. callback은 null이니 bypass된다. 

`enqueueUpdate()`에 전달되는 `current$1`은 `HostRoot`이고, 위에 설명한 update 객체, lane은 업데이트 우선순위를 나타내는 상수이다. 호출하면 `FiberRootNode`를 리턴한다. `enqueueUpdate()`를 따라가보자

```javascript
function enqueueUpdate(fiber, update, lane) {
  var updateQueue = fiber.updateQueue;
  if (updateQueue === null) {
    // Only occurs if the fiber has been unmounted.
    return null;
  }

  var sharedQueue = updateQueue.shared;
  {
    if (currentlyProcessingQueue === sharedQueue && !didWarnUpdateInsideUpdate) {
      error('An update (setState, replaceState, or forceUpdate) was scheduled ' + 'from inside an update function. Update functions should be pure, ' + 'with zero side-effects. Consider using componentDidUpdate or a ' + 'callback.');

      didWarnUpdateInsideUpdate = true;
    }
  }
  ...
}
```

기억을 되돌려보면, `initializeUpdateQueue(uninitializedFiber)`를 호출해서 `HostRoot.updateQueue`에 아래 queue를 할당했다. 

```javascript
var queue = {
  baseState: fiber.memoizedState,
  firstBaseUpdate: null,
  lastBaseUpdate: null,
  shared: {
    pending: null,
    interleaved: null,
    lanes: NoLanes
  },
  effects: null
};
```

이제 이 queue가 `updateQueue`가 된다. 처음 렌더링이 발생하는 것이기 때문에, `currentlyProcessingQueue`는 null이라서 if문은 bypass한다. 계속 이어서 보자.  

```javascript
function enqueueUpdate(fiber, update, lane) {
  ...
  if (isUnsafeClassRenderPhaseUpdate()) {
    // This is an unsafe render phase update. Add directly to the update
    // queue so we can process it immediately during the current render.
    var pending = sharedQueue.pending;

    if (pending === null) {
      // This is the first update. Create a circular list.
      update.next = update;
    } else {
      update.next = pending.next;
      pending.next = update;
    }

    sharedQueue.pending = update; // Update the childLanes even though we're most likely already rendering
    // this fiber. This is for backwards compatibility in the case where you
    // update a different component during render phase than the one that is
    // currently renderings (a pattern that is accompanied by a warning).

    return unsafe_markUpdateLaneFromFiberToRoot(fiber, lane);
  } else {
    return enqueueConcurrentClassUpdate(fiber, sharedQueue, update, lane);
  }
}
```

최초 업데이트기때문에 `isUnsafeClassRenderPhaseUpdate()`는 `false`를 리턴해서 else문으로 간다. 그 전에 **render phase**를 언급하고 넘어갈 필요가 있다. 

리액트 렌더링은 두 단계의 phase로 나누어진다. 하나는 방금 언급한 **render phase**이고 다른 하나는 **commit phase**이다. 

간단히 설명하면 `render phase`는 React가 DOM Tree와 VirtualDOM Tree를 비교하면서 [이전 포스트](https://jasonkang14.github.io/react/what-is-fiber-architecture)에서 언급한 `work-in-progress queue`를 생성해서 화면을 업데이트를 준비하는 과정이다. `commit phase`는 `render phase`에서 작업된 변경상태들을 화면에 반영하는 단계이다. 최초 렌더링이기 때문에 업데이트할게 없고, 따라서 `work-in-progress queue`가 필요 없는 상황이기 때문에, `isUnsafeClassRenderPhaseUpdate()`가 `false`를 리턴하는 것이다. state 변경으로 인해 update가 발생하더라도 문제가 없다면 `false`일 것 같다. 

그럼 이제 `enqueueConcurrentClassUpdate()`를 확인해본다. 

```javascript
function enqueueConcurrentClassUpdate(fiber, queue, update, lane) {
  var interleaved = queue.interleaved;
  if (interleaved === null) {
    update.next = update; 

    pushConcurrentUpdateQueue(queue);
  } else {
    update.next = interleaved.next;
    interleaved.next = update;
  }

  queue.interleaved = update;
  return markUpdateLaneFromFiberToRoot(fiber, lane);
}
```

`fiber`는 여전히 `HostRoot`이고, `queue`는 아래 객체이다. `update`는 업데이트를 발생시킨 timestamp와 lane정보를 가지고있다. 

```javascript
shared: {
  pending: null,
  interleaved: null,
  lanes: NoLanes
},
```

`interleaved`라는 단어는 무언가를 번갈아가면서 한다는 뜻인데, 여기서 어떤 용도인지는 아직 잘 파악이 되지 않는다. else문의 코드로 유추할 때는 하나씩 업데이트를 쳐내기 위해 사용되는 값인 것 같다. 지금은 `interleaved == null`이기 때문에, update.next에 `update`자신을 할당하고, `pushConcurrentUpdateQueue()`를 호출한다. `concurrentQueues`는 array인데, queue이기때문에 업데이트가 필요한 queue가 하나씩 `.push()`되어 끝에 추가되는 구조이다. 

`queue.interleaved`에 `update`를 할당하고, `markUpdateLaneFromFiberToRoot()`를 호출한다. 


```javascript
function markUpdateLaneFromFiberToRoot(sourceFiber, lane) {
  sourceFiber.lanes = mergeLanes(sourceFiber.lanes, lane);
  var alternate = sourceFiber.alternate;

  if (alternate !== null) {
    alternate.lanes = mergeLanes(alternate.lanes, lane);
  }

  {
    if (alternate === null && (sourceFiber.flags & (Placement | Hydrating)) !== NoFlags) {
      warnAboutUpdateOnNotYetMountedFiberInDEV(sourceFiber);
    }
  } // Walk the parent path to the root and update the child lanes.


  var node = sourceFiber;
  var parent = sourceFiber.return;

  while (parent !== null) {
    parent.childLanes = mergeLanes(parent.childLanes, lane);
    alternate = parent.alternate;

    if (alternate !== null) {
      alternate.childLanes = mergeLanes(alternate.childLanes, lane);
    } else {
      {
        if ((parent.flags & (Placement | Hydrating)) !== NoFlags) {
          warnAboutUpdateOnNotYetMountedFiberInDEV(sourceFiber);
        }
      }
    }

    node = parent;
    parent = parent.return;
  }

  if (node.tag === HostRoot) {
    var root = node.stateNode;
    return root;
  } else {
    return null;
  }
}
```

`sourceFiber`는 `HostRoot`이고 `lane`은 상수이다. `sourceFiber.lanes`는 `lane`과 같은 상수인데, `mergeLanes()`는 두 lane에 대해 bitwise OR operator를 사용한 값이다. 

`alternate`도 fiber의 개념인데, commit phase가 지나고 DOM에 변경사항이 DOM Tree에 적용된 fiber를 `current fiber`라고 칭하고, 변경사항이 반영중인 fiber를 `work-in-progress fiber`라고 칭한다. alternate은 각각 서로를 나타낸다. `current fiber`의 alternate은 `work-in-progress fiber`이고, `work-in-progress fiber`의 alternate은 `current fiber`이다. 

`return`도 fiber의 개념인데, VirtualDOM Tree에서 본인의 부모 node에 해당하는 값을 나타낸다. 하지만 return이라는 단어를 사용하는 이유는, 해당 fiber node에서 작업이 끝나면 return, 돌아가서 처리할 node를 나타내기 때문이다. 렌더링에서 업데이트는 자식노드(child node)를 모두 업데이트 한 후에 부모노드(return node)를 업데이트하기 때문에 return이라는 단어를 사용한 것 같다. 개념과 유사하게 `parent`라는 변수에 해당 값을 할당한다. while문을 사용해서 업데이트 할 `lane`을 배정한다고 볼 수 있다. 

lane을 계산하는 과정이 모두 끝나면 최초 렌더링에서는 `sourceFiber`가 `HostRoot`이기 때문에 `HostRoot`의 `stateNode`인 `FiberRootNode`를 리턴한다. 

다시 `updateContainer()`로 돌아가면...


```javascript
// react-dom.development.js

function updateContainer(element, container, parentComponent, callback) {
  ...
  var root = enqueueUpdate(current$1, update, lane);

  if (root !== null) {
    scheduleUpdateOnFiber(root, current$1, lane, eventTime);
    entangleTransitions(root, current$1, lane);
  }

  return lane;
  ...
}
```

`root`가 `FiberRootNode`라서 `null`이 아니기 때문에 아래 두 함수를 실행한다. `scheduleUpdateOnFiber()`부터 보자

```javascript
function scheduleUpdateOnFiber(root, fiber, lane, eventTime) {
  checkForNestedUpdates();

  {
    if (isRunningInsertionEffect) {
      error('useInsertionEffect must not schedule updates.');
    }
  }

  {
    if (isFlushingPassiveEffects) {
      didScheduleUpdateDuringPassiveEffects = true;
    }
  }

  markRootUpdated(root, lane, eventTime);

  if (root === workInProgressRoot) {
    if ( (executionContext & RenderContext) === NoContext) {
      workInProgressRootInterleavedUpdatedLanes = mergeLanes(workInProgressRootInterleavedUpdatedLanes, lane);
    }

    if (workInProgressRootExitStatus === RootSuspendedWithDelay) {
      markRootSuspended$1(root, workInProgressRootRenderLanes);
    }
  }

  ensureRootIsScheduled(root, eventTime);
}
```

`checkForNestedUpdates()`는 `useEffect()`안에서 `setState()`를 실행하는 등으로 인해 무한루프가 발생하는 경우 에러를 던진다. [useInsertionEffect](https://react.dev/reference/react/useInsertionEffect)는 styled-component나 emotion과 같은 CSS-in-JS 라이브러리 author들이 사용하는 훅이라고 한다. 실제로는 사용할 일이 없으니 역시 무시하고 지나간다. 

`isFlushingPassiveEffects`를 이해하려면 **passive effect**를 먼저 알아야한다. **passive** 라는 단어는 수동적이라는 뜻인데, 화면에 렌더링을 마치고 발생하는 업데이트를 뜻한다. flushing 한다는 것은 변경사항을 DOM에 반영하는 것이다. 즉 `isFlushingPassiveEffects`는 렌더링은 끝난 상태이고, **뒤로 미뤄덨던 업데이트를 지금 반영중이냐**를 확인하는 변수이다. 맞다면 `didScheduleUpdateDuringPassiveEffects`라는 flag를 활성화하고 다음단계로 넘어간다. 

`markRootUpdated`는 `RootFiberNode`의 `eventTimes`라는 attribute에 `eventTime`을 할당해서, 값을 바꿔준다. 해당 lane의 업데이트가 언제 요청되었는지 저장해두는 것이다. `root === workInProgressRoot`라는 조건은, 렌더링이 진행중일 때 새로운 업데이트가 들어왔는지를 확인하는 것인데, 지금은 최초렌더링이기 때문에 해당되지 않는다. 따라서 `root` update의 schedule을 마치고 끝이난다. 

상태 업데이트와 관련있는 `beginWork()`라는 함수에 대해 언급하지 않고 마무리한다. 해당 함수에 대해서는, `setState()`가 발생했을 때 리액트가 어떻게 화면을 업데이트 하는지에 대해 공부하면서 작성해보도록 하겠다. 