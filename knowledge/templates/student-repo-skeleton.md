# 學員專題 Repo 骨架（template-repo）

> 手冊的 `template-repo/` 是給各組 fork / 複製的起步 repo。原則：「最小起步只需要這些，後面需要再加就好。」
> 帶學員時：⑤ 環境階段（W7–9）就要建好；README 在 W0 就寫，task.md 到實作期（W10 起）再填才有內容。

---

## 1. 內容一覽與用意

| 檔案 | 內容 | 為什麼要有 |
|---|---|---|
| `README.md` | 一句話描述、為什麼做、六階段架構表、快速開始、團隊、進度連結、License | 面試官打開 GitHub 第一眼看的就是 README；「為誰解決什麼問題」要在第一行 |
| `task.md` | 前置期 / 實作期三階段 / Demo 準備的勾選清單，每項標 `@負責人` | 「讓組員知道誰在做什麼、做到哪了」，任何人快速看進度 |
| `.env.example` | 所有環境變數的鍵名，值留空或放假值 | 讓別人知道要填哪些 key；真正的 `.env` 永遠不 commit |
| `.gitignore` | 排除 secrets、資料、Python / Jupyter / IDE / Airflow / Docker 產物 | 防止 API key 與爬下來的資料進 git（雷區：commit 了 secret 要 revoke＋BFG 清歷史） |
| `requirements.txt` | 涵蓋 8 情境可能用到的套件（見下） | 一個 `pip install -r` 就能跑；學員依題目刪減 |

實作期有需要再逐步加：

```
├── src/                # 程式碼：4 個資料夾＝小組 4 種角色各擁有一個目錄
├── data/               # 資料（加進 .gitignore，只留 README 說明怎麼取得）
├── docs/               # 架構圖、API 文件等——「寫文件不是裝飾，是必要 artifact」
├── tests/              # 測試
├── Dockerfile          # 容器化（課程會教）
└── docker-compose.yml
```

建議的 `docs/` 內容（散見 03、04、06、07 章）：`user-persona.md` / `persona.md`、`pain-points.md`、`data-inventory.md`、`architecture.md`、`data-flow.md`、`er-diagram.md`、`api-contract.md`、`architecture-decisions.md`（ADR）、`lessons-learned.md`、`self-check-w{N}.md`、`retro/final.md`。根目錄另有 `team-charter.md`、`topic-decision.md`、`PLAN.md`、`GUIDE.md`。

---

## 2. README.md 範本（原文）

````markdown
# {專題名稱}

> 一句話描述：為 {誰} 解決 {什麼} 問題。

---

## 為什麼做這個

{用 2-3 句話說明：業務痛點是什麼、現況有多痛、你們想做到什麼}

---

## 系統架構

```
Source → Ingest → Storage → Process → Serve → Observe
```

把上面每個階段換成你們實際用的工具，例如：

| 階段 | 工具 |
|---|---|
| Source | {例：Spotify API / Kaggle CSV / 公開資料} |
| Ingest | Python |
| Storage | MySQL / MongoDB |
| Process | pandas / SQL |
| Serve | FastAPI / Streamlit |
| Observe | Discord webhook / Sentry |

---

## 快速開始

```bash
git clone https://github.com/{你們的team}/{project}.git
cd {project}
cp .env.example .env   # 填入需要的 API keys
pip install -r requirements.txt
python main.py
```

---

## 團隊

| 成員 | 負責範圍 | GitHub |
|---|---|---|
| {Name} | {負責什麼} | @user1 |
| {Name} | {負責什麼} | @user2 |
| {Name} | {負責什麼} | @user3 |

---

## 進度追蹤

見 [task.md](task.md)

---

## License

MIT
````

**協作 SOP 對 README 的要求（05 章）**：一句話描述（為誰解決什麼）、螢幕截圖 / GIF、為什麼做這個、系統架構（一張圖）、快速開始（「5 分鐘內能跑起來」，如 `make dev`）、技術棧、團隊、License。專業版 repo 另需互相連結團隊 repo 與每人 portfolio。

---

## 3. task.md 範本（原文）

```markdown
# 任務追蹤

> 每完成一項就打勾。簡單就好，重點是讓組員知道誰在做什麼、做到哪了。

---

## 前置期

- [ ] 溝通工具 + 看板開好
- [ ] GitHub repo 建好、每人都能 push
- [ ] 題目選定
- [ ] 架構圖畫好

---

## 實作期

### 第一階段：跑通最小可行版

- [ ] {任務 1}  @負責人
- [ ] {任務 2}  @負責人
- [ ] {任務 3}  @負責人

### 第二階段：擴充 + 自動化

- [ ] {任務 4}  @負責人
- [ ] {任務 5}  @負責人

### 第三階段：打磨 + 收尾

- [ ] {任務 6}  @負責人
- [ ] {任務 7}  @負責人

---

## Demo 準備

- [ ] 投影片
- [ ] 講稿 / demo 流程
- [ ] 組內練習至少 1 次
- [ ] README 更新到最新狀態
```

05 章的進階寫法（每項附 issue 編號與 commit hash，便於追溯）：

```markdown
## Sprint 1
- [x] Spotify ingest（@owner）#12 ed4f3a2
- [ ] Streamlit MVP（@owner）#18
```

三階段對應 Sprint 1 跑通 → Sprint 2 擴充＋自動化 → Sprint 3 打磨＋監控；07 評分表 self check 的最低分維度也要寫成 task 進 task.md。

---

## 4. .env.example 的鍵（只列名稱，不含值）

| 區塊 | 鍵 |
|---|---|
| LLM / AI | `OPENAI_API_KEY`、`ANTHROPIC_API_KEY` |
| Database | `MONGODB_URL`、`MONGODB_DB`、`MYSQL_HOST`、`MYSQL_PORT`、`MYSQL_USER`、`MYSQL_PASSWORD`、`MYSQL_DB` |
| Airflow | `AIRFLOW_UID`、`AIRFLOW__CORE__FERNET_KEY`、`AIRFLOW__WEBSERVER__SECRET_KEY` |
| 外部 API（依情境） | `SPOTIFY_CLIENT_ID / SECRET / REDIRECT_URI`、`GOOGLE_MAPS_API_KEY`、`LINE_CHANNEL_ACCESS_TOKEN`、`LINE_CHANNEL_SECRET`、`DISCORD_WEBHOOK_URL`（pipeline 失敗通知） |
| Monitoring | `SENTRY_DSN` |
| App config | `ENV`、`LOG_LEVEL` |

檔頭註解：「複製這份成 .env，填入實際值；.env 不可 commit！（已加進 .gitignore）」。DB host 用 `mongo` / `mysql` 服務名，暗示預設跑在 docker compose 內。

## 5. .gitignore 重點

- **Secrets**：`.env`、`.env.local`、`*.key`、`*.pem`、`credentials.json`、`service-account*.json`
- **Data**：`data/raw/`、`data/processed/`、`data/interim/`、`data/external/`、`*.csv`、`*.parquet`、`*.json.gz`，例外保留 `data/README.md`、`data/sample.csv`
- **其他**：Python 快取與 venv、`.ipynb_checkpoints/`、IDE、Airflow（`airflow.cfg`、`airflow.db`、`logs/`）、Docker pid、OS 檔、`dist/ build/`

## 6. requirements.txt 分組

| 分組 | 套件 |
|---|---|
| Core | python-dotenv、pydantic |
| Data | pandas、numpy、pyarrow |
| DB drivers | pymongo、sqlalchemy、pymysql、psycopg2-binary |
| HTTP / scraping | requests、httpx、beautifulsoup4、playwright、scrapy |
| Pipeline | apache-airflow |
| Analytics / ML | scikit-learn、sentence-transformers |
| LLM | openai、anthropic |
| Serve | streamlit、fastapi、uvicorn、folium |
| Monitoring | sentry-sdk |
| Test / dev | pytest、pytest-cov、pytest-mock、ruff、black、mypy |

（版本皆以 `>=` 下限標示；Airflow 通常另裝在 docker 映像中，學員可視情況移除以免本機安裝衝突。）

---

來源：
- https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook/tree/main/template-repo（`README.md`、`task.md`、`.env.example`、`.gitignore`、`requirements.txt`）
- https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook/blob/main/05-collaboration-sop.md（Repo 結構、文件三件套）
- https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook/blob/main/03-team-process.md（各階段 docs 產出物）
