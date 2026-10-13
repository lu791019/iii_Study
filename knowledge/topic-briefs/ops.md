# 排程、自動化與部署

> 給已經把 pipeline 手動跑通的學員：讀完這份，你能讓它**自己定時跑、失敗會重試會通知、一個指令起整套、搬上雲也不燒錢**。
> 一句心法：**沒上排程的 pipeline 是作業；會自己動、出事你知道的，才是 MVP。**

**流程總覽**：`選排程工具 → 寫 DAG（retries）→ 接上爬蟲 → Compose 一鍵起＋七步驟驗證 → .env／Secret Manager → 本機 → GCE → 多 VM＋Cloud SQL →（Composer）→ 監控告警＋測試＋CI`

主要來源：`stock-crawler-de-course-materials/課程手冊/`（第 07、09–17 章，補充 A／F／G／H／I／L）、`airflow/dags/`、`docker-compose-all.yml`、`.github/workflows/ci.yml`；`hahow-crawler-de-course-materials/airflow/`；`DataDreamers-decrosscareer-student-project-handbook/`（02、05、07）。

## 做完這一份，你會做到

1. 說得出 cron／APScheduler／Airflow／GitHub Actions 各自適合什麼，並替自己的專題選一個（附理由）。
2. 寫出一支有 `retries`、`catchup=False`、失敗告警的 DAG，並在 UI 上 Clear 重跑失敗的 task。
3. 用 `docker compose up -d` 一行起整套服務，跑完七步驟驗證，證明「資料真的落地」。
4. 讓 repo 裡沒有任何真密碼：本機用 `.env`、雲端用 Secret Manager、CI 用 GitHub Secrets。
5. 把系統搬上 GCE、設好預算警告、收工前停機；每個 PR 自動跑測試，README 第一頁有架構圖。

## 為什麼自動化是「作品 → MVP」的分水嶺

| 等級 | 排程與部署的樣貌 | 面試官看到什麼 |
| --- | --- | --- |
| 作業 | 手動跑一次 script，demo 時現場按 Run | 「這只是一支程式」 |
| 作品 | 有排程，但失敗沒人知道；只在自己電腦能跑 | 跟其他人差不多 |
| MVP | 排程＋重試＋歷史紀錄＋失敗通知＋一鍵起＋CI，能講「為什麼選 Airflow 不選 cron」 | 想深入問你怎麼設計 |

- 藍圖原文：「很多人交作業時就停在『手動跑一次』——但 pipeline 的價值在『自動』。」（`DataDreamers…/02-pipeline-blueprint.md` 2️⃣ Ingest）
- 評分表直接計分：實作品質 15–20 分要「有 CI、能 docker compose 一鍵起」；demo 前 self check 有一條「至少有一支監控（pipeline 失敗會通知）」（`07-demo-day-rubric.md`）。
- 💡 面試 5 秒講法：「這條 pipeline 每個交易日 20:00 由 Airflow 觸發，失敗自動重試一次、再失敗發 Discord；整套 `docker compose up` 一行起，每個 PR 跑 CI。」

## Step 1：選排程工具（20 min）

**目標**：用一張表替專題選定排程工具，寫得出「為什麼不選另外三個」。

| | cron | APScheduler | Airflow | GitHub Actions `schedule` |
| --- | --- | --- | --- | --- |
| 誰在計時 | 作業系統服務 | 你的 Python 行程 | 獨立的 scheduler 服務 | GitHub 的機器 |
| 到點執行什麼 | 一條 shell 指令 | 一個 Python 函式 | 一張 DAG（多個 task＋依賴） | 一份 workflow（多個 step） |
| 最小單位 | 1 分鐘 | 秒（多一個 `second` 欄位） | 依 cron，實務以分鐘計 | 約 5 分鐘，尖峰會延遲（估計值，依 GitHub 文件，課程未實測） |
| 依賴／重試／UI／log | ❌ 只能翻 `/var/log/syslog` | 重試自己寫，沒有 UI | ✅ `retries` 一行、Graph／Grid／Logs | 有執行紀錄頁，重試要自己設計 |
| 程式關掉後 | 排程還在 | 排程跟著消失 | 服務在就在 | 不依賴你的機器 |
| 適合 | 單機、一支 script、課堂示範 | 「這支程式就是在跑排程」 | 多步驟 ETL、要補跑、要給別人看歷史 | 免費、無需主機的輕量批次；或只當 CI |
| 課程出處 | 第 09 章前半 | 第 09 章主線 | 第 10–12、17 章 | 補充 I（只用於 CI）；藍圖 2️⃣ 列為排程選項 |

- 一個常見誤解：**Airflow 取代的是 APScheduler，不是 Celery**——Airflow 是編排層（何時跑、誰先誰後、失敗怎麼補），Celery 是執行層（第 10 章）。

**範例：crontab 一行**（第 09 章；髒細節包進腳本，crontab 只寫一行）

```bash
# 每週一到五 18:00 跑一次；輸出附加到 log（cron 沒有畫面，不導出就看不到成敗）
0 18 * * 1-5 ~/stock-crawler/example/cron_producer.sh >> /tmp/cron_producer.log 2>&1
```

⚠️ `echo '...' | crontab -` 會**整份取代**既有排程；示範完記得 `crontab -r`，不然 worker 沒開時任務會一直堆在 RabbitMQ。

**產出**：專題計畫書「工具選型」一列：選了什麼＋一句理由＋不選另外三個的原因。

## Step 2：Airflow 基本功與五塊積木（60 min）

**目標**：看懂 DAG／task／operator／schedule／retries，寫出第一支能自動重試、失敗會通知的 DAG。

**做法：五個核心概念**

| 概念 | 一句話 | 課程寫法（`airflow/dags/example_first_dag.py`） |
| --- | --- | --- |
| DAG | 工作流的容器，本身不做事 | `with DAG(dag_id=..., ...) as dag:` |
| task／operator | 一個步驟／做某類事的方法 | `PythonOperator`、`BashOperator` |
| 依賴 | `a >> b`：a 成功才跑 b | `start_task >> end_task` |
| schedule | cron 語法；`catchup=False` 不補跑歷史 | `"0 18 * * 1-5"`（`stock_crawler_dag.py`） |
| retries | 失敗自動重試 | `retries=1`、`retry_delay=1 分鐘`、`execution_timeout=1 小時` |

**第 11 章五塊積木**

| 積木 | 用途 | 一定要記得的事 | 範例 DAG |
| --- | --- | --- | --- |
| `BranchPythonOperator` | 依條件走不同路（交易日守門、選市場） | 回傳值要一字不差等於下游 task_id；匯合點改 `trigger_rule="none_failed_min_one_success"` | `example_branch_operator_dag.py` |
| XCom | task 之間傳**小**資料 | return 自動推送、`xcom_pull` 拉取；大資料走 DB，DAG 只管流程 | `example_xcom_dag.py` |
| `TriggerDagRunOperator` | DAG 觸發另一個 DAG | 被觸發的 DAG 也要 unpause | `example_trigger_dag_operator_dag.py` |
| `DockerOperator` | 臨時起容器跑任務，依賴完全隔離 | compose 要掛 `/var/run/docker.sock`；臨時容器不讀 `.env`，連線變數要在 `environment` 明給 | `example_docker_operator_dag.py` |
| `DummyOperator` | 佔位、分組、匯合，讓圖好讀 | 不執行任何事 | `example_dummy_tasks_dag.py` |

**範例：最小 DAG（含重試與失敗告警）**——課程 repo 的 DAG **沒有**告警 callback，下面的 `notify_failure` 是補上的寫法（未在課程環境實測）。

```python
import os
from datetime import datetime, timedelta
import requests
from airflow import DAG
from airflow.operators.python import PythonOperator   # Airflow 2.10 寫法

def notify_failure(context):
    url = os.environ.get("ALERT_WEBHOOK_URL")          # 放 .env，不寫死
    if not url:
        print("ALERT_WEBHOOK_URL 未設定，略過告警")      # 降級要印出，不靜默
        return
    ti = context["task_instance"]
    requests.post(url, json={"content": f"❌ {ti.dag_id}.{ti.task_id} 失敗：{ti.log_url}"}, timeout=10)

def crawl():
    from crawler.tasks_crawler_finmind import crawler_finmind   # 重活放進函式，不放頂層
    crawler_finmind("2330")

with DAG(
    dag_id="my_daily_crawl",
    schedule="0 18 * * 1-5",          # 課程用 schedule_interval；Airflow 3 只認 schedule
    start_date=datetime(2024, 1, 1),
    catchup=False,
    max_active_runs=1,
    default_args={"retries": 1, "retry_delay": timedelta(minutes=1),
                  "execution_timeout": timedelta(hours=1),
                  "on_failure_callback": notify_failure},   # 重試用完才會觸發
) as dag:
    PythonOperator(task_id="crawl_2330", python_callable=crawl)
```

- DAG 檔頂層每幾十秒會被 scheduler 重新執行一次——**頂層只放宣告，連 DB、打 API 全包進函式**（第 10 章「想再深入一點」）。

**產出**：`airflow/dags/` 裡一支自己的 DAG；UI 截圖一張：Grid 上看得到一次失敗 → 自動重試 → 綠。

## Step 3：把 Airflow 接上你的爬蟲（45 min）

**目標**：選定三種串法之一，說得出選它的理由。（第 12 章）

| | 串法一：直接爬 | 串法二：發任務給 Celery | 串法三：DockerOperator 跑 producer |
| --- | --- | --- | --- |
| task 內容 | 直接呼叫 `crawler_finmind(stock_id)` | `apply_async(queue="twse")` 發進佇列 | 起容器跑多佇列 producer |
| 誰執行爬蟲 | Airflow 自己 | Celery worker 池 | Celery worker 池 |
| Airflow 要裝爬蟲依賴？ | 要 | 要（要 import 任務物件） | **不要** |
| DAG 等爬完嗎 | 等 | 不等（fire-and-forget） | 不等 |
| 適合 | 量小、流程單純 | 量大、要獨立擴充 worker | 兩邊依賴要徹底隔離 |
| 範例 DAG | `stock_crawler_dag.py` | `stock_crawler_producer_dag.py` | `stock_crawler_docker_producer_dag.py` |

- ⚠️ 串法二、三的 task 全綠 ≠ 爬蟲成功：編排層看不到執行結果，要到 Flower／DB 補驗。
- 演進路徑：初期串法一 → 量大改串法二 → 團隊分工細了改串法三。三者不是選擇題，是長大的三個階段。
- 補充：把 `AIRFLOW__CORE__EXECUTOR` 從 `LocalExecutor` 換成 `CeleryExecutor`，DAG 零改動就能把 task 分到多台 worker（`airflow/docker-compose-airflow-celery-stock.yml`）。

**產出**：計畫書寫明「串法 X，因為 ___」；Airflow Graph 截圖＋DB 查詢結果。

## Step 4：Docker Compose 一鍵起＋七步驟端到端驗證（60 min）

**目標**：一個指令起整套，並**證明會動，不是看起來會動**。（第 13 章）

**做法**：`cp .env.example .env` → `docker build -f airflow/Dockerfile -t stock-airflow:latest .` → `docker compose -f docker-compose-all.yml up -d` → 等 60–90 秒 → 七步驟。

| Step | 驗什麼 | 指令重點 | ✅ 通過標準 |
| --- | --- | --- | --- |
| 1 服務 | 容器都活著 | `docker ps -a` | 全 Up；`airflow-init` 是 `Exited (0)` 屬正常 |
| 2 Web | 入口能開 | `curl` 五個 UI | 都回 200；Metabase 503 再等 30 秒 |
| 3 Logs | 沒有真錯誤 | `docker logs crawler_twse` | worker `ready`、init 出現 `User "admin" created`；啟動初期 `Connection refused` 是暫時現象 |
| 4 發任務 | 任務進得去 | `uv run crawler/producer_multi_queue.py` | 印出 `send task_2330 task` |
| 5 執行 | 有人做 | `docker logs ... \| grep succeeded` | twse、tpex 各自 succeeded |
| 6 資料 | 資料落地 | `SELECT stock_id, COUNT(*) ...` | 兩支股票各數百筆——**succeeded 還不夠，資料在庫裡才算數** |
| 7 編排 | DAG 載入 | `airflow dags list \| grep stock` | 列出 stock DAG；trigger 後回 Step 6 看筆數增加 |

**範例：compose 片段**（節錄自 `docker-compose-all.yml` 的 `airflow-webserver`，環境變數略）

```yaml
  airflow-webserver:
    image: stock-airflow:latest
    command: bash -c "exec airflow webserver"
    ports:
      - "8081:8080"                         # 避開 phpMyAdmin 的 8080
    environment:
      MYSQL_HOST: mysql                     # 容器互連一律用服務名
      GCP_PROJECT_ID: ${GCP_PROJECT_ID:-}   # 從 .env 帶入，沒設就略過 BQ
    volumes:
      - ./airflow/dags:/opt/airflow/dags    # DAG 改了不用重啟，等 30–60 秒
      - ./crawler:/opt/airflow/crawler
    depends_on:
      airflow-init:
        condition: service_completed_successfully   # init 跑完才起
```

- 本機預設值（僅限本機教學，上雲必換）：Airflow `admin / admin`、MySQL `root / 1234`、RabbitMQ `worker / worker`。
- 時間不夠？最小路徑：只起 `mysql` ＋ Airflow 三容器，跑 Step 1、2、6、7。

**產出**：README「一行起」區塊＋七步驟驗證結果（可貼成表）。

## Step 5：設定與機密——`.env` → Secret Manager → GitHub Secrets（30 min）

**目標**：repo 裡零真密碼；同一份 compose 走遍本機與雲端，差異只在 `.env`。（補充 G、第 16 章、補充 I）

| 層級 | 放在哪 | 誰來讀 | 規矩 |
| --- | --- | --- | --- |
| 程式 | `os.environ.get("MYSQL_HOST", "127.0.0.1")` | Python | 找得到就用，找不到用預設 |
| Compose 解析 | `${VAR:-預設}`＋同目錄 `.env` | docker compose | 優先序：shell > `.env` > 預設；先用 `docker compose config` 看替換結果 |
| 容器注入 | `environment:`／`env_file:` | 容器內程式 | 跟 `${}` 替換是兩件事 |
| 雲端 | Secret Manager | VM 服務帳戶（`secretAccessor` 授權） | 部署時取出寫進 `.env`：`echo "MYSQL_PASSWORD=$(gcloud secrets versions access latest --secret=mysql-password)" >> .env`；誰讀過有稽核紀錄 |
| CI／CD | GitHub Secrets（例：`GCP_SA_KEY`） | workflow | 服務帳戶只給最小權限角色 |

- `.gitignore` 擋 `.env*`，repo 只放 `.env.example`（變數名＋假值）。評分表把「沒有 hardcoded secret」「`.env.example` 完整」列為加分項。

**產出**：`.env.example`、`.gitignore`、README 的「設定」段。

## Step 6：部署階梯與成本控制（依階段，每階 1–3 小時，估計值）

**目標**：知道自己的專題該爬到第幾階，並在第一天就裝好「保險絲」。（第 14–17 章、補充 H／L）

| 階 | 樣貌 | 改了什麼 | 費用（課程記載的約略值） | 收工動作 |
| --- | --- | --- | --- | --- |
| 0 本機 | Compose 13 個容器 | — | 0 | `docker compose down` |
| 1 單台 GCE | `e2-standard-2` 跑整套（雲端不跑 Metabase） | 防火牆只放行自己 IP 的 UI port，3306／5672 不開 | 開機約 NT$2.4／小時；20GB 磁碟約 NT$25／月；全天開約 NT$1,600／月 | `gcloud compute instances stop` |
| 2 多 VM＋Cloud SQL | VM1 跑 Airflow＋RabbitMQ、VM2 跑 worker、MySQL 換託管 | 只改 `.env` 幾行；機器間用內部 IP | Cloud SQL `db-f1-micro` 約 US$0.07／小時 | Cloud SQL `--activation-policy=NEVER`＋兩台 VM stop |
| 3 Composer（選修） | 託管 Airflow，DAG 上傳 bucket | 套件、環境變數改由 Composer 設定 | 最小環境每月數百美元起；建置 20–30 分鐘 | **沒有停機選項，只能 `environments delete`** |
| 旁支 | Cloud Run 對外 API（補充 H）；地端資料回填 BigQuery（補充 L） | build → tag → push → deploy | 閒置縮到零 | `gcloud run services delete` |

- **第一件事：預算警告**。帳單 → 預算與警告 → 僅傳送警告、目標 NT$3,000、門檻 50%／90%／100% 寄信（第 14 章 Part B）。$300 美元／90 天試用額度，全天開機的燃燒速度撐不到兩個月（第 14 章，約略值）。
- **指定收工負責人**：多人共用 VM 時「大家都以為別人會關」（第 14 章團體專案上雲）。重開工後外部 IP 會換，要重 patch Cloud SQL 授權網路。
- **架構圖跟著階梯更新**（補充 A）：本機一張、雲端一張；方框用實際容器名、圓柱代表資料儲存、實線是資料流、虛線是觸發；Airflow 四個容器收成一個框。原始檔 `.drawio` 進 repo，範本見 `課程手冊/drawio/本地端架構圖.drawio`、`GCP雲端架構圖.drawio`。
- 專題建議停在第 0–1 階就夠拿滿分；第 2 階以上是加分，砍掉不是失敗。

**產出**：預算警告截圖、雲端架構圖、README 的「部署與收工」段。

## Step 7：監控、排錯、測試與 CI（60 min）

**目標**：出事時 10 分鐘內定位到哪一層；每個 PR 自動跑測試，紅了擋 merge。

**做法 A：監控介面分工＋排錯五步 SOP**（第 07、12 章；由外而內，先定位層、再看細節）

| 順序 | 看哪裡 | 回答什麼 | 典型訊號 |
| --- | --- | --- | --- |
| 0 | Airflow UI（Graph／Grid／Logs） | 哪一次 run、哪個 task 紅了 | Grid 一眼看出哪天失敗；Clear 單獨重跑 |
| 1 | Flower `:5555` | 任務到了嗎？成功或失敗？ | 沒紀錄 → 往下查 |
| 2 | RabbitMQ `:15672` | 訊息進佇列了嗎？堆積嗎？ | Ready 堆高 = 沒人領；Unacked 卡住 = 領了做不完 |
| 3 | Portainer `:9000`（選做） | 容器活著嗎？一直 restart？ | 看 log 找 crash 原因 |
| 4 | phpMyAdmin／SQL | 資料真的寫進去了嗎？ | SUCCESS 但沒資料 → `MYSQL_HOST` 指錯 |
| 5 | `docker logs <容器>` | 以上都看不出來 | 讀最原始的錯誤 |

- 告警：藍圖 6️⃣ Observe 建議 Email／Slack／Discord webhook；課程 DAG 未內建，用 Step 2 的 `on_failure_callback` 補上。雲端另看 Cloud Monitoring／Logging 與帳單。

**做法 B：測試金字塔＋CI**（補充 F、補充 I）

| 層 | 測什麼 | 需要服務 | 課程檔案 | CI 跑不跑 |
| --- | --- | --- | --- | --- |
| 單元 | 函式邏輯、設定覆蓋（mock 掉網路） | ❌ | `tests/test_config.py`、`tests/test_crawler_parsing.py` | ✅ 每次 push／PR |
| 整合 | upsert 冪等（真連 MySQL） | ✅ | `tests/test_upsert_integration.py` | 預設排除；要跑需加 `services:` |
| 端到端 | 整套系統 | ✅ 全部 | Step 4 七步驟 | 人工／腳本 |

**範例：GitHub Actions 骨架**（精簡自 `.github/workflows/ci.yml`；CD 段為補充 I 的路徑示範，課程 repo 未掛）

```yaml
name: CI
on:
  push: { branches: [main] }
  pull_request: { branches: [main] }
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: astral-sh/setup-uv@v5
      - run: uv sync --frozen                                  # 照 uv.lock 還原，環境可重現
      - run: uv run pytest tests/ -m "not integration" -v      # CI 沒 MySQL，排除整合測試
  # deploy: needs: test → auth(secrets.GCP_SA_KEY) → build → push → gcloud run deploy；tag 用 ${{ github.sha }}
```

- 協作 SOP 最低標準：**每個 PR 都要 CI 過，不過不准合**；可再加 ruff／black、`docker compose` 起得來檢查（`05-collaboration-sop.md` §8）。

**產出**：Actions 頁綠勾截圖、告警測試訊息截圖（故意讓 task 失敗一次）、README 的「監控與排錯」段。

## 卡關 → 處方

| 卡關 | 處方 |
| --- | --- |
| Airflow UI 打不開、報 `You need to initialize the database` | 等 `airflow-init` 印出 `User "admin" created`，再 `docker restart airflow-webserver airflow-scheduler` |
| 8080 port 衝突 | phpMyAdmin 也用 8080；單獨跑 Airflow 先關 phpMyAdmin，或用 `docker-compose-all.yml`（Airflow 走 8081） |
| DAG 沒出現在列表 | 看 scheduler log——多半是語法錯誤；改檔後等 30–60 秒 |
| 排程時間差 8 小時 | 明寫 `Asia/Taipei`（APScheduler `timezone=`、compose `AIRFLOW__CORE__DEFAULT_TIMEZONE`）；GitHub Actions cron 是 UTC |
| unpause 了卻沒跑 | 正常：Airflow 等到下一個排程點才跑（批次思維，第 10 章）；想立刻驗證就手動 trigger |
| 分支後 end 也 skipped | end 的 `trigger_rule` 改 `none_failed_min_one_success` |
| producer_dag 全綠但 DB 沒資料 | fire-and-forget：看 Flower 與 worker log；確認 `apply_async(queue=...)` 跟 worker `-Q` 對得上 |
| DockerOperator 容器報 `Connection refused` | 臨時容器不讀 `.env`，在 `environment` 明給 `RABBITMQ_HOST`、`MYSQL_HOST` |
| 改了 `.env` 密碼登入仍失敗 | MySQL 密碼只在 volume 初始化時生效：`down -v` 重建或 `ALTER USER` |
| 本機呼叫 GCP 報 `DefaultCredentialsError` | 重開終端機後 `export GOOGLE_APPLICATION_CREDENTIALS` 失效，重設（補充 L） |
| 停機再開後網址連不上 | 外部 IP 換了：`gcloud compute instances list` 查新 IP，重 patch Cloud SQL 授權網路 |
| 額度燒太快 | 回帳單看「使用費」欄；VM stop、Cloud SQL `NEVER`、Composer 直接 delete |
| 課程 DAG 在 Airflow 3 上壞掉 | 課程用 Airflow 2.10.4；`schedule_interval`、`python_operator` 匯入路徑、`execution_date` 在 3 已移除（第 17 章 Part C） |

## 交付前檢查清單

- [ ] 計畫書寫明排程工具與串法，各附一句「為什麼不選另一個」
- [ ] DAG 有 `retries`、`retry_delay`、`execution_timeout`、`catchup=False`、明寫時區
- [ ] 故意讓一個 task 失敗：看到自動重試，重試用完收到告警
- [ ] DAG 頂層沒有連 DB／打 API 的程式碼
- [ ] `docker compose up -d` 一行起；七步驟驗證全過（Step 6 資料真的落地）
- [ ] repo 搜不到真密碼；有 `.env.example`；`.env` 在 `.gitignore`
- [ ] 上雲者：預算警告已設；收工負責人已指定；`gcloud compute instances list` 顯示 TERMINATED
- [ ] 本機與雲端架構圖都在 README 第一頁，`.drawio` 原始檔進 repo
- [ ] Actions 頁有綠勾；branch protection 要求 CI 通過才能 merge
- [ ] README 有「一行起」「部署與收工」「監控與排錯」三段

打勾 ≥ 8 → 自動化這一塊可以進 Demo Day。

## 來源注意事項（改寫成正式教材前先統一）

| 位置 | 不一致 | 建議 |
| --- | --- | --- |
| 註解與實際不符 | `ci.yml` 檔頭寫「手冊18」「補充C」（實為補充 I／F）；`docker-compose-all.yml` 把 MongoDB 標成「補充F」（實為補充 D）；`hahow-…/airflow/dags/hahow_crawler_*_dag.py` 註解寫「重試 2 次」，程式是 `retries: 1` | 章節以手冊章名為準，參數以程式值為準 |
| Airflow UI port | 單獨 compose 用 8080、`docker-compose-all.yml` 用 8081；hahow 的 phpMyAdmin 用 8000 | 每份文件標明用哪個 compose 檔 |
| 費用數字 | NT$2.4／小時、NT$1,600／月、US$0.07／小時皆為課程撰寫時的估價 | 一律標「約略值」，附實測日期 |
