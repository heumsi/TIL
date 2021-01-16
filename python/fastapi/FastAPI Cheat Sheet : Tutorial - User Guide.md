# FastAPI Cheat Sheet : Tutorial - User Guide

이 문서는 FastAPI 사용법에 대해 "나 보기 편하라고" 만든 문서입니다.  
공식 문서도 매우 훌륭하지만, "Tutorial - User Guide" 에서 당장 필요한 것들만 가져온 일종의 Cheat Sheet 입니다.  
프로젝트 시작 전, 공부 겸 정리해봅니다.

 FastAPI `0.63.0` 버전을 기반으로 합니다.



## Basic Usage

### Install

```bash
$ pip install fastapi[all]
```



### Quick start

```python
# main.py

from fastapi import FastAPI

app = FastAPI()

@app.get("/")
async def root():
    return {"message": "Hello World"}
```

```bash
$ uvicorn main:app --reload
```



## Request 를 받는 법



### Parameters

별도의 Request Body 없이 url 로만 요청을 전달하는 경우입니다.  
이 경우 Parameter 는 크게 2종류로 구분됩니다.

- Path Parameters
- Query Parameters



#### Path Parameters

```python
# ex. GET /items/1
@app.get("/items/{item_id}")
async def read_item(item_id: int):
```



#### Query Parameters

```python
# ex. GET /items/?skip=0&limit=10
@app.get("/items/")
async def read_item(skip: int = 0, limit: int = 10):
```



#### Path & Query Parameters

```python
# ex. GET /users/0/items/5?short=True
@app.get("/users/{user_id}/items/{item_id}")
async def read_user_item(
    user_id: int,  # Path param
    item_id: str,  # Path param
    q: Optional[str] = None,  # Query param (Optional)
    short: bool  # Query param (Required)
):
```



### Request Body

Reqeust Body 에 데이터를 실어 요청을 보내는 경우입니다.  
주로 `POST` , `PATCH`,  `PUT` 요청에서 사용합니다.



#### Reqeust Body

```python
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None

@app.post("/items/")
async def create_item(item: Item):  # json -> Item 변환
```

```json
ex. POST /items/

{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```



#### Request body + path

```python
@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item):
```

```json
PUT /items/1

{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```



#### Request body + path + query

```python
@app.put("/items/{item_id}")
async def create_item(item_id: int, item: Item, q: Optional[str] = None):
```

```json
PUT /items/1?a=1

{
    "name": "Foo",
    "description": "An optional description",
    "price": 45.2,
    "tax": 3.5
}
```



### Query Parameters

Generic validations and metadata:

- `alias`
- `title`
- `description`
- `deprecated`

Validations specific for strings:

- `min_length`
- `max_length`
- `regex`



#### Query Param

```python
from fastapi import Query

@app.get("/items/")
async def read_items(q: Optional[str] = Query(None, min_length=3, max_length=50, regex="^fixedquery$")):
```

- `Query` 의 첫 번째 인자로 `None` 으로 두면 이 인자는 Optional 하고 기본 값으로 `None` 을 가짐.
- 인자를 Optional 하지 않고 Required 하게 주고 싶으면 다음처럼  `...` 키워드를 사용하면 됨.

```python
async def read_items(q: str = Query(..., min_length=3)):
```



#### Query Param List

```python
# GET /items/?q=foo&q=bar
@app.get("/items/")
async def read_items(q: Optional[List[str]] = Query(None)):  # Query를 사용해야 함.
    # q = ["foo", "bar"] 이 됨.
```



#### Declare more metadata

```python
@app.get("/items/")
async def read_items(
    q: Optional[str] = Query(
        None,
        title="Query string",
        description="Query string for the items to search in the database that have a good match",
        min_length=3,
    )
):
```



#### Alias parameters

`item-query` 같이 파이썬에서 허용이 안되는 변수명이라면 `alias` 를 사용할 수 있다.

```python
# GET /items/?item-query=foobaritems
@app.get("/items/")
async def read_items(q: Optional[str] = Query(None, alias="item-query")):
```



#### Deprecating parameters

`deprecated` 된 파라미터의 경우, 이를 표시할 수 있다.

```python
@app.get("/items/")
async def read_items(
    q: Optional[str] = Query(
        None,
        ...
        deprecated=True,
    )
):
```



### Path Parameters

#### Declare metadata

```python
from fastapi import Path

@app.get("/items/{item_id}")
async def read_items(
    item_id: int = Path(..., title="The ID of the item to get"), 
    # `Path` 의 첫 번째로 인자로 항상 `...` 를 줘야 함.
):
```



#### Number validations

```python
from fastapi import Path

@app.get("/items/{item_id}")
async def read_items(
    *,
    item_id: int = Path(..., title="The ID of the item to get", gt=0, le=1000),
    q: str,
):
```



### Request Body

#### Multiple body parameters

```python
class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None


class User(BaseModel):
    username: str
    full_name: Optional[str] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item, user: User):
```

위처럼 여러 개의 BodyParameter 가 존재할 경우, RequestBody에 다음처럼 필드 키를 줘야 함.

```json
PUT /items/1

{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    },
    "user": {
        "username": "dave",
        "full_name": "Dave Grohl"
    }
}
```



#### Singular values in body

```python
from fastapi import Body

@app.put("/items/{item_id}")
async def update_item(
    item_id: int, item: Item, user: User, importance: int = Body(..., gt=0)
):
```

```json
PUT /items/{item_id}

{
    "item": {
        ...
    },
    "user": {
        ...
    },
    "importance": 5
}
```



#### Embed a single body parameter

```python
@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item = Body(..., embed=True)):
```

```json
PUT /items/1

{
    "item": {
        "name": "Foo",
        "description": "The pretender",
        "price": 42.0,
        "tax": 3.2
    }
}
```



#### Field (Validation)

```python
from pydantic import BaseModel, Field

class Item(BaseModel):
    name: str
    description: Optional[str] = Field(
        None, title="The description of the item", max_length=300
    )
    price: float = Field(..., gt=0, description="The price must be greater than zero")
    tax: Optional[float] = None
```



#### Nested Models

```python
from typing import List, Optional, Set
from pydantic import BaseModel, HttpUrl

class Image(BaseModel):
    url: HttpUrl
    name: str


class Item(BaseModel):
    name: str
    description: Optional[str] = None
    price: float
    tax: Optional[float] = None
    tags: Set[str] = set()
    images: Optional[List[Image]] = None


@app.put("/items/{item_id}")
async def update_item(item_id: int, item: Item):
```

```json
PUT /items/{item_id}

{
    "name": "Foo",
    "description": "The pretender",
    "price": 42.0,
    "tax": 3.2,
    "tags": [
        "rock",
        "metal",
        "bar"
    ],
    "images": [
        {
            "url": "http://example.com/baz.jpg",
            "name": "The Foo live"
        },
        {
            "url": "http://example.com/dave.jpg",
            "name": "The Baz"
        }
    ]
}
```



## Response 를 내보내는 법



### Response Model

```python
class UserIn(BaseModel):
    username: str
    password: str
    email: EmailStr
    full_name: Optional[str] = None


class UserOut(BaseModel):
    username: str
    email: EmailStr
    full_name: Optional[str] = None


@app.post("/user/", response_model=UserOut)
async def create_user(user: UserIn):
    return user
```



### Status Code

```python
from fastapi import status

@app.post("/items/", status_code=status.HTTP_201_CREATED)
async def create_item(name: str):
```



## 그 외 Request와 Response 에 대한 것들


### Cookie Parameters

```python
from fastapi import Cookie

@app.get("/items/")
async def read_items(ads_id: Optional[str] = Cookie(None)):
    return {"ads_id": ads_id}
```



### Header Parameters

```python
@app.get("/items/")
async def read_items(user_agent: Optional[str] = Header(None)):
```



### Form Data

```bash
# 아래 라이브러리를 먼저 설치해야 함.
$ pip install python-multipart
```

```python
from fastapi import Form

@app.post("/login/")
async def login(username: str = Form(...), password: str = Form(...)):
```



### Request File

```bash
# 아래 라이브러리를 먼저 설치해야 함.
$ pip install python-multipart
```

```python
from fastapi import File, UploadFile

# Single File
@app.post("/files/")
async def create_file(file: bytes = File(...)):
    return {"file_size": len(file)}


@app.post("/uploadfile/")
async def create_upload_file(file: UploadFile = File(...)):
    return {"filename": file.filename}


# Multiple Files
@app.post("/files/")
async def create_files(files: List[bytes] = File(...)):
    return {"file_sizes": [len(file) for file in files]}


@app.post("/uploadfiles/")
async def create_upload_files(files: List[UploadFile] = File(...)):
    return {"filenames": [file.filename for file in files]}
```



### Handling Errors

#### Use HTTPException

```python
from fastapi import HTTPException


@app.get("/items/{item_id}")
async def read_item(item_id: str):
    items = {"foo": "The Foo Wrestlers"}
    if item_id not in items:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"item": items[item_id]}
```



#### Custom Exception Handler

```python
from fastapi import FastAPI, Request
from fastapi.responses import JSONResponse

# Define Exception
class UnicornException(Exception):
    def __init__(self, name: str):
        self.name = name


# Define Exception Handler
@app.exception_handler(UnicornException)
async def unicorn_exception_handler(request: Request, exc: UnicornException):
    return JSONResponse(
        status_code=418,
        content={"message": f"Oops! {exc.name} did something. There goes a rainbow..."},
    )

# Endpoint
@app.get("/unicorns/{name}")
async def read_unicorn(name: str):
    if name == "yolo":
        raise UnicornException(name=name)
    return {"unicorn_name": name}
```



### Path Operation Configuration

#### Response Status Code

```python
from fastapi import status

@app.post("/items/", response_model=Item, status_code=status.HTTP_201_CREATED)
async def create_item(item: Item):
```



#### Tags

OpenAPI 문서에서 태그 단위로 엔드 포인트를 그루핑 시킴

```python
@app.post("/items/", response_model=Item, tags=["items"])
async def create_item(item: Item):
```



#### Summary and description

```python
@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    description="Create an item with all the information, name, description, price, tax and a set of unique tags",
)
async def create_item(item: Item):
```



#### Description from docstring

docstring에다가 써도 됨. OpenAPI 문서에 마크다운 형태로 파싱해서 자동으로 추가됨.

```python
@app.post("/items/", response_model=Item, summary="Create an item")
async def create_item(item: Item):
    """
    Create an item with all the information:

    - **name**: each item must have a name
    - **description**: a long description
    - **price**: required
    - **tax**: if the item doesn't have tax, you can omit this
    - **tags**: a set of unique tag strings for this item
    """
```



#### Response description

```python
@app.post(
    "/items/",
    response_model=Item,
    summary="Create an item",
    response_description="The created item",
)
async def create_item(item: Item):
```



### JSON Compatible Encoder

```python
from fastapi.encoders import jsonable_encoder

@app.put("/items/{id}")
def update_item(id: str, item: Item):
    fake_db = {}
    
    # The result of calling it is something that can be encoded with the Python standard
    json_compatible_item_data = jsonable_encoder(item)
    fake_db[id] = json_compatible_item_data
```



### Body - Updates

```python
items = {
    "foo": {"name": "Foo", "price": 50.2},
    "bar": {"name": "Bar", "description": "The bartenders", "price": 62, "tax": 20.2},
    "baz": {"name": "Baz", "description": None, "price": 50.2, "tax": 10.5, "tags": []},
}

@app.patch("/items/{item_id}", response_model=Item)
async def update_item(item_id: str, item: Item):
    stored_item_data = items[item_id]
    stored_item_model = Item(**stored_item_data)
    update_data = item.dict(exclude_unset=True)
    updated_item = stored_item_model.copy(update=update_data)
    items[item_id] = jsonable_encoder(updated_item)
    return updated_item
```



### Dependencies



#### Depends

```python
from fastapi import Depends

async def common_parameters(q: Optional[str] = None, skip: int = 0, limit: int = 100):
    return {"q": q, "skip": skip, "limit": limit}


@app.get("/items/")
async def read_items(commons: dict = Depends(common_parameters)):
    return commons


@app.get("/users/")
async def read_users(commons: dict = Depends(common_parameters)):
    return commons
```



#### Classes as Dependencies

```python
from fastapi import Depends

class CommonQueryParams:
    def __init__(self, q: Optional[str] = None, skip: int = 0, limit: int = 100):
        self.q = q
        self.skip = skip
        self.limit = limit

@app.get("/items/")
async def read_items(commons: CommonQueryParams = Depends()):
```





## Security

### OAuth2 with Password, Bearer with JWT tokens

```python
from datetime import datetime, timedelta
from typing import Optional

from fastapi import Depends, FastAPI, HTTPException, status
from fastapi.security import OAuth2PasswordBearer, OAuth2PasswordRequestForm
from jose import JWTError, jwt
from passlib.context import CryptContext
from pydantic import BaseModel

# to get a string like this run:
# openssl rand -hex 32
SECRET_KEY = "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30


fake_users_db = {
    "johndoe": {
        "username": "johndoe",
        "full_name": "John Doe",
        "email": "johndoe@example.com",
        "hashed_password": "$2b$12$EixZaYVK1fsbw1ZfbX3OXePaWxn96p36WQoeG6Lruj3vjPGga31lW",
        "disabled": False,
    }
}


class Token(BaseModel):
    access_token: str
    token_type: str


class TokenData(BaseModel):
    username: Optional[str] = None


class User(BaseModel):
    username: str
    email: Optional[str] = None
    full_name: Optional[str] = None
    disabled: Optional[bool] = None


class UserInDB(User):
    hashed_password: str


pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

app = FastAPI()


def verify_password(plain_password, hashed_password):
    return pwd_context.verify(plain_password, hashed_password)


def get_password_hash(password):
    return pwd_context.hash(password)


def get_user(db, username: str):
    if username in db:
        user_dict = db[username]
        return UserInDB(**user_dict)


def authenticate_user(fake_db, username: str, password: str):
    user = get_user(fake_db, username)
    if not user:
        return False
    if not verify_password(password, user.hashed_password):
        return False
    return user


def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt


async def get_current_user(token: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        username: str = payload.get("sub")
        if username is None:
            raise credentials_exception
        token_data = TokenData(username=username)
    except JWTError:
        raise credentials_exception
    user = get_user(fake_users_db, username=token_data.username)
    if user is None:
        raise credentials_exception
    return user


async def get_current_active_user(current_user: User = Depends(get_current_user)):
    if current_user.disabled:
        raise HTTPException(status_code=400, detail="Inactive user")
    return current_user


@app.post("/token", response_model=Token)
async def login_for_access_token(form_data: OAuth2PasswordRequestForm = Depends()):
    user = authenticate_user(fake_users_db, form_data.username, form_data.password)
    if not user:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect username or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    access_token_expires = timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    access_token = create_access_token(
        data={"sub": user.username}, expires_delta=access_token_expires
    )
    return {"access_token": access_token, "token_type": "bearer"}


@app.get("/users/me/", response_model=User)
async def read_users_me(current_user: User = Depends(get_current_active_user)):
    return current_user


@app.get("/users/me/items/")
async def read_own_items(current_user: User = Depends(get_current_active_user)):
    return [{"item_id": "Foo", "owner": current_user.username}]
```



## Middleware

A "middleware" is a function that works with every **request** before it is processed by any specific *path operation*. And also with every **response** before returning it.

```python
@app.middleware("http")
async def add_process_time_header(request: Request, call_next):
    start_time = time.time()
    response = await call_next(request)
    process_time = time.time() - start_time
    response.headers["X-Process-Time"] = str(process_time)
    return response
```



#### CORS

```python
from fastapi.middleware.cors import CORSMiddleware

origins = [
    "http://localhost.tiangolo.com",
    "https://localhost.tiangolo.com",
    "http://localhost",
    "http://localhost:8080",
]

app.add_middleware(
    CORSMiddleware,
    allow_origins=origins,
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)
```



## SQL (Relational) Databases

```python
# sql_app/database.py

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

SQLALCHEMY_DATABASE_URL = "sqlite:///./sql_app.db"
# SQLALCHEMY_DATABASE_URL = "postgresql://user:password@postgresserver/db"

engine = create_engine(
    SQLALCHEMY_DATABASE_URL, connect_args={"check_same_thread": False}
)
SessionLocal = sessionmaker(autocommit=False, autoflush=False, bind=engine)

Base = declarative_base()
```

```python
# sql_app/models.py

from sqlalchemy import Boolean, Column, ForeignKey, Integer, String
from sqlalchemy.orm import relationship

from .database import Base


class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)

    items = relationship("Item", back_populates="owner")


class Item(Base):
    __tablename__ = "items"

    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    description = Column(String, index=True)
    owner_id = Column(Integer, ForeignKey("users.id"))

    owner = relationship("User", back_populates="items")
```

```python
# sql_app/schemas.py

from typing import List, Optional

from pydantic import BaseModel


class ItemBase(BaseModel):
    title: str
    description: Optional[str] = None


class ItemCreate(ItemBase):
    pass


class Item(ItemBase):
    id: int
    owner_id: int

    class Config:
        orm_mode = True


class UserBase(BaseModel):
    email: str


class UserCreate(UserBase):
    password: str


class User(UserBase):
    id: int
    is_active: bool
    items: List[Item] = []

    class Config:
        orm_mode = True
```

```python
# sql_app/crud.py

from sqlalchemy.orm import Session

from . import models, schemas


def get_user(db: Session, user_id: int):
    return db.query(models.User).filter(models.User.id == user_id).first()


def get_user_by_email(db: Session, email: str):
    return db.query(models.User).filter(models.User.email == email).first()


def get_users(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.User).offset(skip).limit(limit).all()


def create_user(db: Session, user: schemas.UserCreate):
    fake_hashed_password = user.password + "notreallyhashed"
    db_user = models.User(email=user.email, hashed_password=fake_hashed_password)
    db.add(db_user)
    db.commit()
    db.refresh(db_user)
    return db_user


def get_items(db: Session, skip: int = 0, limit: int = 100):
    return db.query(models.Item).offset(skip).limit(limit).all()


def create_user_item(db: Session, item: schemas.ItemCreate, user_id: int):
    db_item = models.Item(**item.dict(), owner_id=user_id)
    db.add(db_item)
    db.commit()
    db.refresh(db_item)
    return db_item
```

```python
# sql_app/main.py

from typing import List

from fastapi import Depends, FastAPI, HTTPException
from sqlalchemy.orm import Session

from . import crud, models, schemas
from .database import SessionLocal, engine

models.Base.metadata.create_all(bind=engine)

app = FastAPI()


# Dependency
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()


@app.post("/users/", response_model=schemas.User)
def create_user(user: schemas.UserCreate, db: Session = Depends(get_db)):
    db_user = crud.get_user_by_email(db, email=user.email)
    if db_user:
        raise HTTPException(status_code=400, detail="Email already registered")
    return crud.create_user(db=db, user=user)


@app.get("/users/", response_model=List[schemas.User])
def read_users(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    users = crud.get_users(db, skip=skip, limit=limit)
    return users


@app.get("/users/{user_id}", response_model=schemas.User)
def read_user(user_id: int, db: Session = Depends(get_db)):
    db_user = crud.get_user(db, user_id=user_id)
    if db_user is None:
        raise HTTPException(status_code=404, detail="User not found")
    return db_user


@app.post("/users/{user_id}/items/", response_model=schemas.Item)
def create_item_for_user(
    user_id: int, item: schemas.ItemCreate, db: Session = Depends(get_db)
):
    return crud.create_user_item(db=db, item=item, user_id=user_id)


@app.get("/items/", response_model=List[schemas.Item])
def read_items(skip: int = 0, limit: int = 100, db: Session = Depends(get_db)):
    items = crud.get_items(db, skip=skip, limit=limit)
    return items
```



## Bigger Applications - Multiple Files

### `APIRouter`

```python
from fastapi import APIRouter

router = APIRouter()


@router.get("/users/", tags=["users"])
async def read_users():
    return [{"username": "Rick"}, {"username": "Morty"}]


@router.get("/users/me", tags=["users"])
async def read_user_me():
    return {"username": "fakecurrentuser"}


@router.get("/users/{username}", tags=["users"])
async def read_user(username: str):
    return {"username": username}
```



### Another module with `APIRouter`

```python
from fastapi import APIRouter, Depends, HTTPException

from ..dependencies import get_token_header

router = APIRouter(
    prefix="/items",
    tags=["items"],
    dependencies=[Depends(get_token_header)],
    responses={404: {"description": "Not found"}},
)


fake_items_db = {"plumbus": {"name": "Plumbus"}, "gun": {"name": "Portal Gun"}}


@router.get("/")
async def read_items():
    return fake_items_db


@router.get("/{item_id}")
async def read_item(item_id: str):
    if item_id not in fake_items_db:
        raise HTTPException(status_code=404, detail="Item not found")
    return {"name": fake_items_db[item_id]["name"], "item_id": item_id}
```



### The main `FastAPI`

```python
from fastapi import Depends, FastAPI

from .dependencies import get_query_token, get_token_header
from .internal import admin
from .routers import items, users

app = FastAPI(dependencies=[Depends(get_query_token)])


app.include_router(users.router)
app.include_router(items.router)
app.include_router(
    admin.router,
    prefix="/admin",
    tags=["admin"],
    dependencies=[Depends(get_token_header)],
    responses={418: {"description": "I'm a teapot"}},
)


@app.get("/")
async def root():
    return {"message": "Hello Bigger Applications!"}
```



## Background Tasks

You can define background tasks to be run *after* returning a response.

```python
from fastapi import BackgroundTasks, FastAPI

app = FastAPI()


def write_notification(email: str, message=""):
    with open("log.txt", mode="w") as email_file:
        content = f"notification for {email}: {message}"
        email_file.write(content)


@app.post("/send-notification/{email}")
async def send_notification(email: str, background_tasks: BackgroundTasks):
    background_tasks.add_task(write_notification, email, message="some notification")
    return {"message": "Notification sent in the background"}
```



## Metadata and Docs URLs

You can customize several metadata configurations in your **FastAPI** application.



### Title, description, and version

```python
from fastapi import FastAPI

app = FastAPI(
    title="My Super Project",
    description="This is a very fancy project, with auto docs for the API and everything",
    version="2.5.0",
)
```



### Metadata for tags

You can also add additional metadata for the different tags used to group your path operations with the parameter `openapi_tags`.

```python
from fastapi import FastAPI

tags_metadata = [
    {
        "name": "users",
        "description": "Operations with users. The **login** logic is also here.",
    },
    {
        "name": "items",
        "description": "Manage items. So _fancy_ they have their own docs.",
        "externalDocs": {
            "description": "Items external docs",
            "url": "https://fastapi.tiangolo.com/",
        },
    },
]

app = FastAPI(openapi_tags=tags_metadata)


@app.get("/users/", tags=["users"])
async def get_users():
    return [{"name": "Harry"}, {"name": "Ron"}]


@app.get("/items/", tags=["items"])
async def get_items():
    return [{"name": "wand"}, {"name": "flying broom"}]
```



## Testing

```python
from fastapi import FastAPI
from fastapi.testclient import TestClient

app = FastAPI()


@app.get("/")
async def read_main():
    return {"msg": "Hello World"}


client = TestClient(app)


def test_read_main():
    response = client.get("/")
    assert response.status_code == 200
    assert response.json() == {"msg": "Hello World"}
```



## Debugging

```python
import uvicorn
from fastapi import FastAPI

app = FastAPI()

if __name__ == "__main__":
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

