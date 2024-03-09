---
title: Next.js에서 서버로 인증하는 방법
date: "2022-08-01T21:41:37.121Z"
template: "post"
draft: false
slug: "/nextjs/authenticate-with-next-auth"
category: "Nextjs"
tags:
  - "Nextjs"

description: "next-auth를 활용해서 서버로 인증하는 방법"
---

[React.js를 Next.js로 이관하는 과정에 대해 설명한 포스트](https://jasonkang14.github.io/nextjs/migrating-from-react)에서 로그인의 성공을 포스트 했다. 그런데 여기에 한가지 문제가 있었다. 기존에 서버에서 받아온 토큰을 세션 스토리지에 저장했었는데, 이 토큰을 사용해서 [getServerSideProps](https://nextjs.org/docs/basic-features/data-fetching/get-server-side-props)에서 데이터를 가져오려니 아래와 같은 에러를 만났다. 

`sessionStorage is not defined`
![session-storage-is-not-defined](https://i.imgur.com/vhmHcLl.png)

생각해보면 당연한 이야기다. 서버는 브라우저가 아니기때문에 스토리지가 존재하지 않는다. 그래서 자연스럽게 쿠키에 데이터를 저장해야했고, 기존 설정들을 바꿔서 쿠키에 데이터를 저장해보기로 했다. 

일단 axios설정에 `withCredentials: true` 를 추가한다
```typescript
const axiosClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_ADDRESS,
  timeout: 8000,
  headers: {
    'Content-type': 'application/json',
  },
  withCredentials: true,   // 여기가 추가된다. 
});
```

[MDN](https://developer.mozilla.org/en-US/docs/Web/API/Request/credentials)에 보면 브라우저에서 쿠키를 사용하려면, 리퀘스트 보낼 때  `credentials: 'include'` 로 해야한다고 나와있는데 axios에서는 위와 같이 설정해야 한다. 그리고 이제 서버에서 보내는 response의 body가 아니라 cookie에 세션을 담아서 전송하는 방식으로 변경한다. 회사의 서버는 fastapi로 되어있다. 

```python
  response = JsonResponse()
  response.set_cookie(
    key="session",
    value=FASTAPI_SESSION,
    samesite='None',
    secure=True,
    httponly=True,
    expires=30*60
  )
```

그런데 저렇게 작업을 해도, response header에 보면 `set-cookie`가 있는데, 개발자도구의 application 탭에서는 쿠키를 확인할 수 없다. 심지어 서버 설정에 `expose_header`인가 라는 설정도 있어서 해봤는데 안된다. 지금 생각으로는 서버와 클라이언트의 도메인이 다르기 때문인걸로 보인다. 그래서 로컬호스트에 도메인을 붙이는 작업을 해볼까 하다가 찾아보니 [Next.js 공식문서](https://nextjs.org/docs/authentication#authentication-providers)에서 [NextAuth.js](https://next-auth.js.org/)라는 라이브러리를 추천하는 것을 보고 사용해보기로 했다. 

다수의 예제들이 소셜로그인을 위한 것인데(신기하게도 네이버와 카카오 예제도 있다) 지금은 회사에서 자체적으로 올린 인증서버를 사용해야 해서 [credentials](https://next-auth.js.org/providers/credentials) provider를 사용해서 작업해보기로 했다. 

일단 시키는대로 `pages/api` 디렉토리에 `auth`라는 디렉토리를 만들고 그 안에 `[...nextauth].ts`라는 파일을 생성한다. 그리고 아래와 같이 작성한다

```typescript
import axios from 'axios';
import NextAuth, { NextAuthOptions } from 'next-auth';
import CredentialsProvider from 'next-auth/providers/credentials';

export const authOptions: NextAuthOptions = {
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        username: {
          label: 'Email',
          type: 'email',
        },
        password: { label: 'Password', type: 'password' },
      },
      async authorize(credentials, req) {
        try {
          const res = await axios.post(process.env.NEXTAUTH_URL || '', credentials);
          if (res.data) {
            return res.data;
          }
        } catch (e) {
          return;
        }

        return null;
      },
    }),
  ],
  session: {
    strategy: 'jwt',
  },
  callbacks: {
    async jwt({ token, user }) {
      token.sessionToken = user?.token;
      return token;
    },
    async session({ session, user, token }) {
      return session;
    },
  },
};

export default NextAuth(authOptions);

```

next auth는 자체적으로 아이디, 비밀번호를 입력하는 `<form>`을 제공하는데, `credentials`에 작성한 값들이 사용된다. 환경변수로 `NEXTAUTH_URL`을 꼭 설정해줘야지 next-auth를 사용한 authentication이 가능하다. 안그러면 계속 자체적으로 만들어진 서버주소 `localhost:3000/api/auth/signin`으로 요청을 보내서 에러가 발생한다. 

설정해둔 인증서버 주소로 요청을 보내면, 서버는 response body에 토큰을 담아서 return한다. authorize가 res.data를 return하면 저 정보는 `jwt` callback의 user에 담긴다.... next-auth의 의도는 서버에서 유저 정보를 리턴하는 것을 의도한 것 같다. 

user.token에 접근하여 자체 인증서버가 발급한 토큰을 token이라는 객체에 sessionToken이라는 key에 담아서 리턴하면 `session` callback의 session에 담기고, 이 session의 session.sessionToken이 쿠키로 들어가게 된다. `sessionToken`에 토큰을 담아야하는 이유는 소스코드를 열어보니 `session.sessionToken`이 `__Secure-next-auth.session-token`이라는 key로 쿠키에 저장되고, 이 쿠키를 next auth에서 제공하는 [getToken()](https://next-auth.js.org/tutorials/securing-pages-and-api-routes#using-gettoken)이라는 함수를 사용해서 가져올 수 있기 때문이다. 

여기서 추가로 설정을 하나 더 해줘야 하는데, 위에서 언급한 것처럼 서버에서 보내준 정보가 `jwt` callback의 user에 담기기 때문인지는 모르겠으나, next-auth는 자체적으로 jwt토큰을 생성한다. 그래서 나는 저 라이브러리를 fork해서 서버에서 제공하는 토큰을 바로 쿠키에 넣도록 수정했다. 

그리고 로그인 버튼을 클릭하면 `authorize`라는 함수가 실행돼서, 함수 로직을 타게 된다. 하지만 이미 만들어둔 로그인 페이지가 있기 때문에, 그 페이지를 사용하고 싶었다. 다행히 next-auth에서는 그 기능을 제공한다. 

```typescript
import { signIn } from 'next-auth/react';

export default function Login () {
  const handleLogin = async () => {
    const loginRes = await signIn('credentials', {
      username,
      password,
      redirect: false,
    });
  };
}

```

저기 `signIn` 이라는 함수를 호출하면 `[...nextauth].ts`에 있는 `authorize`를 호출하는 구조이다. 여기서 `redirect: false`로 해둔 이유는 `redirect: true`인 경우, `NEXTAUTH_URL`이라는 환경변수에 선언한 주소로 redirect되기 때문이다. signIn 함수로 `callbackUrl`이라는 값을 넘길 수도 있는데, `http://`를 포함한 주소를 입력해도 무시하고 `NEXTAUTH_URL`로 이동한다. 따라서 loginRes에서 제공하는 status를 확인해서 아래와 같이 redirect하는 식으로 구현했다. 

```typescript
import { signIn } from 'next-auth/react';
import { useRouter } from 'next/router';

export default function Login () {
  const router = useRouter();

  const handleLogin = async () => {
    const loginRes = await signIn('credentials', {
      username,
      password,
      redirect: false,
    });
  };

  if (loginRes.status === 200) {
    router.push('/dashboard')
  }
}
```

다음 포스트에서는 dashboard페이지에서 서버사이드 렌더링을 어떻게 구현했고, 이 방법이 client side rendering과 비교했을 때 얼마나 효율적인지에 대해 작성할 예정이다.