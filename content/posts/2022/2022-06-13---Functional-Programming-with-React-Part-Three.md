---
title: Functional Programming with React [3] - Pure Component
date: "2022-06-13T12:53:37.121Z"
template: "post"
draft: false
slug: "/react/functional-programming-with-react-part-three"
category: "React"
tags:
  - "React"

description: "리액트 컴포넌트를 pure component로 변환하는 법"
---

함수형 프로그래밍에 대해 공부하면서 [Pure Function](https://jasonkang14.github.io/cs/functional-programming-pure-function) 과 [Currying](https://jasonkang14.github.io/cs/functional-programming-currying)에 대해 학습했다. 이제 `Pure Function`의 개념을 React의 `Pure Component`에 적용시켜보고자 한다.

Pure function에서 중요한 개념 중 하나는, 함수의 인자를 조작할 수 없다는 것이다. 따라서 React Component를 함수라고 볼 때 (함수형 컴포넌트기 때문에), 이와 유사하게 component는 인자로 받게되는 prop을 수정하면 안된다.

사내 세미나 일환으로 발표를 하다보니 예제를 들으면 좋을 것 같아서 회사 레포에서 prop을 변형하는 컴포넌트를 찾아봤더니 아래와 같은 코드가 있었다.

```typescript
interface Props {
  prop: any;
}

const ComponentA = ({ prop }: Props) => {
  const [showStatus, setShowStatus] = useState(false);

  const handleStatusChange = () => {
    setShowStatus(!showStatus);
  };

  return (
    <>
      <ComponentB status={showStatus} />
      <ComponentC status={showStatus} changeStatus={handleStatusChange} />
    </>
  );
};
```

`showStatus`라는 값이 `ComponentB` 와 `ComponentC` 의 prop인데, `ComponentC` 에게는 해당 prop을 변경할 수 있는 함수를 prop으로 내려준다. 따라서 prop을 변형할 수 있기 때문에, pure component가 아니다. 하지만 `showStatus`라는 값은 변경이 필요하고, 두개의 component에 사용된다. 이럴 경우에는 어떻게 해야할까?

`prop`을 수정하면 안된다는 원칙을 지키기 위해 `showStatus`를 [Context API](https://reactjs.org/docs/context.html)를 사용해서 전역변수로 변경했다. `ComponentA`는 더이상 state를 갖지 않는 stateless component가 되고. 자식 component들은 더이상 prop을 수정하지 않도록 수정했다.

[Profiler](https://reactjs.org/docs/profiler.html)를 사용해서 이렇게 하는 것이 무슨 의미가 있는지 검증해보았다.

```typescript
interface Props {
  prop: any;
}

const ComponentA = ({ prop }: Props) => {
  return (
    <Profiler id="component-a" onRender={onRenderCallback}>
      <ComponentB />
      <ComponentC />
    </Profiler>
  );
};
```

`onRenderCallback`은 react 공식문서의 예제를 그대로 가져왔고, 실제 렌더링 시간인 `actualDuration`을 비교했다.

특정 액션을 취하는데 지역변수를 활용한 경우 **6.8** 소요 되었는데, 전역변수로 활용하니 **4.2**로 약 38% 개선되었다. 다른 액션들도 취해보았을 때, 적게는 9%에서 많게는 42%정도 rendering 효율이 개선된 것을 확인할 수 있었다. profiler를 통해서 보면 전반적인 렌더시간도 많이 감소한 것을 볼 수 있다.

![rendering time check via profiler](https://i.imgur.com/8iD6PvO.png)

프런트엔드팀에서 지속적으로 리액트 효율 개선을 위해 많은 시도를 할 예정인데, 공유가 가능한 선에서 블로깅을 계속 해보도록 하겠다.
