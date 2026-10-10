# Repo 盤點：可作為諮詢、業師、講師參考的素材

盤點日期：2026-10-10。公開 repo 讀了 README 與根目錄；私人 repo 未能讀取，只依名稱推測（標「推測」）。README 原文快照在 [sources/](sources/)。

## 推薦優先參考

1. **[DataDreamers-decrosscareer-student-project-handbook](https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook)**：最完整的業師 SOP，含選題、10 階段流程、8 情境、Demo Day 評分表、復盤與履歷 checklist。
2. **[midterm-mvp-template](https://github.com/lu791019/midterm-mvp-template)**：現成工作坊套件，含講師指南、評分表、解答、追蹤週報模板。
3. **[de-course](https://github.com/lu791019/de-course)**：21 集帶狀課的逐集教材與操作手冊。
4. **[stock-crawler-de-course-materials](https://github.com/lu791019/stock-crawler-de-course-materials)**：生產級 DE 架構教學範本，章節手冊最完整。
5. **[job-hunter-v2](https://github.com/lu791019/job-hunter-v2)**：求職輔導工具雛形（面試練習、職缺追蹤），可結合諮詢服務。

## 完整清單

| repo | 公/私 | 內容 | 可作為 | 推薦 |
| --- | --- | --- | --- | --- |
| DataDreamers-decrosscareer-student-project-handbook | 公 | TibaMe DE 班學員專題手冊 | 課程教材、專題範本、諮詢流程 | 高 |
| midterm-mvp-template | 公 | 期中工作坊 starter：CSV → SQLite → LLM → FastAPI | 課程教材、專題範本 | 高 |
| de-course | 公 | 21 集帶狀課：WSL、Git、uv、Docker 與實作手冊 | 課程教材 | 高 |
| stock-crawler-de-course-materials | 公 | 台股爬蟲：Celery、RabbitMQ、Airflow、MySQL、BigQuery、Metabase | 課程教材、專題範本 | 高 |
| hahow-crawler-de-course-materials | 公 | Hahow 爬蟲 pipeline：Airflow、Celery、MySQL、Metabase | 課程教材、專題範本 | 中高 |
| mentoroad | 公 | 導師媒合平台構想，結合生成式 AI 與職涯諮詢理論 | 諮詢流程構想 | 中 |
| job-hunter-v2 | 公 | AI 求職助手：檔案、職缺追蹤、行為面試練習 | 求職工具 | 中 |
| iii_DB102_Project_Hospital_assitant | 公 | 資策會結業專題：檢查排程預測、chatbot、CT 影像辨識 | 轉職案例（Dex 本人） | 中（有敏感資料，見下） |
| dex-agent-os | 公 | 個人 AI 知識與內容生產系統 | 內容產製工具 | 低中 |
| datadevdex | 公 | Claude Code 的 Tech Lead 人格系統 | AI 輔助開發示範 | 低 |
| Titianic_kaggle_RF | 公 | Kaggle 鐵達尼隨機森林 | 入門 ML 範例 | 低 |
| ML100-Days | 公 | ML 百日馬拉松作業 | 學習歷程範例 | 低 |
| iii_HA_modeling_time_inference_ML | 公 | 舊專題的預測程式 | 舊專題片段 | 低 |
| Potentail_Client_Prediction_XGBoost | 公 | XGBoost 二元分類 | 舊練習 | 低 |
| portfolio | 公 | vCard 作品集網站模板 | 學員作品集模板 | 低 |
| de-06-projects | 公 | Git 練習 | 不相關 | 低 |
| de-05-projects | 私 | 推測：第 05 期學員專題 | 學員作品案例 | 未讀 |
| hamastar-yolo-course | 私 | 推測：YOLO 企業內訓課 | 課程教材 | 未讀 |
| channel-learn、channel-learn-planning | 私 | 推測：學習頻道的規劃與內容 | 內容規劃 | 未讀 |
| coop-docgen | 私 | 推測：合作案文件產生器 | 不確定 | 未讀 |

## 注意事項

- **iii_DB102_Project_Hospital_assitant 的 `Resources/CT.csv`（公開）**：約 9.7 萬列醫院檢查紀錄格式的資料；依 Dex 說明為假資料。
- **dex-agent-os（公開）**：含個人日記與 `.claude/settings.local.json`。諮詢紀錄不要 commit 進公開 repo。
- **midterm-mvp-template**：`_instructor/` 的解答是公開的，學員看得到。
- **爬蟲課程 repo**：README 中的 `root / 1234` 是本機開發預設值，風險低，但教學時要提醒不可用於正式環境。
