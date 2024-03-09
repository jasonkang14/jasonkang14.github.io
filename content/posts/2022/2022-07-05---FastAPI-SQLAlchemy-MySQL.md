---
title: FastAPI - SQLAlchemy & MySQL
date: "2022-07-05T21:21:37.121Z"
template: "post"
draft: false
slug: "/fastapi/database-using-mysql"
category: "FastAPI"
tags:
  - "FastAPI"

description: "SQLAlchemy를 사용해서 FastAPI를 MySQL과 연결하는 방법"
---

FastAPI 공식문서의 tutorial에서 [User Guide - SQL Databases](https://fastapi.tiangolo.com/tutorial/sql-databases/)항목을 참고한다. FastAPI 공식문서에 DB연동은 두가지 방법이 소개되어있다. 하나는 [SQLAlchemy](https://www.sqlalchemy.org/)를 사용하는 법이고 다른 하나는 [Peewee](http://docs.peewee-orm.com/en/latest/)를 사용하는 방법이다. 둘중에 고민하다가 예전에 원티드+에서 봤던 영상에 따르면 원티드에서는 SQLAlchemy를 사용한다고 해서, SQLAlchemy로 연동하고자 한다. 

공식문서에서 제시한 대로 파일들을 만들어서 정보를 입력할 생각이다. 차이가 있다면 여기는 `sql_app`이라는 디렉토리를 생성하는데, 나는 이미 `app` 이라는 디렉토리에 `main.py`가 존재하기 때문에, 기존의 `app` 디렉토리 안에 `sql`이라는 디렉토리를 만들어서 데이터베이스 연동과 관련된 코드들을 집어넣으려고 한다. 현재 디렉토리 tree는 아래와 같은 형태이다

```
app
├── __init__.py
├── main.py
├── alembic
│   ├── env.py
│   ├── script.py.mako
│   └── versions
├── alembic.ini
├── routers
│   ├── __init__.py
│   └── users.py
└── sql
    ├── __init__.py
    ├── crud.py
    ├── database.py
    ├── models.py
    └── schemas.py
```

`crud.py`가 가장 위에 있지만 공식문서에 나와있는 순서대로 따라가보기로 한다. 
```python
# database.py
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = f'mysql+mysqlconnector://{MYSQL_USER}:{MYSQL_PASSWORD}@{MYSQL_SERVER_NAME}:3306/{DATABASE_NAME}' 

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, 
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()
```

데이터베이스는 MySQL을 사용할 예정인데, database와 query 공부를 MySQL로 하기위해 MySQL 서버 하나를 [Naver Cloud Platform](https://www.ncloud.com/)에 띄워뒀기 때문이다. f-string 안에 변수들은 import된 곳은 없지만 환경변수이다. 지금은 로컬에서 테스트중이라 하드코딩 해뒀는데 참고하시라고 변수로 적어둔다. 

다음은 `models.py`이다. 공식문서에는 두 개의 테이블을 선언하고 1:N으로 엮는다. 하지만 이전 포스트에서 설명한 것 처럼 사용자 테이블을 먼저 만들고 회원가입/로그인 먼저 구현할 생각이기 때문에 user table만 생성한다. 컬럼도 내가 필요한대로 몇 개 더 추가한다.

```python
# models.py

from sqlalchemy import (
    Boolean,
    Column,
    Date,
    Integer,
    String,
)

from .database import Base

class User(Base):
    __tablename__ = "users"

    id        = Column(Integer, primary_key=True, index=True)
    email     = Column(String, unique=True, index=True)
    password  = Column(String)
    is_active = Column(Boolean, default=True)
    phone     = Column(String, unique=True)
    birth     = Column(Date, unique=True)
    gender    = Column(String, default='M')
```

python은 역시 align해줘야 보기좋다. 다음은 `schemas.py`이다. 회원가입이나 로그인 요청을 받을 때 request body에 들어갈 항목들 정도로 봐주면 된다. [Nest.js](https://nestjs.com/)로 프로젝트를 해봤다면 `dto`와 유사한 개념이라고 생각하시면 되겠다. 사실 굳이 필요는 없는데 [pydantic](https://pydantic-docs.helpmanual.io/)을 사용해서 타입체킹을 하기 위함이다. University College London과 마이크로소프트가 발표한 [논문](https://earlbarr.com/publications/typestudy.pdf)에 따르면, 자바스크립트 기준으로 볼 때 타입스크립트를 사용하면 버그가 15%정도 줄어든다고 한다. 회원가입(CREATE) 기준으로 작성한다.

```python
from pydantic import BaseModel

class UserBase(BaseModel):
    email: str
    phone: str
    birth: str
    gender: str


class UserCreate(UserBase):
    password: str


class User(UserBase):
    id: int
    is_active: bool

    class Config:
        orm_mode = True
```

마지막으로 `crud.py` 이다. 회원가입 기준으로 `create_user`만 작성한다. 비밀번호 로직 등등은 수정할 예정이다. 

```python
# crud.py
from sqlalchemy.orm import Session

from . import models, schemas

def create_user(db: Session, user: schemas.UserCreate):
    fake_hashed_password = user.password + "notreallyhashed"
    db_user = models.User(
        email    = user.email,
        password = fake_hashed_password,
        phone    = user.phone,
        birth    = user.birth,
        gender   = user.gender
    )

    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user
```

이제 이 정보들을 `main.py` 로 import한다.

```python
# main.py

from fastapi import Depends, FastAPI, Request, Response

from .routers import users

from sqlalchemy.orm import Session
from .sql.database import SessionLocal, engine
from .sql import crud, models, schemas

models.Base.metadata.create_all(bind=engine)

app = FastAPI()

app.include_router(users.router)

@app.middleware("http")
async def db_session_middleware(request: Request, call_next):
    response = Response("Internal server error", status_code=500)
    try:
        request.state.db = SessionLocal()
        response = await call_next(request)
    finally:
        request.state.db.close()
    return response


def get_db(request: Request):
    return request.state.db


@app.get("/")
async def root():
    return "Hello World!"
```

`get_db` 함수는 router에 선언된 함수들의 dependency로 추가 될 예정이다. 아마도 나중에 `dependencies.py`파일로 옮길 것 같다. 하지만 이상태로 서버를 실행하면 에러가 발생하는데, 바로 서버에 데이터베이스와 테이블 정보가 없기 때문이다. django에서 migration과 유사한 기능을 하는 파이썬 패키지가 필요하다. 검색하다가 SQLAlchemy와 관련이 있는 것 같은 [Alembic](https://alembic.sqlalchemy.org/en/latest/)을 사용하기로 한다. 

`alembic`사용법은 [다음 포스트](https://jasonkang14.github.io/fastapi/offline-database-migration-with-alembic)에서 다뤄보도록 하겠다.