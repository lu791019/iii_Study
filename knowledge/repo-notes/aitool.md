# aitool

> **私人 repo**｜大型工作集，部分子專案為**客戶案**｜2026-10-16 只讀各子專案 README、docs、規則檔與目錄結構，未細讀程式
> 本筆記不放客戶名稱、裝置連線資訊、設定檔內容與內部團隊數字。

## 一句話用途

Dex 的 AI 工程工作集，收了 4 個子專案：自然語言查資料庫（Text-to-SQL）、文件 OCR、中文／台語語音辨識微調、基礎設施設備的影像偵測與預測性維護 MLOps。

## 子專案一覽

| 子專案 | 做什麼 | 技術棧 | 架構重點 |
| --- | --- | --- | --- |
| `sqltotext` | 自然語言 → SQL → 執行 → 解釋 → 圖表 | FastAPI、LangChain、SQL Server（pyodbc）、Ollama／OpenAI／Copilot、ApexCharts | 先做**問題分類**（一般諮詢 ／ 數據諮詢）再分流；地端模式多一步 Text-to-SQL；schema 快取獨立成微服務 |
| `documentocr` | 一般、表格、發票三種 OCR 流水線 | FastAPI、Tesseract、EasyOCR、LayoutParser、docTR、Ollama 文字校正 | 多引擎**競賽協調器**；依文字密度（5 級）自動選參數；版面偵測 → 裁切 → 細部結構 → 辨識 → 整合 |
| `taiwanse` | Whisper 中文／台語微調與推論 | Hugging Face、PEFT（LoRA）、Gradio、MLflow、Airflow、nginx、Docker | README 指向他人的開源專案，推論為以開源專案為底改編；含 YouTube 音訊爬蟲建資料集 |
| `Tunneldetection` | 客戶案：攝影機串流即時物件偵測＋設備生命週期預測 | FastAPI、YOLO-NAS、OpenCV、BotSORT／OC-SORT、H2O-3、SHAP、APScheduler、WebSocket | DDD 四層（presentation → application → domain → infrastructure）；多攝影機 session 管理；MLOps 規劃含驗證關卡與回滾 |

## 可借用的知識精華

1. **Text-to-SQL 先分類再查詢**：不是每個問題都需要 SQL；分類器把「一般問題」直接回答，減少錯誤 SQL。
2. **schema 快取**：資料庫結構放記憶體，不必每次請 LLM 重讀 schema。
3. **多引擎競賽＋優雅降級**：同一張圖讓兩個 OCR 引擎都跑、取較佳結果；某引擎不可用就自動切換。
4. **依輸入特性自動調參**：先偵測文字密度，再選對應的 OCR 參數組。
5. **LoRA 參數高效微調**：消費級 GPU 也能微調 Whisper；訓練、推論、Gradio 介面、資料爬蟲分 script。
6. **模型上線的三道驗證關卡**：新模型誤差不得比舊模型差超過一定比例、預測分布偏移（KL 散度）低於閾值、近期資料回測命中率達標；不過關就不部署，可一鍵回滾。
7. **Golden Test Set／Hard Negative Set／影子模式**：每次重訓都用固定考卷考試，專門收集最容易誤判的樣本，新舊模型並跑比對。
8. **延遲與品質的取捨寫成設定檔**：JPEG 品質、跳幀間隔、推論前 resize 都可調，附「快速模式／極速模式」範例。
9. **色彩空間統一**：讀進來立刻 BGR → RGB，寫出去才轉回 BGR，中間全程 RGB，避免辨識錯誤。
10. **Dev log 三分類**：已修復／嘗試過但 revert／保留但效果有限，連失敗方案也記。

## AI 協作痕跡

- 規則檔有三種工具：`.kiro/steering/*.md`（含依檔案路徑自動載入的規則）、`.amazonq/rules/`、Claude Code 的 `For Claude: REQUIRED SUB-SKILL: Use superpowers:executing-plans` 計畫。
- MLOps 計畫的 task 一律是「寫失敗測試 → 確認失敗 → 實作 → 確認通過 → commit」。
- Sprint backlog 開頭附「名詞對照表（給非技術背景的讀者）」，並寫容量規劃原則：維運與新功能工時按比例切、每天留緩衝。

## 適合的教學案例／學員專題

- **教學案例**：Text-to-SQL 的分流設計、OCR 前處理與多引擎、LoRA 微調流程、DDD 分層、MLOps 驗證關卡（Golden Test Set、漂移偵測、回滾）。
- **學員專題**：
  - 🟢「公開資料 Text-to-SQL 問答」：SQLite＋公開資料集＋問題分類。
  - 🟡「收據／表格 OCR → 結構化 CSV」：用自己拍的假收據。
  - 🔴「影像偵測服務＋模型驗證關卡」：公開資料集＋預訓練模型。
- ⚠️ 影像偵測子專案是客戶案：教學只講架構與方法，不展示客戶簡報、場域資料與設定檔。
