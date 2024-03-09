---
title: "Nock을 활용한 React Query 테스트"
date: "2023-05-29T20:35:37.121Z"
template: "post"
draft: false
slug: "/react/mock-http-request-with-mock"
category: "React"
tags:
  - "React"

description: "custom hook을 mocking하지 않고 HTTP request를 테스트하는 방법"
---

[지난 포스트](https://jasonkang14.github.io/react/unit-test-with-jest)에 이어서 이번에는 HTTP Request를 테스트 해보려고 합니다. 저는 전역변수 및 상태관리를 편하게 하기 위해 [React Query](https://tanstack.com/query/v4/)를 사용중인데요, 리액트 쿼리에서는 [테스트를 위한 페이지](https://tanstack.com/query/v4/docs/react/guides/testing)를 별도로 제공하고 있습니다.

리액트 쿼리는, 커스텀 훅을 사용해서 HTTP request를 처리하는 것이 일반적인데, 커스텀 훅을 mocking하지 않고, http request를 mocking하기를 권장하고 있습니다. 

공식문서에서 권장하는 테스트 방향은 아래와 같습니다. 

```typescript
const queryClient = new QueryClient();
const wrapper = ({ children }) => (
  <QueryClientProvider client={queryClient}>
    {children}
  </QueryClientProvider>
);

const expectation = nock('http://example.com')
  .get('/api/data')
  .reply(200, {
    answer: 42
  });

const { result, waitFor } = renderHook(() => useFetchData(), { wrapper });

await waitFor(() => {
  return result.current.isSuccess;
});

expect(result.current.data).toEqual({answer: 42});
```

간단하게 요약하자면

1. 테스트 용도로 별도의 `QueryClientProvider`와 `queryClient`를 생성하고, 
2. nock을 사용해 http request를 mocking하고
3. 기존에 작성된 커스텀 훅에서 해당 http request를 호출한 것처럼 해서 처리해라

입니다. 

[nock](https://github.com/nock/nock)은 Node.js를 위한 HTTP mocking 및 테스팅 라이브러리입니다. 이 라이브러리는 HTTP 요청을 가로채고, 설정에 따라 응답을 반환함으로써 개발자가 애플리케이션에서 외부 서비스에 대한 의존성을 제거할 수 있게 도와줍니다.

Nock을 사용하면, 개발자는 실제 HTTP 서버가 아닌, Nock 객체를 사용해 테스트를 수행할 수 있습니다. 이는 테스트의 속도를 높이고, 네트워크 연결 문제나 외부 서비스의 장애가 테스트 결과에 영향을 주는 것을 방지할 수 있습니다.

그럼 이제 로그인 성공사례를 테스트해보겠습니다. 

```typescript
it('이메일과 비밀번호를 사용해서 성공적으로 로그인 할 수 있다', async () => {
  nock(process.env.VITE_API_ADDRESS ?? '')
    .post(`/${endpoints.login}`, { username: 'registered', password: 'password' })
    .reply(200, { token: 'AUTH_TOKEN' });

  const { result } = renderHook(() => useLogin(), { wrapper: TestWrapper });

  act(() => {
    result.current.mutate({
      username: 'registered',
      password: 'password',
    });
  });

  await waitFor(() => expect(result.current.isSuccess).toBe(true));

  expect(result.current.data).toEqual({ token: 'AUTH_TOKEN' });
});
```


`useLogin()`이라는 커스텀 훅이 호출하는 login API를 nock을 사용해서 mocking하고, react query의 `isSuccess`가 true인지를 확인하는 코드입니다. 성공적으로 http 요청이 200을 반환하면, mocking된 response가 제대로 return되었는지 확인합니다. 

![success-test-success](https://i.imgur.com/WlOnrGL.png)

테스트는 성공적으로 통과했습니다. 

response body가 중복되기 때문에, 아래처럼 변수로 선언해서 사용하는 것이 어떠냐고 생각하실 수도 있습니다. 

```typescript
const successResponse = {
  token: 'AUTH_TOKEN'
}

nock(process.env.VITE_API_ADDRESS ?? '')
    .post(`/${endpoints.login}`, { username: 'registered', password: 'password' })
    .reply(200, successResponse);

...

expect(result.current.data).toEqual(successResponse);

```

하지만 [assert와 expect는 하드코딩 하는 것이 더 좋다는 글](https://jojoldu.tistory.com/615)을 읽고 저는 expect에 해당하는 항목들은 모두 하드코딩 하고 있습니다. 관심이 있으신 분은 링크된 포스트에 상세한 예제와 함께 설명되어있으니 참고하시면 좋겠습니다. 

이제 실패 케이스를 테스트합니다. 

```typescript
it('로그인 실패 시 에러메세지가 나타난다', async () => {
  nock(process.env.VITE_API_ADDRESS ?? '')
    .post(`/${endpoints.login}`, { username: 'not-registered', password: 'password' })
    .reply(401, {});

  const { result } = renderHook(() => useLogin(), { wrapper: TestWrapper });

  act(() => {
    result.current.mutate({
      username: 'not-registered',
      password: 'password',
    });
  });

  await waitFor(() => expect(result.current.isError).toBe(true));
});
```

성공케이스와 유사하게 작성하고, http status code를 401로 처리했습니다. 테스트 케이스는 통과하지만 한가지 어색한 부분이 있습니다. axios를 try...catch로 감싸기 때문에, custom hook에서 400대 에러가 발생하면 `console.error()`에 값이 찍힙니다. 

![success-with-error-message](https://i.imgur.com/lacvbiA.png)

사실상 성공인데 빨간색 메세지가 `Error`라는 값과 나타나니 당황스럽습니다. 위에서 테스트 용도로 별도의 `QueryClientProvider`를 생성한다고 언급했는데요, 해당 `QueryClientProvider`의 Logger에서 error는 출력하지 않는 것으로 수정하도록 합니다. 

```typescript
setLogger({
  log: console.log,
  warn: console.warn,
  error: () => {},
});
```

`console.log()`와 `console.warn()`은 표시되지만, error는 화면에 출력되지 않습니다. 다시 테스트를 돌려봅니다. 

![error-case-success-without-error-message](https://i.imgur.com/ZlvY65L.png)

`console.error()`가 사라진 것을 확인할 수 있습니다. 

logger를 disable하는 방법은 리액트 쿼리 버전에 따라 다르니 공식문서를 확인해주시고, 만약 `setLogger` 옵션을 사용하고 싶지 않다거나, 별도의 `QueryClientProvider`를 사용하지 않고, 실제 운영 환경에서 사용되는 코드를 같이 사용하고 싶다면 아래와 같이 `console`을 mocking할 수 있습니다. 

```typescript
beforeAll(() => {
  jest.spyOn(console, 'log').mockImplementation(() => {});
  jest.spyOn(console, 'warn').mockImplementation(() => {});
  jest.spyOn(console, 'error').mockImplementation(() => {});
});

afterAll(() => {
  jest.restoreAllMocks();
});
```

모든 테스트 파일마다 해당 mocking을 해줘야하니 불편할 수 있겠지만, 선택사항이라고 생각합니다. 개인프로젝트에서 테스트코드를 검증(?)했으니 이제 회사 코드에도 테스트코드를 작성하려고 합니다. 레거시 코드에 테스트를 작성하는게 쉽지 않을 수 있지만, 리팩토링 전 필수라고 생각해서 조금씩 진행할 예정입니다. 작업하다 의미있는 시행착오를 만난다면 글로 공유드리겠습니다. 