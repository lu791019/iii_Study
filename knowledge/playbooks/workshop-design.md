# 實作工作坊設計手冊（Workshop Playbook）

> 這份手冊讓你照 Dex 的方式，設計一場 3 小時左右的「每人做出一個可跑 MVP」實作工作坊。
> 範本來源：TibaMe 雲端資料工程師班的期中「LLM × DE MVP Builder 實戰工作坊」（midterm-mvp-template）。

## 1. 設計心法（先讀這段）

| 原則 | 意思 | 在範本裡怎麼落地 |
|------|------|----------------|
| 低門檻、高天花板 | 保底讓每個人都有產出，強的人不無聊 | Starter Notebook 有提示與骨架；Section 9-10（Dashboard）當回家作業 |
| 個人實作、團隊智囊 | 每人獨立做完整條線，組內互相 debug | 13 人分 3 組；同組可選不同題，交流時看見不同資料與清洗策略 |
| 做到哪就 Demo 到哪 | 不要求全部完成，降低轉職者的挫折感 | 每段都有「基礎完成」與「⏱ 時間不夠？只做 X」 |
| 一個作品持續升級 | 工作坊產出不是一次性練習，是後續課程的種子 | v0 Notebook → v1 repo → Docker → Airflow → GCP 的版本階梯 |
| 需求由講師給定 | 初學者用 AI 釐清需求太花時間，把時間留給 ETL 和 LLM | 每題預寫 `requirements_spec.md`（v4.0 從「四關卡」改成「三關卡」的理由） |
| 先求能跑，再求漂亮 | 任何外部依賴都要有備案 | LLM 有 fallback 規則版、CSV 放 repo 裡、SQLite 不需網路 |

定位用語：「引導式挑戰賽」——比純工作坊有挑戰感，比黑客松安全。名稱刻意叫「挑戰賽」不叫「黑客松」，避免初學者覺得「我不夠格」。

## 2. 時間盒設計（以 3 小時為例）

骨架：**講解 10–20 min → 實作 25–45 min → 交流 5–10 min**，循環三次，最後 Demo + 收尾。

| 時段 | 分鐘 | 類型 | 內容 | Notebook |
|------|------|------|------|---------|
| 13:30-13:50 | 20 | 講解 | 開場：投影 demo 一條完整 pipeline + 規則 + 題目導覽 | |
| 13:50-14:15 | 25 | 實作 1 | clone → 環境 → 選題 → 讀需求 | Section 0 |
| 14:15-14:20 | 5 | 交流 1 | 組員依 spec 的「交流引導」互問 4 題 | |
| 14:20-14:30 | 10 | 講解 | ETL + 各題清洗難點 | |
| 14:30-15:15 | 45 | 實作 2 | ETL：raw → cleaned → SQL 統計（最長一段） | Section 1-3 |
| 15:15-15:25 | 10 | 講解 | LLM prompt + fallback + API | |
| 15:25-15:55 | 30 | 實作 3 | LLM → 文件 → FastAPI | Section 4-8 |
| 15:55-16:05 | 10 | 交流 2 | 閃電秀預演：組內 2–3 人各 3 min | |
| 16:05-16:20 | 15 | Demo | 每組推派 1 人 × 3 min，嚴格計時 | |
| 16:20-16:30 | 10 | 收尾 | 升級路線 + Next Step + Q&A | |

時間分配經驗值（早期全天版企劃）：講解約 35%、實作約 43%、Demo 約 17%、休息約 5%。第一段講解可以長（要跑完整 demo），之後每段講解壓在 10–20 分鐘。

**設計規則**

- 最長的實作段給「核心技能」（這裡是 ETL），不是給新奇的部分（LLM）。
- 每段實作開始前，投影片上要有一張「實作 N Time」：時間、Section 範圍、基礎完成、進階。
- 交流不是休息：一定給「問什麼」的引導題，寫在每題 spec 底部。
- Demo 前一定有組內預演，正式 Demo 才會精煉。

## 3. 「段落 → 交付物 → ✅ 完成標準」模式

每個段落（Step / Section）都寫成三件事。學員看任何一頁都知道「做什麼、交什麼、做到哪算完成」。

```markdown
## Step 2：ETL Pipeline（45 min）
> 對應簡報 Slide 15 ｜ Notebook Section 1-3

| Section | 做什麼 | 交付物 |
|---------|--------|--------|
| 1 | Extract — 讀 CSV → 寫入 raw 表 | pipeline.db 的 raw 表 |
| 2 | Transform — 清洗 → 寫入 cleaned 表 | cleaned 表 |
| 3 | 統計分析 | processed/*.csv |

> ✅ 基礎完成：pipeline.db 有 raw + cleaned 表、processed/ 有統計 CSV、有一個能 Demo 講的數字
```

同一套標準要同時出現在四個地方，而且文字一致：README、START_HERE 速查卡、投影片、Notebook 每段開頭（🎯 完成這段後你應該有）。

### Notebook 內部的結構

| 元素 | 寫法 |
|------|------|
| 段落開頭 | `### 🎯 完成這段後你應該有：` 條列產物 + `> ⏱ 時間不夠？只做 Step X-1 和 X-4` |
| 每個 Step | `### Step 1-1 🟢 簡單：讀取 CSV` → 💡 概念一句 → 💡 **範例**（用「學生名單」等無關情境示範語法）→ 🎯 現在對你的資料做同樣的事 → ✅ 預期：2,000 筆、9 個欄位 |
| 難度標記 | `🟢` 開放式（範例教語法，自己套用）、`🟡` 半骨架（別的情境示範，你翻譯成自己的欄位）、`🔴` 完整骨架（填關鍵欄位名和 SQL）、`（不需要改）` 直接跑 |
| 檢查點 | `### 🏁 清洗檢查點`、`### ✅ 檢查點：三表驗證`——直接跑，全部 ✅ 才往下 |
| 進階探索 | Section 3 後放「💡 你還可以分析什麼？」5 個點子，並說明「都可以用一個 GROUP BY 搞定」 |
| 封裝難的部分 | API 呼叫、批次分析寫成 helper（`llm_analyze()`、`run_batch_analysis()`），學員只負責「呼叫、看結果、寫入 DB」 |

### 兩種難度的 Notebook

- `pipeline_starter.ipynb`：程式碼留 `???` 讓學員填空。
- `pipeline_challenge.ipynb`：同樣結構，但只給「# 相關程式碼：」註解，學員自己寫完整一行。
- `_instructor/solutions/topic_N_solution.ipynb`：完整答案，開頭標「⚠️ 講師用完整答案」。

## 4. 題目與需求規格設計

### 題庫可行性檢查（每題開放前都要過）

- [ ] 有講師預備的 sample CSV（範本每題 2,000 筆，從原始資料取樣），避免現場下載失敗
- [ ] 有明確的 raw → processed → LLM output
- [ ] 最低版本能在一段實作時間內完成
- [ ] 有可展示的最後產出（表、標籤、摘要、API、Dashboard）
- [ ] 有 API / 網路失敗時的 fallback
- [ ] 可在後續課程升級成 GitHub / Docker / Airflow / GCP 版本

### 題庫組成

- 6 題對應 6 個產業情境（零售 POS、電商比價、音樂串流、叫車熱點、求職薪資、不動產），每題都對應一種資料顧問角色。
- 難度要有梯度並明說：題 4 要 merge 兩張表「最複雜」；題 6 是中文欄位 + 民國年。不知道選哪題的人，引導選「最直覺」的題 1 或題 5。
- 題目總表固定欄位：`# | 題目 | 資料 | 一句話 | 需求連結`。

### requirements_spec.md 結構

情境 → 你的角色 → 使用者 → 資料來源（檔案表 + 欄位表）→ 今日 MVP Scope（ETL / LLM）→ 預期產出 → 成功指標 → 交流引導。
完整模板與填寫範例見 [templates/requirements-spec.md](../templates/requirements-spec.md)。

重點：

- **情境用老闆的一句話開場**：「老闆說：『我們有大量的銷售交易紀錄，但沒人在分析……』」。
- **使用者要寫到頻率與決策**：「每週看報告，決定進貨策略和行銷重點」。
- **欄位表要標出陷阱**：「注意：含逗號，需轉數字」「民國年格式」。
- **成功指標要可口說**：「能說出銷售額最高的 3 個品類和 3 個國家」＋「建議有數據支撐（不是空話）」。

## 5. Repo 與檔案配置

```
workshop-repo/
├── README.md              ← 完整說明（Step 1-4、架構圖、完成標準、AI 規範、升級表、文件索引）
├── START_HERE.md          ← 現場速查卡：「不用看完整 README，照這張卡走就好」
├── requirements.txt / .env.example
├── data/raw/topic_N/      ← 每題自成一包：spec + CSV + starter + challenge + api.py + app.py + processed/ + output/
├── docs/                  ← README 模板、AI prompts、題庫、資料來源、tech_decision、upgrade_plan、pipeline.mmd
├── scripts/               ← 自動測試所有 solution notebook + API（考前回歸測試）
├── _advanced/             ← 後續課程才用的 Dockerfile / compose
└── _instructor/           ← 講師引導手冊、講稿、TA brief、學員手冊、事前通知、評分表、追蹤機制、簡報、solutions
```

- 每題資料夾自成一包，學員只需要看一個資料夾。
- `scripts/test_midterm_topics.py`：把 solution notebook 複製到暫存目錄執行，再啟動 FastAPI 打 endpoint——確保 6 題都能跑通再上課。

## 6. 環境：Colab 優先、本地備援

| 情境 | 做法 |
|------|------|
| Colab | Section 0 偵測 `google.colab`，自動 `git clone` repo 並 `os.chdir` 到該題資料夾 |
| 本地 | `jupyter notebook data/raw/topic_N/pipeline_starter.ipynb`，或 VS Code |
| API Key | 先讀 Colab Secrets（左側 🔑），再讀本地 `.env`；都沒有就印「⚠️ 無 API Key，使用 fallback（不影響完成度）」 |
| 儲存 | SQLite（Python 內建、零安裝、離線可跑） |
| API | FastAPI：建議另開 Terminal 跑 `uvicorn api:app --reload --port 8000`；notebook 內 threading 啟動易遇 port 被占用 |
| 事前測試 | 事前通知請學員在 Colab 跑 `import pandas as pd; print(pd.__version__)` |

## 7. 備案策略

| 風險 | 備案 |
|------|------|
| 沒有 / 用完 LLM API Key | `llm_analyze(text, api_key)`：有 key 呼叫 API，失敗或沒 key 自動走 `_llm_fallback()` 關鍵字規則版，回傳同樣的 JSON 結構 |
| LLM 回傳不是純 JSON | helper 只取 `{...}` 區段再 `json.loads`；prompt 加「請只回傳 JSON」 |
| 跑太慢 | `BATCH_SIZE` 改 20；預設只分析前 50 筆 |
| Wi-Fi 不穩 | 資料都在 repo、SQLite 離線可跑；講師帶備用 API Key |
| Colab 打不開 | 本地 Jupyter 或 VS Code |
| 有人完全做不出來 | 開 solution 帶著跑，確保至少有產出；api.py / app.py 本身就是 solution |
| 時間不夠 | Demo 從 3 分鐘縮成 2 分鐘；FastAPI 改回家做 |

原則：**fallback 版本要讓流程完整跑通**——學員用規則版一樣能做出 analyzed 表，完成度不受影響。

## 8. 講師材料組（`_instructor/`）

| 文件 | 給誰 | 內容 |
|------|------|------|
| facilitator_guide.md | 主講 + TA | 活動概覽、T-1 週到當天的準備清單、詳細時間表、開場逐字稿、巡場重點、卡關引導句、Demo、收尾稿、備案、物資 |
| speaker_notes.md | 主講 | 依 Slide 編號的講稿，每張標秒數（30s、45s） |
| ta_briefing.md | TA | 「5 分鐘讀完就能上場」：角色、時間軸、巡場 SOP、介入／不介入、QA 表、計時、不需要做的事 |
| student_handbook.md | 學員 | 活動是什麼、流程、Step by Step、AI 規範、遇到問題怎麼辦、你會帶走什麼 |
| student_pre_event_notice.md | 學員 | 帳號與 Colab 準備、上午報告指引（核心 1–5 項 + 延伸 6–8 項）、可複製的報告模板、FAQ |
| project_report_rubric.md | 講師 | 上午專題報告評估表 + 回饋模板 |
| final_project_tracking.md | 講師 | 期中到期末的追蹤節奏、週報模板、燈號 |
| solutions/ | 講師 | 每題完整答案 |

講師手冊的通用模板見 [templates/instructor-guide.md](../templates/instructor-guide.md)。

### 巡場與引導

- 以 T+10 / T+25 / T+40 分鐘設觀察點，每點寫「觀察什麼」和「介入時機」。
- 引導句用提問，不直接給答案：「你現在在哪個資料夾？用 `os.getcwd()` 看看」「先用 `df.dtypes` 看看欄位是什麼型別」。
- TA 的介入訊號：盯螢幕不動超過 3 分鐘、一直 Google、表情困惑不舉手、整組都卡（叫講師）。
- 不介入：正在打字、在跟組員討論、在用 ChatGPT。
- TA 不做：講課、幫打程式、評分。

## 9. 評分

### 工作坊 Demo（內部，不公開排名）

| 維度 | 權重 |
|------|------|
| Pipeline 完成度（ETL + SQLite 三表） | 30% |
| AI 應用品質（prompt + 結果） | 25% |
| 作品打包（README + upgrade_plan） | 20% |
| 洞察品質（有數字支撐的發現） | 15% |
| Demo 表達（3 分鐘講清楚） | 10% |

Demo 結構（每 30 秒一段）：問題 → 資料與清洗 → LLM 做了什麼 → show output → 下一步升級 → 快問快答。
講師點評每人挑一句，例如「如果資料量變 10 倍，你的 pipeline 會怎麼調整？」「這個 LLM 分類結果你同意嗎？有沒有分錯的？」

### 專題報告 rubric（1–4 分 × 5 維度，總分 20）

| 維度 | 權重 | 4 優秀長什麼樣 |
|------|------|---------------|
| 問題定義 | 25% | 問題精準，有商業價值論述 |
| 資料規劃 | 25% | 已做 EDA，了解品質與限制 |
| 技術架構 | 25% | 架構合理且有備案 |
| 分工與時程 | 15% | 有里程碑和驗收標準 |
| 表達清晰度 | 10% | 流暢且有圖表輔助 |

回饋三段式：**肯定**（你們的 ___ 做得很好）→ **建議**（如果能再 ___）→ **連結下一步**（下午可以先用 MVP 驗證 ___）。

## 10. 解答（solution）處理方式

- 答案放 `_instructor/solutions/`，README 標明「講師用（學員不需看）」；repo 是公開的，所以把它視為「卡關時的最後一道保底」，不是秘密。
- 部分元件刻意直接給 solution：`api.py`、`app.py`（Dashboard）。目的是讓學員「改路徑就能跑」，把時間留給核心段落。
- 講師開場用 solution 投影跑一次完整 pipeline，讓學員先看見終點。
- 上課前用自動化腳本跑過所有 solution，確保每題端到端可跑。

## 11. 課後追蹤（期中 → 期末）

| 頻率 | 方式 | 內容 |
|------|------|------|
| 每週 | 各組在群組回報 | 週報模板，1 分鐘填完 |
| 每 2 週 | 講師文字回饋 | 標出風險 |
| 期末前 2 週 | 線上 check-in 15 min/組 | 確認能 Demo、解決卡關 |

週報模板：

```
📋 第 __ 週進度（__月__日）
組別：_____
✅ 本週完成：
🔜 下週目標：
🚧 卡關/風險：（沒有就寫「無」）
📊 完成度自評：_____ %
```

- 講師只看「卡關/風險」欄。
- 燈號：🟢 正常（按讚不介入）／🟡 落後 1 週或卡關仍在嘗試（群組給方向）／🔴 落後 2 週以上或卡死（約 15 分鐘線上 check-in）。
- 里程碑以「驗收標準」寫：第 2 週結束前一定要有東西能跑。
- 收尾時要明講「今天的個人 MVP 就是期末團專的原型」，並預告期末多兩個評分項：團隊協作、技術升級深度。

## 12. AI 使用規範（工作坊通用）

| 用法 | 是否允許 |
|------|---------|
| AI 生成 pandas / SQL | ✅ 允許 |
| AI debug 錯誤 | ✅ 允許 |
| LLM API 做分類 / 摘要 | ✅ 鼓勵 |
| AI 依模板產出文件草稿 | ✅ 鼓勵 |
| 完全貼上不了解的程式碼 | ⚠️ 不建議（Demo 時要能說明） |

搭配 `docs/ai_prompts.md` 提供編號 prompt（MVP Scope、資料理解、pandas/SQL、Debug、LLM 分析、分類、Pipeline Documentation）。最重要的是「文件 prompt」：學員貼上欄位、清洗邏輯、前後筆數、SQL、LLM 設定、endpoints，AI 依模板產出 10 節的 Pipeline Documentation，課後直接改成 repo README。

## 13. 工作坊設計 Checklist

**定位與目標**
- [ ] 一句話寫出今天的產出（例：`CSV → pandas → SQLite → LLM → FastAPI`）
- [ ] 寫明學員已學／未學，工作坊只用已學技能 + 一個新體驗
- [ ] 定義 MVP 版本階梯，標出「最低完成」與「推薦完成」
- [ ] 想好這個產出在後續課程怎麼被回收、升級

**題目**
- [ ] 每題通過可行性檢查（資料在 repo、可在時限內做完、有 fallback、有可展示產出）
- [ ] 每題有 requirements_spec（情境、角色、使用者、資料、MVP scope、產出、成功指標、交流引導）
- [ ] 題目有難度梯度並明講難點

**時間與教材**
- [ ] 時間表以分鐘切，講解 ≤ 20 min、最長實作給核心技能、每段後有交流
- [ ] 每段寫「做什麼 → 交付物 → ✅ 基礎完成」，README／速查卡／投影片／Notebook 一致
- [ ] Notebook 每段有 🎯 完成標準、⏱ 時間不夠的最小路徑、🟢🟡🔴 難度標記、檢查點
- [ ] 難接的部分包成 helper；可直接給的元件（API、Dashboard）給 solution
- [ ] starter / challenge / solution 三個版本
- [ ] 自動化腳本跑過所有 solution

**環境與備案**
- [ ] Colab 自動設定 + 本地路徑都測過
- [ ] API Key 多來源讀取，沒 key 也能完成
- [ ] 風險表：網路、API、Colab、進度落後、時間不夠

**人與流程**
- [ ] 講師手冊（準備清單、時間表、逐字稿、巡場、引導句、備案、物資）
- [ ] 講稿（依 slide 標秒數）、TA brief（5 分鐘讀完）
- [ ] 學員手冊 + 事前通知（帳號、環境測試、要準備的報告）
- [ ] Demo 格式、計時方式、點評句、內部評分表
- [ ] 課後追蹤：週報模板、燈號、里程碑、check-in 問題
- [ ] 所有文件的時間、檔名、Demo 長度同步（改一處要全改）

## 參考來源

- [README.md](https://github.com/lu791019/midterm-mvp-template/blob/main/README.md)、[START_HERE.md](https://github.com/lu791019/midterm-mvp-template/blob/main/START_HERE.md)
- [_instructor/facilitator_guide.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/facilitator_guide.md)、[speaker_notes.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/speaker_notes.md)、[ta_briefing.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/ta_briefing.md)
- [_instructor/student_handbook.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/student_handbook.md)、[student_pre_event_notice.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/student_pre_event_notice.md)
- [_instructor/project_report_rubric.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/project_report_rubric.md)、[final_project_tracking.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/final_project_tracking.md)
- [_instructor/midterm_workshop_proposal.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/midterm_workshop_proposal.md)（早期全天 8 題版，已標為歷史文件）
- [data/raw/topic_1/requirements_spec.md](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_1/requirements_spec.md) 等 6 題、[pipeline_starter.ipynb](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_1/pipeline_starter.ipynb)、[pipeline_challenge.ipynb](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_1/pipeline_challenge.ipynb)
- [docs/topic_catalog.md](https://github.com/lu791019/midterm-mvp-template/blob/main/docs/topic_catalog.md)、[docs/ai_prompts.md](https://github.com/lu791019/midterm-mvp-template/blob/main/docs/ai_prompts.md)、[docs/README_TEMPLATE.md](https://github.com/lu791019/midterm-mvp-template/blob/main/docs/README_TEMPLATE.md)、[docs/repo_format.md](https://github.com/lu791019/midterm-mvp-template/blob/main/docs/repo_format.md)
- [scripts/test_midterm_topics.py](https://github.com/lu791019/midterm-mvp-template/blob/main/scripts/test_midterm_topics.py)
