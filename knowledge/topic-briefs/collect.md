# Brief：資料收集（Source → Ingest 的前半段）

> 用途：給撰寫教學文件的人當素材。成品文件結構＝開頭一句話 → 流程總覽 → 編號步驟（目標／做法表／短 Python／產出）→ 交付前檢查清單。
> 縮寫：**DD** = `DataDreamers-decrosscareer-student-project-handbook/`、**SC** = `stock-crawler-de-course-materials/`、**HH** = `hahow-crawler-de-course-materials/`、**MT** = `midterm-mvp-template/`（補充來源，資料規格寫法最完整）。
> 帳密只出現本機預設值；數字凡是來源沒有核實的，都標「約略值／估計值」。

---

## 0. 開頭一句話（建議稿）

> 這份給剛要動手做專題的轉職學員：讀完你能從業務問題倒推出要哪些資料、挑到「最省力又合法」的來源、寫出一頁資料規格，並把第一批 Raw 資料原封不動存下來。
>
> 心法：**有 API 就別用爬蟲；用了爬蟲也不要寫死。**（DD `02-pipeline-blueprint.md` Source 段「選型原則」）

## 1. 流程總覽（一行箭頭）

```
業務問題 → 需要的資料清單 → 候選來源盤點 → 五維度評估 + 徒手試抓 → 資料規格 → 可重跑的收集程式 → Raw 落地（含 metadata）
```

- 位置：六階段藍圖 `Source → Ingest → Storage → Process → Serve → Observe` 的前兩格（DD `02-pipeline-blueprint.md`）；ETL 的 **E**。
- 下一份文件（資料爬蟲）接手「收集程式的工程化」：參數化 → Celery → 失敗處理 → upsert。

## 2. 做完這份，你會做到（可驗證動詞句）

1. 依「API ＞ 檔案下載 ＞ DB 匯出 ＞ 爬蟲」的優先序，替自己的題目列出 2–3 個候選來源並說出取捨理由。
2. 用五個維度（授權、更新頻率、欄位、規模、穩定性）替每個來源打燈號。
3. 徒手打一次 API，印出筆數、欄位、型別與日期範圍。
4. 寫一頁資料規格（來源、授權、筆數、欄位與型別、主鍵、更新頻率、已知坑）。
5. 寫一支會分頁、會重試、有限速的收集程式。
6. 把原始回應加時間戳存進 `data/raw/`，並確認它被 `.gitignore` 排除。

---

## 3. 核心觀念 A：來源優先序（由上往下選）

| 優先 | 來源形式 | 範例（repo 內出現過的） | 為什麼排這裡 | 出處 |
|---|---|---|---|---|
| 1 | 政府／機構開放資料（API 或 CSV） | data.gov.tw、內政部實價登錄、NYC TLC、CFPB、台北 ETC | 授權清楚、欄位穩定、不用帳號或只要 key | DD `scenarios/scenario-3,5,8`；MT `docs/data_sources.md` |
| 2 | 官方／商業 API | FinMind、Spotify Web API、NewsAPI、Google Maps Places | 有文件、有版本、有 rate limit 可預期；要管 key 與配額 | SC `課程手冊02`；DD `scenario-4,7,8` |
| 3 | 檔案下載（Kaggle、UCI、公開 CSV/parquet/zip） | Kaggle Restaurant Order、UCI Online Retail、KKBOX、Spotify Charts | 最快拿到大量資料；缺點是「死資料」，看不出 pipeline 價值 | DD `scenario-1,3,4`；DD `scenario-1` 雷區「用 Kaggle 資料就直接 demo」 |
| 4 | 資料庫匯出 | 企業 ERP MySQL dump、`pandas.read_sql()` | 學員專題少見；實務上最常見 | DD `02-pipeline-blueprint.md` Source 表 |
| 5 | 網站背後的 JSON API（非公開文件） | Hahow `api.hahow.in/api/products/search`、蝦皮公開 search endpoint | 比解析 HTML 穩，但沒有文件、隨時可能改 | HH `data_ingestion/hahow_crawler_course.py`；DD `scenario-2` |
| 6 | 爬 HTML／動態頁（最後手段） | PTT（requests+BeautifulSoup）、momo（Scrapy）、PChome／104（Playwright） | 成本最高：反爬、改版、法律灰色地帶 | `de-course/ep02/crawlers/ptt_crawler.py`；DD `scenario-2,6` |
| — | Streaming | Kafka、Pub/Sub | 進階，學期專題通常不碰 | DD `02-pipeline-blueprint.md` |

- 補強做法：Kaggle 歷史資料＋「模擬即時」資料（Faker 生門市 POS）混用，才看得出 pipeline 價值（DD `scenario-1-restaurant-pos.md` 資料源表＋雷區）。
- 「被動拉 vs 主動抓」：API 是被動拉、爬蟲是主動抓（DD `02-pipeline-blueprint.md` Source 表用語，可沿用）。

## 4. 核心觀念 B：評估一個來源的五個維度

| 維度 | 要回答的問題 | 去哪查 | 🔴 紅燈訊號 |
|---|---|---|---|
| 授權 | 能不能用？能不能公開展示？能不能商用？ | 資料集頁的 License（CC BY 4.0、CC0、政府資料開放授權條款）、網站 ToS、robots.txt | 沒寫授權、ToS 禁止自動化、要轉售（DD `scenario-6`「不要做成 SaaS 開賣」） |
| 更新頻率 | 多久有新資料？要排程嗎？ | API 文件、資料集更新紀錄 | 一次性靜態檔卻宣稱「即時」；RSS 部分 source 不更新（DD `scenario-7` 雷區） |
| 欄位 | 有沒有我要的欄位？型別、單位、編碼？ | 徒手抓一筆、看 `df.dtypes` | 日期是字串（SC `課程手冊06` 線上三步）、民國年（MT 題 6）、數字含逗號（MT 題 3）、「面議」薪資（DD `scenario-6`） |
| 規模 | 一次多少筆？每天多少？ | 試抓＋看 `_metadata.count` 之類的總數欄 | 單機吃不下又沒分區（DD `scenario-5` 雷區「BigQuery 沒分區」） |
| 穩定性 | 會不會改版、限流、要登入、token 過期？ | 試抓多次、看錯誤訊息 | 免費版限流（SC `課程手冊02`「FinMind 免費版有請求上限」）、OAuth token 1 小時過期（DD `scenario-4` 雷區）、反爬很硬（DD `scenario-6`） |

- 依規模與頻率挑儲存／排程的決策樹直接沿用 DD `02-pipeline-blueprint.md`「工具選型」思考點 1、2（<1 GB → MySQL/MongoDB；每天 → cron；每小時 → Airflow）。

## 5. 核心觀念 C：八情境資料來源對照（每情境一列）

| # | 情境 | 主來源（取得方式） | 補充來源 | 規模（來源所載，約略值） | 收集面的坑 | 出處 |
|---|---|---|---|---|---|---|
| 1 | 餐飲 POS | Kaggle Restaurant Order（下載 CSV） | UCI Online Retail（CSV）、Faker 模擬門市 POS | ~50 萬筆；模擬每門市每日 200–500 筆 | 只用 Kaggle 看不出 pipeline 價值 | DD `scenarios/scenario-1-restaurant-pos.md` |
| 2 | 電商比價 | momo（Scrapy + rotating UA）、PChome（Playwright） | 蝦皮公開 search API、自家 SKU 對照表（模擬） | 1000 SKU × 4 平台 | 反爬、robots.txt、`data/raw/` 不能 commit | DD `scenario-2-ecommerce.md` |
| 3 | 數位金融 | CFPB Complaint DB（開放 API） | Kaggle Bank Complaints（CSV）、PTT 銀行版（爬蟲） | 上百萬筆英文；PTT 每月 ~500 筆 | 個資去識別化 | DD `scenario-3-fintech.md` |
| 4 | 音樂串流 | Spotify Web API（OAuth + REST） | Spotify Charts（CSV）、Kaggle KKBOX | 數百萬筆聽歌紀錄 | OAuth refresh、rate limit（圖上標 30/sec，估計值） | DD `scenario-4-music.md` |
| 5 | 叫車交通 | NYC TLC（下載 parquet） | Uber Movement、台北 ETC（data.gov.tw API） | 月 ~1 GB／數百萬筆 | 檔案大，要分區 | DD `scenario-5-ride.md` |
| 6 | 求職媒合 | 104（Playwright） | CakeResume（Scrapy）、勞動部薪資統計（API/Excel） | 每天 2–5 萬職缺 | 反爬硬、灰色地帶、去重 | DD `scenario-6-recruit.md` |
| 7 | 新聞輿情 | NewsAPI（免費 100 req/day，來源所載未核實） | Google News RSS、PTT 八卦（爬蟲）、Threads 第三方 API | PTT 5000+/日 | 同則新聞多家轉載要去重；監控 source health | DD `scenario-7-news.md` |
| 8 | 房地產 | 內政部實價登錄（每季 zip／API） | data.gov.tw POI、Google Maps Places API（要 key） | 每季 ~10 萬筆 | 欄位多、民國年、API quota | DD `scenario-8-realestate.md` |

- 情境總表（Source／Storage／Process／Serve 一列一情境）在 DD `02-pipeline-blueprint.md`「怎麼套用到 8 情境」，可直接引用。
- 課程內建範例：股價＝FinMind API（SC），線上課程＝Hahow 網站 JSON API（HH）。

## 6. 核心觀念 D：資料規格（data spec）長什麼樣

最佳範本是 MT `docs/data_sources.md`（每題一張「項目／說明」表）與 MT `data/raw/topic_1/requirements_spec.md`（情境 → 使用者 → 資料來源表 → 欄位型別表 → MVP scope → 預期產出 → 成功指標）。建議教學文件給這張表當模板：

| 項目 | 填什麼 | 範例（MT 題 1） |
|---|---|---|
| 檔案／表名 | Raw 落地位置 | `data/raw/topic_1/orders.csv` |
| 筆數 | 實測筆數＋取樣方式 | 2,000 筆（從 106 萬筆取樣） |
| 來源 | 連結 | Kaggle: Online Retail II UCI |
| 授權 | 授權名稱 | CC BY 4.0 |
| 欄位 | 名稱（中文）＋型別 | `invoice_date`（str，需轉 datetime） |
| 唯一鍵 | 哪幾欄決定「同一筆」 | 股價＝`stock_id + date`（SC `課程手冊06`）；Hahow 課程＝`id`（HH `data_ingestion/mysql.py`） |
| 更新頻率／抓取時間 | 多久抓一次、哪個時區 | 週一到週五 18:00（SC 補充J Step 6 DAG） |
| 已知坑 | 型別、編碼、單位 | 民國年、含逗號數字（MT `docs/data_sources.md` 注意事項） |
| 用途 | ETL 用途／分析用途 | 銷售排行、客戶分析 |

> 一個小提醒：唯一鍵一定要在規格階段就決定。新聞資料沒有天然唯一鍵，「重跑時用什麼判斷重複」要自己定（SC 補充J 常見疑問「新聞資料也適用這條演進線嗎？」）。

## 7. 核心觀念 E：法律與倫理

| 議題 | 規則 | 出處 |
|---|---|---|
| robots.txt | 爬之前先看，遵守 Disallow | DD `scenario-2-ecommerce.md`「重要：合規 disclaimer」；DD `04-pain-points-industry-map.md` 情境 2 雷區 |
| 身分與速率 | 加 `User-Agent` 表明身份（可附 email）；每秒 < 1 request；「你不是要做 DDoS」 | DD `scenario-2`；DD `04-pain-points-industry-map.md` |
| 授權／商用 | 不商用沒授權的資料；學習、demo、履歷 OK，不要做成 SaaS 開賣 | DD `scenario-2`、`scenario-6`「重要：法律風險」 |
| 個資 | 不 publish 個人資料；客訴內容姓名／電話／卡號 mask；即使公開資料也去識別化 | DD `scenario-3`「重要：個資處理」、`scenario-6`、`scenario-8` |
| 可識別資訊 | 房地產不公開個人／物件可識別資訊；預測要標「模型估算」 | DD `scenario-8-realestate.md`「重要：法律與倫理」 |
| 爬到的內容 | `.gitignore` 加 `data/raw/`，不要 commit 爬到的內容 | DD `scenario-2` ⑤ 環境 |
| 金鑰 | `.env` 一定要 gitignore；程式只讀環境變數 | DD `scenario-3` ⑤；SC `crawler/config.py`、補充G |
| 本 repo 規則 | 不 commit 學員個資、真實客戶資料、帳密；範例用公開資料集或假資料 | `/home/user/iii_Study/CLAUDE.md` |

## 8. 核心觀念 F：Raw 永遠保留

- 「剛開始很容易只開一張 MySQL table，原始資料每次直接覆蓋掉 —— 結果分析跑錯時，連回去重算的機會都沒有。」→ **Raw 永遠保留**，只在 Silver／Gold 層清洗（DD `02-pipeline-blueprint.md` Storage 段）。
- 分層命名：Raw/Bronze → Cleaned/Silver → Curated/Gold；情境實例：`raw_complaints / classified`（DD scenario-3）、`raw_jobs / jobs_clean / skills_index`（scenario-6）、`raw scrape → product_match → price_intelligence`（scenario-2）。
- ELT 的理由就是「原始資料保留、轉換邏輯可隨時重跑」（DD `02-pipeline-blueprint.md` ETL vs ELT）。
- 程式面：整理層回傳新 DataFrame、不改輸入，「原始資料保持不動，要比對整理前後差在哪隨時可以做」（SC 補充J Step 1）；資料庫存 API 原始值，衍生欄位分析時再算（SC `課程手冊06` 線上段）。
- 歷史要另存：Hahow 課程主表 upsert、銷售快照表 insert 並帶 `captured_at`（HH `data_ingestion/mysql.py` `course_sales_table`）；DD scenario-2 雷區「沒紀錄歷史價格 → 沒辦法分析趨勢」。

---

## 9. 編號步驟（建議時間為估計值）

### Step 1：從業務問題倒推需要的資料（20 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 把「我想做 X」翻成「我需要哪幾欄、多長時間、多細顆粒度」 | ① 寫一句 persona＋痛點（DD `04-pain-points-industry-map.md`）② 列出要回答的 3 個問題 ③ 每個問題反推欄位與時間範圍 ④ 標出唯一鍵候選 | `docs/data_needs.md`：問題 × 欄位 × 時間範圍 × 顆粒度 |

### Step 2：盤點候選來源並排優先序（30 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 每個資料需求至少 2 個候選來源 | ① 先查 §5 情境表 ② 依 §3 優先序由上往下找 ③ 有 API 就停，不要先想爬蟲 ④ 找不到 API 時，開 DevTools → Network 看網站是否有 JSON 端點（Hahow 就是這種，HH `hahow_crawler_course.py`） | 候選來源表（來源／形式／連結） |

### Step 3：五維度評估＋徒手試抓（40 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 用證據而不是印象決定來源 | ① 每個來源照 §4 打 🟢🟡🔴 ② 不開 Docker、不寫 class，先用一段 inline Python 打一次（SC `課程手冊02`「熱身：先徒手打一次 FinMind API」）③ 記下筆數、欄位、型別、日期範圍、錯誤訊息 | 評估表＋試抓輸出截圖 |

```python
# 改寫自 SC 課程手冊02 熱身段：徒手試抓 + 快速評估
import requests, pandas as pd

URL = "https://api.finmindtrade.com/api/v4/data"
params = {"dataset": "TaiwanStockPrice", "data_id": "2330",
          "start_date": "2025-01-01", "end_date": "2025-06-17"}
resp = requests.get(URL, params=params, timeout=10)
data = resp.json()
if resp.status_code != 200:
    print("失敗：", data.get("msg"))          # 限流、參數錯都在 msg 裡
else:
    df = pd.DataFrame(data["data"])
    print("筆數：", len(df))
    print("欄位型別：\n", df.dtypes)            # date 是 object（字串）→ 規格要記
    print("日期範圍：", df["date"].min(), "~", df["date"].max())
    print("空值：\n", df.isna().sum())
```

### Step 4：寫資料規格（30 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 讓組員、業師、未來的你都看得懂資料長什麼樣 | 照 §6 模板逐欄填；授權與唯一鍵不能空；已知坑寫進去 | `docs/data_sources.md`（MT 同名檔為範本） |

### Step 5：寫可重跑的收集程式（60 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 同一支程式換參數就能抓別的範圍，失敗會重試、不會打爆對方 | ① 參數化（來源、日期、類別都從參數或環境變數來，SC `crawler/config.py`）② 分頁：先讀總數再算頁數（HH `hahow_crawler_course.py` 用 `_metadata.limit/count`）③ 重試：只對暫時性錯誤（429、5xx、timeout）④ 每頁之間 sleep ⑤ 失敗要印出來，不要靜默 | `src/collect.py` |

```python
# 分頁 + 重試 + 限速；分頁邏輯改寫自 HH data_ingestion/hahow_crawler_course.py
import math, time, requests
from requests.adapters import HTTPAdapter
from urllib3.util.retry import Retry

def make_session() -> requests.Session:
    s = requests.Session()
    retry = Retry(total=3, backoff_factor=1,              # 1s、2s、4s 退避
                  status_forcelist=[429, 500, 502, 503, 504])
    s.mount("https://", HTTPAdapter(max_retries=retry))
    s.headers["User-Agent"] = "course-project-bot (contact: you@example.com)"
    return s

def fetch_all(category: str, delay: float = 1.0) -> list[dict]:
    s = make_session()
    url = "https://api.hahow.in/api/products/search"
    first = s.get(url, params={"category": "COURSE", "groups": category}, timeout=10).json()
    pages = math.ceil(first["_metadata"]["count"] / first["_metadata"]["limit"])
    rows = []
    for p in range(1, pages + 1):
        r = s.get(url, params={"category": "COURSE", "groups": category, "page": p}, timeout=10)
        r.raise_for_status()                              # 重試用完仍失敗 → 直接報錯
        rows += r.json()["data"]["courseData"]["products"]
        time.sleep(delay)                                 # 原 repo 0.1s；DD 建議 < 1 req/s
    return rows
```

### Step 6：Raw 落地＋metadata（20 min）

| 目標 | 做法 | 產出 |
|---|---|---|
| 原始回應原封不動存下來，日後可重算、可追溯 | ① 路徑含來源與抓取日期 ② 檔名含時間戳，不覆蓋 ③ 存原始 JSON，不先轉 DataFrame ④ 旁邊寫一份 metadata（來源 URL、參數、筆數、抓取時間 UTC）⑤ `data/raw/` 加進 `.gitignore` | `data/raw/<source>/<YYYYMMDD>/*.json` ＋ `_meta.json` |

```python
# Raw 保存：時間戳檔名 + metadata，不覆蓋舊檔
import json
from datetime import datetime, timezone
from pathlib import Path

def save_raw(rows: list[dict], source: str, params: dict) -> Path:
    now = datetime.now(timezone.utc)                     # 一律存 UTC（HH 也用 timezone.utc）
    folder = Path("data/raw") / source / now.strftime("%Y%m%d")
    folder.mkdir(parents=True, exist_ok=True)
    stem = f"{source}_{now.strftime('%Y%m%dT%H%M%SZ')}"
    path = folder / f"{stem}.json"
    path.write_text(json.dumps(rows, ensure_ascii=False), encoding="utf-8")
    meta = {"source": source, "params": params, "count": len(rows),
            "fetched_at": now.isoformat(), "file": path.name}
    (folder / f"{stem}_meta.json").write_text(json.dumps(meta, ensure_ascii=False, indent=2))
    print(f"已寫入 {path}（{len(rows)} 筆）")
    return path
```

---

## 10. 卡關 → 處方

| 卡關 | 處方 | 出處 |
|---|---|---|
| 找不到 API，第一反應想寫爬蟲 | 先查開放資料平台與 Kaggle；再開 DevTools 看有沒有 JSON 端點 | DD `02-pipeline-blueprint.md` 選型原則 |
| API 回非 200、印出一行錯誤 | 暫時 `print(data)` 看原始 `msg` | SC `課程手冊02` 排錯表 |
| 回傳空 DataFrame | 代碼錯或該期間沒資料 → 用確定有資料的代碼（2330）測 | SC `課程手冊02` 排錯表 |
| 抓幾次就被擋（限流） | 減少數量、加 sleep、註冊拿 token（放 `.env`） | SC `課程手冊02` 排錯表 |
| token 一小時就過期、demo 翻車 | 實作 OAuth refresh | DD `scenario-4` 雷區 |
| 日期是字串，入庫變 text | 規格寫明型別，整理層轉 `date` | SC 補充J Step 1；SC `課程手冊06` |
| 民國年、含逗號數字 | 規格「已知坑」欄記下，清理時轉換 | MT `docs/data_sources.md` 注意事項 |
| CSV 中文亂碼 | 存檔用 `utf-8-sig` | SC `課程手冊05` 排錯表 |
| Raw 被覆蓋，算錯沒法重來 | 時間戳檔名 + Raw 層只增不改 | DD `02-pipeline-blueprint.md` Storage |
| 同則新聞／同職缺算多次 | 規格階段先定唯一鍵 | DD `scenario-6,7` 雷區 |
| 只有 Kaggle 死資料，看不出 pipeline 價值 | 混一份 Faker 模擬即時資料或接一個會更新的 API | DD `scenario-1` 雷區 |
| 不小心 commit 了爬到的內容或 key | `.gitignore` 加 `data/raw/`、`.env`；已推上去就換 key | DD `scenario-2` ⑤、`scenario-3` ⑤ |

## 11. 交付前檢查清單

- [ ] 每個資料需求都對應到一個來源，且能說出「為什麼不用爬蟲」或「為什麼只能爬」。
- [ ] 每個來源的授權已查證並寫進規格（CC BY、CC0、政府開放授權或 ToS 結論）。
- [ ] 要爬的網站已看過 robots.txt，程式有 User-Agent 與限速（建議 < 1 req/s）。
- [ ] 徒手試抓過一次，筆數、欄位、型別、日期範圍有實際輸出可貼。
- [ ] `docs/data_sources.md` 已完成，唯一鍵、更新頻率、已知坑都有填。
- [ ] 收集程式沒有寫死的清單或日期，換參數就能抓別的範圍。
- [ ] 重試只針對暫時性錯誤，且有上限；失敗會印出來。
- [ ] Raw 以時間戳檔名落地、從不覆蓋，旁邊有 metadata。
- [ ] `data/raw/`、`.env` 已在 `.gitignore`；repo 內沒有個資、帳密、真實客戶資料。
- [ ] 規格裡的數字都是實測值；未核實的標「約略值／估計值」。

---

## 12. 給撰寫者的備註

- 來源衝突：HH 程式每頁 `time.sleep(0.1)`（約 10 req/s），DD scenario-2 建議每秒 < 1 request、範例 commit 寫 2s sleep。教材應以 DD 為準，並把 HH 當「可以改進的地方」。
- FinMind 免費額度、NewsAPI 100 req/day、Spotify 30/sec 都只出自教材文字，repo 內未實測 → 一律標估計值或請學員查官方文件。
- FinMind token：SC `crawler/tasks_crawler_finmind_mongo.py` 以 `"token": ""` 參數示範，正式版應從環境變數讀。
- 本機預設值（只限教材示意）：MySQL `root / 1234`、RabbitMQ `worker / worker`（SC `crawler/config.py`、HH `data_ingestion/config.py`）。
