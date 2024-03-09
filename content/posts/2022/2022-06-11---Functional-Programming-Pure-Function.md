---
title: Functional Programming - Pure Function
date: "2022-06-11T23:14:37.121Z"
template: "post"
draft: false
slug: "/cs/functional-programming-pure-function"
category: "CS"
tags:
  - "CS"

description: "함수형 프로그래밍에서 순수함수의 역할은 무엇인가?"
---

# TL;DR

### A pure function always returns the same output if the same inputs are given without having any side effects.

영어로 쓰는게 말이 좀 더 쉬운 것 같아서 TL;DR만 영어로 작성한다. 블로그는 영어로 쓰는게 편하긴 하지만, 사내 세미나용이기 때문에 한국말로 써본다.

[Toss SLASH 22](https://toss.im/slash-22)에 frontend 개발자들이 참고하면 좋을만한 내용이 많았다. 현재 회사에 시니어가 없는 상황이라, 프로젝트 코드를 고도화 할 아이디어들을 다양한 컨퍼런스를 통해서 얻는 중인데, 그중에 `Effective Component`를 적용해보기로 했다. 그리고 리팩토링 하는김에, 함수형 프로그래밍을 조금 더 잘 적용하면 어떨까 싶어 공부하게 되었다.

우선 기본개념인 순수함수에 대해서 알아보자.

순수함수는

1. input이 같다면 항상 같은 output을 return 해야하고,
2. side effect가 없어야한다.

#### 항상 같은 output을 return 한다는 것은 무슨 뜻인지 예제를 통해 같이 살펴보자

```javascript
function add(a, b) {
  return a + b;
}

console.log(add(3, 5));
console.log(add(3, 5));
console.log(add(3, 5));
```

몇 번을 호출하던 항상 8을 return하고, 언제 호출하던 항상 8을 return한다. 즉 input이 같다면 항상 같은 output을 return하기 때문에, 순수함수라고 할 수 있다. 그렇다면 input이 같아도 다른 output을 return하는 함수는 어떤경우일까?

```javascript
let c = 10;
function addTwo(a, b) {
  return a + b + c;
}

console.log(addTwo(1, 2));
console.log(addTwo(1, 2));
console.log(addTwo(1, 2));
```

이번에도 비슷하다. 항상 13을 return한다. 그런데 갑자기 `c`의 값이 바뀐다면 어떻게 될까?

```javascript
let c = 10;
function addTwo(a, b) {
  return a + b + c;
}

console.log(addTwo(1, 2));
console.log(addTwo(1, 2));
c = 20;
console.log(addTwo(1, 2));
```

`c` 의 값이 바뀌면서 갑자기 23을 return한다. input이 같더라도 항상 같은 output을 return하지 않기 때문에, `addTwo`는 순수함수가 아니다.

#### 그렇다면 side effect는 무엇인가?

side effect는 함수 밖의 외부의 상태를 변경하거나, 들어온 인자의 상태를 변경하는 것이다.

```javascript
let c = 10;
function addThree(a, b) {
  c = b;
  return a + b;
}
```

위 함수는 항상 같은 값을 return하긴 하지만, 함수 밖 `c`라는 변수에 영향을 미친다. 이런 것들이 side-effect이다. 따라서 side-effect를 가진 위 함수는 순수함수라고 볼 수 없다. 그렇다고 side-effect가 늘 나쁜 것은 아니다. 가끔은 side-effect를 의도하는 경우도 있다(함수형 프로그래밍에 어긋나긴 하지만)

들어온 인자의 상태를 변경하는 건 어떤 경우가 있을까?

```javascript
let obj = { val: 10 };

function changeObj(obj, b) {
  obj.val += b;
}
```

위 함수는 `obj`라는 인자를 변형시키기 때문에 순수함수가 아니다. 함수형 프로그래밍에서는 외부의 값, 또는 기존 객체를 변형하지 않고 새로운 객체를 return한다.

```javascript
let obj = { val: 10 };

function createNewObj(obj, b) {
  return { val: obj.val + b };
}
```

이런식으로.
다음에는 [currying](https://jasonkang14.github.io/cs/functional-programming-currying)에 대해 알아본다
