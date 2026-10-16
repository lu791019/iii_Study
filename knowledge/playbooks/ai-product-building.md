# 教學手冊：Dex 打造 AI 工具與產品的做法

> 這份文件讓之後協助 Dex 備課、帶專題或延續開發的人（或 AI），知道 Dex 怎麼從一個痛點做到能跑的 AI 工具：常用架構、技術選型、怎麼跟 Claude Code 協作、從需求到 MVP 的步驟，以及每個 repo 可以拿來教什麼。
> 整理自 7 個工具／產品 repo（2026-10-16 讀取）。標示規則：**【repo】** 是 repo 裡實際寫的；**【推論】** 是本文件從多個 repo 歸納出的提案，repo 沒有明講。
> 私人 repo 只摘做法與教訓，不放客戶名稱、網址、帳號、金額。各 repo 細節見 [../repo-notes/](../repo-notes/)。

> 一句心法：**先讓主幹在沒有 AI 的情況下也能用，AI 是加值，不是地基。**

## 做完這份閱讀，你會做到

- 用一行箭頭說出 Dex 工具的共通架構，並對應到每個 repo
- 解釋 Dex 偏好的技術棧，以及「為什麼選 A 不選 B」
- 照 Dex 的文件組（`implementation_plan.md`、`task.md`、For Claude 接手文件）開一個新專案
- 把任一 repo 變成一堂課或一個學員專題題目

---

## 1. 七個 repo 一覽

| repo | 公/私 | 一句話 | 主流程 |
| --- | --- | --- | --- |
| castflow | 公 | Podcast 音訊 → 6 平台社群文案 → 排程直發 | 上傳 → 壓縮 → Whisper → Claude（風格 DNA）→ Meta API |
| social-scraper | 公 | LINE 傳連結，自動爬取、摘要、寫回 Sheet／Notion | LINE → Patchright／httpx → OCR／STT → `claude --print` → Sheet／Notion → LINE 推播 |
| job-hunter-v2 | 公 | AI 求職助手：個人檔案、職缺、行為面試錄影練習 | 表單 → FastAPI → LLM 出題 → 瀏覽器錄影 → 回放 |
| podmind | 私人 | Podcast 雙語對照逐字稿＋AI 摘要（聽／讀／掃三種 view） | RSS → mlx-whisper → Claude 段落翻譯＋摘要 → Markdown／Next.js |
| filmer | 私人 | 用錄音或文字留觀影感受，靠語意配對遇見同頻的人 | 錄音 → Whisper → embedding → pgvector → Feed／推薦 |
| coop-docgen | 私人 | 客戶案：表單（可附錄音）→ 套官方範本 → Word／PDF → 雲端歸檔 | 表單 → schema 驗證 → docxtpl 套版 → 預覽／產出 → 歸檔；錄音 → Gemini 草稿回填 |
| aitool | 私人 | AI 工作集：Text-to-SQL、文件 OCR、中台語 ASR、影像檢測 MLOps | 依子專案而異（見 repo-notes） |

---

## 2. 常見架構：收集 → 處理 → LLM → 介面 → 落地

**【推論】** 7 個 repo 幾乎都能套進同一條管線：

```
收集（爬蟲／RSS／上傳／錄音／表單） → 處理（壓縮、轉錄、OCR、清洗） → LLM（摘要、翻譯、擷取、生成） → 介面（Web／LINE Bot／表單） → 落地（DB、Sheet、Notion、檔案歸檔、社群發布）
```

| 階段 | castflow | social-scraper | podmind | coop-docgen | aitool（舉例） |
| --- | --- | --- | --- | --- | --- |
| 收集 | 音訊上傳 | LINE 連結、Patchright 爬 IG／Threads | RSS feed | 網頁表單、錄音 | 資料庫 schema、文件影像、攝影機串流 |
| 處理 | ffmpeg.wasm 瀏覽器端壓縮 | ffmpeg 抽音軌、mlx-whisper、圖片 OCR | ffmpeg 裁剪、mlx-whisper | schema 驗證、法規檢核 | 影像前處理、多 OCR 引擎競賽 |
| LLM | Claude 生成 6 平台文案 | `claude --print` 摘要＋關鍵點 | Claude 段落翻譯＋TL;DR | Gemini 一次產逐字稿＋欄位草稿 | 問題分類 → Text-to-SQL → 結果解釋 |
| 介面 | Next.js Dashboard | LINE Bot | Markdown → Next.js 三 view | React 動態表單＋即時預覽 | FastAPI＋簡易 Web UI、Gradio |
| 落地 | Supabase、Threads／FB 發布 | Google Sheet、Notion | Supabase（規劃） | Word／PDF、雲端儲存 | 圖表、報告、模型版本 |

### 反覆出現的設計模式【repo】

| 模式 | 做法 | 出現在 |
| --- | --- | --- |
| **LLM 供應商抽象層** | 定一個介面（`generate()`、`generate_json()` 或 Protocol），各家實作一個 class；換供應商不動 router 與前端 | job-hunter-v2（Factory）、castflow（`provider: cli / anthropic / openai / gemini / auto`）、coop-docgen（`DraftProvider`）、aitool Text-to-SQL（local／cloud 模式） |
| **儲存抽象層** | 開發用本地檔案，正式換雲端，只換一個實作 | job-hunter-v2（Local → S3）、coop-docgen（Local → 雲端儲存） |
| **本地優先、零 API 成本** | 轉錄用 mlx-whisper、LLM 走 `claude -p` 吃訂閱額度；擴散時才換雲端 API | social-scraper、podmind、castflow（CLI 模式） |
| **AI 是加值，主幹可獨立運作** | 後端沒設 AI 時前端整塊隱藏，純手填照常；AI 草稿要人工逐欄確認 | coop-docgen |
| **防幻覺三層** | prompt 明令「只填聽得到的」＋後端丟掉 schema 外的鍵＋前端只覆蓋有值欄位 | coop-docgen |
| **Schema 當單一事實來源** | 後端 Pydantic 定欄位 → 輸出 JSON Schema → 前端動態渲染表單；新增文件類型前端零修改 | coop-docgen |
| **失敗不靜默** | 歸檔失敗照樣給下載，但明確警示使用者；降級要印出來 | coop-docgen、aitool OCR（引擎不可用自動切換） |
| **成本護欄** | 每日 AI 呼叫上限、逾量回 429 並導引手填；雲端預算警示；每日爬取上限 | coop-docgen、social-scraper |
| **併發與佇列** | 同時最多 N 筆，其餘排隊；先回「已收到」，背景處理完再推播 | social-scraper |

---

## 3. 技術選型偏好【repo】

| 層 | Dex 常用 | 出現在 | repo 寫的理由 |
| --- | --- | --- | --- |
| Web 全端 | Next.js 15 App Router＋Tailwind＋shadcn/ui＋Supabase | castflow、podmind | SSR＋API Routes 一體，MVP 不需獨立後端；DB＋Storage＋Auth 一站式 |
| 前後端分離 | React／Vite／TypeScript＋FastAPI | job-hunter-v2、coop-docgen | FastAPI 自帶 Swagger；前端 build 後由 FastAPI 同源服務，部署成單容器 |
| 跨平台 App | Flutter＋Riverpod＋GoRouter，另開 SEO 站 | filmer | 一人團隊要一套 codebase；Flutter web SEO 不足，所以內容頁分出去 |
| Python 工具鏈 | uv、pytest、ruff | social-scraper、coop-docgen | — |
| 語音轉文字 | mlx-whisper（本地）／Groq Whisper／OpenAI Whisper API／Gemini 單次呼叫 | podmind、social-scraper／castflow／filmer／coop-docgen | 自用階段零成本 → 雲端快、便宜 → 要部署時才上 API |
| LLM | `claude -p`、Claude Sonnet、Gemini（Vertex AI）、Ollama 本地 | 多數 repo | 「Sonnet 而非 Opus」：摘要任務品質夠用，成本低很多 |
| 爬蟲 | Patchright（反偵測 Playwright）、httpx＋readability、官方 REST API | social-scraper | 「先用 API 拿能拿的，不夠才上爬蟲」；不登入，零封號風險 |
| 排程 | Supabase pg_cron、APScheduler、手動觸發 | castflow、aitool | Vercel Hobby Cron 一天只能一次，不夠分鐘級 |
| 部署 | Vercel、Cloud Run（`--source .` 代建）、本機＋ngrok | castflow、coop-docgen、social-scraper | 閒置縮到 0、免費額度內 |

**選型原則【repo】**

1. **熟悉工具優先**：「重用熟悉工具 = 零學習成本」（podmind ADR-002）。
2. **MVP 先自用、零或低成本**：先 dog-fooding 驗證，再談擴散。
3. **產品獨立性**：可擴散的產品不依賴第三方當資料源或儲存層，「擴散時會被掐脖子」（podmind）。
4. **砍到能交付**：Auth 先規劃不實作、圖片手動上傳不做 AI 生圖、直發只做兩個平台（castflow）。
5. **每個選擇都寫「拒絕了什麼、為什麼」**：ADR 的 Alternatives Rejected 欄。

---

## 4. 怎麼跟 Claude Code／AI 協作開發

### 4.1 每個 repo 都有的文件組【repo】

| 文件 | 內容 | 例子 |
| --- | --- | --- |
| `implementation_plan.md` | Goal Description → **User Review Required**（開工前要人拍板的決策與假設）→ Proposed Changes（檔案結構、`[NEW]`／`[MODIFY]`）→ Verification Plan | castflow、social-scraper、podmind、filmer |
| `task.md` | 依 Phase／Section 分組的 checkbox；完成的 task 後面附 commit hash；spike 發現直接寫在 task 底下 | social-scraper、coop-docgen |
| `docs/plans/YYYY-MM-DD-<主題>-design.md`＋`-plan.md` | 設計文件與逐 task 實作計畫分開；`PLAN.md` 只當索引 | social-scraper、job-hunter-v2、coop-docgen |
| `GUIDE.md` | 給開發者的操作手冊：快速啟動、結構、慣例、API、測試、排錯 | job-hunter-v2、coop-docgen、social-scraper |
| `CLAUDE.md` | 專案級開發規範：方法論、技術棧、命名、測試覆蓋率、產品價值觀 | filmer |
| `SESSION-RESUME.md` | 「下次重啟 session 時 Claude 第一個要讀的檔案」 | podmind |
| `decisions.md`／思考紀錄 | ADR：Context／Decision／Rationale／Alternatives Rejected；外加暫緩想法與未回答問題 | podmind、filmer |

### 4.2 For Claude 接手文件怎麼寫【repo】

- 計畫開頭固定一行：`> For Claude: REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.`，再接 **Goal／Architecture／Tech Stack** 三行（job-hunter-v2、aitool）。
- 先寫 **「Codebase Conventions (Read First)」**：model 寫法、service 是函式不是 class、測試 fixture 在哪，避免 AI 自己發明風格（job-hunter-v2）。
- 每個 Task 列 **Files：Create／Modify（含行號）／Test**，步驟固定：寫失敗測試 → 確認失敗 → 實作 → 確認通過 → commit（job-hunter-v2、aitool MLOps task）。
- `implementation_plan.md` 也可以當 **Handoff 進度快照**：已完成、下一步、「已修改但未 commit 的檔案」、已知平台雷區（job-hunter-v2、coop-docgen）。
- **SESSION-RESUME 的兩張表**（podmind）：
  - 「Dex 說 → Claude 應該做」：「接續 X」→ 讀本檔；「我想先討論」→ 探索模式，不要急著派 executor。
  - 語氣訊號字典：「你覺得呢」「可以深入討論」＝**探索模式**，給選項不拍板；「先 X」「落地」「開工」＝**結案模式**，立刻產出。
- **反模式紀錄**：把 AI 犯過的判斷錯誤寫進 VISION 或 resume 檔（例如「過早拍板」「假設共用資源就是好的」），下次 session 先讀。

### 4.3 其他協作手法【repo】

| 手法 | 做法 | 出處 |
| --- | --- | --- |
| 多角色 AI 會議 | 讓 AI 扮演 CPO／CMO／CTO／CFO／教學總監開多輪會議，收斂成 VISION 與 ADR；討論夠了由 Dex 叫停「先把規劃落地，不要一直開會」 | podmind |
| SDD → BDD → TDD | spec 提取 API Contract → 每端點 ≥3 個 Given/When/Then（happy path、驗證失敗、權限）→ 紅綠重構 → SDD Review 比對 spec | filmer |
| Subagent 分工 | 主 session 做 spec 與測試；實作、spec review、code review 各派 subagent | filmer |
| 多工具規則檔 | 同一套規範分別寫成 `CLAUDE.md`、`.kiro/steering/*.md`（可依 fileMatch 載入）、`.amazonq/rules/` | filmer、aitool |
| Dev log 三分類 | 「已修復」／「嘗試過但已 revert」／「保留但效果有限」，連失敗的方案也記，避免重試 | aitool |
| 給設計 AI 的需求書 | 需求書寫清使用者、畫面、硬約束、已定案行為、現況問題、參考範例「看什麼」；提示詞分輪、一次一張畫面；「不要發明不存在的功能」、用真實內容不用 lorem ipsum；回饋要指著具體元素講 | coop-docgen |
| 名詞對照表 | 給非技術讀者的文件，第一次出現的術語附白話解釋（Recall、漂移偵測、影子模式） | aitool |

---

## 5. 從需求到 MVP 的流程

**【推論】** 把 filmer、podmind、castflow、coop-docgen 的實際順序合起來，是下面 8 步。每步都有 repo 例子。

| 步 | 目標 | 做法 | 產出 | repo 例子 |
| --- | --- | --- | --- | --- |
| 1 | 痛點與定位 | 競品痛點表（產品 ／ 痛點）、一句話定位、proto-persona、JTBD | `product-strategy.md`、README 開頭 | filmer、podmind、castflow |
| 2 | 價值觀當決策過濾器 | 寫 3–5 條「A > B」 | CLAUDE.md 或 VISION.md 的價值觀段 | filmer「真實 > 完美」、podmind「加 buff 不取代」 |
| 3 | 決策與不做清單 | 每個決策寫理由與被拒絕的替代方案；MVP 砍掉的功能也列出來 | `decisions.md`、思考紀錄 | podmind ADR-004、filmer D002–D007 |
| 4 | Spike 驗證關鍵假設 | 先把要驗的假設寫成可量測的條件，用最小單檔腳本跑 | spike 腳本＋報告＋「Phase 1 修正清單」 | podmind（0.5 天，6 個驗證點過 5 個 → GO）、social-scraper（先 spike 出 IG JSON 結構） |
| 5 | 實作計畫與人工確認 | `implementation_plan.md` 的 User Review Required 先讓人拍板 | plan＋task.md | 全部 |
| 6 | 先跑通 happy path | 最小閉環先通，加值功能排後面 | 能 demo 的主流程 | castflow「風格 DNA 移到 Phase 5」；coop-docgen「純手填是主幹，AI 是加值」 |
| 7 | 前置作業提早做 | 要審核、要申請的外部帳號放 Phase 1 | — | castflow「Meta Developer App 在 Phase 1 建立，避免 Phase 4 才發現」 |
| 8 | 上線、控成本、收回饋 | e2e 實測、預算警示、使用者現場回饋寫回 task.md | 部署＋回饋清單 | coop-docgen「社場回饋」段 |

### 常見坑與對策【repo】

| 你遇到的狀況 | 出處 | 怎麼解 |
| --- | --- | --- |
| 計畫寫的技術和實際做的不一樣（計畫寫 GPT-4o，實作改成 Claude） | castflow `PLAN.md` vs README | 改選型時同步改計畫；或在計畫標「⚠️ 已過時」 |
| 總覽寫 Phase 5 是 0%，GUIDE 寫已完成 | job-hunter-v2 | 進度只在 task.md 定義一次，其他文件引用它 |
| 模型代號被下架，API 回 404 | coop-docgen | 模型名放環境變數，不寫死在程式 |
| 中文 Whisper 段落碎片化、出現簡體字、節目名認錯、幻覺 | podmind spike | 合併過短段落、`--initial-prompt` 注入專有名詞、OpenCC 轉繁、`temperature 0` |
| 並行呼叫 `claude -p` 斷線，翻譯只完成一部分 | podmind spike | 改循序＋retry |
| Whisper API 單檔 25MB 上限 | castflow | 瀏覽器端 ffmpeg.wasm 壓成 64kbps mono 16kHz |
| 雲端服務帳號無法在個人雲端硬碟建檔 | coop-docgen | 改用物件儲存；或用組織共用雲端硬碟 |
| 每次部署都留一個映像，累積後超出免費額度 | coop-docgen | 設映像清理政策（保留最新幾個） |
| docx 轉 PDF 後中文變空白方框 | coop-docgen | 容器裝中文字型並設字型對映 |
| 爬蟲瀏覽器和本機 Chrome 不能同時開；ngrok 每次重啟換網址 | social-scraper | 寫進 README 注意事項與排錯表 |
| 部署到錯的雲端專案 | coop-docgen | 部署前先確認作用中的 CLI 設定檔 |

---

## 6. 可以拿來教什麼

| repo | 可以拿來教什麼 |
| --- | --- |
| castflow | 一條完整的「音訊 → LLM → 自動發布」閉環，加上 Prompt 分層（模板 → 規則 → 風格 DNA → 範例）與排程狀態機（draft → scheduled → published → failed） |
| social-scraper | 爬蟲倫理與路線選擇（官方 API → 無頭瀏覽器 → 攔截 API）、LINE Bot webhook、佇列併發，以及「零 API 成本」的本地 LLM 管線 |
| job-hunter-v2 | FastAPI 模組化結構（models／schemas／service／router）、LLM Factory 抽象層，以及寫給 Claude 照著做的 TDD 計畫 |
| podmind | 從產品哲學到 spike：怎麼把假設寫成可量測的驗證點、ADR 怎麼寫，以及 session 接手文件 |
| filmer | 產品策略文件（定位聲明、persona、JTBD）、「不做什麼」決策紀錄、SDD → BDD → TDD |
| coop-docgen | 真實客戶案的 MVP：schema 驅動表單、AI 加值不綁架主幹、防幻覺、雲端成本控管、上線後收回饋迭代 |
| aitool | Text-to-SQL 的問題分類分流、多引擎 OCR 競賽、DDD 分層的 CV 推論服務與 MLOps 驗證關卡 |

### 學員專題題目建議【推論】

依 [student-project-mentoring.md](student-project-mentoring.md) 的專題分級，從簡到難：

| 難度 | 題目 | 對應 repo | 最小路徑 |
| --- | --- | --- | --- |
| 🟢 | 連結收藏摘要 Bot：LINE／Telegram 傳連結 → 摘要 → 寫進 Sheet | social-scraper | 只做一般網頁＋GitHub，先不碰 IG |
| 🟢 | 公開資料 Text-to-SQL 問答：自然語言 → SQL → 圖表 | aitool（Text-to-SQL） | SQLite＋公開資料集，先只做「資料問題 ／ 一般問題」分類 |
| 🟡 | Podcast／課程錄音 → 逐字稿 → 重點與社群草稿 | castflow、podmind | 不做自動發布，只產 Markdown |
| 🟡 | 表單 → 官方範本 → Word 產生器（用假資料、公開範本） | coop-docgen | 一種文件、一個範本、無 AI |
| 🟡 | 面試練習工具：題庫 → 錄影 → 回放 | job-hunter-v2 | 不做 AI 評估 |
| 🔴 | 語意配對社群：文字 → embedding → pgvector 相似推薦 | filmer | 純文字、不做錄音 |
| 🔴 | 影像偵測服務＋模型驗證關卡（Golden Test Set、回滾） | aitool | 用公開資料集與預訓練模型 |

---

## 7. 寫教材時的隱私提醒

- coop-docgen 與 aitool 部分子專案是客戶案：教學一律寫「某 NGO 客戶」「某基礎設施客戶」，不寫客戶名、網址、雲端專案名、合約內容。
- 私人 repo 的 `.env`、設定檔裡有帳號或裝置連線資訊，**不要複製進教材**；範例一律用 `your-xxx` 佔位。
- 引用 repo 裡的數字（成本、耗時、測試數）時，標「repo 自述」或「約略值」。

下一步 → 各 repo 細節看 [../repo-notes/](../repo-notes/)；要把題目寫成計畫書，用 [../templates/project-plan.md](../templates/project-plan.md)。
