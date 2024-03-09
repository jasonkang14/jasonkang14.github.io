---
title: FastAPI - 회원가입
date: "2022-07-17T21:21:37.121Z"
template: "post"
draft: false
slug: "/fastapi/signup-view"
category: "FastAPI"
tags:
  - "FastAPI"

description: "FastAPI로 회원가입을 진행한다"
---

디렉토리 구조도 잡았으니 이제 본격적으로 API를 만들어본다. 
[FastAPI 디렉토리 구조](https://jasonkang14.github.io/fastapi/directory-structure)에서 언급한 것처럼, `pydantic`의 `BaseModel`만 request body로 인식하기 때문에, `schemas/user.py`에서 request body의 type을 먼저 설정한다

```python
# schemas/user.py
from pydantic import BaseModel

class UserCreate(BaseModel):
    email: str
    password: str
    gender: str
    phone: str
    birth: str
```

사이드 목적상 많은 개인정보는 필요없고, 나이와 성별정도로만 데이터를 저장해보려고한다. 지역구를 저장하는게 의미가 있을 수도 있지만, 정확하게 받아올 수 있는 정보가 아니니 생략한다.

`routers/user.py`에 클라이언트로부터 request를 받을 인터페이스를 설정한다.
```python
# routers/user.py

from fastapi             import APIRouter, Depends, HTTPException
from sqlalchemy.orm      import Session

from schemas   import user as user_schema

from .deps import get_db, Message

# 각 router/XXX.py에 선언한 router들을 routers.py에서 import한 후, 해당 router를 main.py에서 import한다.
router = APIRouter()

# 아래와 같이 response 종류들을 선언해줘야 swagger에 나타난다.
signup_responses = {
    201: {'model': Message, 'description': '회원가입 성공'},
    409: {'model': Message, 'description': 'email 또는 phone이 중복되는 경우'},
}


@router.post("/signup", status_code=201, responses={**signup_responses})
def signup(
    signup_info: user_schema.UserCreate,
    db: Session = Depends(get_db),
):
    # bcrypt를 사용해서 비밀번호를 단방향 hashing하는 과정을 거친다. 코드는 공식문서 참고
    # signup_info.password를 덮어쓰는 이유는 crud/user.py에서도 UserCreate schema를 사용하기 때문이다. 
    signup_info.password = get_password_hash(signup_info.password)

    # db에 저장한다.
    create_user(signup_info, db)

    # 특이하게 dicitionary로 리턴해도 정상 작동한다.
    return {'msg': 'SIGNUP_SUCCESS'}
```

이제 sqlalchemy를 사용해서 데이터베이스에 저장하는 과정을 살펴본다. 
```python
# crud/user.py
def create_user(
    signup_info: UserCreate,
    db: Session,
) -> User:
    db_obj = User(
                username = signup_info.username,
                password = signup_info.password,
                phone    = signup_info.phone,
                gender   = signup_info.gender,
                birth    = signup_info.birth
            )
    db.add(db_obj)
    db.commit()

    return 
```


`db.add()`를 하면 깃헙으로 따지면 스테이징이 되고, `db.commit()`을 해야 실제로 디비에 `INSERT`된다. 공식문서 예제에서는 `.commit()` 후에 `.refresh()`를 해준다.
`.refresh()`를 하는 이유는, 다른 어딘가에서 데이터베이스에 커밋이 발생했을 때, 내가 원하는 객체가 변경될 수 있기 때문이다(회원가입에는 크게 해당하지 않지만 누군가 갑자기 사용자 정보를 바꾼다던지의 문제)
그렇다면 해당 객체가 내가 원하는 상태의 객체가 아닐 수 있기 때문에 데이터베이스를 read해서 최신화 하는 것이다. 
회원가입 후 사용자 정보를 따로 클라이언트에 리턴하는 절차가 없기 때문이 refresh는 생략한다.

curl로 시도해보면 잘 작동하는 것을 볼 수 있다.
![signup-via-curl](https://i.imgur.com/eBZM75g.png) 

서버 로그도 잘 찍히는 것을 볼 수 있다.
![signup-success-from-uvicorn](https://i.imgur.com/laT8gEb.png)

이제 API가 하나 만들어졌으니, 클라이언트를 만들어보도록 하겠다.