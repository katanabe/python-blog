# python-blog

FastAPI + React SPA のブログアプリ。

## 技術スタック

- **Backend**: FastAPI, SQLModel, SQLite, JWT認証
- **Frontend**: React 19, TypeScript, Vite, TanStack Query, React Router

## セットアップ

### 前提

- [mise](https://mise.jdx.dev/) がインストールされていること

### Backend

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
pre-commit install
```

### Frontend

```bash
cd frontend
npm install
```

## 開発サーバー起動

```bash
# Backend (ポート 8000)
cd backend && source .venv/bin/activate
uvicorn app.main:app --reload

# Frontend (ポート 5173)
cd frontend
npm run dev
```

API ドキュメント: http://localhost:8000/docs

## テスト

```bash
cd backend && source .venv/bin/activate
pytest
```
