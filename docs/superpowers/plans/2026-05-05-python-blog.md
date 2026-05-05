# Python Blog Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** FastAPI + SQLite バックエンド / React SPA フロントエンドで認証付きブログアプリを構築する。

**Architecture:** バックエンドはFastAPI REST API（JWT認証、SQLite+SQLModel）、フロントエンドはVite+React SPA（TanStack Query、React Router）。完全に分業された構成で、バックエンドはポート8000、フロントエンドはポート5173で動作する。バックエンドはTDDで実装し、フロントエンドはテストなし。

**Tech Stack:** FastAPI, SQLModel, python-jose, passlib, pytest, React 19, TypeScript, Vite, TanStack Query v5, React Router v7

---

## File Map

```
python-blog/
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app, CORS, router登録
│   │   ├── database.py          # SQLiteエンジン, Sessionファクトリ
│   │   ├── auth.py              # JWT生成/検証, パスワードハッシュ, get_current_user依存
│   │   ├── models/
│   │   │   ├── __init__.py
│   │   │   ├── user.py          # Userテーブル (SQLModel)
│   │   │   └── post.py          # Postテーブル (SQLModel)
│   │   ├── schemas/
│   │   │   ├── __init__.py
│   │   │   ├── auth.py          # RegisterRequest, LoginRequest, TokenResponse
│   │   │   └── post.py          # PostCreate, PostUpdate, PostResponse
│   │   └── routers/
│   │       ├── __init__.py
│   │       ├── auth.py          # POST /auth/register, POST /auth/login
│   │       └── posts.py         # GET/POST /posts, GET/PATCH/DELETE /posts/{id}
│   ├── tests/
│   │   ├── conftest.py          # インメモリSQLite fixture, TestClient fixture
│   │   ├── test_auth.py         # register/login テスト
│   │   └── test_posts.py        # posts CRUD + 権限テスト
│   └── requirements.txt
└── frontend/
    ├── index.html
    ├── vite.config.ts
    ├── tsconfig.json
    ├── package.json
    └── src/
        ├── main.tsx             # QueryClientProvider, StrictMode
        ├── App.tsx              # BrowserRouter, Routes定義
        ├── lib/
        │   ├── api.ts           # fetch wrapper (Authorization header付与)
        │   └── auth.ts          # localStorage token 操作
        ├── components/
        │   └── ProtectedRoute.tsx  # 未認証なら /login にリダイレクト
        └── pages/
            ├── Home.tsx
            ├── Auth/
            │   ├── Login.tsx
            │   └── Register.tsx
            └── Posts/
                ├── Index.tsx
                ├── Show.tsx
                ├── New.tsx
                └── Edit.tsx
```

---

## Task 1: プロジェクトスキャフォールド

**Files:**
- Create: `python-blog/backend/requirements.txt`
- Create: `python-blog/frontend/package.json`
- Create: `python-blog/frontend/tsconfig.json`
- Create: `python-blog/frontend/vite.config.ts`
- Create: `python-blog/frontend/index.html`

- [ ] **Step 1: ディレクトリ構造を作成する**

```bash
mkdir -p python-blog/backend/app/models
mkdir -p python-blog/backend/app/schemas
mkdir -p python-blog/backend/app/routers
mkdir -p python-blog/backend/tests
mkdir -p python-blog/frontend/src/lib
mkdir -p python-blog/frontend/src/components
mkdir -p python-blog/frontend/src/pages/Auth
mkdir -p python-blog/frontend/src/pages/Posts
touch python-blog/backend/app/__init__.py
touch python-blog/backend/app/models/__init__.py
touch python-blog/backend/app/schemas/__init__.py
touch python-blog/backend/app/routers/__init__.py
touch python-blog/backend/tests/__init__.py
```

- [ ] **Step 2: `backend/requirements.txt` を作成する**

```
fastapi==0.111.0
uvicorn[standard]==0.30.1
sqlmodel==0.0.19
python-jose[cryptography]==3.3.0
passlib[bcrypt]==1.7.4
pydantic[email]==2.7.1
pytest==8.2.2
httpx==0.27.0
```

- [ ] **Step 3: Python 仮想環境を作り依存をインストールする**

```bash
cd python-blog/backend
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

- [ ] **Step 4: `frontend/package.json` を作成する**

```json
{
  "name": "python-blog-frontend",
  "version": "0.0.1",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build"
  },
  "dependencies": {
    "react": "^19.0.0",
    "react-dom": "^19.0.0",
    "react-router-dom": "^7.0.0",
    "@tanstack/react-query": "^5.0.0"
  },
  "devDependencies": {
    "@types/react": "^19.0.0",
    "@types/react-dom": "^19.0.0",
    "@vitejs/plugin-react": "^4.0.0",
    "typescript": "^5.0.0",
    "vite": "^5.0.0"
  }
}
```

- [ ] **Step 5: `frontend/tsconfig.json` を作成する**

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "moduleResolution": "bundler",
    "jsx": "react-jsx",
    "strict": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "skipLibCheck": true
  },
  "include": ["src"]
}
```

- [ ] **Step 6: `frontend/vite.config.ts` を作成する**

```typescript
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

export default defineConfig({
  plugins: [react()],
  server: { port: 5173 },
})
```

- [ ] **Step 7: `frontend/index.html` を作成する**

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Python Blog</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

- [ ] **Step 8: フロントエンド依存をインストールする**

```bash
cd python-blog/frontend
npm install
```

- [ ] **Step 9: コミットする**

```bash
cd python-blog && git init && git add . && git commit -m "chore: project scaffold"
```

---

## Task 2: データベースモデル

**Files:**
- Create: `backend/app/database.py`
- Create: `backend/app/models/user.py`
- Create: `backend/app/models/post.py`

- [ ] **Step 1: `app/database.py` を作成する**

```python
from sqlmodel import create_engine, Session, SQLModel

DATABASE_URL = "sqlite:///./blog.db"
engine = create_engine(DATABASE_URL, connect_args={"check_same_thread": False})


def create_db_and_tables() -> None:
    SQLModel.metadata.create_all(engine)


def get_session():
    with Session(engine) as session:
        yield session
```

- [ ] **Step 2: `app/models/user.py` を作成する**

```python
from datetime import datetime
from sqlmodel import SQLModel, Field


class User(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    email: str = Field(unique=True, index=True)
    hashed_password: str
    created_at: datetime = Field(default_factory=datetime.utcnow)
```

- [ ] **Step 3: `app/models/post.py` を作成する**

```python
from datetime import datetime
from sqlmodel import SQLModel, Field


class Post(SQLModel, table=True):
    id: int | None = Field(default=None, primary_key=True)
    title: str
    body: str
    author_id: int = Field(foreign_key="user.id", index=True)
    created_at: datetime = Field(default_factory=datetime.utcnow)
    updated_at: datetime = Field(default_factory=datetime.utcnow)
```

- [ ] **Step 4: コミットする**

```bash
git add backend/app/database.py backend/app/models/
git commit -m "feat: database setup and SQLModel models"
```

---

## Task 3: スキーマ (Pydantic リクエスト/レスポンス型)

**Files:**
- Create: `backend/app/schemas/auth.py`
- Create: `backend/app/schemas/post.py`

- [ ] **Step 1: `app/schemas/auth.py` を作成する**

```python
from pydantic import BaseModel, EmailStr


class RegisterRequest(BaseModel):
    email: EmailStr
    password: str


class LoginRequest(BaseModel):
    email: EmailStr
    password: str


class TokenResponse(BaseModel):
    access_token: str
    token_type: str = "bearer"
```

- [ ] **Step 2: `app/schemas/post.py` を作成する**

```python
from datetime import datetime
from pydantic import BaseModel


class PostCreate(BaseModel):
    title: str
    body: str


class PostUpdate(BaseModel):
    title: str | None = None
    body: str | None = None


class PostResponse(BaseModel):
    id: int
    title: str
    body: str
    author_id: int
    created_at: datetime
    updated_at: datetime
```

- [ ] **Step 3: コミットする**

```bash
git add backend/app/schemas/
git commit -m "feat: Pydantic schemas for auth and posts"
```

---

## Task 4: 認証ユーティリティ

**Files:**
- Create: `backend/app/auth.py`

- [ ] **Step 1: `app/auth.py` を作成する**

```python
from datetime import datetime, timedelta

from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPAuthorizationCredentials, HTTPBearer
from jose import JWTError, jwt
from passlib.context import CryptContext
from sqlmodel import Session

from app.database import get_session
from app.models.user import User

SECRET_KEY = "change-me-in-production"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 60 * 24

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
bearer_scheme = HTTPBearer()


def hash_password(password: str) -> str:
    return pwd_context.hash(password)


def verify_password(plain: str, hashed: str) -> bool:
    return pwd_context.verify(plain, hashed)


def create_access_token(user_id: int) -> str:
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    return jwt.encode({"sub": str(user_id), "exp": expire}, SECRET_KEY, algorithm=ALGORITHM)


def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(bearer_scheme),
    session: Session = Depends(get_session),
) -> User:
    try:
        payload = jwt.decode(credentials.credentials, SECRET_KEY, algorithms=[ALGORITHM])
        user_id = int(payload["sub"])
    except (JWTError, KeyError, ValueError):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid token")
    user = session.get(User, user_id)
    if not user:
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="User not found")
    return user
```

- [ ] **Step 2: コミットする**

```bash
git add backend/app/auth.py
git commit -m "feat: JWT auth utilities and get_current_user dependency"
```

---

## Task 5: テストインフラ (conftest.py)

**Files:**
- Create: `backend/tests/conftest.py`

- [ ] **Step 1: `tests/conftest.py` を作成する**

インメモリSQLiteを使い、テストごとに独立したDBを作る。`get_session` 依存を差し替えることで本番DBに一切触れない。

```python
import pytest
from fastapi.testclient import TestClient
from sqlmodel import Session, SQLModel, StaticPool, create_engine

from app.database import get_session
from app.main import app


@pytest.fixture(name="session")
def session_fixture():
    engine = create_engine(
        "sqlite://",
        connect_args={"check_same_thread": False},
        poolclass=StaticPool,
    )
    SQLModel.metadata.create_all(engine)
    with Session(engine) as session:
        yield session
    SQLModel.metadata.drop_all(engine)


@pytest.fixture(name="client")
def client_fixture(session: Session):
    def override_get_session():
        yield session

    app.dependency_overrides[get_session] = override_get_session
    with TestClient(app) as client:
        yield client
    app.dependency_overrides.clear()
```

- [ ] **Step 2: `app/main.py` を最小限で作成する (テストが import できる状態)**

```python
from fastapi import FastAPI

app = FastAPI(title="Python Blog API")
```

- [ ] **Step 3: コミットする**

```bash
git add backend/tests/conftest.py backend/app/main.py
git commit -m "test: pytest fixtures with in-memory SQLite"
```

---

## Task 6: 認証エンドポイント (TDD)

**Files:**
- Create: `backend/tests/test_auth.py`
- Create: `backend/app/routers/auth.py`
- Modify: `backend/app/main.py`

- [ ] **Step 1: 失敗するテストを書く (`tests/test_auth.py`)**

```python
def test_register_success(client):
    response = client.post("/auth/register", json={"email": "user@example.com", "password": "secret"})
    assert response.status_code == 201
    data = response.json()
    assert "access_token" in data
    assert data["token_type"] == "bearer"


def test_register_duplicate_email(client):
    client.post("/auth/register", json={"email": "user@example.com", "password": "secret"})
    response = client.post("/auth/register", json={"email": "user@example.com", "password": "other"})
    assert response.status_code == 400


def test_login_success(client):
    client.post("/auth/register", json={"email": "user@example.com", "password": "secret"})
    response = client.post("/auth/login", json={"email": "user@example.com", "password": "secret"})
    assert response.status_code == 200
    assert "access_token" in response.json()


def test_login_wrong_password(client):
    client.post("/auth/register", json={"email": "user@example.com", "password": "secret"})
    response = client.post("/auth/login", json={"email": "user@example.com", "password": "wrong"})
    assert response.status_code == 401


def test_login_unknown_email(client):
    response = client.post("/auth/login", json={"email": "noone@example.com", "password": "secret"})
    assert response.status_code == 401
```

- [ ] **Step 2: テストを実行してすべて FAIL することを確認する**

```bash
cd backend && source .venv/bin/activate
pytest tests/test_auth.py -v
```

Expected: ERROR または FAILED (router が未定義)

- [ ] **Step 3: `app/routers/auth.py` を実装する**

SQLModel の `session.select()` を使って User を検索し、登録/ログインを処理する。

```python
from fastapi import APIRouter, Depends, HTTPException, status
from sqlmodel import Session, select

from app.auth import create_access_token, hash_password, verify_password
from app.database import get_session
from app.models.user import User
from app.schemas.auth import LoginRequest, RegisterRequest, TokenResponse

router = APIRouter(prefix="/auth", tags=["auth"])


@router.post("/register", response_model=TokenResponse, status_code=201)
def register(body: RegisterRequest, session: Session = Depends(get_session)):
    existing = session.exec(select(User).where(User.email == body.email)).first()
    if existing:
        raise HTTPException(status_code=status.HTTP_400_BAD_REQUEST, detail="Email already registered")
    user = User(email=body.email, hashed_password=hash_password(body.password))
    session.add(user)
    session.commit()
    session.refresh(user)
    return TokenResponse(access_token=create_access_token(user.id))


@router.post("/login", response_model=TokenResponse)
def login(body: LoginRequest, session: Session = Depends(get_session)):
    user = session.exec(select(User).where(User.email == body.email)).first()
    if not user or not verify_password(body.password, user.hashed_password):
        raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED, detail="Invalid credentials")
    return TokenResponse(access_token=create_access_token(user.id))
```

- [ ] **Step 4: `app/main.py` を更新して auth ルーターを登録する**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.database import create_db_and_tables
from app.routers import auth

app = FastAPI(title="Python Blog API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.on_event("startup")
def on_startup() -> None:
    create_db_and_tables()


app.include_router(auth.router)
```

- [ ] **Step 5: テストを実行してすべて PASS することを確認する**

```bash
pytest tests/test_auth.py -v
```

Expected: 5 passed

- [ ] **Step 6: コミットする**

```bash
git add backend/
git commit -m "feat: auth endpoints (register, login) with TDD"
```

---

## Task 7: Posts エンドポイント (TDD)

**Files:**
- Create: `backend/tests/test_posts.py`
- Create: `backend/app/routers/posts.py`
- Modify: `backend/app/main.py`

- [ ] **Step 1: 失敗するテストを書く (`tests/test_posts.py`)**

```python
import pytest


@pytest.fixture
def auth_headers(client):
    client.post("/auth/register", json={"email": "author@example.com", "password": "secret"})
    res = client.post("/auth/login", json={"email": "author@example.com", "password": "secret"})
    return {"Authorization": f"Bearer {res.json()['access_token']}"}


@pytest.fixture
def other_headers(client):
    client.post("/auth/register", json={"email": "other@example.com", "password": "secret"})
    res = client.post("/auth/login", json={"email": "other@example.com", "password": "secret"})
    return {"Authorization": f"Bearer {res.json()['access_token']}"}


def test_list_posts_empty(client):
    response = client.get("/posts")
    assert response.status_code == 200
    assert response.json() == []


def test_create_post(client, auth_headers):
    response = client.post("/posts", json={"title": "Hello", "body": "World"}, headers=auth_headers)
    assert response.status_code == 201
    data = response.json()
    assert data["title"] == "Hello"
    assert data["body"] == "World"
    assert "id" in data
    assert "author_id" in data


def test_create_post_unauthorized(client):
    response = client.post("/posts", json={"title": "Hello", "body": "World"})
    assert response.status_code == 403


def test_get_post(client, auth_headers):
    post = client.post("/posts", json={"title": "Hello", "body": "World"}, headers=auth_headers).json()
    response = client.get(f"/posts/{post['id']}")
    assert response.status_code == 200
    assert response.json()["title"] == "Hello"


def test_get_post_not_found(client):
    response = client.get("/posts/9999")
    assert response.status_code == 404


def test_list_posts(client, auth_headers):
    client.post("/posts", json={"title": "First", "body": "Body1"}, headers=auth_headers)
    client.post("/posts", json={"title": "Second", "body": "Body2"}, headers=auth_headers)
    response = client.get("/posts")
    assert response.status_code == 200
    assert len(response.json()) == 2


def test_update_post(client, auth_headers):
    post = client.post("/posts", json={"title": "Hello", "body": "World"}, headers=auth_headers).json()
    response = client.patch(f"/posts/{post['id']}", json={"title": "Updated"}, headers=auth_headers)
    assert response.status_code == 200
    assert response.json()["title"] == "Updated"
    assert response.json()["body"] == "World"


def test_update_post_forbidden(client, auth_headers, other_headers):
    post = client.post("/posts", json={"title": "Hello", "body": "World"}, headers=auth_headers).json()
    response = client.patch(f"/posts/{post['id']}", json={"title": "Hack"}, headers=other_headers)
    assert response.status_code == 403


def test_delete_post(client, auth_headers):
    post = client.post("/posts", json={"title": "Hello", "body": "World"}, headers=auth_headers).json()
    response = client.delete(f"/posts/{post['id']}", headers=auth_headers)
    assert response.status_code == 204
    assert client.get(f"/posts/{post['id']}").status_code == 404


def test_delete_post_forbidden(client, auth_headers, other_headers):
    post = client.post("/posts", json={"title": "Hello", "body": "World"}, headers=auth_headers).json()
    response = client.delete(f"/posts/{post['id']}", headers=other_headers)
    assert response.status_code == 403
```

- [ ] **Step 2: テストを実行してすべて FAIL することを確認する**

```bash
pytest tests/test_posts.py -v
```

Expected: ERROR または FAILED (router が未定義)

- [ ] **Step 3: `app/routers/posts.py` を実装する**

```python
from datetime import datetime

from fastapi import APIRouter, Depends, HTTPException, status
from sqlmodel import Session, select

from app.auth import get_current_user
from app.database import get_session
from app.models.post import Post
from app.models.user import User
from app.schemas.post import PostCreate, PostResponse, PostUpdate

router = APIRouter(prefix="/posts", tags=["posts"])


@router.get("", response_model=list[PostResponse])
def list_posts(session: Session = Depends(get_session)):
    return session.exec(select(Post)).all()


@router.get("/{post_id}", response_model=PostResponse)
def get_post(post_id: int, session: Session = Depends(get_session)):
    post = session.get(Post, post_id)
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Post not found")
    return post


@router.post("", response_model=PostResponse, status_code=201)
def create_post(
    body: PostCreate,
    session: Session = Depends(get_session),
    current_user: User = Depends(get_current_user),
):
    post = Post(**body.model_dump(), author_id=current_user.id)
    session.add(post)
    session.commit()
    session.refresh(post)
    return post


@router.patch("/{post_id}", response_model=PostResponse)
def update_post(
    post_id: int,
    body: PostUpdate,
    session: Session = Depends(get_session),
    current_user: User = Depends(get_current_user),
):
    post = session.get(Post, post_id)
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Post not found")
    if post.author_id != current_user.id:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Not authorized")
    for key, value in body.model_dump(exclude_unset=True).items():
        setattr(post, key, value)
    post.updated_at = datetime.utcnow()
    session.add(post)
    session.commit()
    session.refresh(post)
    return post


@router.delete("/{post_id}", status_code=204)
def delete_post(
    post_id: int,
    session: Session = Depends(get_session),
    current_user: User = Depends(get_current_user),
):
    post = session.get(Post, post_id)
    if not post:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail="Post not found")
    if post.author_id != current_user.id:
        raise HTTPException(status_code=status.HTTP_403_FORBIDDEN, detail="Not authorized")
    session.delete(post)
    session.commit()
```

- [ ] **Step 4: `app/main.py` に posts ルーターを追加する**

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware

from app.database import create_db_and_tables
from app.routers import auth, posts

app = FastAPI(title="Python Blog API")

app.add_middleware(
    CORSMiddleware,
    allow_origins=["http://localhost:5173"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)


@app.on_event("startup")
def on_startup() -> None:
    create_db_and_tables()


app.include_router(auth.router)
app.include_router(posts.router)
```

- [ ] **Step 5: テストを全件実行して PASS することを確認する**

```bash
pytest tests/ -v
```

Expected: 14 passed (auth 5 + posts 9)

- [ ] **Step 6: 開発サーバーを起動して Swagger UI を確認する**

```bash
uvicorn app.main:app --reload
# ブラウザで http://localhost:8000/docs を開く
```

- [ ] **Step 7: コミットする**

```bash
git add backend/
git commit -m "feat: posts CRUD endpoints with TDD"
```

---

## Task 8: フロントエンド — API クライアント + 認証ユーティリティ

**Files:**
- Create: `frontend/src/lib/auth.ts`
- Create: `frontend/src/lib/api.ts`

- [ ] **Step 1: `src/lib/auth.ts` を作成する**

```typescript
const TOKEN_KEY = "token"

export function getToken(): string | null {
  return localStorage.getItem(TOKEN_KEY)
}

export function setToken(token: string): void {
  localStorage.setItem(TOKEN_KEY, token)
}

export function removeToken(): void {
  localStorage.removeItem(TOKEN_KEY)
}

export function isAuthenticated(): boolean {
  return !!getToken()
}
```

- [ ] **Step 2: `src/lib/api.ts` を作成する**

```typescript
import { getToken } from "./auth"

const BASE_URL = "http://localhost:8000"

export class ApiError extends Error {
  constructor(
    public status: number,
    message: string,
  ) {
    super(message)
  }
}

export async function apiFetch<T>(path: string, options: RequestInit = {}): Promise<T> {
  const token = getToken()
  const headers: HeadersInit = {
    "Content-Type": "application/json",
    ...(token ? { Authorization: `Bearer ${token}` } : {}),
    ...(options.headers ?? {}),
  }
  const res = await fetch(`${BASE_URL}${path}`, { ...options, headers })
  if (!res.ok) {
    const body = await res.json().catch(() => ({ detail: res.statusText }))
    throw new ApiError(res.status, body.detail ?? res.statusText)
  }
  if (res.status === 204) return undefined as T
  return res.json() as Promise<T>
}
```

- [ ] **Step 3: コミットする**

```bash
git add frontend/src/lib/
git commit -m "feat: frontend API client and auth utilities"
```

---

## Task 9: フロントエンド — ルーティング + ProtectedRoute

**Files:**
- Create: `frontend/src/components/ProtectedRoute.tsx`
- Create: `frontend/src/App.tsx`
- Create: `frontend/src/main.tsx`

- [ ] **Step 1: `src/components/ProtectedRoute.tsx` を作成する**

```typescript
import { Navigate, Outlet } from "react-router-dom"
import { isAuthenticated } from "../lib/auth"

export default function ProtectedRoute() {
  return isAuthenticated() ? <Outlet /> : <Navigate to="/login" replace />
}
```

- [ ] **Step 2: `src/App.tsx` を作成する**

`/posts/new` は `/:id` より前に定義して literal マッチを優先する。

```typescript
import { BrowserRouter, Routes, Route } from "react-router-dom"
import ProtectedRoute from "./components/ProtectedRoute"
import Home from "./pages/Home"
import Login from "./pages/Auth/Login"
import Register from "./pages/Auth/Register"
import PostsIndex from "./pages/Posts/Index"
import PostsShow from "./pages/Posts/Show"
import PostsNew from "./pages/Posts/New"
import PostsEdit from "./pages/Posts/Edit"

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/login" element={<Login />} />
        <Route path="/register" element={<Register />} />
        <Route path="/posts" element={<PostsIndex />} />
        <Route path="/posts/:id" element={<PostsShow />} />
        <Route element={<ProtectedRoute />}>
          <Route path="/posts/new" element={<PostsNew />} />
          <Route path="/posts/:id/edit" element={<PostsEdit />} />
        </Route>
      </Routes>
    </BrowserRouter>
  )
}
```

- [ ] **Step 3: `src/main.tsx` を作成する**

```typescript
import { StrictMode } from "react"
import { createRoot } from "react-dom/client"
import { QueryClient, QueryClientProvider } from "@tanstack/react-query"
import App from "./App"

const queryClient = new QueryClient()

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
    </QueryClientProvider>
  </StrictMode>,
)
```

- [ ] **Step 4: コミットする**

```bash
git add frontend/src/
git commit -m "feat: frontend routing and ProtectedRoute"
```

---

## Task 10: 認証ページ (Login / Register)

**Files:**
- Create: `frontend/src/pages/Auth/Register.tsx`
- Create: `frontend/src/pages/Auth/Login.tsx`

- [ ] **Step 1: `src/pages/Auth/Register.tsx` を作成する**

```typescript
import { useState } from "react"
import { useNavigate, Link } from "react-router-dom"
import { apiFetch } from "../../lib/api"
import { setToken } from "../../lib/auth"

export default function Register() {
  const navigate = useNavigate()
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [error, setError] = useState("")

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setError("")
    try {
      const data = await apiFetch<{ access_token: string }>("/auth/register", {
        method: "POST",
        body: JSON.stringify({ email, password }),
      })
      setToken(data.access_token)
      navigate("/posts")
    } catch (err) {
      setError(err instanceof Error ? err.message : "Registration failed")
    }
  }

  return (
    <div>
      <h1>Register</h1>
      {error && <p style={{ color: "red" }}>{error}</p>}
      <form onSubmit={handleSubmit}>
        <div>
          <label>Email<br />
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
          </label>
        </div>
        <div>
          <label>Password<br />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} required />
          </label>
        </div>
        <button type="submit">Register</button>
      </form>
      <p>Already have an account? <Link to="/login">Login</Link></p>
    </div>
  )
}
```

- [ ] **Step 2: `src/pages/Auth/Login.tsx` を作成する**

```typescript
import { useState } from "react"
import { useNavigate, Link } from "react-router-dom"
import { apiFetch } from "../../lib/api"
import { setToken } from "../../lib/auth"

export default function Login() {
  const navigate = useNavigate()
  const [email, setEmail] = useState("")
  const [password, setPassword] = useState("")
  const [error, setError] = useState("")

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setError("")
    try {
      const data = await apiFetch<{ access_token: string }>("/auth/login", {
        method: "POST",
        body: JSON.stringify({ email, password }),
      })
      setToken(data.access_token)
      navigate("/posts")
    } catch (err) {
      setError(err instanceof Error ? err.message : "Login failed")
    }
  }

  return (
    <div>
      <h1>Login</h1>
      {error && <p style={{ color: "red" }}>{error}</p>}
      <form onSubmit={handleSubmit}>
        <div>
          <label>Email<br />
            <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} required />
          </label>
        </div>
        <div>
          <label>Password<br />
            <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} required />
          </label>
        </div>
        <button type="submit">Login</button>
      </form>
      <p>No account? <Link to="/register">Register</Link></p>
    </div>
  )
}
```

- [ ] **Step 3: コミットする**

```bash
git add frontend/src/pages/Auth/
git commit -m "feat: login and register pages"
```

---

## Task 11: Posts ページ

全ページで共通の型:
```typescript
type Post = {
  id: number
  title: string
  body: string
  author_id: number
  created_at: string
  updated_at: string
}
```

**Files:**
- Create: `frontend/src/pages/Home.tsx`
- Create: `frontend/src/pages/Posts/Index.tsx`
- Create: `frontend/src/pages/Posts/Show.tsx`
- Create: `frontend/src/pages/Posts/New.tsx`
- Create: `frontend/src/pages/Posts/Edit.tsx`

- [ ] **Step 1: `src/pages/Home.tsx` を作成する**

```typescript
import { Link } from "react-router-dom"
import { isAuthenticated } from "../lib/auth"

export default function Home() {
  return (
    <div>
      <h1>Python Blog</h1>
      <nav>
        <Link to="/posts">Posts</Link>
        {" | "}
        {isAuthenticated() ? (
          <Link to="/posts/new">New Post</Link>
        ) : (
          <Link to="/login">Login</Link>
        )}
      </nav>
    </div>
  )
}
```

- [ ] **Step 2: `src/pages/Posts/Index.tsx` を作成する**

```typescript
import { useQuery } from "@tanstack/react-query"
import { Link } from "react-router-dom"
import { apiFetch } from "../../lib/api"
import { isAuthenticated } from "../../lib/auth"

type Post = { id: number; title: string; body: string; author_id: number; created_at: string; updated_at: string }

export default function PostsIndex() {
  const { data: posts, isLoading, error } = useQuery<Post[]>({
    queryKey: ["posts"],
    queryFn: () => apiFetch<Post[]>("/posts"),
  })

  if (isLoading) return <p>Loading...</p>
  if (error) return <p>Error: {error.message}</p>

  return (
    <div>
      <h1>Posts</h1>
      {isAuthenticated() && <Link to="/posts/new">New Post</Link>}
      <ul>
        {posts?.map((post) => (
          <li key={post.id}>
            <Link to={`/posts/${post.id}`}>{post.title}</Link>
          </li>
        ))}
      </ul>
      <Link to="/">Home</Link>
    </div>
  )
}
```

- [ ] **Step 3: `src/pages/Posts/Show.tsx` を作成する**

```typescript
import { useQuery, useMutation, useQueryClient } from "@tanstack/react-query"
import { useParams, useNavigate, Link } from "react-router-dom"
import { apiFetch } from "../../lib/api"
import { isAuthenticated } from "../../lib/auth"

type Post = { id: number; title: string; body: string; author_id: number; created_at: string; updated_at: string }

export default function PostsShow() {
  const { id } = useParams<{ id: string }>()
  const navigate = useNavigate()
  const queryClient = useQueryClient()

  const { data: post, isLoading, error } = useQuery<Post>({
    queryKey: ["posts", id],
    queryFn: () => apiFetch<Post>(`/posts/${id}`),
  })

  const deleteMutation = useMutation({
    mutationFn: () => apiFetch(`/posts/${id}`, { method: "DELETE" }),
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] })
      navigate("/posts")
    },
  })

  if (isLoading) return <p>Loading...</p>
  if (error) return <p>Error: {error.message}</p>
  if (!post) return <p>Not found</p>

  return (
    <div>
      <h1>{post.title}</h1>
      <p>{post.body}</p>
      <Link to="/posts">Back</Link>
      {isAuthenticated() && (
        <>
          {" | "}<Link to={`/posts/${post.id}/edit`}>Edit</Link>
          {" | "}<button onClick={() => deleteMutation.mutate()}>Delete</button>
        </>
      )}
    </div>
  )
}
```

- [ ] **Step 4: `src/pages/Posts/New.tsx` を作成する**

```typescript
import { useState } from "react"
import { useNavigate } from "react-router-dom"
import { useMutation, useQueryClient } from "@tanstack/react-query"
import { apiFetch } from "../../lib/api"

type Post = { id: number; title: string; body: string; author_id: number; created_at: string; updated_at: string }

export default function PostsNew() {
  const navigate = useNavigate()
  const queryClient = useQueryClient()
  const [title, setTitle] = useState("")
  const [body, setBody] = useState("")

  const createMutation = useMutation({
    mutationFn: (data: { title: string; body: string }) =>
      apiFetch<Post>("/posts", { method: "POST", body: JSON.stringify(data) }),
    onSuccess: (post) => {
      queryClient.invalidateQueries({ queryKey: ["posts"] })
      navigate(`/posts/${post.id}`)
    },
  })

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    createMutation.mutate({ title, body })
  }

  return (
    <div>
      <h1>New Post</h1>
      {createMutation.error && <p style={{ color: "red" }}>{createMutation.error.message}</p>}
      <form onSubmit={handleSubmit}>
        <div>
          <label>Title<br />
            <input value={title} onChange={(e) => setTitle(e.target.value)} required />
          </label>
        </div>
        <div>
          <label>Body<br />
            <textarea value={body} onChange={(e) => setBody(e.target.value)} required />
          </label>
        </div>
        <button type="submit" disabled={createMutation.isPending}>Create</button>
      </form>
    </div>
  )
}
```

- [ ] **Step 5: `src/pages/Posts/Edit.tsx` を作成する**

```typescript
import { useState, useEffect } from "react"
import { useNavigate, useParams } from "react-router-dom"
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query"
import { apiFetch } from "../../lib/api"

type Post = { id: number; title: string; body: string; author_id: number; created_at: string; updated_at: string }

export default function PostsEdit() {
  const { id } = useParams<{ id: string }>()
  const navigate = useNavigate()
  const queryClient = useQueryClient()
  const [title, setTitle] = useState("")
  const [body, setBody] = useState("")

  const { data: post } = useQuery<Post>({
    queryKey: ["posts", id],
    queryFn: () => apiFetch<Post>(`/posts/${id}`),
  })

  useEffect(() => {
    if (post) {
      setTitle(post.title)
      setBody(post.body)
    }
  }, [post])

  const updateMutation = useMutation({
    mutationFn: (data: { title: string; body: string }) =>
      apiFetch<Post>(`/posts/${id}`, { method: "PATCH", body: JSON.stringify(data) }),
    onSuccess: (updated) => {
      queryClient.invalidateQueries({ queryKey: ["posts"] })
      navigate(`/posts/${updated.id}`)
    },
  })

  function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    updateMutation.mutate({ title, body })
  }

  return (
    <div>
      <h1>Edit Post</h1>
      {updateMutation.error && <p style={{ color: "red" }}>{updateMutation.error.message}</p>}
      <form onSubmit={handleSubmit}>
        <div>
          <label>Title<br />
            <input value={title} onChange={(e) => setTitle(e.target.value)} required />
          </label>
        </div>
        <div>
          <label>Body<br />
            <textarea value={body} onChange={(e) => setBody(e.target.value)} required />
          </label>
        </div>
        <button type="submit" disabled={updateMutation.isPending}>Update</button>
      </form>
    </div>
  )
}
```

- [ ] **Step 6: 両サーバーを起動して動作確認する**

ターミナル1 (backend):
```bash
cd python-blog/backend && source .venv/bin/activate
uvicorn app.main:app --reload
```

ターミナル2 (frontend):
```bash
cd python-blog/frontend && npm run dev
```

ブラウザで `http://localhost:5173` を開き確認:
- `/register` でユーザー登録 → `/posts` にリダイレクト
- `/posts/new` で記事作成 → 詳細ページへ
- `/posts` に一覧表示
- `/posts/:id/edit` で編集
- 削除後 `/posts` に戻る
- 未ログインで `/posts/new` → `/login` にリダイレクト

- [ ] **Step 7: コミットする**

```bash
git add frontend/src/pages/
git commit -m "feat: all post pages (index, show, new, edit)"
```
