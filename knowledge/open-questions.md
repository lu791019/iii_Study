# 待 Dex 確認

整理知識庫時發現、需要 Dex 決定或補充的事項。確認後更新對應文件並刪除該列。

| 主題 | 問題 | 相關文件 |
| --- | --- | --- |
| Sprint 長度 | 手冊 00 章寫每週 planning、每 2 週 review；03 章同時寫「2 週一個 sprint」與「3 sprints × 3 週」。統一用哪個？ | playbooks/student-project-mentoring.md |
| Sprint retro | 00、06 章說不做 sprint retro，05 章排程有每週 retro，評分表要求至少 3 次 sprint retro 紀錄。採哪個？ | playbooks/student-project-mentoring.md、templates/demo-day-rubric.md |
| WIP 上限 | 每人同時進行 2 張還是 3 張卡？ | playbooks/student-project-mentoring.md |
| 帶狀課集數 | de-course 只有 EP01–EP10 手冊；EP13–16、EP19、EP21 的主題為推測。 | playbooks/course-design.md |
| 期中工作坊 | 產出檔名（report.md 或 pipeline_doc.md）、實作段數（2 或 3）、Demo 時長（2 或 3 分鐘）、資料筆數（START_HERE 寫 10 萬筆，實為 2,000 筆） | playbooks/workshop-design.md |
| job-hunter-v2 | AI 面試回饋尚無 prompt 與評分規則、職缺追蹤沒有前端頁、JD 出題按鈕停用。諮詢時是否使用？ | playbooks/career-coaching.md |
| 範例題目 | requirements-spec 的填寫範例（公共自行車站點）為新編，欄位需以實際下載資料核對。 | templates/requirements-spec.md |
| 台股爬蟲課 ci.yml | 註解寫「手冊18／補充C」，實際對應補充 I／F；CI 只跑測試，沒有接部署 job。 | 「CI/CD 與軟體工程」文件 |
| 台股爬蟲課 補充 H | 說 config.py 會從 Secret Manager 讀機密，但程式那段是註解掉的。 | 「CI/CD 與軟體工程」文件 |
| 台股爬蟲課 worker | 正式 `crawler/worker.py` 的 `acks_late` 註解掉，只有 `worker_demo.py` 開啟。 | 「資料爬蟲」文件 |
| 台股爬蟲課 補充 C | 分區段筆數（530；p2024 1415／p2025 637）與前段 320 筆對不上；mock 資料註解寫 30 個交易日，章節寫 32。 | 「資料庫進階」文件 |
| 台股爬蟲課 補充 K | close_model 指標前後不一致（4.12/0.9986 與 4.15/0.9985）；INFORMATION_SCHEMA 範例用 `region-us`，但 dataset 在 asia-east1。 | 「BigQuery 進階與雲地混合」文件 |
| 台股爬蟲課 補充 B | 沒有實測的速度數字，文件中的秒數為估計值。 | 「並行與分散式」文件 |
| Hahow 爬蟲 | 每頁只停 0.1 秒，DataDreamers 情境 2 建議每秒 1 次以內。 | 「資料爬蟲」文件 |
