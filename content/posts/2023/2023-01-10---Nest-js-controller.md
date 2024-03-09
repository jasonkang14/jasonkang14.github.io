---
title: Nest.js 의 Controller
date: "2023-01-10T14:35:37.121Z"
template: "post"
draft: false
slug: "/nestjs/what-is-controller"
category: "Nestjs"
tags:
  - "Nestjs"

description: "Nest.js Controller가 클라이언트의 request를 핸들링하는 방법"
---

### TL;DR
controller는 클라이언트로부터 request를 받아서 response를 return하는 역할을 담당한다. 

nest init을 하니 아래와 같은 기본 컨셉 설명이 있었다. 

[OOP(Object Oriented Programming)](https://jasonkang14.github.io/cs/object-oriented-programming), [FP(Functional Programming)](https://jasonkang14.github.io/cs/functional-programming-pure-function), FRP(Functional Reactive Programming)을 혼합해서 사용한다고 한다. OOP와 FP는 아는데 FRP는 잘 모르겠다. 다음 포스트에서 한 번 작성해보도록 하겠다. 

Node.js의 등장으로 자바스크립트가 점점 좋아지고 다양한 라이브러리들이 등장했지만, 아키텍쳐 문제는 해결하지 못했다고 한다. 따라서 Nest.js는 이를 해결하고자 한다. 

```npm
npm i -g @nestjs/cli
nest new project-name
```

위 명령어를 실행하면 npm을 사용해서 nestjs 를 글로벌로 설치하고, create-react-app과 유사하게 새로운 nest 프로젝트를 생성할 수 있다. 

overview 섹션을 따라하면서 어떤식으로 동작하는지 이해해보고자 한다. 

위 명령어를 실행하면 아래 구조의 파일들이 생성된다

```
src
├── app.controller.spec.ts
├── app.controller.ts
├── app.module.ts
├── app.service.ts
└── main.ts
```

대략적인 구조로 봤을 때는 `service`가 `view`의 역할을 하는 MVC 패턴을 따르는 것으로 보인다. 

```typescript
// main.ts

import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  await app.listen(3000);
}
bootstrap();
```

`NestFactory` 클래스는 application instance를 생성할 수 있는 다양한 method들을 제공한다. `create()`를 호출하면 application object를 리턴하고, 이 객체가 nest.js를 구동하는 다양한 method들을 지원한다. 

Nest.js는 Express를 기반으로 만들어졌는데, 퍼포먼스 향상을 위해 Fastify를 사용할 수도 있다고 한다. Express는 간단한 WebSocket 서버를 만들기 위해 사용한 적이 있는데 Fastify는 처음본다. 문서만 읽어보기로는 더 빠르다고 한다. 

이제 본격적으로 controller에 대해 살펴본다

![nest-js-controller](https://i.imgur.com/s6s39y1.png)

routing을 지원해서 클라이언트에게 특정 엔드포인트로 요청을 받는다. 특정 클래스가 controller임을 선언하기 위해서 `@Controller` decorator를 사용한다. basic controller의 구조는 아래와 같다. routing하는 법, response를 return하는 법, request 객체에 접근하는 법들을 아래 예제코드에 추가하도록 하겠다. 

```typescript

// cats.controller.ts

import {
    Body,
    Controller, 
    Delete, 
    Get, 
    HttpCode,
    Param,
    Query,
    Redirect
} from '@nestjs/common';

// 해당 controller가 요청을 받는 route. 
// `/cats`로 들어오는 request는 모두 여기서 처리한다
@Controller('cats') 
export class CatsController {
    // http method 괄호 안에는 경로가 들어간다.
    @Get() 
    findAll(): string { // 함수 이름
        return 'This action returns all cats';
    }

    // `/cats/profile`로 들어오는 request를 처리한다
    @Get('profile')
    findProfile(): string { // 함수 이름
        return 'This action returns a cat profile';
    }

    // abcd, ab_cd, abecd 등의 경로를 지원하나고 한다. 
    // 왜 쓰는지는 이해는 안된다.   
    @Delete('ab*cd') 
    @HttpCode(204) // custom http status code를 지정해줄 수 있다. default는 200
    findWild(): string {
        return 'This action is wild'
    }

    // route parameter로 값을 받아올 수 있다.
    @Patch(':catId')
    updateCat(@Param('catId') catId: number): string {
        return 'this updates a cat'
    }

    @Get('docs')
    @Redirect('https://docs.nestjs.com', 302)  // redirect
    getDocs(@Query('version') version) { // query parameter
        if (version && version === '5') {
            return { url: 'https://docs.nestjs.com/v5/' };
        }
    }

    @Post()
    async create(@Body() createCatDto: CreateCatDto) { // DTO를 선언해서 request body로 사용한다 
        return 'This action adds a new cat';
    }
}
```

Express의 `Response`객체를 사용해서 return할 수도 있다고 하는데, 왜 굳이 그렇게까지 해야하는지 모르겠다. response객체를 컨트롤해서 다양한 커스텀이 가능하다고 하는데. 지금 상황에서는 필요성을 느끼지 못하겠다.