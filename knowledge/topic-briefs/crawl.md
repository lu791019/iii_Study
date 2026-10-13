# Brief：資料爬蟲（從一支腳本到分散式、可重跑的爬蟲）

> 用途：給撰寫教學文件的人當素材。成品文件結構＝開頭一句話 → 流程總覽 → 編號步驟（目標／做法表／短 Python／產出）→ 交付前檢查清單。
> 縮寫：**SC** = `stock-crawler-de-course-materials/`、**HH** = `hahow-crawler-de-course-materials/`、**DD** = `DataDreamers-decrosscareer-student-project-handbook/`、**DE** = `de-course/`（靜態 HTML 範例）。
> 帳密只寫本機預設值：RabbitMQ `worker / worker`、MySQL `root / 1234`（SC `crawler/config.py`、HH `data_ingestion/config.py`）。

---

## 0. 開頭一句話（建議稿）

> 這份給已經能用 `requests` 抓到資料的學員：讀完你能把一支「能跑就好」的爬蟲，一步步改成參數化、可分散、失敗會重試、重跑不會重複的資料管線，而且每一步都能驗證沒改壞。
>
> 心法：**Celery 換掉的是「誰來執行、在哪裡執行」，不是「執行什麼」。**（SC `課程手冊/補充J` 這一篇你學到了）

## 1. 流程總覽（一行箭頭）

```
徒手打一次 → 函式參數化（不寫死）→ 抽出一顆任務 → producer → RabbitMQ → worker → 多佇列分流 → retry / acks_late → upsert 入庫 → Airflow 定時發任務
```

| 階段 | 換掉的東西 | 沒動的東西 | 出處 |
|---|---|---|---|
| 單支腳本 → 參數化函式 | 寫死的清單、日期、連線字串 | 抓取邏輯 | SC 補充J Step 0–2 |
| 參數化 → Celery | 誰來執行（迴圈 → worker） | 任務主體（只加 `@app.task`、`.delay`） | SC 補充J Step 5；SC `課程手冊02` |
| 單佇列 → 多佇列 | 任務送去哪排隊（`apply_async(queue=)`、`-Q`） | worker 程式碼 | SC `課程手冊03` |
| 不處理失敗 → retry／acks_late | 失敗時訊息的命運 | 抓取邏輯 | SC `課程手冊04` |
| append → upsert | 寫入方式（複合主鍵＋`on_duplicate_key_update`） | 抓取與派工 | SC `課程手冊05`、`06` |
| 人工發任務 → Airflow | 誰呼叫 producer | 所有爬蟲程式 | SC 補充J Step 6 |

## 2. 做完這份，你會做到

1. 判斷目標網站是靜態 HTML、背後有 JSON API，還是動態頁，並選對工具。
2. 把寫死的爬蟲改成 `crawl(stock_id, start_date, end_date)` 這種「輸入全靠參數」的任務。
3. 啟動 worker、用 producer 發任務，在 worker log 看到 `succeeded`（不只是 `received`）。
4. 用兩條佇列分流，並用 `--scale` 加開 worker。
5. 說出 `self.retry()` 和 `Reject(requeue=True)` 的差別，以及為什麼開了 `acks_late` 任務就必須冪等。
6. 用 upsert 寫入資料庫，重跑三次筆數不變。

---

## 3. 先判斷頁面型態：靜態 HTML vs API vs 動態頁

| 型態 | 怎麼認出來 | 工具 | repo 範例 | 穩定度 |
|---|---|---|---|---|
| 公開 API（有文件） | 官方文件、需要 dataset／token 參數 | `requests.get(url, params=...)` → `resp.json()` → `pd.DataFrame` | FinMind（SC `crawler/tasks_crawler_finmind.py`） | 🟢 最穩 |
| 網站背後的 JSON API | DevTools → Network → Fetch/XHR 看到 JSON 回應 | 同上，加 `User-Agent`；照回應的分頁欄位翻頁 | Hahow `api.hahow.in/api/products/search`，用 `_metadata.limit/count` 算頁數（HH `data_ingestion/hahow_crawler_course.py`） | 🟡 沒文件、可能改 |
| 靜態 HTML | 「檢視原始碼」就看得到資料 | `requests` + `BeautifulSoup(resp.text, "html.parser")` + `find_all` | PTT：`cookie: over18=1`、`div.r-ent`（DE `ep02/crawlers/ptt_crawler.py`） | 🟡 改版就壞 |
| 動態頁（JS 渲染） | 原始碼沒資料、要捲動或登入 | Playwright／Selenium（＋sleep）；Scrapy 做大量靜態 | PChome、104（DD `scenario-2`、`scenario-6`） | 🔴 成本最高、反爬最硬 |

- 選型原則：**用 API 就別用爬蟲，用爬蟲也不能寫死**（DD `02-pipeline-blueprint.md` Source 段）。
- 爬蟲任務是 **I/O 密集**：時間花在等 `requests.get`，可開遠高於核心數的併發，Celery pool 首選 `gevent`（SC `課程手冊02` 想再深入；SC 補充B pool 表）。

## 4. 防封鎖禮儀（也是合規底線）

| 項目 | 做法 | 出處 |
|---|---|---|
| robots.txt | 爬之前先看，遵守 Disallow | DD `scenario-2-ecommerce.md` 合規 disclaimer |
| 速率限制 | 每秒 < 1 request；範例 commit「respect robots.txt and add 2s sleep」；104 用 sleep + random | DD `scenario-2`、`scenario-6` 雷區 |
| Headers | `User-Agent` 表明身份（可附 email）；PTT 要帶 `cookie: over18=1` | DD `scenario-2`；DE `ptt_crawler.py` |
| Token／配額 | FinMind 免費版有請求上限，大量抓要註冊 token（放環境變數） | SC `課程手冊02` 排錯表 |
| 退避 | 429／timeout 屬暫時性故障 → retry，間隔遞增 | SC `課程手冊04` 情境 1 |
| 併發上限 | 「你不是要做 DDoS」：worker 併發 × worker 數 ＝ 對方承受的請求數 | DD `04-pain-points-industry-map.md` |
| 監控 | 記錄爬取成功率、反爬 alert | DD `scenario-2`、`scenario-6` Observe |
| 爬到的內容 | `data/raw/` 進 `.gitignore`，不 commit | DD `scenario-2` ⑤ |

## 5. hahow-crawler 與 stock-crawler 的差異

| 面向 | stock-crawler（SC） | hahow-crawler（HH） |
|---|---|---|
| 資料來源 | FinMind 公開 API（有文件、dataset 參數） | Hahow 網站背後 JSON API（無文件，帶瀏覽器 UA） |
| 分頁 | 不分頁，用 `start_date / end_date` 控制量 | 先打一次拿 `_metadata.count / limit`，`math.ceil` 算頁數再逐頁打 |
| 任務顆粒度 | 一支股票一顆任務（`stock_id`） | 一個分類一顆任務（`category`，12 類）；課程與文章兩種任務 |
| 佇列分流 | `twse` / `tpex`（上市／上櫃） | `hahow_course` / `hahow_article`（`producer_crawler_hahow_by_queue.py`） |
| 入庫 | Ch05 `to_sql(append)` 故意製造重複 → Ch06 複合主鍵 `stock_id + date` upsert；另有 MongoDB `update_one(upsert=True)` 版 | 主表 `hahow_course` 以 `id` upsert；`hahow_course_sales` 只 insert、帶 `captured_at` 做歷史快照（`data_ingestion/mysql.py`） |
| 時間處理 | 整理層 `pd.to_datetime(...).dt.date`（補充J） | `hahow_crawler_common.to_datetime()` 把 `...T...Z` 轉 datetime；`uploaded_at` 用 UTC |
| 失敗處理 | `worker_demo.py` + `tasks_demo_fail.py` 四情境示範；正式 `worker.py` 的 `acks_late` 註解掉 | 無 retry、無 acks_late |
| 限速 | 無 sleep（5 支股票量小） | 每頁 `time.sleep(0.1)` |
| 演進教材 | 17 章手冊＋補充 A–L，`example/evolution/step0–6` | README＋`*_course.py` → `*_optimized.py` → `*_optimized_sales.py` 三版對照 |
| 本機服務 | MySQL `mydb`、phpMyAdmin :8080 | MySQL `hahow`、phpMyAdmin :8000 |
| 環境變數名 | `WORKER_ACCOUNT`、`MYSQL_ACCOUNT` | `WORKER_USERNAME`、`MYSQL_USERNAME` |

> 教學建議：SC 教「每一步只換一件事」的演進；HH 當「換一個資料源，同一套骨架照樣成立」的練習題。

## 6. 七步拆解（SC 補充J）

兩條判準：**拆的依據是「會為了不同理由改變」**；**每一步都要能執行**，輸出與前一步相同。

| 步驟 | 這一步做的事 | 沒動的東西 | 驗收 | 檔案 |
|---|---|---|---|---|
| Step 0 | 單體：一個函式包辦決定抓什麼、呼叫 API、存檔；設定寫死 | — | 產生 CSV | `example/evolution/step0_single_file.py` |
| Step 1 | 同檔拆成抓、整理、存三個函式，`main()` 串接；補上整理階段 | 檔案數、儲存行為 | CSV／MySQL 結果同 Step 0 | `step1_functions.py` |
| Step 2 | 抽出一顆任務 `crawl(stock_id, start, end)`（樞紐） | 三個階段函式 | 輸出同 Step 1；三個可分散條件成立 | `step2_task_function.py` |
| Step 3 | 三函式各自成檔：client／transformer／repository＋config 讀環境變數 | 函式內容 | 只改 `STORAGE=csv,mysql`，其他檔不動 | `step3_modules/` |
| Step 4 | task 與 producer 分家，`build_jobs()` 決定顆粒度 | 四層內容 | 輸出同 Step 3，仍無 Celery | `step4_task/` |
| Step 5 | 接 Celery＋RabbitMQ：加 `worker.py`、`@app.task()`、`.delay()` | 任務主體 | worker log 出現 `succeeded` | `step5_celery/` |
| Step 6 | Airflow 定時發任務，DAG 不放爬蟲本身 | 所有爬蟲程式 | DAG 綠＋worker `succeeded`＋DB 有新資料 | `step6_airflow/` |

- 任務可分散的三條件（Step 2）：① 輸入全由參數帶入 ② 不依賴其他任務結果 ③ 不回傳結果，直接寫進儲存層。
- 判準一對照（改動理由 → 只動哪層）：範圍變了→producer；來源變了→client；資料形狀變了→transformer；儲存目標變了→repository；環境變了→config；執行方式變了→worker；觸發時機變了→DAG。
- 反向驗證：拿掉 `@app.task()` 與 `.delay`，程式仍能單機跑。

---

## 7. 編號步驟（建議時間為估計值）

### Step 1：判斷型態＋徒手打一次（20 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 確認 API 能通、看懂回傳結構，再談工程化 | ① 依 §3 判斷型態 ② 不經 Celery、不開 Docker，用 inline Python 打一次（SC `課程手冊02` 熱身）③ `print(data.keys())` 看外層結構（FinMind：`msg / status / data`） | 一份能印出 DataFrame 的試抓輸出 |

### Step 2：參數化＋拆三層（45 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 不寫死；抓、整理、存各管各的 | ① 反例：HH `example/hahow.py` 第 8 行在函式內寫死 `category = "programming"`，參數形同虛設 ② 設定集中到 `config.py`，用 `os.environ.get(鍵, 預設值)` ③ client 只抓、transformer 只整理（回傳新 DataFrame）、repository 只存 | `config.py`、`client.py`、`transformer.py`、`repository.py` |

```python
# 改寫自 SC example/evolution/step3_modules/config.py + client.py
import os
import pandas as pd
import requests

FINMIND_URL = os.environ.get("FINMIND_URL", "https://api.finmindtrade.com/api/v4/data")
FINMIND_DATASET = os.environ.get("FINMIND_DATASET", "TaiwanStockPrice")
FINMIND_TOKEN = os.environ.get("FINMIND_TOKEN", "")      # 不寫進程式碼

def fetch_stock_price(stock_id: str, start_date: str, end_date: str) -> pd.DataFrame:
    """只負責拿資料：不知道資料會被整理成什麼、存到哪裡。"""
    params = {"dataset": FINMIND_DATASET, "data_id": stock_id,
              "start_date": start_date, "end_date": end_date, "token": FINMIND_TOKEN}
    resp = requests.get(FINMIND_URL, params=params, timeout=10)
    resp.raise_for_status()            # 與 repo 不同：拋例外，讓上層決定要不要 retry
    return pd.DataFrame(resp.json()["data"])
```

### Step 3：接上 producer／broker／worker（45 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 任務從「當場執行」變成「送進佇列由 worker 執行」 | ① `worker.py` 宣告 `broker`（`pyamqp://帳號:密碼@主機:埠/`）與 `include` ② 任務加 `@app.task()` ③ producer 改 `.delay()` ④ 參數只用可序列化型別（字串、數字、list、dict），不能傳 DataFrame 或連線（SC 補充J Step 5） | worker `[tasks]` 列出任務；log 有 `succeeded` |

- 角色比喻：Producer＝點餐客人、Broker＝出單機、Worker＝廚師（SC `課程手冊01`）。
- `include` 漏掉 → `Received unregistered task`；用 `celery -A` 啟動要寫完整套件路徑並放 `__init__.py`（SC 補充J Step 5）。

### Step 4：多佇列分流與擴充（30 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 某類暴量不卡住另一類；哪條忙就只加哪條 | ① producer 用 `.s(...).apply_async(queue=...)`（`.delay()` 不能指定佇列）② worker 加 `-Q twse`、`--hostname=twse@%h` ③ `docker compose -f compose-advanced/docker-compose-worker-network.yml up -d --scale crawler_twse=3`（compose 不可寫死 `container_name`） | 兩條佇列各自被對應 worker 消費 |

```python
# 改寫自 SC crawler/producer_multi_queue.py 與 HH producer_crawler_hahow_by_queue.py
from crawler.tasks_crawl import crawl    # 示意路徑：Step 5 的 crawl 任務

JOBS = [  # build_jobs()：任務顆粒度在這裡決定
    {"stock_id": "2330", "queue": "twse"},
    {"stock_id": "00679B", "queue": "tpex"},
]

for job in JOBS:
    crawl.s(stock_id=job["stock_id"], start_date="2025-01-02",
            end_date="2025-06-17").apply_async(queue=job["queue"])
    print(f"送出 {job['stock_id']} → {job['queue']}")
# worker 端：celery -A crawler.worker worker -l info --hostname=twse@%h -Q twse
```

- worker 能隨意加減的三原因：無狀態、不開對外 port、共用 broker（SC `課程手冊03`）。

### Step 5：失敗處理 retry／acks_late（40 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 暫時性故障自動重試；worker 被殺任務不遺失 | ① 暫時性（timeout、429）→ `self.retry()`，有 `max_retries` 上限 ② 永久性（毒訊息）→ 不要 `Reject(requeue=True)` 無限循環，丟棄或進 DLQ ③ 行程死亡 → `acks_late=True` + `task_reject_on_worker_lost=True` ④ 開了 acks_late 就可能重跑 → 寫入必須冪等（Step 6） | 故意失敗的實驗紀錄：worker log＋RabbitMQ Ready/Unacked |

```python
# 改寫自 SC crawler/tasks_demo_fail.py 情境 1 + 補充J step5_celery/task.py
import requests
from crawler.worker import app                       # 以下為示意路徑，依專案調整
from crawler.client import fetch_stock_price
from crawler.transformer import transform
from crawler.repository import upsert_stock_price

@app.task(bind=True, acks_late=True, max_retries=3, default_retry_delay=5)
def crawl(self, stock_id: str, start_date: str, end_date: str):
    print(f"{stock_id} 第 {self.request.retries + 1} 次嘗試")
    try:
        raw = fetch_stock_price(stock_id, start_date, end_date)
    except requests.RequestException as exc:              # 暫時性錯誤才重試
        raise self.retry(exc=exc, countdown=5 * 2 ** self.request.retries)
    if raw.empty:
        print(f"{stock_id} 沒有資料，不整理也不寫入")
        return
    upsert_stock_price(transform(raw))                    # 冪等：重跑也不會重複
```

| | `self.retry()` | `Reject(requeue=True)` |
|---|---|---|
| 訊息行為 | 發**新任務**回佇列 | **原訊息**放回佇列 |
| 次數上限 | 有（`max_retries`） | 沒有，毫秒級無限循環 |
| 最終結果 | SUCCESS 或 FAILURE | 永遠留在佇列 |

（SC `課程手冊04`「retry vs requeue 對照」；Airflow `default_args.retries` 重試的是「發送任務」，不是爬蟲本身，SC 補充J Step 6、`airflow/dags/stock_crawler_producer_dag.py` 第 74 行）

### Step 6：upsert 入庫，做到冪等（40 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 每天重抓、失敗重試、補資料重跑都不會越跑越髒 | ① 照 SC `課程手冊06` 走一遍：無主鍵表 append 兩輪 → 筆數翻倍 → 有主鍵表 append → 1062 報錯 → 換 upsert ② 主鍵＝規格裡的唯一鍵（股價 `stock_id + date`；Hahow `id`）③ 歷史快照另開 insert-only 表並帶 `captured_at`（HH） ④ 多 worker 同時寫 → 用 DB 層 upsert，不用 DataFrame 層「讀回→去重→清空→寫回」 | 重跑三次筆數不變的截圖 |

```python
# 改寫自 SC 課程手冊06 + HH data_ingestion/mysql.py upload_data_to_mysql_upsert（改成批次）
from sqlalchemy import BigInteger, Column, Date, Float, MetaData, String, Table, create_engine
from sqlalchemy.dialects.mysql import insert

engine = create_engine("mysql+pymysql://root:1234@127.0.0.1:3306/mydb")  # 本機預設值；正式從 config 讀
metadata = MetaData()
stock_price = Table("TaiwanStockPrice", metadata,
    Column("stock_id", String(50), primary_key=True),   # 複合主鍵 = 唯一身分
    Column("date", Date, primary_key=True),
    Column("Trading_Volume", BigInteger), Column("open", Float), Column("max", Float),
    Column("min", Float), Column("close", Float), Column("spread", Float))

def upsert_stock_price(df):
    metadata.create_all(engine, tables=[stock_price])       # 表不存在才建
    rows = df[[c.name for c in stock_price.columns]].to_dict(orient="records")
    stmt = insert(stock_price).values(rows)
    stmt = stmt.on_duplicate_key_update(
        {c.name: stmt.inserted[c.name] for c in stock_price.columns if not c.primary_key})
    with engine.begin() as conn:                            # 一個 transaction
        conn.execute(stmt)
    print(f"UPSERT {len(rows)} 筆")
```

- `on_duplicate_key_update` 是 MySQL 專用；PostgreSQL 用 `ON CONFLICT`；MongoDB 用 `update_one(filter, {"$set": doc}, upsert=True)`（SC `crawler/tasks_crawler_finmind_mongo.py`）。
- 寫入前 `df.replace({pd.NaT: None, np.nan: None})`，否則 NaN 寫不進 MySQL（HH `hahow_crawler_course_optimized_sales.py`）。

### Step 7：觀測、排錯、上排程（30 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 證明會動，不是看起來會動 | ① 排錯 SOP 由外而內：Flower → RabbitMQ → Portainer → phpMyAdmin → `docker logs`（SC `課程手冊07`）② Airflow DAG 只發任務、不跑爬蟲；DAG 綠只代表「送出」，要再看 worker `succeeded` 與 DB 筆數（SC 補充J Step 6） | 四介面同步觀察截圖＋DB 筆數查詢 |

---

## 8. 卡關 → 處方（整合自各章排錯表）

| 你遇到的狀況 | 原因 | 怎麼解 | 出處 |
|---|---|---|---|
| worker 印出一行錯誤、不是表格 | API 回非 200（限流、參數錯） | 暫時 `print(data)` 看 `msg` | SC `課程手冊02` |
| 印出空 DataFrame | 代碼錯或期間沒資料 | 用 2330 測 | SC `課程手冊02` |
| 抓幾支就被擋 | 免費限流 | 減量、加 sleep、拿 token | SC `課程手冊02` |
| `Received unregistered task` | `worker.py` 的 `include` 漏了模組 | 補 include，重開 worker | SC 補充J Step 5 |
| 任務送出去永遠沒人做 | worker 沒加對應 `-Q`，只吃預設 `celery` 佇列 | worker 加 `-Q twse` | SC `課程手冊03` |
| 想分流卻用 `.delay()` | `.delay()` 不能指定佇列 | `.s(...).apply_async(queue=...)` | SC `課程手冊03` |
| 佇列名稱對不上（`twse` vs `twes`） | 打錯字 | 兩邊名稱完全一致 | SC `課程手冊03` |
| `--scale` 報 container name 衝突 | compose 寫死 `container_name` | 用 `compose-advanced/` 網路版 | SC `課程手冊03` |
| `Reject` 好像沒效果 | 用了沒開 acks_late 的 `worker.py` | 用 `-A crawler.worker_demo` | SC `課程手冊04` |
| worker 被同一則訊息佔住 | `Reject(requeue=True)` 毒訊息無限循環 | 設重試上限／丟棄／DLQ | SC `課程手冊04` |
| `Access denied for user` | 帳密或 host 不符 | 本機預設 root / 1234；確認容器在跑 | SC `課程手冊05` |
| Docker worker 連不到 DB | 容器內仍用 127.0.0.1 | compose 設 `MYSQL_HOST=mysql` | SC `課程手冊05` |
| 重跑筆數變多 | append、沒主鍵或主鍵設錯 | 複合主鍵＋upsert | SC `課程手冊06` |
| `on_duplicate_key_update` 語法錯 | MySQL 專用 | import `sqlalchemy.dialects.mysql.insert` | SC `課程手冊06` |
| 逐列 upsert 很慢 | `iterrows()` 一列一列寫 | 改批次 upsert | SC `課程手冊06` |
| 任務 SUCCESS 但查不到資料 | `MYSQL_HOST` 沒設對／寫錯表 | 查 phpMyAdmin | SC `課程手冊07` |
| Airflow 全綠但資料沒進來 | DAG 只代表任務已送出 | 看 worker log、Flower、DB 筆數 | SC 補充J Step 6 |
| 不同分類資料互相覆蓋 | `to_sql(mode="replace")` | 改 upsert（HH 程式內註解） | HH `hahow_crawler_course_optimized_sales.py` |
| 104／電商爬太快被 ban | 無限速、UA 固定 | sleep + random + rotating UA | DD `scenario-6`、`scenario-2` 雷區 |

## 9. 交付前檢查清單

- [ ] 已說明來源型態（API／JSON 端點／靜態／動態），並說得出為什麼不用更簡單的方式。
- [ ] 看過 robots.txt；程式有 User-Agent、限速（建議 < 1 req/s）、timeout。
- [ ] 函式內沒有寫死的清單、日期、URL、帳密；設定全在 `config.py` 讀環境變數。
- [ ] 任務參數只用可序列化型別，且滿足三條件（參數化、不互相依賴、不回傳結果）。
- [ ] worker `[tasks]` 列得出任務；log 看得到 `succeeded`，不只是 `received`。
- [ ] 至少兩條佇列分流，或寫明為什麼只需要一條。
- [ ] 暫時性錯誤有 retry 且有上限；開了 `acks_late` 的任務寫入是冪等的。
- [ ] 資料表有主鍵（唯一鍵），重跑三次筆數不變。
- [ ] 拿掉 `@app.task()` 與 `.delay` 仍能單機執行（反向驗證）。
- [ ] `.env`、`data/raw/`、`output/` 不進 git；repo 內只有本機預設值，沒有真實帳密。

---

## 10. 給撰寫者的備註

- 限速衝突：HH 每頁 `time.sleep(0.1)`（約 10 req/s，估計值）與 DD scenario-2「每秒 < 1 request」不一致，教材以 DD 為準，HH 當改進題。
- SC 正式版 `crawler/worker.py` 第 44 行 `task_acks_late` 是註解狀態；acks_late 只在 `worker_demo.py` 開。教材若說「正式版有 acks_late」會是錯的。
- SC 正式版 `crawler/tasks_crawler_finmind.py` 仍是抓、整理、存同檔（相當於補充J Step 1–3 之間），補充J「七步與課程正式程式的對照」已明講，可直接引用。
- 本 brief 的四段程式都是改寫，不是 repo 原文：Step 2 的 client 改成 `raise_for_status()` 以配合 retry；Step 6 把 repo 的逐列 upsert 改成批次。寫進教材前要實測，並註明實測版本。
- phpMyAdmin 埠號兩個 repo 不同（SC :8080、HH :8000），寫指令時別混用。
