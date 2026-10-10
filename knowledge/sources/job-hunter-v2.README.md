# Job Hunter

AI 驅動的求職助手 — 產生客製化履歷、模擬行為面試錄影練習、取得 AI 回饋。

AI-powered job hunting assistant — generate tailored resumes, practice behavioral interviews with video recording, and get AI-driven feedback.

## Features / 功能

- **認證系統** — Email 註冊登入 + JWT 認證
- **個人檔案** — 管理學歷、經歷、技能、作品集
- **職缺追蹤** — 手動新增或 URL 解析職缺
- **面試練習** — 行為面試模擬 + 瀏覽器錄影
  - 題庫模式（30+ 精選題目）或根據 JD 由 AI 動態產生
  - 沉浸式全螢幕，限時讀題 + 倒數計時
  - 錄製、預覽、重錄答題影片
  - 回顧歷史練習，48 小時內可回放錄影
- **AI 履歷產生** — 根據個人背景與目標職缺產生客製化履歷 *(規劃中)*
- **AI 面試評估** — 語音轉文字 + AI 回饋分析 *(規劃中)*

## Tech Stack / 技術棧

| 層級 | 技術 |
|------|------|
| **前端** | React 19, TypeScript, Vite, Tailwind CSS v4, Zustand, React Hook Form + Zod |
| **後端** | FastAPI, SQLAlchemy 2.0 (async), Alembic, Pydantic v2 |
| **資料庫** | SQLite (開發) / PostgreSQL 15 (正式) |
| **AI** | OpenAI + Anthropic (Factory Pattern 抽象層) |
| **儲存** | Local filesystem (開發) / S3 (正式) |
| **基礎設施** | Docker Compose (PostgreSQL + Redis) |

## Getting Started / 快速開始

### 環境需求

- Python 3.11+
- Node.js 18+
- Docker *（選用，用於 PostgreSQL + Redis）*

### Backend 後端

```bash
cd backend
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

cp ../.env.example .env
# 編輯 .env — 設定 SECRET_KEY，選填 OPENAI_API_KEY / ANTHROPIC_API_KEY

alembic upgrade head
uvicorn app.main:app --reload
# API: http://localhost:8000
# Swagger 文件: http://localhost:8000/docs
```

### Frontend 前端

```bash
cd frontend
npm install
npm run dev
# App: http://localhost:5173
```

### Docker（選用）

```bash
docker-compose up -d
# 更新 .env 中的 DATABASE_URL：
# DATABASE_URL=postgresql+asyncpg://postgres:postgres@localhost:5432/jobhunter
```

## Project Structure / 專案結構

```
backend/
  app/
    auth/           # 認證（JWT、註冊、登入）
    profile/        # 個人檔案（學歷、經歷、技能）
    portfolio/      # 作品集（檔案上傳、連結）
    jobs/           # 職缺管理（手動輸入 + URL 解析）
    interview/      # 面試練習（Session、題目、錄影）
    llm/            # LLM 抽象層（OpenAI / Anthropic）
    common/         # 儲存抽象層（Local / S3）
  tests/            # pytest-asyncio 測試
  alembic/          # 資料庫遷移

frontend/
  src/
    pages/          # Landing、Login、Register、Profile、Interview
    components/     # AuthGuard、Layout、VideoRecorder
    stores/         # Zustand（auth、interview）
    api/            # Axios client + API 函式
    types/          # TypeScript 型別定義
```

## API Endpoints / API 端點

| 路徑前綴 | 模組 | 說明 |
|----------|------|------|
| `/api/v1/auth` | 認證 | 註冊、登入、取得當前使用者 |
| `/api/v1/profile` | 個人檔案 | 學歷、經歷、技能 CRUD |
| `/api/v1/portfolio` | 作品集 | 檔案/連結上傳、列表、刪除 |
| `/api/v1/jobs` | 職缺 | 職缺列表 CRUD |
| `/api/v1/interview` | 面試 | Session、題目、錄影管理 |

啟動後端後，可至 `/docs` 查看完整互動式 API 文件。

## Environment Variables / 環境變數

| 變數 | 必填 | 說明 |
|------|------|------|
| `SECRET_KEY` | Yes | JWT 簽名金鑰 |
| `DATABASE_URL` | Yes | SQLite 或 PostgreSQL 連線字串 |
| `OPENAI_API_KEY` | No | OpenAI API 金鑰（AI 功能） |
| `ANTHROPIC_API_KEY` | No | Anthropic API 金鑰（AI 功能） |
| `DEFAULT_LLM_PROVIDER` | No | `openai` 或 `anthropic` |
| `CORS_ORIGINS` | No | 允許的來源（預設 localhost:5173） |
| `STORAGE_BACKEND` | No | `local` 或 `s3` |

詳見 [`.env.example`](.env.example)。

## Testing / 測試

```bash
cd backend
source .venv/bin/activate
python -m pytest tests/ -v
```

## Development Status / 開發進度

| Phase | 說明 | 狀態 |
|-------|------|------|
| 1 | 認證 + 個人檔案 | Done |
| 1.5 | 作品集、LLM、職缺、前端基礎 | Done |
| 5 | 面試練習（後端 + 前端） | Done |
| 5.5 | UI/UX 優化 | Done |
| 2 | LLM 進階整合 | Planned |
| 3 | 職缺 URL 解析 | Planned |
| 4 | AI 履歷產生 | Planned |
| 6 | 付費系統（Stripe） | Planned |
| 7 | AI 面試評估 | Planned |
| 8 | 整合與打磨 | Planned |

## License

MIT
