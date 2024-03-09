---
title: Functional Programming - Currying
date: "2022-06-12T18:14:37.121Z"
template: "post"
draft: false
slug: "/cs/functional-programming-currying"
category: "CS"
tags:
  - "CS"

description: "함수형 프로그래밍에서 커링(currying)은 무엇인가?"
---

# TL;DR

### Currying is a way to transform a function to decoponse multiple arguments into a sequence of functions with a single argument

순수함수에 대해 이해했으니, 이제 currying에 대해 알아본다. 일반적으로 함수를 호출할 때 아래와 같이 호출한다.

```javascript
func(a, b, c);
```

currying하면 아래와 같이 호출한다.

```javascript
func(a)(b)(c);
```

호출하는 방식이 변하려면, 함수의 구조가 변경되어야 하는데. 아래와 같다.

```javascript
function func(a, b, c) {
  return a + b + c;
}
```

위 함수에 currying을 적용하면,

```javascript
function func(a) {
  return function (b) {
    return function (c) {
      return a + b + c;
    };
  };
}
```

이렇게된다.

### 그럼 왜 currying을 해야하나?

1. 함수 실행 전 필요한 변수들을 모두 받았는지 확인할 수 있다.
2. 같은 인자를 여러번 넘기는 것을 방지할 수 있다.
3. 함수를 나누면서 하나의 함수가 하나의 역할만 할 수 있도록 구현한다(순수함수)
4. higher-order function(HOC)를 만들기 위해 사용된다.
5. 가독성에 도움이 된다.

어쩌면 currying을 적용한 함수가 가독성이 떨어진다고 볼 수도 있다. 그래서 자바스크립트에서 기본으로 제공하지는 않지만, currying을 적용할 수 있는 utility function을 만들어서 사용하는 방법도 있다.

```javascript
function _curry(func) {
  return function curried(...args) {
    if (f.length !== args.length) {
      return curried.bind(null, ...args);
    }
    return func(...args);
  };
}

function add(a, b, c) {
  return a + b + c;
}

add(1, 2, 3);

const curriedAdd = curry(add);
curriedAdd(1)(2)(3);
```

이런식으로 가능하다. 사실 가독성보다는 순수함수의 규칙을 지킨다는 것에 더 큰 의의가 있다고 생각한다. 다음에는 리액트에서 중요한 개념인 Higher-Order Component와 유사한 개념인 Higher-Order Function에 대해 알아보도록 하자.
