---
title: Hello World from Fast API
date: "2022-07-02T16:21:37.121Z"
template: "post"
draft: false
slug: "/fastapi/hello-world"
category: "FastAPI"
tags:
  - "FastAPI"

description: "FastAPI로 Hello World!"
---

2년만에 사이드 프로젝트를 시작한다. 2년 전에 사이드 프로젝트를 두 개 했었는데, 그 때는 무조건 잘 될거라고 판단하고 앱을 만드는데만 집중했다. 그래서 빨리 찍어내기 위해서 익숙한 기술로 개발했다. 프로젝트는 망했고(앱 정책상 하나는 짤리고, 하나는 게시도 못함) 나에게 남은 건 짤렸던 앱의 클라우드 비용뿐이었다. 그래서 이번엔 평소에 관심있던 기술로 사이드를 진행하고, 블로그라도 남겨보려고 한다. 

사이드에 사용하고자 하는 추천 모델이 python으로 구현되기 때문에, 언어는 python으로 정했다. 그리고 그동안 말만 들어봤던 FastAPI로 서버를 구현하려고 한다. 회사에 쿠버네티스를 구축하면서 Microservice Architecture를 적용하려고 하는데, 그 중에 FastAPI로 하나의 microservice를 구현할 예정이라서 일에도 도움이 될 것 같다. 모든 절차는 공식문서를 따른다 

그래도 나름 규모가 있는--장고를 기준으로 하면 앱이 여러개인--서비스를 만들 예정이기 때문에, 공식문서의 [Bigger Applications](https://fastapi.tiangolo.com/tutorial/bigger-applications/)를 참고했다. 디렉토리 구조는 아래와 같이 가져갈 예정이다. 공식문서에 나온 구조에서 admin과 dependencies.py를 제외하고 따라해본다.

admin은 필요 없는 기능인 것 같고--지금 보기에는--dependencies 는 http 통신할 때 토큰을 가져오는 코드인데, 아직 로그인 구현도 안 된 상태에서 토큰을 가져오는 코드는 빌요 없는 것 같다. 

```
app
├── __init__.py
├── main.py
└── routers
    ├── __init__.py
    └── users.py

```

그리고 이제 시키는대로 세부 내용들을 작성한다. 우선 `routers` 부터 시작한다. 공식문서 중에서는 `users.py`를 작성한다. 추후에 DB연동할 때 사용자 테이블을 먼저 연동하고, 그와 관련된 회원가입//로그인 등의 기능을 먼저 구현하고 싶기 때문이다. 장고를 처음 접했던 때를 생각하면, 그렇게 하고나면 어느정도 구동 원리가 이해될 것이라고 생각한다. `app`과 `routers` 디렉토리 밑에 `__init__.py`가 있는 이유는, 파이썬 모듈마냥 사용하고 싶기 때문이다. 

공식문서를 잘 따라서 `main.py`를 아래와 같이 구현한다. `users.py`는 공식문서대로 구현한다

```python
# main.py

from fastapi import FastAPI

from .routers import users

app = FastAPI()

app.include_router(users.router)

@app.get("/")
async def root():
    return "Hello World!"
```

그냥 튜토리얼을 따라하는 것과 조금 차이가 있는데, 기본 튜토리얼은 파일 하나짜리라서 서로 import하는게 없기 때문에 parent가 없어서 상대경로 import에서 계속 에러를 뱉어낸다. 그래서 최상단에 저 app이라는 디렉토리가 있어야 하고, 서버를 구동하는 명령어도 기본 튜토리얼과 차이가 있다. 

`uvicorn app.main:app --reload`

app 디렉토리에서, 해당 디렉토리안에 있는 `main.py`에 선언된 `app`을 실행하는 코드이다. 자주 사용할 명령어인데 반복해서 적기 귀찮아서 `run.sh`파일을 만들고 그 안에 넣어두었다. 

![hello-world-from-fast-api](https://i.imgur.com/Fypz0l7.png)

hello world가 매우 잘 작동하고, routers에 있는 것도 잘 작동한다. 
공식문서를 그대로 따라하면 `http://localhost:8000`으로 get request를 보내면 `main.py`에 있는 값이 아니라 `routers.users`에 있는 경로로 요청을 보내는데, 이건 `main.py`에 선언된 경로보다 `routers.users`이 먼저 import되기 때문인 것 같다. 

사이드에는 [GraphQL](https://graphql.org/)도 적용해보려고 한다. GraphQL역시 회사 개발팀에서 4분기 목표로 잡은 내용인데, 미리 공부해서 팀원들과 공유하려고 한다. 연말 출시 목표인데 별 탈 없이 잘 진행할 수 있기를...
