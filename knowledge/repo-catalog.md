# Repo 盤點

Dex 所有 repo 的用途與對應的知識庫文件。盤點日期：2026-10-16。每個 repo 的重點整理在 [repo-notes/](repo-notes/)；私人 repo 只整理方法與結構，不含客戶機密。

## 優先參考

1. [DataDreamers 學員專題手冊](https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook)：最完整的業師流程。
2. [stock-crawler-de-course-materials](https://github.com/lu791019/stock-crawler-de-course-materials)：業界等級 DE 教學專案，已轉成 8 塊積木與 5 份進階文件。
3. [midterm-mvp-template](https://github.com/lu791019/midterm-mvp-template)：工作坊套件。
4. channel-learn／channel-learn-planning（私）：cohort 教學平台與其決策紀錄。
5. [daydreamdex-astro](https://github.com/lu791019/daydreamdex-astro)：個人品牌網站，人物檔案的主要來源。

## 教學與課程

| repo | 公/私 | 用途 | 筆記 | 延伸文件 |
| --- | --- | --- | --- | --- |
| [DataDreamers-decrosscareer-student-project-handbook](https://github.com/lu791019/DataDreamers-decrosscareer-student-project-handbook) | 公 | TibaMe DE 班學員專題手冊：專題三級、10 階段流程、8 情境、Demo Day 評分 | — | [student-project-mentoring.md](playbooks/student-project-mentoring.md) |
| [midterm-mvp-template](https://github.com/lu791019/midterm-mvp-template) | 公 | 期中工作坊 starter：CSV → SQLite → LLM → FastAPI，含講師指南與評分表 | — | [workshop-design.md](playbooks/workshop-design.md) |
| [de-course](https://github.com/lu791019/de-course) | 公 | 21 集帶狀課教材：WSL、Git、uv、Docker 與實作手冊 | — | [course-design.md](playbooks/course-design.md) |
| [stock-crawler-de-course-materials](https://github.com/lu791019/stock-crawler-de-course-materials) | 公 | 台股爬蟲 DE 課：Celery、RabbitMQ、Airflow、MySQL、BigQuery、GCP，17 章 + 補充 A–L | — | [de-project-course.md](playbooks/de-project-course.md) |
| [hahow-crawler-de-course-materials](https://github.com/lu791019/hahow-crawler-de-course-materials) | 公 | Hahow 課程爬蟲 pipeline：Airflow、Celery、MySQL、Metabase | — | [crawl.md](topic-briefs/crawl.md) |
| 企業內訓 YOLO 課 | 私（企業內訓課） | 為某企業內訓客戶開的影像物件偵測（YOLO）課，repo 收第 3–4 場的教材、學員包、講師手冊與改版規格。第 3 場用 Vibe Coding 把訓練好的 ONNX 模型做成瀏… | [筆記](repo-notes/corporate-yolo-course.md) | [corporate-training.md](playbooks/corporate-training.md) |
| channel-learn | 私 | Dex 的多租戶 cohort 教學平台的程式碼與內容。學員從練基本功（題庫、瀏覽器內跑碼、真人批改）一路走到帶完專題，最後做出求職作品集；Discord 社群和職涯工具都接在平台… | [筆記](repo-notes/channel-learn.md) | — |
| channel-learn-planning | 私 | channel-learn 教學平台的規劃書、決策紀錄、grill 質詢紀錄、競品分析、內容盤點、上線 QA 與交接文件，也就是平台的大腦；程式在 channel-learn。 | [筆記](repo-notes/channel-learn-planning.md) | [content-channel.md](playbooks/content-channel.md) |
| de-05-projects | 私 | 從名稱推測是第 05 期 DE 班的學員專題 repo，但目前幾乎是空的。 | [筆記](repo-notes/de-05-projects.md) | — |

## 個人品牌與職涯服務

| repo | 公/私 | 用途 | 筆記 | 延伸文件 |
| --- | --- | --- | --- | --- |
| [daydreamdex-astro](https://github.com/lu791019/daydreamdex-astro) | 公 | 14 篇長文、三個服務頁（職涯諮詢、顧問、講座）、影評。是 Dex 公開自我介紹最完整、最新的來源。 | [筆記](repo-notes/daydreamdex-astro.md) | — |
| [daydreamdex](https://github.com/lu791019/daydreamdex) | 公 | Dex 當 CEO，主 session 當幕僚長（CoS），調度 8 位 CXO（CMO、CPO、CLO 顧問總監、EDU 教學總監、CTO、COO、CFO、TechConsult… | [筆記](repo-notes/daydreamdex.md) | — |
| [lu791019-github-io](https://github.com/lu791019/lu791019.github.io) | 公 | Back-End Developer & Data Engineer、Programming Instructor and Career Development Advisor。是… | [筆記](repo-notes/lu791019-github-io.md) | — |
| [mentoroad](https://github.com/lu791019/mentoroad) | 公 | 仿 Topmate／ADPList 的導師預約平台雛形，願景是讓導師與門生互相串連、角色可以隨時互換，並把生涯教練的引導理論做成生成式 AI 引導機器人。網站把它列為實作成績「Me… | [筆記](repo-notes/mentoroad.md) | — |
| [job-hunter-v2](https://github.com/lu791019/job-hunter-v2) | 公 | 管理個人檔案與作品集、追蹤職缺、用題庫或 JD 出題做行為面試錄影練習並回放；AI 履歷與 AI 面試評估仍在規劃中。 | [筆記](repo-notes/job-hunter-v2.md) | [career-coaching.md](playbooks/career-coaching.md) |

## AI 工具與產品

| repo | 公/私 | 用途 | 筆記 | 延伸文件 |
| --- | --- | --- | --- | --- |
| [dex-agent-os](https://github.com/lu791019/dex-agent-os) | 公 | 把每天的工作紀錄、學習輸入、會議與諮詢逐字稿，用 `claude --print` 加工成日記、洞察，再轉成 Threads、Facebook、部落格、電子報、短影音等多頻道草稿，… | [筆記](repo-notes/dex-agent-os.md) | — |
| [datadevdex](https://github.com/lu791019/datadevdex) | 公 | 有技術立場、會記住你分享過的做法，並把具體經驗蒸餾成可被推翻的技術原則。在 [daydreamdex](daydreamdex.md) 裡擔任 CTO 的大腦。 | [筆記](repo-notes/datadevdex.md) | — |
| aitool | 私 | 自然語言查資料庫（Text-to-SQL）、文件 OCR、中文／台語語音辨識微調、基礎設施設備的影像偵測與預測性維護 MLOps。 | [筆記](repo-notes/aitool.md) | [ai-product-building.md](playbooks/ai-product-building.md) |
| coop-docgen | 私 | 逐字稿＋依 schema 擷取欄位 → 套用草稿 → 人工逐欄確認 | [筆記](repo-notes/coop-docgen.md) | — |
| podmind | 私 | 用中英雙語對照逐字稿＋AI 摘要，同時支援「聽得更好」（增強聆聽）和「不聽也能跟上」（替代聆聽）。 | [筆記](repo-notes/podmind.md) | — |
| filmer | 私 | 用錄音或文字留下觀影感受，系統用語意配對讓你遇見感受相近的人。 | [筆記](repo-notes/filmer.md) | — |
| [castflow](https://github.com/lu791019/castflow) | 公 | 上傳音訊 → AI 轉錄 → 依個人風格生成 6 平台文案 → 排程直發 Threads 與 Facebook。 | [筆記](repo-notes/castflow.md) | — |
| [social-scraper](https://github.com/lu791019/social-scraper) | 公 | `scraper/`（各平台）、`media/`（OCR、轉錄）、`services/`（Sheet、摘要、Notion）、`line_webhook/`。 | [筆記](repo-notes/social-scraper.md) | — |

## 工作與產業專案

| repo | 公/私 | 用途 | 筆記 | 延伸文件 |
| --- | --- | --- | --- | --- |
| [Grid-Edge](https://github.com/lu791019/Grid-Edge) | 公 | 智慧電網的邊緣端程式：依設定定時用 Modbus、SNMP、API 讀取設備資料，經 MQTT 送上雲端，並接收雲端下發的 Modbus 控制指令。 | [筆記](repo-notes/Grid-Edge.md) | — |
| [Grid-Backend](https://github.com/lu791019/Grid-Backend) | 公 | 智慧電網的雲端後端：訂閱邊緣端與氣象站的 MQTT 資料存進 InfluxDB，用 PostgreSQL 管設備設定，並提供查詢 API 與告警信。 | [筆記](repo-notes/Grid-Backend.md) | — |
| [Sails-backend](https://github.com/lu791019/Sails-backend) | 公 | 能源場域的資料接收後端（aisails）：接儲能系統（ESS）、Skysails 風機、光達風場、雷擊、氣象等設備資料，存進 InfluxDB 與 PostgreSQL。 | [筆記](repo-notes/Sails-backend.md) | — |
| [Anomaly_Deteciotn_for_Client](https://github.com/lu791019/Anomaly_Deteciotn_for_Client) | 公 | 客戶合約的異常（風險）偵測：從 SQL Server 撈資料、做編碼與特徵，產出每日風險名單。 | [筆記](repo-notes/Anomaly_Deteciotn_for_Client.md) | — |
| [Potentail_Client_Prediction_XGBoost](https://github.com/lu791019/Potentail_Client_Prediction_XGBoost) | 公 | 潛在客戶預測的 XGBoost 二元分類，特徵選擇、編碼與資料切分都由 XML 設定檔控制。 | [筆記](repo-notes/Potentail_Client_Prediction_XGBoost.md) | — |
| [airflow_heroku_jupyter](https://github.com/lu791019/airflow_heroku_jupyter) | 公 | 把 Airflow（加 JupyterLab）部署到 Heroku 的設定範本，另附一張工作上影像辨識結果入倉的流程圖。 | [筆記](repo-notes/airflow_heroku_jupyter.md) | — |

## 轉職期與比賽作品

| repo | 公/私 | 用途 | 筆記 | 延伸文件 |
| --- | --- | --- | --- | --- |
| [iii_DB102_Project_Hospital_assitant](https://github.com/lu791019/iii_DB102_Project_Hospital_assitant) | 公 | 資策會大數據班結業專題「Hospital Assistant」的總 repo，用醫院檢查資料預測檢查時間、用 LINE Bot 推播，加上 CT 影像辨識與健康文章推薦。 | [筆記](repo-notes/iii_DB102_Project_Hospital_assitant.md) | [archive-hospital-study.md](archive-hospital-study.md) |
| [iii_HA_modeling_time_inference_ML](https://github.com/lu791019/iii_HA_modeling_time_inference_ML) | 公 | 醫院專題的「檢查時間預測」建模紀錄，用 XGBoost、Bagging、Stacking 搭配兩種類別編碼做比較。 | [筆記](repo-notes/iii_HA_modeling_time_inference_ML.md) | — |
| [iii_HA_data_pipeline_Spark_Streaming_with_kafka](https://github.com/lu791019/iii_HA_data_pipeline_Spark_Streaming_with_kafka) | 公 | 醫院專題的即時預測管線：病患資料經 Kafka 進 Spark Streaming，套用預先訓練的 scikit-learn 模型，算出儀器使用時間再送回 Kafka／HDFS。 | [筆記](repo-notes/iii_HA_data_pipeline_Spark_Streaming_with_kafka.md) | — |
| [iii_HA_Image_Recognition_DL](https://github.com/lu791019/iii_HA_Image_Recognition_DL) | 公 | 醫院專題的 CT 影像辨識模組，用 keras-frcnn（Faster R-CNN）做遷移學習，並用 Kafka 把上傳的影像送去辨識、結果回傳 Flask 網頁。 | [筆記](repo-notes/iii_HA_Image_Recognition_DL.md) | — |
| [iii_HA_recommend_system](https://github.com/lu791019/iii_HA_recommend_system) | 公 | 醫院專題的健康文章推薦模組：爬健康網站文章、中文斷詞分群，再依使用者點擊紀錄做協同過濾推薦。 | [筆記](repo-notes/iii_HA_recommend_system.md) | — |
| [iii_HA_linechatbot](https://github.com/lu791019/iii_HA_linechatbot) | 公 | 醫院專題的病患端 LINE 聊天機器人，可以查預約檢查、依身體部位建議掛號科別，並推薦健康文章。 | [筆記](repo-notes/iii_HA_linechatbot.md) | — |
| [Docker-compose_environment_DB](https://github.com/lu791019/Docker-compose_environment_DB) | 公 | 資策會大數據班用的 Docker Compose 環境集合，一份檔案起 ELK、Kafka、Hadoop、Spark、MySQL、MongoDB。 | [筆記](repo-notes/Docker-compose_environment_DB.md) | — |
| [2019_Esuan_bank_TBrian_CreditCardDetection](https://github.com/lu791019/2019_Esuan_bank_TBrian_CreditCardDetection) | 公 | 玉山銀行 × T-Brain 2019 信用卡盜刷偵測競賽的分析筆記與 Notebook。 | [筆記](repo-notes/2019_Esuan_bank_TBrian_CreditCardDetection.md) | — |
| [2020-Datathon-Demo-by-Python-Django](https://github.com/lu791019/2020-Datathon-Demo-by-Python-Django) | 公 | 2020 Datathon 的 Demo 網站：用發票與天氣資料做 RFM 分析，依天氣推薦適合推播飲品的會員，包成 Django 儀表板。 | [筆記](repo-notes/2020-Datathon-Demo-by-Python-Django.md) | — |
| [2020-presidential-hackathon.taiwan.gov](https://github.com/lu791019/2020-presidential-hackathon.taiwan.gov) | 公 | 2020 總統盃黑客松的提案書：給銀髮族的「語音地圖」App，結合開放資料、推薦系統與給社工單位的健康儀表板。 | [筆記](repo-notes/2020-presidential-hackathon.taiwan.gov.md) | — |
| [ML100-Days](https://github.com/lu791019/ML100-Days) | 公 | 線上「機器學習百日馬拉松」的每日作業，從 EDA、特徵工程、傳統模型一路做到 Keras 與神經網路基礎。 | [筆記](repo-notes/ML100-Days.md) | — |
| [Titianic_kaggle_RF](https://github.com/lu791019/Titianic_kaggle_RF) | 公 | Kaggle 鐵達尼號存活預測的隨機森林練習。 | [筆記](repo-notes/Titianic_kaggle_RF.md) | — |

## 其他

- 空或內容極少：Potentail_Client_Prediction_2.0、book、de-06-projects（Git 練習）、portfolio（模板原文）。
- Fork 或他人專案：AI-100x-SE-Research-Lab、datathon、art-for-all、pandas_exercises、Allison94/NKR202_DramaRadar、TernMayDay/NodeJs_Frontend_2024、pycontw/pycontw-program-2024（未整理）。
- 尚未讀取的私人 repo：bigzuozhen、photosearch、ESG 系列、Modbus 系列、test-golang-application、Closed_List、Crawler100-Days。
