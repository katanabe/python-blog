# Python Blog App — Design Spec

Date: 2026-05-05

## Overview

A blog application built with FastAPI (backend REST API) and React SPA (frontend), using SQLite for storage and JWT for authentication. This is the third entry in a series of blog apps exploring different stacks (rails-blog, hono-inertia-blog), and represents the most decoupled frontend/backend architecture of the three.

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | FastAPI, Python 3.12+ |
| ORM | SQLModel (SQLAlchemy + Pydantic unified) |
| Database | SQLite |
| Auth | JWT (python-jose, passlib/bcrypt) |
| Frontend | React 19, TypeScript, Vite |
| Data fetching | TanStack Query v5 |
| Routing | React Router v7 |

## Directory Structure

```
python-blog/
├── backend/
│   ├── app/
│   │   ├── main.py          # FastAPI app, CORS, router registration
│   │   ├── database.py      # SQLite engine, session dependency
│   │   ├── auth.py          # JWT encode/decode, password hashing
│   │   ├── models/
│   │   │   ├── user.py      # User table
│   │   │   └── post.py      # Post table
│   │   ├── schemas/
│   │   │   ├── auth.py      # RegisterRequest, LoginResponse, TokenPayload
│   │   │   └── post.py      # PostCreate, PostUpdate, PostResponse
│   │   └── routers/
│   │       ├── auth.py      # /auth/register, /auth/login
│   │       └── posts.py     # /posts CRUD
│   └── requirements.txt
└── frontend/
    ├── src/
    │   ├── main.tsx
    │   ├── lib/
    │   │   ├── api.ts        # fetch wrapper with auth header
    │   │   └── auth.ts       # token storage (localStorage)
    │   ├── pages/
    │   │   ├── Home.tsx
    │   │   ├── Posts/
    │   │   │   ├── Index.tsx
    │   │   │   ├── Show.tsx
    │   │   │   ├── New.tsx
    │   │   │   └── Edit.tsx
    │   │   └── Auth/
    │   │       ├── Login.tsx
    │   │       └── Register.tsx
    │   └── components/
    │       └── ProtectedRoute.tsx
    ├── package.json
    └── vite.config.ts
```

## Data Models

### User
| Field | Type | Notes |
|-------|------|-------|
| id | int | primary key |
| email | str | unique |
| hashed_password | str | bcrypt |
| created_at | datetime | |

### Post
| Field | Type | Notes |
|-------|------|-------|
| id | int | primary key |
| title | str | |
| body | str | |
| author_id | int | FK → User |
| created_at | datetime | |
| updated_at | datetime | |

## API Endpoints

| Method | Path | Auth | Description |
|--------|------|------|-------------|
| POST | /auth/register | — | ユーザー登録 |
| POST | /auth/login | — | JWT トークン取得 |
| GET | /posts | — | 記事一覧 |
| GET | /posts/{id} | — | 記事詳細 |
| POST | /posts | Bearer | 記事作成 |
| PATCH | /posts/{id} | Bearer | 記事更新（作成者のみ） |
| DELETE | /posts/{id} | Bearer | 記事削除（作成者のみ） |

## Authentication Flow

1. User registers via `POST /auth/register` → password hashed with bcrypt
2. User logs in via `POST /auth/login` → receives JWT access token
3. Frontend stores token in `localStorage`
4. Authenticated requests send `Authorization: Bearer <token>` header
5. FastAPI dependency `get_current_user` decodes JWT and injects user into route handlers
6. Edit/delete endpoints check `post.author_id == current_user.id`

## Frontend Routing

```
/               → Home (公開)
/posts          → Posts/Index (公開)
/posts/:id      → Posts/Show (公開)
/posts/new      → Posts/New (要ログイン)
/posts/:id/edit → Posts/Edit (要ログイン)
/login          → Auth/Login
/register       → Auth/Register
```

`ProtectedRoute` コンポーネントがトークン有無を確認し、未ログインなら `/login` にリダイレクト。

## Error Handling

- バリデーションエラー: FastAPI の自動 422 レスポンス (Pydantic)
- 認証エラー: 401 Unauthorized
- 権限エラー: 403 Forbidden
- Not Found: 404

## Development Setup

- Backend: `uvicorn app.main:app --reload` (port 8000)
- Frontend: `vite dev` (port 5173)
- CORS: backend が `localhost:5173` を許可
- Swagger UI: `http://localhost:8000/docs` で自動生成

## Testing Strategy (TDD)

実装はテストを先に書いてから本実装する TDD フローで進める。

### Backend (pytest)
- `pytest` + `httpx` (AsyncClient) でエンドポイントのインテグレーションテスト
- テスト用 SQLite はインメモリ (`:memory:`) を使い、各テストで独立
- テスト対象:
  - 認証: register/login の正常系・異常系
  - posts CRUD: 一覧・詳細・作成・更新・削除
  - 権限: 他ユーザーの記事を編集・削除できないこと

### TDD フロー
```
テストを書く (RED) → 実装する (GREEN) → リファクタ (REFACTOR)
```
各エンドポイント・コンポーネントをこの順で実装する。

## Out of Scope

- 画像アップロード
- コメント機能
- ページネーション
- メール認証
- リフレッシュトークン
