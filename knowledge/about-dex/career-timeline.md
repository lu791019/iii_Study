# Dex 的技術歷程：從公開 repo 重建

> 給轉職諮詢與業師用：讀完這份，你能用 Dex 本人的 repo 當證據，講一個「非本科、從半導體走到 AI」的轉職故事，並知道每段經歷可以拿哪個 repo 給學員看。
> 整理日期：2026-10-16。人物背景、公司與職稱以 [profile.md](profile.md) 為準；技能整理見 [expertise-map.md](expertise-map.md)；單一 repo 的細節見 [../repo-notes/](../repo-notes/)。

```
半導體缺陷分析 → 自學 Python + ML 百日馬拉松 → 資策會大數據班專題 → Big Data Engineer → 金融 DE → DE & 後端（能源 IoT）→ 講師／業師 → AI 技術總監
```

## 日期怎麼判讀

| 來源 | 可信度 | 說明 |
| --- | --- | --- |
| 公開網站 about 頁、舊版履歷頁 | 高 | 公司與在職期間，整理在 [profile.md](profile.md) |
| repo 內容裡的日期（檔名 `_0905`、migration 時間戳、套件版本） | 中高 | 最接近「實際做的時間」 |
| `git log -1` 的日期 | 中 | 本次 clone 是 shallow clone，每個 repo 只看得到最後一個 commit，多數是「Add files via upload」，代表**上傳日**，不一定是開發日 |

⚠️ 上傳日和開發日常常差很多：`iii_HA_data_pipeline_Spark_Streaming_with_kafka` 上傳於 2020-12，但模型檔名是 2019-09；`2020-Datathon-Demo-by-Python-Django` 上傳於 2024-01，套件版本卻是 2020 年秋天的。下表的「年份」以內容日期為主，括號附上傳日。

## 依年份的技術歷程

| 年份 | 專案／repo | 情境 | 技術 | 學到什麼 |
| --- | --- | --- | --- | --- |
| 2016–2019 | （無 repo） | 工作：半導體廠 Defect Analyst Engineer | 公司內部工具 | 工具「帶不走」，成為轉職動機（Dex 部落格〈Python轉職指南〉） |
| 2019-01～12 | [ML100-Days](../repo-notes/ML100-Days.md)（上傳 2019-12） | 自學：線上 ML 百日馬拉松作業，檔名日期 0123 → 1220 | pandas、scikit-learn、特徵工程、Keras、梯度下降、反向傳播、optimizer | 用每日作業把 ML 與 DL 基礎從 EDA 走到神經網路 |
| 2019 上半～09（推論） | [iii_DB102_Project_Hospital_assitant](../repo-notes/iii_DB102_Project_Hospital_assitant.md)（上傳 2019-08）＋ archive 醫院專題 | 資策會 × 緯育 TibaMe「大數據與資料分析實體班」結業專題，6 人小組 | pandas、MySQL、Elasticsearch、MongoDB、scikit-learn、Keras、LINE Bot | 第一次把資料整理、資料庫、ML、DL、聊天機器人串成一個產品故事 |
| 2019-07～09 | [iii_HA_modeling_time_inference_ML](../repo-notes/iii_HA_modeling_time_inference_ML.md)（上傳 2019-09） | 同上專題：檢查時間預測 | XGBoost、GridSearchCV、Bagging、Stacking（Python 與 R）、One-Hot 與 Mean Encoding | 同一題比較多種模型與編碼方式 |
| 2019-08～09 | [iii_HA_recommend_system](../repo-notes/iii_HA_recommend_system.md)（上傳 2020-03） | 同上專題：健康文章推薦 | requests、BeautifulSoup、ThreadPool 爬蟲、jieba、K-Means、Naive Bayes、協同過濾、MySQL | 爬蟲 → 斷詞分群 → 依點擊紀錄推薦的完整鏈 |
| 2019-09 | [iii_HA_linechatbot](../repo-notes/iii_HA_linechatbot.md)（上傳 2019-09） | 同上專題：病患端介面 | Flask、line-bot-sdk、MariaDB、Docker Compose、serveo tunnel | 把模型結果透過 LINE 推給使用者；第一次用 Docker Compose 包整套服務 |
| 2019-09 | [iii_HA_Image_Recognition_DL](../repo-notes/iii_HA_Image_Recognition_DL.md)（上傳 2019-09） | 同上專題：CT 影像辨識 | Keras Faster R-CNN 遷移學習、Kafka producer／consumer、Flask | 用開源模型做遷移學習，並用 Kafka 串到 Web |
| 2019-09 | [iii_HA_data_pipeline_Spark_Streaming_with_kafka](../repo-notes/iii_HA_data_pipeline_Spark_Streaming_with_kafka.md)（上傳 2020-12） | 同上專題：即時預測管線 | HDFS、PySpark RDD、Spark Streaming（DStream）、Kafka、broadcast sklearn 模型 | README 自述「初次將 Spark 應用於實務」：從 pandas 版改到 RDD 版，再改到 Streaming 版 |
| 2019-09 | [Docker-compose_environment_DB](../repo-notes/Docker-compose_environment_DB.md)（上傳 2019-09） | 資策會課程環境 | Docker Compose：ELK、Kafka、Zookeeper、Hadoop、Spark、MySQL、MongoDB | 把大數據課的整套環境寫成 compose 檔 |
| 2019-09 | [Titianic_kaggle_RF](../repo-notes/Titianic_kaggle_RF.md)（上傳 2019-09） | 自學：Kaggle 入門 | scikit-learn RandomForest | 第一個完整的分類題 |
| 2019 秋 | [2019_Esuan_bank_TBrian_CreditCardDetection](../repo-notes/2019_Esuan_bank_TBrian_CreditCardDetection.md)（上傳 2020-03） | 比賽：玉山 × T-Brain 信用卡盜刷偵測 | 欄位分析、衍生變數、IsolationForest、RandomForest、LightGBM、F1 | 極度不平衡資料要看 F1，不是 accuracy |
| 2020-01～02 | [Anomaly_Deteciotn_for_Client](../repo-notes/Anomaly_Deteciotn_for_Client.md)（上傳 2020-03） | 工作（推論：線上教育公司 Big Data Engineer 時期） | SQL Server、TargetEncoder、One-Class SVM、SMOTE + SVC、DEV／STG／PROD 設定 | 從比賽 Notebook 走到要每天跑、要分環境的正式腳本 |
| 2020-05 | [2020-presidential-hackathon.taiwan.gov](../repo-notes/2020-presidential-hackathon.taiwan.gov.md) | 黑客松：總統盃提案（只有提案書） | 構想：開放資料、推薦系統、AWS ML、儀表板 | 用「緣起 → 市場 → 對象 → 功能 → 未來」寫提案 |
| 2020-08 | [Potentail_Client_Prediction_XGBoost](../repo-notes/Potentail_Client_Prediction_XGBoost.md)（上傳 2020-08） | 工作（推論：潛在客戶預測） | XGBoost、xgbfir、RandomUnderSampler、XML 設定檔、SQLAlchemy | 把特徵選擇、one-hot、切分日期寫成設定檔，模型程式可重用 |
| 2020 下半 | [2020-Datathon-Demo-by-Python-Django](../repo-notes/2020-Datathon-Demo-by-Python-Django.md)（上傳 2024-01） | 比賽：2020 Datathon（主辦單位 repo 未註明） | Django、Azure SQL（pyodbc）、RFM、Docker、gunicorn、nginx | 分析結果做成可操作的 Web Demo，不只交 Notebook |
| 2021-01 | [airflow_heroku_jupyter](../repo-notes/airflow_heroku_jupyter.md)（上傳 2021-01） | 自學＋工作流程圖（推論） | Airflow、Celery、Redis、Postgres、Heroku；流程圖：GPU 伺服器 → HDFS → Hive 分區表 | 排程工具上雲；把工作上的影像辨識結果入倉流程畫成泳道圖 |
| 2021 | （無 repo；`book` 為空） | 工作：純網銀 Data Engineer（約 3 個月） | — | 公開 repo 無對應證據 |
| 2024-01～04 | [Sails-backend](../repo-notes/Sails-backend.md)（上傳 2024-11） | 工作（推論：DE & 後端時期的能源專案） | Node.js、Express、JWT、Swagger、MQTT、FTP、InfluxDB、PostgreSQL、Sequelize migration、Jest | 接儲能、風機、光達、雷擊等多種設備資料；時序資料進 InfluxDB |
| 2024-07～11 | [Grid-Edge](../repo-notes/Grid-Edge.md)、[Grid-Backend](../repo-notes/Grid-Backend.md)（上傳 2024-11） | 工作（推論：智慧電網專案） | TypeScript、Hono、Modbus、SNMP、MQTT、BullMQ、Redis、PouchDB、InfluxDB、PostgreSQL、Azure IoT、Jest | 邊緣端讀設備、雲端收資料與告警的兩層架構 |
| 2024-10 | [mentoroad](../repo-notes/mentoroad.md)、portfolio、[lu791019.github.io](../repo-notes/lu791019-github-io.md) | 個人：諮詢平台構想、個人頁 | Next.js、tRPC、Prisma（mentoroad） | 開始把職涯諮詢做成產品構想 |
| 2026-03 | [dex-agent-os](../repo-notes/dex-agent-os.md)、[datadevdex](../repo-notes/datadevdex.md)、[daydreamdex](../repo-notes/daydreamdex.md)、[job-hunter-v2](../repo-notes/job-hunter-v2.md)、[castflow](../repo-notes/castflow.md)、[social-scraper](../repo-notes/social-scraper.md) | 個人 AI 系統與產品（AI 技術總監時期） | Claude Code agent、LLM API、Whisper、FastAPI、Next.js、Supabase | 從「做模型」轉向「做 AI 應用與 agent 工作流」 |
| 2026-06～10 | midterm-mvp-template、hahow-crawler-de-course-materials、de-course、stock-crawler-de-course-materials、DataDreamers 手冊 | 講師與專題業師 | Airflow、Celery、RabbitMQ、MySQL、BigQuery、FastAPI、LLM | 把十年做過的東西改寫成可教的教材（見 [../playbooks/](../playbooks/)） |

## 轉職案例講法（給學員參考）

> 一句話版本：「我不是本科，先在半導體做缺陷分析；用半年的進修班做出一個串滿 ML、資料庫和聊天機器人的專題轉進資料工程；做了五年資料管線和後端之後，才回頭把 AI 做成應用。」

### 階段 1：半導體 QA → 轉職準備（2016–2019）

在半導體廠做缺陷分析，用的是公司內部工具，換公司就用不上，這是他想轉的原因。2019 年他先邊上班邊自學，留下 ML 百日馬拉松的每日作業，接著辭職上半年的大數據實體班，和 6 人小組做出醫院助理專題：檢查時間預測、LINE 推播、CT 影像辨識、健康文章推薦，一次把多種技術串起來。

- 證據：[ML100-Days](../repo-notes/ML100-Days.md)、[iii_DB102_Project_Hospital_assitant](../repo-notes/iii_DB102_Project_Hospital_assitant.md) 與 5 個 `iii_HA_*` 子 repo、[archive-hospital-study.md](../archive-hospital-study.md)
- 給學員的重點：專題不是單一 Notebook，是「一個痛點 + 一條能 Demo 的管線」。（推論）他能進大數據職位，靠的是 Spark Streaming + Kafka 這種「資料工程味」最重的那一塊，不是模型分數。

### 階段 2：資料工程 / 軟體（2019–2025）

第一份資料工作在線上教育公司當 Big Data Engineer，repo 裡看得到他把 Notebook 改成分 DEV／STG／PROD、有 log、讀設定檔的正式腳本，也畫了 GPU 伺服器 → HDFS → Hive 分區表的入倉流程（推論：這些是工作專案）。之後做過純網銀資料工程，再到大型電子製造集團做資料工程與後端，2024 年的 repo 已經是 TypeScript／Node.js 的 IoT 後端：Modbus、MQTT、InfluxDB、migration、Jest 測試一應俱全（推論：對應能源與 ESG 相關專案）。

- 證據：[Anomaly_Deteciotn_for_Client](../repo-notes/Anomaly_Deteciotn_for_Client.md)、[Potentail_Client_Prediction_XGBoost](../repo-notes/Potentail_Client_Prediction_XGBoost.md)、[airflow_heroku_jupyter](../repo-notes/airflow_heroku_jupyter.md)、[Sails-backend](../repo-notes/Sails-backend.md)、[Grid-Edge](../repo-notes/Grid-Edge.md)、[Grid-Backend](../repo-notes/Grid-Backend.md)
- 給學員的重點：轉職後的成長不是換更炫的模型，是**從「跑得出結果」變成「每天穩定跑、壞了查得到」**。比賽與黑客松（玉山、總統盃、Datathon）是這段期間維持手感與作品的方式。

### 階段 3：AI（2025–2026）

在學院做行銷與技術雙棲之後，他成為 AI 技術總監；2026 年的個人 repo 幾乎都是 AI 應用：Podcast 轉多平台文案、LINE 傳連結自動摘要、AI 求職助手、用 Claude Code 搭的 agent 系統。同時他把過去的資料工程經驗寫成課程與業師手冊，教學本身也成了作品。

- 證據：[castflow](../repo-notes/castflow.md)、[social-scraper](../repo-notes/social-scraper.md)、[job-hunter-v2](../repo-notes/job-hunter-v2.md)、[dex-agent-os](../repo-notes/dex-agent-os.md)、[datadevdex](../repo-notes/datadevdex.md)、[daydreamdex](../repo-notes/daydreamdex.md)、[../playbooks/](../playbooks/) 各教材
- 給學員的重點：（推論）2019 年他就學過 DL，但真正做 AI 是在有資料工程與後端底子之後；**AI 應用 = 資料管線 + API + 一個 LLM 呼叫**，前兩塊才是大多數工作量。

### 三段對照：學員可以抄的做法

| 階段 | Dex 當時做的事 | 學員可以怎麼做 |
| --- | --- | --- |
| 1 | 在職自學 + 半年實體班 + 團隊專題 | 先用公開課程打基礎，再用一個能 Demo 的團隊專題當敲門磚 |
| 2 | 工作中把 Notebook 改成正式腳本，下班參加比賽 | 把專題升級到「有排程、有 log、有測試」；比賽當作品，不當排名 |
| 3 | 用既有的資料與後端能力做 AI 應用，並把經驗寫成教材 | 先做小而完整的 AI 工具，寫 README 與架構圖，讓 repo 自己講故事 |

## 待確認

| 主題 | 問題 |
| --- | --- |
| 半導體職稱 | 各處寫法不同（QA、設備工程師、Defect Analyst）；[profile.md](profile.md) 建議以「Defect Analyst Engineer」為準。學員版故事用「半導體 QA」是否可以？ |
| 進修班期間 | 部落格說「半年」，美光離職為 2019/01、第一份資料工作為 2019/11；實體班確切月份未能從 repo 核實（估計 2019 上半到 2019-09）。 |
| 工作 repo 的歸屬 | Anomaly、Potentail_Client_Prediction_XGBoost、airflow 流程圖、Sails、Grid 是否為工作專案、是否已取得公開許可，repo 未說明（皆為推論）。 |
| 2020 Datathon | 主辦單位與名次未在 repo 註明。 |
| 2025 年 | 學院行銷與技術雙棲期間沒有公開技術 repo，故事中這段要靠 Dex 口述補。 |

## 未建立筆記的 repo

| repo | 原因 |
| --- | --- |
| Potentail_Client_Prediction_2.0 | 空 repo，沒有任何 commit |
| book | 只有一行標題的 README（最後 commit 2021-05） |
| stock-crawler-de-course-materials、hahow-crawler-de-course-materials、de-course、midterm-mvp-template | 已整理在 [../playbooks/](../playbooks/)，本次只用來排時間軸 |
| portfolio | 網站模板（內容仍為模板原文）；個人頁見 [lu791019-github-io](../repo-notes/lu791019-github-io.md) |
