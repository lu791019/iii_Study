# Dex 教材風格觀察（工作坊與帶狀課）

來源：midterm-mvp-template（README、START_HERE、_instructor/、docs/、notebook markdown）與 de-course（README、集數 README、EP05–EP10 與各操作手冊）。

1. **開頭一句話講「這份給誰、讀完能做到什麼」。** 「給主講師 + TA 用：照著這份跑就能完成下午 3 小時活動。」「活動前一天或當天早上 10:00 前給 TA 看。5 分鐘讀完就能上場。」

2. **整條流程濃縮成一行箭頭，反覆出現。** `CSV → pandas → SQLite (raw/cleaned/analyzed) → SQL → LLM → FastAPI` 出現在 README、速查卡、學員手冊、Notebook 標題、講稿。EP 手冊的「涵蓋」也用 → 串：「啟動 RabbitMQ → 啟動 Celery Worker → 發第一個任務 → …」。

3. **時間都切到分鐘，並寫在標題上。** 「Step 2：ETL Pipeline（45 min）」、時間表每列都有「分鐘」欄；講稿每張投影片標秒數：「[Slide 3] 做到哪算完成 (45s)」。

4. **「做到哪算完成」一定寫出來，用 ✅ / ☐ / 🎯。** 「> ✅ 基礎完成：pipeline.db 有 raw + cleaned 表、processed/ 有統計 CSV、有一個能 Demo 講的數字」；Notebook：「### 🎯 完成這段後你應該有：」；EP 手冊結尾：「## 本集完成清單」。

5. **永遠給「時間不夠」的最小路徑，降低挫折感。** 「> ⏱ 時間不夠？只做 Step 1-1 和 1-4（讀 CSV + 存入 raw 表），跳過探索。」「做不完沒關係，repo 帶走課後繼續」「完成到哪就 Demo 到哪，沒有人會全部做完」。

6. **表格多於段落。** 對照、選項、FAQ、風險、物資、評分全部是表；FAQ 固定「症狀 | 原因 | 解法」，引導句是「卡關 | 引導句」，速查是「手冊 | 內容 | 什麼時候看」。

7. **Emoji 當作功能標記，不當裝飾。** 難度 `🟢 簡單 / 🟡 中等 / 🔴 較難 / （不需要改）`；環境 `🪟 Windows / 🍎 macOS / 🐧 WSL / ✅ 驗證 / ⚠️ 注意 / ❌ 常見錯誤`；燈號 `🟢 正常 / 🟡 注意 / 🔴 需介入`；投影片 `🏭 業界概念` vs `📋 我們今天的…`。

8. **用無關情境示範語法，再要學員「翻譯」到自己的資料。** 「💡 **範例**：如果要讀一份學生名單：`students = pd.read_csv(...)` → 🎯 現在對 `orders.csv` 做同樣的事 → ✅ 預期：2,000 筆、9 個欄位」。

9. **每一步都附預期結果。** Notebook：「✅ 預期：United Kingdom 營收最高」；EP 手冊：`sudo service docker status  # 預期：Active: active (running)`、「看到台積電（2330）的股價 DataFrame = FinMind API 能通」。

10. **引導用提問，不直接給答案。** 「你現在在哪個資料夾？用 `os.getcwd()` 看看」「指著 Notebook 的提示問『這段你看到了嗎？』」；TA 守則：「❌ 幫學員寫程式碼（引導他看提示，不是幫他打字）」。

11. **口語、短句、直接對學員說「你」。** 「你不用講課，你的工作是：巡場、觀察、引導、計時。」「照這張卡走就好。」「你等等要做的就是這件事。」

12. **先講痛點，再給工具，最後一句類比收束。** uv：先列「專案 A 需要 Python 3.9…全域安裝 → 版本衝突」→ 工具比較表 →「一句話：**uv 就是 Python 版的 nvm + npm**」。EP08：「一個 Worker 就是瓶頸：1000 個任務只能一個一個慢慢做。」

13. **先跑通簡化版，再上正式版。** 「stock-crawler 的設計是『每個概念都有簡化版先跑通』」，配「簡化版（先跑通）| 正式版（再深入）」表；Celery 先跑「只 print」的假任務。

14. **把學員作品放進長期升級故事。** 「同一個專案，從期中原型開始…每學一個新技術就升級一次」；「這就是你履歷上的第一個 Data Pipeline 專案。」；「你的 GitHub repo 就是作品集。」

15. **業界術語對照到學員正在做的事。** 「你們今天會建三張表：raw、cleaned、analyzed。業界叫 Medallion Architecture——Bronze、Silver、Gold。」「這就是 12-Factor App 的 Config 原則。」

16. **每個外部依賴都有 fallback，並明講「不影響完成度」。** 「⚠️ 無 API Key，使用 fallback（不影響完成度）」；備案表「API Key 額度用完 → 全部切 fallback 規則版」。

17. **誠實標註版本、實測與過期內容。** 「所有指令都在 Docker 29.6.1 / Compose v5.2.0 / Ubuntu 22.04 實測通過」「最後更新：2026-06-27」；舊企劃書開頭標「⚠️ **歷史文件**：本企劃書為早期規劃版本（全天 8 題），僅供參考」，並指向正式版本。

18. **把「為什麼這樣設計」寫進企劃。** 「為什麼不做純黑客松？學員是轉職者 + 初學者…」「名稱用『挑戰賽』而非『黑客松』——既有挑戰的興奮感，又不會讓初學者覺得『我不夠格參加』」；v4.0 說明為何由講師直接提供 `requirements_spec.md`。

19. **講師端的「最省力」設計。** 「講師只需看『卡關/風險』欄位，有問題的才需要介入。」「建議：LINE 群組回報 + 講師自己用一份 spreadsheet 記錄燈號。最簡單。」

20. **回饋固定句型。** 報告回饋三段：「肯定：你們的 ___ 做得很好 → 建議：如果能再 ___ → 連結下午：可以把 ___ 的想法先用 MVP 驗證」；常見回饋表如「問題太大 →『先縮小範圍，期末再擴展』」。

**同步風險（寫新教材時要避免）**：同一活動的多份文件出現不一致——學員手冊寫產出 `output/report.md`、README 寫 `output/pipeline_doc.md`；事前通知寫「2 段實作、每人 2 分鐘 Demo」，其他文件是 3 段實作、3 分鐘；講師手冊的巡場段落仍用「實作 1（ETL，45 min）」的舊編號；START_HERE 寫「10 萬筆交易」，資料實為 2,000 筆；de-course README 連到不存在的 `Docker實作手冊.md`（實際檔名 `Docker實作操作手冊.md`）。

## 參考來源

- https://github.com/lu791019/midterm-mvp-template ：README.md、START_HERE.md、_instructor/（facilitator_guide.md、speaker_notes.md、ta_briefing.md、student_handbook.md、student_pre_event_notice.md、project_report_rubric.md、final_project_tracking.md、midterm_workshop_proposal.md）、docs/ai_prompts.md、data/raw/topic_1/pipeline_starter.ipynb
- https://github.com/lu791019/de-course ：README.md、ep01/README.md、操作手冊_環境建置.md、Docker安裝教學手冊.md、Docker實作操作手冊.md、EP05_實作操作手冊.md ～ EP10_實作操作手冊.md
