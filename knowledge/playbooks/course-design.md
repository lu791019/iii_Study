# 帶狀課設計手冊（21 集 Course Playbook）

> 這份手冊整理 Dex 的「TibaMe 雲端資料工程師」21 集帶狀課怎麼排、每集手冊怎麼寫、環境建置怎麼教。
> 用途：之後設計新的長期課程（或幫某一集寫手冊）時，照這套結構走。

---

## 1. 整體架構

課程分兩大段，用**同一個專案貫穿**，每學一個技術就升級一次：

| 階段 | 集數 | 教材 repo | 核心任務 |
|------|------|-----------|---------|
| Season 0：基礎建置 | EP01–04 | de-course | 把開發環境建好：WSL → VS Code → Git → uv → Package → Docker → Compose |
| Season 1+：爬蟲系統 | EP05 起 | stock-crawler（主線）或 hahow-crawler（擇一） | 一支爬蟲長成分散式 pipeline：Celery → MySQL → 排程 → Airflow → GCP |

另外在期中插入一場實作工作坊（見 [workshop-design.md](workshop-design.md)），工作坊的個人 MVP 會被帶狀課回收、升級。

### 主線故事（每集都在同一個系統上加一層）

```
一支爬蟲 → Package → 容器 → Producer/RabbitMQ/Worker → 多 Worker 多 Queue
→ 寫入 MySQL → retry 與環境變數 → 測試 → 整合 → BI / 排程 / Airflow
→ GCP VM → BigQuery → Cloud SQL 多台 VM → 每日資料線（Airflow / Composer）
```

---

## 2. 集數表

> 標記：**已確認** = repo 內有該集手冊或明確寫出集數；**推測** = repo 沒有直接寫，依章節順序推論，使用前請向 Dex 確認。

| EP | 主題 | 目標 | 產出 | 依據 |
|----|------|------|------|------|
| EP01 | ETL 觀念 + WSL + VS Code + Git | 在 Linux 上寫程式、用 VS Code 編輯、用 Git 版控 | WSL Ubuntu、VS Code + WSL Extension、clone 的 de-course、ETL 架構圖（diagrams.net） | 已確認 |
| EP02 | Git + GitHub + uv + Module/Package | 每個專案有獨立環境；把腳本整理成 package | `pyproject.toml`、`uv.lock`、`crawlers/` package、`uv run` 跑通 PTT 爬蟲 | 已確認 |
| EP03 | Docker 安裝 + Dockerfile | 從零裝 Docker Engine，把爬蟲包成 image | Docker Engine（WSL）/ Colima（Mac）、能 build/run 的 Dockerfile | 已確認 |
| EP04 | Docker Compose + 完整 infra | 一行指令起多個服務 | `docker compose up -d` 起 4 個服務（RabbitMQ、Flower、MySQL、phpMyAdmin） | 已確認 |
| EP05 | 爬蟲概念 + stock-crawler 入門 | 複習 Docker，跑第一個 FinMind 爬蟲並讀懂程式 | clone stock-crawler、FinMind API 回傳 DataFrame | 已確認 |
| EP06 | Celery 入門 | 把爬蟲變成可派送的任務 | Producer → RabbitMQ → Worker 跑通、Flower 監控、Worker 容器化 | 已確認 |
| EP07 | RabbitMQ 深入 + 四個 Web 管理介面 | 「把眼睛練起來」，看懂系統在做什麼 | 會用 RabbitMQ UI / Flower / phpMyAdmin / Portainer、排錯 SOP | 已確認 |
| EP08 | 多 Worker + 多 Queue | 水平擴展與任務分流 | twse / tpex 分流、`docker compose --scale` | 已確認 |
| EP09 | MySQL + 爬蟲入庫 | 資料真正存進關聯式資料庫 | 建庫建表、Python 連線、爬蟲寫入、SQL 驗證 | 已確認 |
| EP10 | Celery 進階 | 失敗能自己復原；一份程式跑遍所有環境 | retry（指數退避）、`config.py` 環境變數、local vs Docker 切換、完整系統預演 | 已確認 |
| EP11 | 單元測試 Unit Test | 改 code 不怕弄壞 | pytest、Mock、為 config / task 加測試 | 已確認（EP10 下集預告） |
| EP12 | 系統整合 | 一鍵啟動全部服務並端到端驗證 | 整合 compose + 七步驟驗證 | 已確認集數（EP10 稱預演為「EP12 整合的彩排」）；內容對應手冊13 為推測 |
| EP13–16 | 資料出口與編排：Metabase、APScheduler、Airflow 基礎／進階 Operator／接爬蟲 | 讓資料被看見、讓 pipeline 自己跑 | 儀表板、排程、DAG | 推測（對應課程手冊 08–12，集數分配未見於 repo） |
| EP17 | GCP 開通與雲端部署 | 系統上雲（VM、防火牆、IAM、小組協作） | 雲端 VM 跑起整套系統、BigQuery 雙寫開始運轉 | 已確認（手冊14） |
| EP18 | 系統搬家：多台 VM + Cloud SQL | 拆開服務、資料庫改託管 | 兩台 VM、Cloud SQL、Secret Manager | 已確認（手冊16） |
| EP19 | BigQuery 資料倉儲 | OLTP → OLAP、raw/stage/app 三層 | BigQuery 分層 + Looker Studio 儀表板 | 推測（手冊14 寫「下一堂回頭做第 15 章」） |
| EP20 | 每日資料線：Airflow 排程與 Composer | 雲端資料線每天自動跑 | 每個交易日自動觸發的 DAG、Composer 對照、資源清理 | 已確認（手冊17，「課程最後一章」） |
| EP21 | 期末收尾 / Demo | 期末團專展示 | — | 推測（repo 未寫） |

另有補充章（選讀）：架構圖 draw.io、同步/非同步、MySQL 深入、ACID/CAP、MySQL to FastAPI、Unit Test、.env、Cloud Run、CI/CD、七步拆解、BigQuery 深入、雲地混合。補充章的字母順序就是建議閱讀順序，並寫明「建議閱讀時點」（例如「手冊05 之後」）。

---

## 3. 每集手冊的固定結構

範本：`EP05_實作操作手冊.md` ～ `EP10_實作操作手冊.md`。

```markdown
# EP06 實作操作手冊：Celery 入門 — 把爬蟲變成可派送的任務

> 對象：已完成 EP05（已 clone stock-crawler、跑過 FinMind API）的學員
> 涵蓋：啟動 RabbitMQ → 啟動 Worker → 發第一個任務 → 包成 task → Flower 監控 → Docker 化 Worker
> 實作專案：<repo 連結>
> 所有指令在 WSL Ubuntu 環境實測

## 使用前提        ← 上集回顧一句 + 本集要解決的問題 + 環境自檢指令（含預期輸出）+「沒做到回哪一集補」
### 先建立心智模型 ← 一張 ASCII 圖或表格（Producer → RabbitMQ → Worker）
## 目錄
## 第一部分：…    ← 每部分拆成 Step 1、Step 2…，每步「指令 → ✅ 預期輸出」
…
## 本集完成清單    ← 6–8 個 checkbox，每項都是可驗證的動作
## 常見問題（Troubleshooting） ← 症狀 | 原因 | 解法
## 下集預告：EP07  ← 本集停在哪 + 下集 3–4 個學習點
```

**寫法重點**

- 開頭四行固定：對象（前置集數）、涵蓋（用 → 串起本集流程）、實作專案、「所有指令實測」聲明。
- 「使用前提」用三段話說故事：上集做到哪 → 現實的問題是什麼 → 本集要做什麼。例：「一個 Worker 就是瓶頸：1000 個任務只能一個一個慢慢做。」
- 每個 Step 都附預期輸出（`# 預期：Active: active (running)`），讓學員自己判斷成功與否。
- 對照表大量出現：`docker run vs docker compose`、本機 `127.0.0.1` vs Docker 服務名、`if_exists` 三種模式。
- 結尾一定有「收工：關閉服務」，避免學員電腦一直開著容器。
- 每集都可以單獨補課：前提段落指回需要的集數或手冊。

### 每集 README（集數資料夾內）

`ep01/README.md` 這類短版：本集目標一句 → 「本集做了什麼」表（段落 | 內容）→ 環境步驟 → 「EP0X 結束後的專案狀態」表（檔案 | 說明）→ 詳細手冊連結 → 常見問題表。

### 跨集參考手冊

長篇主題獨立成手冊，集數 README 只連結：環境建置（WSL/Linux/VS Code/Git/uv/Package）、Git 協作、Docker 安裝、Docker/Compose 操作、Docker 實作。課程 README 有「手冊 | 內容 | 什麼時候看」索引表。

---

## 4. 環境建置的教學順序

| 順序 | 主題 | 為什麼放這裡 | 教學設計重點 |
|------|------|-------------|-------------|
| 1 | ETL 觀念 + 架構圖 | 先看見終點，後面每個工具才有位置 | EP01 開頭講 Extract → Transform → Load 與整門課架構 |
| 2 | WSL + Ubuntu（Mac 用 Terminal） | 業界開發與部署都在 Linux | 先確認虛擬化已開啟；處理「開 WSL 卻在 `/mnt/c/...`」→ `cd ~` |
| 3 | Linux 基礎指令 | 後面所有操作都在 Terminal | pwd / ls / cd / mkdir / touch / cat / cp / mv / rm 速查表 |
| 4 | VS Code + WSL Extension | 讓編輯器連進 Linux | `code .` 開專案；必裝 Extension 表（WSL、Python、Git Graph） |
| 5 | Git + GitHub | clone 課程 repo、之後交作業 | 每個操作同時給 Terminal 指令與 VS Code GUI 兩種做法；Git Graph 看歷史 |
| 6 | uv | 解決「我的電腦可以跑」 | 先講痛點（專案 A/B 版本衝突）→ 工具比較表 → 為什麼直接用 uv →「uv 就是 Python 版的 nvm + npm」→ 傳統 vs uv 對照表 |
| 7 | Module / Package | 把腳本整理成可 import 的結構 | Level 1 一個 .py → Level 2 加 `if __name__` → Level 3 加 `__init__.py` |
| 8 | Docker Engine | 環境一致、服務隔離 | 裝 Docker Engine 不裝 Desktop；Mac 用 Colima；FAQ 26 題 |
| 9 | Docker Compose | 多服務一鍵起 | 先 `docker run` 跑 Nginx，再換 compose，最後做比較表 |

**環境標記**（操作手冊_環境建置.md）：🪟 Windows PowerShell、🍎 macOS、🐧 WSL/Mac Terminal、📁 目前目錄、✅ 驗證（每步必做）、⚠️ 注意、❌ 常見錯誤。學員一眼知道指令要在哪裡打。

**課前預載（老師必看）**：列出每集會下載的 image 與大小，標出「EP04 才是下載爆發點」，給 `docker pull` 預載清單。原則：小東西現場 demo（展現完整流程），大 image 課前先 pull（把時間花在講解，不是等下載）。

---

## 5. 教學法模式

| 模式 | 做法 | 例子 |
|------|------|------|
| 同一專案貫穿 | 每集在同一系統上加一層，學員最後有一個長期作品 | 期中 MVP → 帶狀課 ETL 架構 → Docker → Airflow → GCP |
| 先跑通簡化版，再深入正式版 | 每個概念先有「只 print」的版本 | `tasks.py`（只 print）→ `tasks_crawler_finmind.py`（真的爬）；手動執行 → APScheduler |
| 心智模型先行 | 動手前先畫角色圖 | Producer（發任務）→ RabbitMQ（排隊）→ Worker（執行） |
| 故意弄壞 | 讓學員看見失敗時系統的反應 | 寫一個會失敗的 task 觀察 retry 2/4/8 秒；故意殺 worker 看訊息去向 |
| 觀察工具先於擴充 | 先學會「看」，再加複雜度 | EP07 不加 Worker，先練四個 Web 介面與排錯順序 |
| 排錯 SOP | 給固定檢查順序 | Flower → RabbitMQ → Portainer → phpMyAdmin |
| 對照表 | 新舊做法並排，說清楚取捨 | 整合版 vs 拆開版 compose；local vs Docker 設定 |
| 痛點驅動 | 每集開頭講「現在的做法哪裡不夠」 | 「資料只是 print 出來，看過就消失」→ EP09 入庫 |
| 實測聲明 | 寫明在哪個版本實測過 | 「所有指令都在 Docker 29.6.1 / Compose v5.2.0 / Ubuntu 22.04 實測通過」 |
| 帶得走的資產 | 把方法本身變成學員能帶走的東西 | 「這套驗證方法本身，就是你帶得走的資產」 |
| 雙路線 | 同主題提供兩個專案擇一 | stock-crawler（漸進式）／hahow-crawler（Airflow + BigQuery 完整版） |

---

## 6. 設計一門新帶狀課的 Checklist

- [ ] 一句話寫出整門課最後的系統長什麼樣，並畫成架構圖
- [ ] 選一個貫穿全程的專案，列出每集在它身上加什麼
- [ ] 分 Season / Phase，每段有明確終點（環境好了、資料進 DB、上雲了）
- [ ] 每集一份實作手冊，套用第 3 節結構（對象 → 前提 → 心智模型 → 目錄 → Step + 預期輸出 → 完成清單 → 疑難排解 → 下集預告）
- [ ] 每個新概念都有「簡化版先跑通」
- [ ] 環境建置照第 4 節順序，每步附 ✅ 驗證
- [ ] 長篇主題獨立成跨集手冊，README 放「什麼時候看」索引
- [ ] 列出課前預載清單（大檔、image）
- [ ] 選讀補充章標「建議閱讀時點」，不影響主線
- [ ] 集數對照表放在 README，手冊開頭寫「本章對應 EPxx」，避免集數與章號混淆
- [ ] 每集結尾都有關閉服務的步驟

---

## 參考來源

- [de-course/README.md](https://github.com/lu791019/de-course/blob/main/README.md)
- [ep01/README.md](https://github.com/lu791019/de-course/blob/main/ep01/README.md)、[ep02/README.md](https://github.com/lu791019/de-course/blob/main/ep02/README.md)、[ep03-04/README.md](https://github.com/lu791019/de-course/blob/main/ep03-04/README.md)
- [操作手冊_環境建置.md](https://github.com/lu791019/de-course/blob/main/操作手冊_環境建置.md)、[Git_GitHub開發協作手冊.md](https://github.com/lu791019/de-course/blob/main/Git_GitHub開發協作手冊.md)
- [Docker安裝教學手冊.md](https://github.com/lu791019/de-course/blob/main/Docker安裝教學手冊.md)、[Docker_Compose開發操作手冊.md](https://github.com/lu791019/de-course/blob/main/Docker_Compose開發操作手冊.md)、[Docker實作操作手冊.md](https://github.com/lu791019/de-course/blob/main/Docker實作操作手冊.md)
- [EP05](https://github.com/lu791019/de-course/blob/main/EP05_實作操作手冊.md) ～ [EP10_實作操作手冊.md](https://github.com/lu791019/de-course/blob/main/EP10_實作操作手冊.md)
- EP11 之後的集數對照參考 stock-crawler 的 README 與 `課程手冊/課程手冊14、16、17`：https://github.com/lu791019/stock-crawler-de-course-materials
- 期中工作坊與帶狀課的銜接：https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/midterm_workshop_proposal.md
