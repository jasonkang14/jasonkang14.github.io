---
title: FastAPI - 디렉토리 구조 변경
date: "2022-07-15T12:21:37.121Z"
template: "post"
draft: false
slug: "/fastapi/directory-structure"
category: "FastAPI"
tags:
  - "FastAPI"

description: "FastAPI 디렉토리 구조를 재설정한다"
---

FastAPI 공식문서의 튜토리얼을 따라해보면 알겠지만, 각 섹션이 모여서 거대한 하나의 어플리케이션을 만드는게 아니라, 모든 파트가 나눠져있다.
따라서 이들은 모든 코드를 main.py안에서 해결한다. 

하지만 실제 프로그램은 그렇게 돌아가지 않는다. 적절히 분산해서 import해서 사용하는 것이 일반적이다. 

디렉토리 구조를 고민하던 와중에, 그동안 작성했던 [FastAPI 시리즈](https://jasonkang14.github.io/tag/fast-api)를 보신 CTO님께서 회사에서 진행하는 신규 프로젝트에서 FastAPI를 적용해보자고 하셨다. 직장인 특성상 회사일을 하게되면 개인 프로젝트보다 조금 더 열심히 하게되는 특성이 있다. 열심히 구글링했고, 결국 FastAPI maintainer가 공개한 프로젝트 구조 예제를 찾았다. 그를 기반으로 조금 확장해서 디렉토리 구조를 아래와 같이 변경했다. 

```bash
.
├── README.md
├── alembic
│   ├── README
│   ├── env.py
│   ├── script.py.mako
│   └── versions
├── alembic.ini
├── core
│   ├── __init__.py
│   ├── auth.py
│   └── config.py
├── crud
│   ├── __init__.py
│   └── user.py
├── database
│   ├── __init__.py
│   ├── models.py
│   └── session.py
├── main.py
├── requirements.txt
├── routers
│   ├── __init__.py
│   ├── deps.py
│   ├── router.py
│   └── user.py
├── run.sh
└── schemas
    ├── __init__.py
    └── user.py
```

1. alembic - database migration을 담당한다.
    - 하나의 database에 여러명이 migration 하는 경우 `alembic_version` table을 확인해서 에러를 수정해야 한다.
    - 또한 `versions` 디렉토리 내의 파일을 수정해서 에러를 해결할 수도 있다.

2. core
    - 주요 기능(?) 들을 넣는다
    - 일단은 인증/인가를 관장하는 `auth.py`와 환경변수를 담당하는 `config.py`를 생성했다.
    - 나중에 firebase와 관련된 기능들도 추가할 예정이다.

3. crud
    - database manipulation을 담당한다
    - Create, Read, Update, Delete

4. routers 
    - controller에 해당한다고 보면 되겠다.
    - interface를 규정하고, 적절하게 crud를 실시한다.
    - database dependency를 `deps.py`에 선언한다

5. schemas  
    - database로 연결하는 schema들을 선언한다.
    - 회사에서 프로젝트 진행하면서 확인하니 `pydantic`의 `BaseModel`을 사용하지 않으면 해당 값을 request body로 인식하지 못하고 query parameter로 받아들이는 문제가 있다. 
