---
title: Functional Programming with React [4] - Currying
date: "2022-06-16T18:53:37.121Z"
template: "post"
draft: false
slug: "/react/functional-programming-with-react-part-four"
category: "React"
tags:
  - "React"

description: "리액트 이벤트 핸들러에 currying을 적용하는 법"
---

함수형 프로그래밍에 대해 공부하면서 [Pure Function](https://jasonkang14.github.io/cs/functional-programming-pure-function) 과 [Currying](https://jasonkang14.github.io/cs/functional-programming-currying)에 대해 학습했다. 이제 `currying`의 개념을 React의 이벤트핸들러에 적용시켜보고자 한다.

단순하게 말하면 currying은 함수에 여러 인자를 동시에 넘겨주는게 아니라 sequence로 넘겨주는 것이다.
이벤트 핸들러에 커링을 적용하면 코드가 조금 단순해질 수 있는데, 예제를 통해서 살펴보도록 하자.

```typescript
function ComponentA() {
  const handleClick = (item: string) => {
    alert(item);
  };

  const handleClickWithEvent = (
    e: React.MouseEvent<HTMLButtonElement>,
    item: string
  ) => {
    alert(item);
  };

  return (
    <>
      <button onClick={(e) => handleClickWithEvent(e, "시작")}>시작</button>
      <button onClick={() => handleClick("종료")}>종료</button>
    </>
  );
}
```

위와 같이 선언하는 이유는 화살표 함수를 사용하지 않으면 render에서 바로 함수가 실행되기 때문이다. [리액트 공식문서](https://reactjs.org/docs/handling-events.html#passing-arguments-to-event-handlers)에서도 argument를 이벤트핸들러에 넘기기 위해서는 위와 같이 작성하라고 되어있다. 추가로 타입스크립트의 경우에는 단순히 `handleClick("종료")`와 같이 작성했을 때는 타입에러도 발생한다.

그리고 두 버튼은 같은 일을 하는데 이벤트를 인자로 사용하는 점에만 차이가 있다. 이 경우 커링을 활용하면 같은 이벤트 핸들러를 재사용 할 수 있다. 위 코드는 아래와 같이 수정이 가능하다.

```typescript
function ComponentA() {
  const handleClick =
    (item: string) => (event: React.MouseEvent<HTMLButtonElement>) => {
      alert(item);
    };

  return (
    <>
      <button onClick={handleClick("시작")}>연결 시작</button>
      <button onClick={handleClick("종료")}>연결 종료</button>
    </>
  );
}
```

직전에 화살표 함수를 사용하지 않고 이벤트핸들러에 인자를 넘겨주면 함수가 바로 실행된다고 했다. 하지만 이렇게 작성한 경우에는 그렇지 않다. 왜냐하면 currying을 적용한 함수의 경우에는 함수의 인자들이 넘어와야 실행이 되는데, 해당 버튼을 클릭하기 전에는 이벤트가 해당 이벤트 핸들러에 전달되지 않기 때문이다.

이벤트 핸들러가 정상적으로 작동하는 또 다른 이유는 자바스크립트의 [closure](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures) 때문이다. `onClick`에 `handleClick("시작")`을 할당하는 순간, event를 인자로 받는 inner function은 외부 함수인 `item`에 접근할 수 있다. 따라서 클릭 이벤트를 해당 이벤트 핸들러에 인자로 전달할 때, outer function의 scope를 가지고 있기 때문에 정상적으로 `alert`가 작동하는 것이다.

이벤트를 직접적으로 이벤트 핸들러에 인자로 넘겨주지는 않지만, MDN의 [Practical Closures](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Closures#practical_closures)섹션에서도 이벤트에 currying을 사용할 것을 권장한다. 어떠한 일이 일어날지를 정의한 함수를 선언한 다음, 해당 함수를 이벤트 핸들러로 할용하는 것이다.

```typescript
function makeSizer(size) {
  return function () {
    document.body.style.fontSize = size + "px";
  };
}

var size12 = makeSizer(12);
var size14 = makeSizer(14);
var size16 = makeSizer(16);

document.getElementById("size-12").onclick = size12;
document.getElementById("size-14").onclick = size14;
document.getElementById("size-16").onclick = size16;
```

위 예제를 리액트스럽게 변환하면 아래와 같다

```typescript

function SizeComponent {
  const makeSizer = (size)  => () => {
    document.body.style.fontSize = size + "px";
  }

  return (
    <>
      <button id="size-12" onClick={makeSizer(12)}>12</button>
      <button id="size-14" onClick={makeSizer(14)}>14</button>
      <button id="size-16" onClick={makeSizer(16)}>16</button>
    </>
  )
}
```
