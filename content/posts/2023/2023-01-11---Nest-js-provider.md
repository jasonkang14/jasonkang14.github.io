---
title: Nest.js 의 Provider
date: "2023-01-11T14:35:37.121Z"
template: "post"
draft: false
slug: "/nestjs/what-is-provider"
category: "Nestjs"
tags:
  - "Nestjs"

description: "Dependency Injection을 통해 Nest.js Provider를 사용하는 방법에 대해"
---

### TL;DR
Provider는 다양한 서비스와 데이터를 관리하는 역할을 담당한다. 

service, repository, factory등등 다양한 Nest 클래스들이 provider의 역할을 할 수 있다고 한다. provider의 핵심 아이디어는 `dependency injection`이다. 이를 통해 클래스들은 서로 관계를 형성할 수 있다. 

![nest-provider-diagram](https://i.imgur.com/BAWS1Es.png)

## Services
일반적으로 데이터베이스에 접근해서 read/write/update/delete 등을 담당하는 역할로 보인다. 

```typescript
// cats.service.ts

import { Injectable } from '@nestjs/common';
import { Cat } from './interfaces/cat.interface'; // entity의 type

@Injectable() // provider는 inject 가능함
export class CatsService {
  private readonly cats: Cat[] = [];

  create(cat: Cat) {
    this.cats.push(cat);
  }

  findAll(): Cat[] {
    return this.cats;
  }
}
```

Nest.js의 Inversion of Control(IoC) 컨테이너가 application내에서 사용되는 객체의 dependency를 관리한다. `@Injectable` decorator를 사용해서 IoC 컨테이너에게 해당 클래스도 injection이 가능한 provider라는 것을 알려주는 것이다. 

선언한 `service`는 `controller`에서 사용/호출 된다. service가 데이터베이스에 접근하는 역할을 담당하기 때문에(예제 코드에 데이터베이스는 없지만), client로부터 read/write request가 들어오면, service에 있는 method를 호출하는 것이다. controller관련 포스트에 작성했던 controller를 service를 사용하는 방식으로 수정해보겠다. 

```typescript
// cats.controller.ts

import { Controller, Get, Post, Body } from '@nestjs/common';
import { CreateCatDto } from './dto/create-cat.dto';
import { CatsService } from './cats.service';
import { Cat } from './interfaces/cat.interface';

@Controller('cats')
export class CatsController {
  constructor(private catsService: CatsService) {}

  @Post()
  async create(@Body() createCatDto: CreateCatDto) {
    this.catsService.create(createCatDto);
  }

  @Get()
  async findAll(): Promise<Cat[]> {
    return this.catsService.findAll();
  }
}
```

`CatsService`가 contructor를 통해 controller에 `inject`된 것을 확인할 수 있다. `private` syntax를 사용해서 해당 service가 외부로 유출되지 않고 해당 controller 안에서만 사용될 수 있도록 한다. `private`을 앞에 붙이기 때문에 nest가 해당 클래스를 바로 인스턴스화해서 controller 내에서 사용할 수 있도록 한다. 이건 typescript가 제공하는 기능이라는데 다음에 찾아봐야겠다. 

dependency injection은 Nest.js 밖에서도 중요한 개념으로 보인다. Nest.js 공식문서는 [Angular 공식문서](https://angular.io/guide/dependency-injection)에 dependency injection이 잘 설명되어 있다고 한다. 

## Optional Providers

클래스가 application configuration의 기준을 미치지 못하는 경우, `@Optional` 데코레이터를 활용해서 default값을 주는데 사용되는 것 같다. 

```typescript

import { Injectable, Optional, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  constructor(@Optional() @Inject('HTTP_OPTIONS') private httpClient: T) {}
}
```

## Property-based injection

provider를 constructor 밖에서 inject 할 수도 있다. 

```typescript
import { Injectable, Inject } from '@nestjs/common';

@Injectable()
export class HttpService<T> {
  @Inject('HTTP_OPTIONS')
  private readonly httpClient: T;
}
```

## Provider Registration

provider를 다른 provider나 controller에서 inject해서 사용하기 위해서는 module에 등록해줘야한다. 

```typescript
// app.module.ts

import { Module } from '@nestjs/common';
import { CatsController } from './cats/cats.controller';
import { CatsService } from './cats/cats.service';

@Module({
  controllers: [CatsController],
  providers: [CatsService],
})
export class AppModule {}
```

위 예제코드에서는 `app.module.ts`에 provider를 선언했기 때문에 application어디서든 사용할 수 있을 것으로 추정된다. 하지만 뒤에 나오는 module을 활용해서 관심사별로 모듈을 나눈다면, provider를 `export`해줘야 다른 모듈에서 해당 provider를 사용할 수 있다. 다음장이 module이라 거기에 나올 것 같다. 