# 學員專題帶領手冊（專題業師 Playbook）

> 用途：未來以 Dex 的助理身份帶 TibaMe 雲端資料工程師養成班的小組專題時，照這份判斷「學員現在在哪、該做什麼、該交什麼、卡住怎麼回」。
> 核心心法（原文）：**「業界專案不是把技術秀出來，是把情境、痛點、解法(最優解和最快解)、遇到的技術問題和如何處理等講清楚。」**
> 手冊目標：從 0 到 1 建立「面試作品集」，升級成「業界最低標準等級的專案」。

---

## 1. 專題的三種等級（帶學員往第三級推）

| 等級 | 樣貌 | 面試時的結果 | 往上一級的關鍵動作 |
|---|---|---|---|
| 作業 | 跑得起 Jupyter Notebook，沒架構圖，demo 只能手動執行 | 面試官比較難看到亮點 | 拆模組、畫 6 階段架構圖、上排程 |
| 作品 | 有敘述、能持續執行、技術棧豐富 | 「跟其他人差別不大」 | 補「為什麼選 A 不選 B」、監控、歷史紀錄 |
| MVP（demo） | 有架構圖、有歷史紀錄、有監控和下一步行動、能講選型理由 | 想深入了解專案 | 履歷化（見第 10 節） |

Demo Day 分數對照同一套邏輯：<60 學作業、60–74 學作品、75–89 業界 demo、90+ top tier（詳見 `templates/demo-day-rubric.md`）。

---

## 2. Data Pipeline 六階段藍圖

**一句話**：所有資料管線都是 Source → Ingest → Storage → Process → Serve → Observe，8 情境與業界系統骨架相同，差別只在每階段選型。

- **兩種視角**：架構視角＝六階段（pipeline 長什麼樣）；行為視角＝ETL / ELT（資料怎麼動、何時轉換）。
- **ETL vs ELT**：ETL 先轉再進倉（嚴格 schema、批次）；ELT 先全部入倉再用倉儲算力轉（BigQuery / Snowflake / Databricks），原始資料保留、轉換可重跑。Storage ⇄ Process 順序因此可換。
- **DE 的價值**：把業務需求變成可運作的資料流——程式碼正確，且有高品質資料持續進來。

| 階段 | 常見選項 | Dex 的原則 / 提醒 |
|---|---|---|
| ① Source | API、爬蟲、檔案下載（Kaggle CSV / zip）、DB 匯出、Streaming（進階） | 有 API 就別用爬蟲；爬蟲不能寫死（參數抽成 function） |
| ② Ingest | 手動 requests（POC）、Airflow / cron / GitHub Actions、Kafka / PubSub、Webhook + Cloud Function | pipeline 的價值在「自動」，能排程就排起來 |
| ③ Storage | Raw/Bronze（MongoDB / GCS / S3 / parquet）→ Silver（MySQL / Postgres）→ Gold（BigQuery / dbt model） | **Raw 永遠保留**，只在 Silver / Gold 清洗；不要每次覆蓋 |
| ④ Process | pandas（<10GB）、SQL、dbt、Spark、LLM API | 不要 1000 行 notebook，拆 staging → intermediate → mart |
| ⑤ Serve | Streamlit / Tableau / Looker、FastAPI、LINE / Discord Bot、報告、推播 | 跟著「使用者怎麼用」走，不是「我會什麼」 |
| ⑥ Observe | Airflow UI / Sentry、dbt tests / Great Expectations、Uptime Robot、Billing、webhook alert | 最常被忽略，卻是從「能跑」變「能放心交給你」的關鍵 |

### 工具選型三個思考點

| 思考點 | 分支 |
|---|---|
| 資料規模（每天進多少） | <1GB → MySQL / MongoDB；1–100GB → BigQuery / PostgreSQL；>100GB → BigQuery / Snowflake / Spark |
| 更新頻率 | 每天 / 每週 → cron；每小時 / 15 分 → Airflow；即時 → Kafka / PubSub |
| 使用者是誰 | 工程師 / 自己 → Python 腳本或 SQL；非技術同事 → Streamlit / Tableau；一般用戶 → LINE Bot / 網頁；其他系統 → FastAPI |

> 前提：團隊專題**優先用課程所學工具或其進階**（Python、Pandas、SQL / NoSQL、Airflow、Docker 等），選型樹是理解用。

---

## 3. 小組 10 階段流程（TibaMe 20 週）

節奏：前 8–9 週基礎課＋專題前置（每週 2–3 hr）；後 11 週進階課＋專題實作（每週 8–12 hr）。前置期各階段時間刻意重疊、可並行。

| 階段 | 週次 | 目標 | 主要活動 | 產出物 | 卡關處方 / 建議 |
|---|---|---|---|---|---|
| ① 啟動 | W1–3 | 確定組員、選定題目 | 互相認識＋強項盤點（1.5h）、8 情境選題（1.5h） | `team-charter.md`（強弱項、可投入時數）、`topic-decision.md`、共用通訊工具 | 都選熱門題→看痛點地圖，哪題對「你過去職涯」最有故事；選題比技術重要；不要為了平均而平均，讓強項極大化 |
| ② Discovery | W2–4（約 8h） | 誰在用、為什麼用、痛在哪、資料怎麼來 | 訪談 / 看報告（3h）、persona 與市場（2h）、盤點資料來源（3h） | `docs/user-persona.md`（1–2 個具體 persona）、`docs/pain-points.md`（3–5 個可量化痛點）、`docs/data-inventory.md`（含取得難度） | 沒目標用戶→你自己就是用戶；痛點抽象→加數字「每天平均花 X 分鐘做 Y」 |
| ③ 設計 | W4–6 | 畫出系統長什麼樣 | draw.io / Excalidraw / Mermaid / dbdiagram.io | `docs/architecture.md`、`docs/data-flow.md`、`docs/er-diagram.md`、`docs/api-contract.md` | 「whiteboard 1 小時省 10 小時 code」；不會畫→抄藍圖換工具名；schema 卡→先想要回答什麼 query 反推欄位 |
| ④ 分工 | W6（約 2h） | 每人清楚 ownership，但沒人是孤島 | 4 角色＋RACI | RACI 矩陣、模組 owner | 每人主擔 1＋副擔 1；每人最多 1 個 R；沒人想當 Product Owner→面試最常被問，一起扛 |
| ⑤ 環境 | W7–9 | 基建一次架好 | repo、channels、PM 工具、branch 規範 | repo（一般版 / 專業版）、`main/develop/feature/*/fix/*`、第一個 PR 跑通 CI、README / PLAN / task.md 三件套 | 慢慢架，不要硬塞一天 |
| ⑥-1 Sprint 1 | W10–12 | 端到端跑通最小可行 pipeline | Source → Storage → 一個結果（哪怕 print） | 一條完整的線 | 「醜沒關係」；別花 3 週把爬蟲調到完美，先跑通整條再優化 |
| ⑥-2 Sprint 2 | W13–15 | 擴充＋自動化 | 加排程、資料源、分析 | 每天自動跑、dashboard 雛形 | 進度落後→**砍 scope 不是熬夜** |
| ⑥-3 Sprint 3 | W16–18 | 打磨＋監控 | UI、監控、文件 | 別人能自己跑起來 | 有人神隱→馬上 escalate 給教練；sprint 中禁 refactor |
| ⑦ 整合 | W17–19 | 各模組串成一條 pipeline | `docker compose up` 一鍵起、schema validation、端到端跑一次、CI 過 | 整合 checklist 全勾 | 模組對不上→回看 api-contract，沒有就現在補；跑一次 10 分鐘→先有再快 |
| ⑧ 上線 | W17–19 | 讓別人在 demo 之外能自己玩 | Cloud Run / Vercel / Render / Streamlit Cloud / GCE | live URL | 監控最低門檻：失敗至少 LINE / Email 通知、dashboard 顯示「最後更新時間」、health check endpoint |
| ⑨ Demo | W19–20 | 兩種版本都能講 | 業務版 5 分、技術版 10 分；組內 dry run ≥3 次 | 投影片 <10 頁、螢幕錄影備份 | 每段有人 backup；翻車那位不講話 |
| ⑩ 復盤 | W20 | 把「做過」轉成「學到 X」 | 個人 final retro → 小組 retro | `docs/retro/final.md` | 「復盤完才算真正畫上句點」 |

### 4 種角色（角色是職責不是人，一人可兼）

| 角色 | 職責 | 適合的人 |
|---|---|---|
| Data Ingest Owner | 資料來源 / API / 資料品質 | 細心、抗錯耐性高 |
| Pipeline Owner（ETL Engineer） | Airflow / Docker / 排程 | 喜歡自動化 |
| Analytics Owner | SQL / pandas / 模型 | 數理直覺好 |
| Product Owner | Streamlit / API / UX | 會講故事、會 demo |

### Demo 兩種結構

| 業務版 5 分（老闆 / 面試官 / 投資人） | 技術版 10 分（同行 / 教練 / 工程主管） |
|---|---|
| 30s 痛點「現在要花 X 分鐘 / 元」→ 60s 方案 → 2m live demo → 60s 影響「省了 / 多了 X」→ 30s CTA | 30s 問題＋用戶 → 2m 架構＋選型理由 → 3m live demo → 2m 踩過的 3 個坑 → 1m 監控＋復盤 → 1m 未來迭代 → 30s Q&A |

---

## 4. 前置期每週節奏與小作業（學員最常問「現在該做什麼」）

手冊章節順序 ≠ 週次順序：第 3 週讀 04 痛點地圖，第 4 週才讀 03 團隊流程。

| 週 | 章節 | 小作業（做完打勾＝本週過關） | 交付 |
|---|---|---|---|
| 0 | 00 敏捷＋工具選型 | 組隊；開溝通群（≥ #announce / #standup / #help / 模組 channels）＋看板；約 daily check-in 時間（文字非同步）；約每週固定 Planning / Review 時段（30–60 分） | 工具開好 |
| 1 | 01 GitHub SOP | 每人帳號＋2FA＋git config（email 與 GitHub 一致）；團隊 repo 建好、每人能 push；branch protection（PR + 1 approve） | repo URL 貼 LINE 群 |
| 2 | 02 管線藍圖 | 畫 6 階段架構圖，每階段標工具＋一句為什麼（Mermaid 可請 AI 協助） | 架構圖 |
| 3 | 04 痛點地圖 | 選定題目；寫 1 個具體 persona（禁止「年輕人」）＋3–5 個可量化痛點；一句「從 A 變成 B」 | `docs/persona.md` 的 PR 連結 |
| 4 | 03 10 階段 | 開一次團隊會議：固定 sync 時間、看板建 3–5 張卡且每張有負責人、決定 po 到群組 | 看板截圖＋sync 時間 |

實作期常駐：天天看 05 協作 SOP；Sprint 2 結束先用 07 評分表 self check 一次（寫 `docs/self-check-w{N}.md`、框出最低 2 個維度寫進 task.md）；demo 前 24 小時再跑最後 checklist。

**Repo 兩種開法**：🌱 一般版＝組長個人帳號開 public repo＋加 collaborator（限制：owner 只有組長、銷帳號會死連結、commit graph 集中組長；降風險：README 註明代管、`Co-authored-by:`、每人定期 fork 備份）。🚀 專業版＝Free Organization 主 repo＋每人 `{username}/portfolio` 互相連結，履歷只放個人 portfolio。**沒選擇障礙就選一般版**，之後可 Transfer。命名：`{組名}-{專題名稱}-{情境簡寫}`。

---

## 5. 敏捷與工具選擇

**方法：Scrumban**（Scrum 5 個 ceremonies 對 3 人組太重、純 Kanban 沒節奏易拖延；業界 3–10 人新創最常用）。
「敏捷不是『速度快』，是『願意接受我們一開始想的不完全對，然後快速調整』。」

| 項目 | 做法 |
|---|---|
| 三大支柱 | 看板＋WIP limit、Sprint Planning、Sprint Review |
| 輔助 | 異步 daily check-in（昨天 / 今天 / 卡；沒卡也要明說「沒卡」）、學期末 final retro |
| 不做 | 每週 sprint retro（易疲乏）、Velocity / Story Points |
| Sprint Planning（45–60 分，嚴格計時） | 0–10 回顧上週 → 10–30 從 Backlog 拖卡（owner / 估時 / acceptance criteria）→ 30–50 風險（誰忙、API 額度）→ 50–60 結論 |
| Sprint Review（60 分） | 0–15 PO 講做了什麼 → 15–35 小 demo → 35–50 Q&A → 50–60 決定進下個 backlog 的反饋 |
| 估時與點數 | S 半天＝1 點、M 1–2 天＝2 點、L 3 天＝4 點；超過 L 要拆 |
| 完成率 | 完成點數 / 預算點數；≥80% 可加量、60–80% 維持、<60% 減量並檢討是否估太樂觀 |
| 優先級 | P0 must（不准砍）、P1 should、P2 nice、P3 delight；砍順序 P3→P2→P1；P0+P1 ≤ 預算 70% |
| 該砍 trigger | sprint 過半剩餘 >60% → 砍 P3 / P2；P0 卡 in-progress >2 天 → 拉教練＋拆卡；連 2 sprint <60% → 直接砍 30%；demo 前 1 週 P1 未完 → 砍光 P2/P3 |

「砍 scope 不是失敗 — 是聰明。業界 PM 一年砍幾十次。」

| 類別 | 推薦 | 理由 |
|---|---|---|
| 溝通 | **Discord**（備案 LINE＋Google Drive） | 學習曲線適中、多 channel / thread、機器人整合；Slack 免費版歷史只留 90 天；LINE 無 channel / thread |
| 專案管理 | **Notion**（文件＋task＋會議混合）或 **Trello**（5 分鐘上手的乾淨看板）；全員熟 GitHub 則 GitHub Projects | 選型樹：第一次跑→Trello；愛寫文件→Notion；99% 在 GitHub→GitHub Projects；想體驗業界→Jira |
| Trello 五欄 | Backlog / This Sprint / In Progress（限 3）/ Review / Done | 卡片：動詞開頭 title、acceptance criteria、一位 assignee、due date、labels |
| 不能省的 5 件事 | 每 task 有主負責人、有簡單完成條件、狀態流轉清楚、每週檢查 In Progress（超過一週＝暗藏問題）、commit / PR 引用 issue 編號 | — |
| nice-to-have 自動化 | GitHub → Discord 通知、standup bot、Sentry → #alerts、Actions 週報 | — |

---

## 6. 協作 SOP 精華（Git / PR / Code Review）

「協作能力比技術深淺更難在履歷上看見，但在面試裡卻會被一眼看穿。」面試官看 GitHub 順序：README → commits（是不是只有 update / fix）→ PR（有沒有 review）→ Issue。

| 主題 | 規範 |
|---|---|
| Repo 起步 | `.env.example`、`.gitignore`、`README.md`、`task.md`、`requirements.txt`；實作期再加 `src/`（4 角色各一目錄）、`data/`（gitignore，只留 README 說明取得方式）、`docs/`、`tests/`、Dockerfile、compose |
| Branch | `main` 永遠可部署、禁直推；`develop` 主幹；`feature/ fix/ docs/ refactor/{owner}-{topic}` |
| 規矩 | branch protection（PR + ≥1 approve）；一個 PR 一個小目標；PR 一週內合掉；不准 force push 共用 branch |
| Commit | Conventional Commits：`feat / fix / docs / refactor / test / chore / perf / ci`，body 寫原因，footer `Closes #12` |
| PR template | 做什麼（一句）/ 為什麼 / 怎麼測試 / 截圖或 log / 影響範圍（schema、API、.env）/ Reviewer checklist / `Closes #` |
| Issue | 三種：bug / feature / question；「Issue 不是聊天，是未來翻得到的決策紀錄」 |
| Review 順序 | commit message → PR description（有回答為什麼嗎）→ 邏輯正確性 → 品質（命名 / 拆分 / DRY）→ 測試 |
| Review prefix | `[blocker]` 必改、`[suggest]` 建議由你決定、`[nit]` 龜毛不擋合 |
| 不該做 | 只說「寫法不好」不給改法；在 PR 開戰（改語音）；自己 approve 自己 |
| CI 最低門檻 | pytest、ruff / black、（mypy）、docker compose 起得來；**每個 PR CI 不過不准合** |
| 文件 | README（W0）、PLAN.md（W2 設計完）、task.md（實作開工）、GUIDE.md（給接手者，半途與最後更新） |
| Secret 外洩 | 立刻 revoke → BFG 清歷史 → 重發；預防：`.env.example`＋`.env` 進 gitignore |

---

## 7. 8 情境地圖

選題原則：「你不是在選好玩的題目，是在選想去的業界。」選一個跟過去工作 / 生活最有連結的——「故事比技術重要」。不確定就看哪題「過去職涯」最有故事。

| # | 情境 | 痛點（業務） | 業界職位 | 建議 pipeline（Source → Storage → Process → Serve） | 往屆作品 | 雷區 |
|---|---|---|---|---|---|---|
| 1 | 餐飲連鎖 POS 庫存＋熱銷 | 老闆要「今天該不該叫貨、叫多少」不是月報 | 餐飲集團 DE / 數據分析師、POS SaaS DE | Kaggle / UCI＋Faker 模擬 POS → MySQL → pandas（7 日 SMA＋安全庫存）→ Streamlit 店長 / 區督 / 總部三視圖；Airflow 6am | 美味餐廳隨拾取、食喪地圖、找咖小幫手、水果價格預測 | 只做銷售報表太薄，要「決策建議」；只用 Kaggle 要混模擬即時資料 |
| 2 | B2C 電商競品比價＋動態定價 | 「我的價格在市場排第幾、要不要調」 | 電商平台 DE、品牌商數據組 | momo / PChome / 蝦皮爬蟲（Scrapy / Playwright）→ MySQL → SQL＋embedding SKU 對齊＋定價建議 → FastAPI＋Streamlit；Airflow 每小時 | 水果價格預測 | robots.txt、速率 <1 req/s；SKU 用「商品名稱完全相等」對齊會對不到 95%；沒存歷史價格 |
| 3 | 數位金融客訴效率 | 「客訴量上升嗎、哪類、誰負責」 | 純網銀 / 銀行數位部 DE、保險科技、FinTech | CFPB API＋PTT → MongoDB → LLM 分類＋情緒（1–5 紅黃綠）→ Streamlit＋LINE 推播 SLA 警示；LLM cost watcher | 反詐騙 LINE Bot、打造自己的 ETF | 每筆用大模型成本爆；沒去識別化；分類精度沒評估；prompt hardcode |
| 4 | 音樂串流推薦 | 「用戶留多久、什麼讓他們多停留」 | 串流音樂 / Podcast / 串流影音 DE | Spotify API（OAuth）＋Charts＋Kaggle KKBOX → MongoDB → pandas＋sklearn（CF → audio feature 分群 → embedding）→ Streamlit＋Spotify login | KKBOX & Spotify 比較、SteamTrend | 推「最熱門」是排行榜不是推薦；OAuth token 過期翻車；features 沒 normalize |
| 5 | 叫車交通熱點＋調度 | 「司機被叫不到的時段在哪、要不要加價」 | 叫車 / 物流 / 外送平台 DE | NYC TLC＋台北 ETC → BigQuery（H3 hex、15 分 bucket）→ BigQuery GIS＋LSTM / Prophet / XGBoost → FastAPI＋Streamlit 地圖 | 國道車流 LSTM、國旅推薦 | BigQuery 沒分區 cost 爆；lat/lng 直接 group by；訓練資料 <1 個月；地圖點太多 |
| 6 | 求職媒合職缺趨勢＋薪資 | 求職者要「學這技能薪水多少、市場缺多少人」 | 人力銀行 / HR Tech DE、企業 HR 數據組 | 104 / CakeResume 爬蟲＋勞動部統計 → MongoDB → LLM 技能抽取＋薪資 parsing → Streamlit / Looker Studio | 求職 360°、求職求值 | 爬太快被擋；taxonomy 太細；沒去重；「面議 / 年薪」沒 normalize；不要做成 SaaS 開賣 |
| 7 | 媒體新聞輿情監測 | 「品牌今天被罵了嗎、誰在罵、要不要回應」 | 公關公司 / 媒體 / 行銷代理 DE | NewsAPI / RSS / PTT / Threads → MongoDB → 兩段式 LLM（便宜模型篩、疑似負面再用貴模型）＋主題 cluster → Streamlit＋LINE Bot；Airflow 每 30 分 | 社群輿情、夜市水軍王、遊戲輿情、巴哈姆特輿情、NASA LineBot（最多參考的情境） | LLM cost 控制；閾值固定（假日 baseline 不同）；轉載沒去重；RSS source health |
| 8 | 不動產房價趨勢 | 「這區明年漲跌、現在進場合不合理」 | 房仲 / 房屋平台 DE、PropTech、銀行房貸部 | 實價登錄 zip＋data.gov.tw POI＋TGOS 地址轉座標 → MySQL＋PostGIS → pandas＋Prophet / LSTM＋周邊評分 → Streamlit＋Folium | 房價走勢密碼、六都寵物便利度 | 用總價不用單坪價；沒去異常值；沒考慮季節性；預測要標「模型估算」不可承諾漲 |

每個情境文件都提供同一組素材，帶學員時可直接引用：一句話定位、量化目標、Mermaid 六階段圖、資料源表、10 階段重點、三個 sprint 的具體範圍、範例 commit log / PR / issue / retro、進階挑戰、雷區、法律與倫理、20 週時程。

**跨情境共通 5 痛點**（有對到面試就有得講）：資料品質（dbt tests / GE、資料健康度）、管線可靠性（排程＋失敗推播＋retry）、資料新鮮度（最後更新時間＋過期警示）、Cost 控制（算過 cloud bill）、權限 / 個資（去識別化、敏感資料不入 git）。

**面試 5 秒講法公式**：「我做了一個 [情境]，從 [來源] 抓資料，每天自動跑出 [產出]。」

---

## 8. FAQ 與常見卡關（Dex 的回法濃縮）

| 學員問題 / 狀態 | Dex 的回法 |
|---|---|
| 手冊太多看不懂，不知怎麼開始 | 先知道自己在哪個階段，只看那階段的章節。流程簡化成三段：前置（架構練習→組隊→選題→GitHub→工具）、實作（每週分配與追蹤）、收尾（整合→優化→Demo→演練）。前置期先「完成兩件事、開始一件事」：建 GitHub、選好溝通與 PM 工具；同時開始想題目 |
| 先建 GitHub 還是先選工具？ | 建議先定溝通與 PM 工具，再用它們協作建 GitHub，把 GitHub 建置當團隊「熱身任務」 |
| 題目怎麼定？ | 漸進收斂：W2 學藍圖時發散 3–5 個方向 → W3 用痛點地圖收斂 → W4 排協作節奏 |
| Scrumban、WIP、Planning 名詞太多 | 先抓一個核心：每週跟組員對一次進度＋用看板追誰在做什麼；跑起來自然懂 |
| 不知道下一篇看哪 | 回 README「手冊閱讀方法」表，用「你的狀態」找必讀與下一步 |
| Author identity unknown / Permission denied (publickey) / protected branch | 回 git config；SSH key 沒加；不能直推 main，開 branch 走 PR |
| 我們沒有目標用戶 / 找不到資料 / 痛點太抽象 | 你自己就是用戶；看藍圖 Source 段與 8 情境；加數字 |
| 大家都想當 lead / 有人攬太多 | 沒有 lead 只有 4 種角色；每人最多 1 個 R |
| 進度落後 / 有人神隱 / 一直 refactor | 砍 scope 不熬夜；馬上 escalate 給教練；sprint 中禁 refactor |

README 的閱讀路徑表（狀態 → 必讀 → 下一步）：沒用過 GitHub → 01 → 05；沒組好隊 → 03 啟動段 → 00；選題中 → 04 → scenarios；不知如何設計 → 02 → 對應 scenario；動工卡關 → 05＋00 → 03 衝刺段；快 demo → 07 → 06；想升級履歷 → 06 → 07 self-check；第一次來 → README＋看 1 部學長姐 demo（回答：解什麼業務問題、用哪些技術、我能做得更好的地方）。

---

## 9. 復盤

- **預設不做 sprint retro**，只做學期末 final retro（專題結束後 1 週內）；進階組可加 KPT / 4Ls / Sailboat。
- 個人 final retro 六題：3 件最有價值的事（寫思維 / 流程 / 心法，不寫工具名）、做得最好的 1 件、重來會改的 3 件、下次想加強的技能（具體可量化）、給每位組員 1 條 feedback、給 3 個月後自己的信。
- 小組：共識 3 件、分歧（不強求統一）、一句總教訓給下一屆。存 `docs/retro/final.md`。
- 「寫『都很好』『都很順』＝沒做 retro，誠實才有用。」
- 模板全文見 `templates/retro-and-resume-checklist.md`。

---

## 10. 課後與履歷升級路徑

| 時間 | 重點 | 動作 |
|---|---|---|
| 第 1 個月 W1 | repo 重整 | 重寫 README、加 20 秒 GIF、審 commit message（爛的 squash）、加 LICENSE |
| W2 | 架構圖補齊 | draw.io 重畫、`docs/architecture-decisions.md`（ADR：為什麼 A 不選 B）、`docs/lessons-learned.md`（5 個坑） |
| W3 | 寫 1 篇 blog | 「我做 X 專題踩過的 N 個坑」，發 Medium / DEV，分享到 LinkedIn / Threads |
| W4 | 影片化 | 5–10 分鐘業務版 demo，YouTube unlisted，嵌履歷 / LinkedIn |
| 第 2–3 個月 | 技術深化（選 1–2） | dbt＋BigQuery、Streaming、MLOps、雲端認證、LLM Engineering（RAG / Agent） |
| 第 4–6 個月 | 拓展（選 1） | 擴成 SaaS（5 人試用、收 $99 / 年驗證）、Open source 化（PyPI）、加入 OSS（good first issue） |

- 面試三種長度：5 秒（電梯）/ 30 秒（痛點→方案→影響→技術棧）/ 5 分鐘（問題→架構與選型→demo→2 個坑→未來）。
- 履歷專案欄：✅ 動詞開頭 ✅ 有數字 ✅ 有具體技術 ✅ 連結可驗證；❌「使用 Python 完成了一個專案」「學習了很多」「跟組員合作」。
- 「這是業界 senior 看你的差距 — 不是工具，是這份累積。」

---

## 11. 帶學員時的注意事項（手冊內不一致處）

| 項目 | 不一致 | 建議口徑 |
|---|---|---|
| Sprint 長度 | 00 章 Planning 每週一次、Review 標「每 2 週」；03 章寫「雙週 sprint」但又「3 個 sprint × 3 週」 | 實作期 3 個 sprint × 3 週；Planning / Review 每週固定一次 |
| Sprint retro | 05 章日程有「每週日 retro」；00 / 06 章預設不做；07 評分表卻要求「Sprint retro 紀錄至少 3 次」 | 預設 final retro；想拿協作 / 復盤滿分的組加做 sprint retro |
| WIP limit | 00 章看板圖寫「限 2/人」、規則寫「最多 3 張」 | 每人 ≤3 |
| 章節編號 | 07 評分表把 02 藍圖稱「01」、05 SOP 稱「03」 | 以檔名為準 |

---

來源（GitHub base：https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook）：
`README.md`、`00-agile-and-tools.md`、`01-github-setup-sop.md`、`02-pipeline-blueprint.md`、`03-team-process.md`、`04-pain-points-industry-map.md`、`05-collaboration-sop.md`、`06-retro-and-after.md`、`07-demo-day-rubric.md`、`FAQ.md`、`scenarios/scenario-1-restaurant-pos.md` ~ `scenarios/scenario-8-realestate.md`
