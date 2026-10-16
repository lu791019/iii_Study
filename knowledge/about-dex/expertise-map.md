# Dex 的技能地圖

> 給排課、配對學員與寫教材的人：讀完這份，你會知道 Dex 每個領域做到什麼程度、證據在哪個 repo、適合拿來教什麼。
> 整理日期：2026-10-16。時間脈絡見 [career-timeline.md](career-timeline.md)，人物背景見 [profile.md](profile.md)。

## 證據強度怎麼看

| 標記 | 意思 |
| --- | --- |
| 🟢 工作級 | 有分環境設定、log、migration、測試或 Docker 部署，看得出是要長期跑的系統 |
| 🟡 專題級 | 完整串起一條流程、能 Demo，但多為 Notebook 或單機腳本 |
| ⚪ 練習級 | 課程作業、模板 fork、提案書 |

「工作級」的 repo 多數是推論為工作專案（repo 本身沒有寫），見 [career-timeline.md](career-timeline.md) 的「待確認」。

## 總覽

| 領域 | 最強證據 | 強度 | 一句話 |
| --- | --- | --- | --- |
| 資料工程 | Spark Streaming 專題、Hive 入倉流程、台股爬蟲課 | 🟢 | 從 Hadoop／Spark 時代一路做到 Airflow、BigQuery |
| 爬蟲 | 健康文章 ThreadPool 爬蟲、台股與 Hahow 爬蟲課、social-scraper | 🟢 | 從單支 requests 到 Celery 分散式與瀏覽器自動化 |
| ML/DL | 醫院檢查時間預測、異常偵測、XGBoost 潛在客戶 | 🟡～🟢 | 結構化資料的分類／迴歸最扎實；DL 停在遷移學習 |
| 後端與 API | Grid-Backend、Sails-backend、job-hunter-v2 | 🟢 | Node.js／TypeScript 與 FastAPI 都有正式專案 |
| DevOps／雲 | 各 repo 的 Docker Compose、Heroku、Azure、GCP | 🟡～🟢 | 會用 Docker 包整套服務；CI/CD 的公開證據較少 |
| IoT／工控 | Grid-Edge、Grid-Backend、Sails-backend | 🟢 | Modbus、SNMP、MQTT、時序資料庫的實戰經驗 |
| AI 應用 | castflow、social-scraper、job-hunter-v2、agent 系統 | 🟡～🟢 | LLM、Whisper、Claude Code agent 工作流 |

## 細表

### 資料工程

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| pandas ETL、時間欄位處理 | [iii_DB102_Project_Hospital_assitant](../repo-notes/iii_DB102_Project_Hospital_assitant.md)、archive 醫院專題 | 🟡 | 「髒資料 → 可建模資料」的第一課；用舊專題當反例講檔名與資料夾規劃 |
| 多種資料庫寫入（MySQL、Elasticsearch、MongoDB） | [iii_DB102_Project_Hospital_assitant](../repo-notes/iii_DB102_Project_Hospital_assitant.md)、[iii_HA_recommend_system](../repo-notes/iii_HA_recommend_system.md) | 🟡 | SQL vs NoSQL 怎麼選；同一份資料寫進三種庫的差別 |
| HDFS、PySpark RDD、Spark Streaming、Kafka | [iii_HA_data_pipeline_Spark_Streaming_with_kafka](../repo-notes/iii_HA_data_pipeline_Spark_Streaming_with_kafka.md) | 🟡 | 「pandas 版 → RDD 版 → Streaming 版」的漸進改寫，很適合講為什麼要分散式 |
| Hadoop、Hive 分區表 | [airflow_heroku_jupyter](../repo-notes/airflow_heroku_jupyter.md)（流程圖） | 🟢（推論） | 檔案落地 → HDFS → Hive Table → Partition Table 的入倉流程 |
| Airflow、Celery、RabbitMQ、BigQuery、Metabase | stock-crawler-de-course-materials、hahow-crawler-de-course-materials、[airflow_heroku_jupyter](../repo-notes/airflow_heroku_jupyter.md) | 🟢 | 業界等級 DE 管線（見 [../playbooks/de-project-course.md](../playbooks/de-project-course.md)） |
| 時序資料（InfluxDB）＋關聯式設定（PostgreSQL） | [Grid-Backend](../repo-notes/Grid-Backend.md)、[Sails-backend](../repo-notes/Sails-backend.md) | 🟢 | 量測值與設定檔分開存；OLTP／時序庫的分工 |
| 資料治理、DataOps | （公開 repo 無；依網站自述曾任企業內訓講師） | — | 需 Dex 補素材 |

### 爬蟲

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| requests + BeautifulSoup | [iii_HA_recommend_system](../repo-notes/iii_HA_recommend_system.md)、archive 醫院專題 | 🟡 | 入門爬蟲；爬完寫進 MongoDB |
| ThreadPool 併發、log、計時 decorator | [iii_HA_recommend_system](../repo-notes/iii_HA_recommend_system.md)（`ThreadPool_crawler_final.py`） | 🟡 | 單執行緒 → 併發的第一步；用 decorator 量時間 |
| Celery + RabbitMQ 分散式爬蟲 | stock-crawler-de-course-materials、hahow-crawler-de-course-materials | 🟢 | 任務佇列、重試、冪等寫入（見 [../topic-briefs/crawl.md](../topic-briefs/crawl.md)） |
| 瀏覽器自動化、OCR、語音轉文字 | [social-scraper](../repo-notes/social-scraper.md) | 🟡 | 社群平台這類動態頁面怎麼抓，以及抓完怎麼用 LLM 摘要 |

### ML/DL

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| 迴歸：多模型比較、GridSearchCV、Stacking、Bagging | [iii_HA_modeling_time_inference_ML](../repo-notes/iii_HA_modeling_time_inference_ML.md)、archive 醫院專題 | 🟡 | 同一題比較多種模型與編碼（One-Hot vs Mean Encoding）；Python 與 R 各做一次 |
| 不平衡分類：F1、IsolationForest、LightGBM | [2019_Esuan_bank_TBrian_CreditCardDetection](../repo-notes/2019_Esuan_bank_TBrian_CreditCardDetection.md) | 🟡 | 為什麼盜刷題不能看 accuracy |
| 異常偵測：One-Class SVM、SMOTE、TargetEncoder | [Anomaly_Deteciotn_for_Client](../repo-notes/Anomaly_Deteciotn_for_Client.md) | 🟢（推論） | 從 Notebook 到每天跑的風險名單腳本 |
| XGBoost + 設定檔驅動的特徵工程 | [Potentail_Client_Prediction_XGBoost](../repo-notes/Potentail_Client_Prediction_XGBoost.md) | 🟢（推論） | 把特徵選擇、編碼、切分寫成 XML 設定，模型程式不用改 |
| 文字：jieba、K-Means、Naive Bayes、embedding | [iii_HA_recommend_system](../repo-notes/iii_HA_recommend_system.md) | 🟡 | 中文斷詞與文章分群 |
| 推薦：item-based 協同過濾、RFM | [iii_HA_recommend_system](../repo-notes/iii_HA_recommend_system.md)、[2020-Datathon-Demo-by-Python-Django](../repo-notes/2020-Datathon-Demo-by-Python-Django.md) | 🟡 | 點擊紀錄依新舊加權算相似度；RFM 分群 |
| DL：Keras、CNN、Faster R-CNN 遷移學習 | [ML100-Days](../repo-notes/ML100-Days.md)、[iii_HA_Image_Recognition_DL](../repo-notes/iii_HA_Image_Recognition_DL.md) | ⚪～🟡 | DL 基礎概念；用開源模型做遷移學習。⚠️ 公開證據停在 2019，教進階 DL 前要先更新 |
| 入門分類 | [Titianic_kaggle_RF](../repo-notes/Titianic_kaggle_RF.md) | ⚪ | Kaggle 第一題 |

### 後端與 API

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| Flask + LINE Bot webhook | [iii_HA_linechatbot](../repo-notes/iii_HA_linechatbot.md) | 🟡 | 把結果推到使用者手機的最短路徑 |
| Django 儀表板 | [2020-Datathon-Demo-by-Python-Django](../repo-notes/2020-Datathon-Demo-by-Python-Django.md) | 🟡 | 用開源模板快速做 Demo，重點放在資料而不是 UI |
| Node.js／Express、JWT、Swagger | [Sails-backend](../repo-notes/Sails-backend.md) | 🟢 | REST API 加驗證與文件 |
| TypeScript／Hono、Sequelize migration、Jest | [Grid-Backend](../repo-notes/Grid-Backend.md) | 🟢 | migration 管理 schema 演進；用 mock 寫單元測試 |
| FastAPI、SQLAlchemy、Alembic | job-hunter-v2、midterm-mvp-template | 🟢 | Python 後端與資料 API（見 [../playbooks/workshop-design.md](../playbooks/workshop-design.md)） |

### DevOps／雲

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| Docker Compose 多服務環境 | [Docker-compose_environment_DB](../repo-notes/Docker-compose_environment_DB.md)、[iii_HA_linechatbot](../repo-notes/iii_HA_linechatbot.md) | 🟡 | 一份 compose 起整套 ELK／Kafka／Hadoop 環境 |
| Dockerfile、gunicorn、nginx | [2020-Datathon-Demo-by-Python-Django](../repo-notes/2020-Datathon-Demo-by-Python-Django.md) | 🟡 | Python Web 部署三件套 |
| Docker image 版本化、私有 registry | [Grid-Edge](../repo-notes/Grid-Edge.md)、[Grid-Backend](../repo-notes/Grid-Backend.md)（`docker-compose.template.yml` 用 `IMAGE_TAG`） | 🟢 | 用 tag 部署與回滾 |
| Heroku（Procfile） | [airflow_heroku_jupyter](../repo-notes/airflow_heroku_jupyter.md)、[2020-Datathon-Demo-by-Python-Django](../repo-notes/2020-Datathon-Demo-by-Python-Django.md) | ⚪ | PaaS 概念（Heroku 已無免費方案，教學改用其他平台） |
| Azure（SQL、IoT Hub）、GCP（BigQuery）、AWS（構想） | 2020-Datathon、[Grid-Backend](../repo-notes/Grid-Backend.md)、stock-crawler、2020-presidential-hackathon | 🟡 | 三朵雲都碰過；GCP 的教學素材最完整 |
| WSL、Git、uv | de-course | 🟢 | 環境建置教學（見 [../playbooks/course-design.md](../playbooks/course-design.md)） |

### IoT／工控

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| Modbus 讀寫、SNMP、BMS API | [Grid-Edge](../repo-notes/Grid-Edge.md) | 🟢 | 邊緣端怎麼定時讀設備；寫入指令為什麼要進佇列（BullMQ + Redis） |
| MQTT 發布與訂閱 | [Grid-Edge](../repo-notes/Grid-Edge.md)、[Grid-Backend](../repo-notes/Grid-Backend.md)、[Sails-backend](../repo-notes/Sails-backend.md) | 🟢 | 邊緣 → 雲端的訊息通道；控制指令反向下發 |
| 離線設定快取（PouchDB） | [Grid-Edge](../repo-notes/Grid-Edge.md) | 🟢 | 網路斷了邊緣設備還能照設定跑 |
| 能源設備資料（儲能、風機、光達、雷擊、柴油發電機） | [Sails-backend](../repo-notes/Sails-backend.md) | 🟢 | 廠商特規格式（如需用廠商程式轉檔）怎麼包成 Docker 處理 |
| 氣象資料串接、告警信 | [Grid-Backend](../repo-notes/Grid-Backend.md) | 🟢 | 外部資料排程拉取、閾值告警 |

### AI 應用

| 技術 | 熟練證據 repo | 強度 | 可以教什麼 |
| --- | --- | --- | --- |
| 規則式聊天機器人 | [iii_HA_linechatbot](../repo-notes/iii_HA_linechatbot.md) | 🟡 | AI 之前的版本：關鍵字 → 回覆；和 LLM 版做對照 |
| LLM 串接（OpenAI、Anthropic 抽象層） | [job-hunter-v2](../repo-notes/job-hunter-v2.md)、midterm-mvp-template | 🟢 | Factory Pattern 換供應商；CSV → SQLite → LLM → FastAPI |
| 語音轉文字 + 文案生成 | [castflow](../repo-notes/castflow.md) | 🟡 | Whisper 轉錄 → 多平台文案 → 排程發布 |
| LINE webhook + 爬取 + 摘要 | [social-scraper](../repo-notes/social-scraper.md) | 🟡 | 2019 年 LINE Bot 的 2026 升級版，適合當「同一個點子七年後怎麼做」 |
| Claude Code agent、多角色工作流、記憶設計 | [dex-agent-os](../repo-notes/dex-agent-os.md)、[datadevdex](../repo-notes/datadevdex.md)、[daydreamdex](../repo-notes/daydreamdex.md) | 🟡 | 個人 AI 工作系統；skill 與 subagent 的分工 |

## 適合配對的學員類型

| 學員想走的方向 | Dex 最能幫的地方 | 帶他看的 repo |
| --- | --- | --- |
| 資料工程 | 管線設計、排程、從作業升級到 MVP | iii_HA_data_pipeline_Spark_Streaming_with_kafka → stock-crawler-de-course-materials |
| 資料科學／ML | 結構化資料建模、評估指標、Notebook 轉腳本 | iii_HA_modeling_time_inference_ML → Anomaly_Deteciotn_for_Client |
| 後端 | API、migration、測試、Docker | Grid-Backend、job-hunter-v2 |
| 製造或能源業轉職 | IoT 協定與時序資料，接得上產業背景 | Grid-Edge、Sails-backend |
| AI 應用 | 把 LLM 包進一個完整的小產品 | midterm-mvp-template、castflow、social-scraper |

⚠️ 較弱或缺公開證據的地方：進階 DL（2019 後無公開 repo）、前端（主要用模板）、CI/CD 部署 job（見 [../open-questions.md](../open-questions.md) 台股爬蟲課 ci.yml）、資料治理（只有網站自述）。
