# Brief：數據分析與視覺化

> 用途：之後要寫成教學文件的素材。結構照「開頭一句 → 流程總覽 → 編號步驟（目標／做法表／範例／產出）→ 交付前檢查清單」排好。
> 來源代號：**[SC]** stock-crawler-de-course-materials｜**[HH]** hahow-crawler-de-course-materials｜**[DD]** DataDreamers-decrosscareer-student-project-handbook｜**[MT]** midterm-mvp-template
> 帳密只出現課程的**本機預設值**（root / 1234、metabase_ro / metabase），正式環境一律走 `.env`。

---

## 0. 開頭一句與心法（候選）

- 開頭一句：這份給 cleaned 表已經做好、但還不知道「要算什麼、怎麼讓人看懂」的轉職學員。讀完你會從一個業務問題出發，交出一張 Dashboard、一支 API，加上一個面試現場 demo 得出來的數字。
- 心法候選：
  - 「BI 工具就是資料的**出口**：讓不會寫 SQL 的人也能拉圖表、看趨勢、做決策。」（[SC] `課程手冊/課程手冊08 - Metabase BI視覺化.md`）
  - 「選型跟著『使用者怎麼用』走，不是跟著『我會什麼』走。」（[DD] `02-pipeline-blueprint.md` 5️⃣ Serve）
  - 「基礎完成＝raw ＋ cleaned ＋統計 CSV ＋**一個能 Demo 講的數字**。」（[MT] `README.md` Step 2）

## 1. 流程總覽（一行箭頭）

```
業務問題 → 指標定義 → EDA（pandas）→ SQL 分析（GROUP BY／視窗函數）→ VIEW／app 表 → 圖表與 Dashboard → API／報告 → 一個能 demo 的數字
```

- 本機版：`MySQL（mydb）→ VIEW → Metabase（唯讀帳號）`（[SC] 手冊08）
- 雲端版：`BigQuery stage → app（MA5/MA20、大盤摘要）→ Looker Studio 四區塊`（[SC] `課程手冊15 - BigQuery資料倉儲.md` Step 4–5）
- 期中版：`cleaned 表 → SQL 統計 → processed/*.csv → matplotlib → FastAPI /stats`（[MT] Section 3、8）

**做完這一章，你會做到**
1. 把一句業務問題拆成有公式、有粒度、有時間窗的指標。
2. 用 `describe`／`value_counts`／`groupby` 在 10 分鐘內摸清楚資料，找出長尾和異常。
3. 寫出 GROUP BY、HAVING，以及 `LAG`／移動平均（MA5、MA20）視窗函數。
4. 判斷什麼時候用 VIEW、什麼時候用實體表，並把結果包成 BI 讀的那一層。
5. 照「問題 → 圖」選對圖，用唯讀帳號接 Metabase，組出一面 Dashboard。
6. 用 FastAPI 開一支查詢端點，在 30 秒內講完你的數字。

---

## 2. 步驟

### Step 1：業務問題 → 指標（20 min）

- **目標**：每張圖都對應一個問題，每個數字都有定義。
- **做法：指標定義卡**（問題／指標／公式／粒度／來源）

| 業務問題 | 指標 | 公式（SQL） | 粒度 | 出處 |
|---|---|---|---|---|
| 哪些商品最暢銷？ | 營收 Top 20 | `SUM(total_amount)`，依 description 分組 | 商品 | [MT] topic_1 Step 3-1 |
| 哪些客戶最有價值？ | 客戶消費額 Top 10 | `SUM(total_amount)`，依 customer_id 分組 | 客戶 | [MT] topic_1 Step 3-4 |
| 各國市場表現？ | 客戶數、訂單數、營收、佔比 | `COUNT(DISTINCT customer_id)`、`SUM(...)/總和` | 國家 | [MT] topic_1 Step 3-2 |
| 叫車熱點在哪、幾點？ | 上車次數 | `COUNT(*)`，依 pickup_zone、hour 分組 | 地區×時段 | [MT] topic_4 Step 3-1/3-2 |
| 哪個縣市房價高？ | 每坪均價（建議看中位數） | `總價萬 / 面積坪` | 縣市 | [MT] topic_6 Step 3-1 |
| 課程每天賣幾份？ | 日增銷量、7 日平均營收 | `sold_num - LAG(sold_num)`；`AVG(revenue) OVER 7 列` | 課程×日 | [HH] `data_ingestion/hahow_bigquery_data_transform.py` |
| 股價趨勢轉強了嗎？ | MA5 對 MA20（黃金交叉） | `AVG(close) OVER (... 4／19 PRECEDING)` | 股票×交易日 | [SC] 手冊15 ③ |

- 寫指標時要確認四件事：分子分母各是什麼？時間窗多長？要不要去重？累計值還是增量？（[HH] 的 `sold_num` 是累計數，直接加總會重複計算）
- 成功指標要寫成講得出口的句子：「能說出銷售額最高的 3 個品類和 3 個國家」「庫存建議有數據支撐（不是空話）」（[MT] `data/raw/topic_1/requirements_spec.md`）。
- **產出**：3–5 張指標定義卡，放在 `docs/` 或 README。

### Step 2：EDA，先看資料長什麼樣（20 min）

- **目標**：在畫圖之前，先知道分佈、極端值和缺口。
- **做法**

| 招式 | 回答什麼 | 出處 |
|---|---|---|
| `df.describe()` | 平均和中位數差很多嗎？→ 長尾；min ≤ 0？→ 異常 | [MT] Step 1-2 |
| `df["col"].value_counts().head(10)` | 類別分佈；前幾名佔多少 | [MT] Step 1-3 |
| `df["col"].nunique()` | 有幾種商品、客戶、股票 | [MT] Step 1-3 |
| `df.groupby("k")["v"].agg([...])` | 分組比較 | [MT] Step 1-3 |
| `df.sample(5)` | 隨機看幾筆原貌 | [MT] Step 1-3 |
| `df["v"].plot.hist(bins=30)`／箱型圖 | 分佈形狀、離群值 | （補充，Section 9 的分佈互動） |
| `isna().sum()` | 缺值集中在哪幾欄 | [SC] `example/pandas.ipynb` |

- **範例 A：pandas groupby ＋畫圖**（改寫自 [MT] topic_1 Step 3-2/3-3）

```python
import pandas as pd, matplotlib.pyplot as plt
df = pd.read_sql("SELECT * FROM cleaned_orders", conn, parse_dates=["invoice_date"])  # SQLite 讀回日期是字串
print(df["total_amount"].describe())             # mean 遠大於 50% → 少數大單拉高平均
top = (df.groupby("country")
         .agg(customers=("customer_id", "nunique"), orders=("invoice_id", "nunique"),
              revenue=("total_amount", "sum"))
         .sort_values("revenue", ascending=False).head(10))
top["share"] = (top["revenue"] / df["total_amount"].sum()).round(3)   # 佔比，demo 用
ax = top["revenue"].plot.barh(figsize=(8, 4), title="各國營收 Top 10")
ax.invert_yaxis(); plt.tight_layout(); plt.savefig("processed/country_revenue.png")
top.to_csv("processed/country_stats.csv")
```

- **產出**：`processed/*.csv`、1–2 張圖，以及 3 條觀察（寫「我看到 ＿＿，所以想查 ＿＿」）。

### Step 3：SQL 分析：GROUP BY 到視窗函數（30 min）

- **目標**：把分析邏輯放進 SQL，讓 BI 和 API 共用同一份。
- **做法**

| 需求 | 寫法 | 重點 | 出處 |
|---|---|---|---|
| 分組彙總 | `GROUP BY ... ORDER BY ... LIMIT` | 每組壓成一列 | [MT] Step 3-1 |
| 篩選分組結果 | `HAVING COUNT(*) > 1` | WHERE 篩列，HAVING 篩組 | [SC] 手冊06 練習 2 |
| 不重複計數 | `COUNT(DISTINCT customer_id)` | BigQuery 的 Materialized View 不支援 | [MT] Step 3-2；[SC] 手冊15 |
| 前一筆 | `LAG(close) OVER (PARTITION BY stock_id ORDER BY trade_date)` | 第一列是 NULL | [SC] 手冊15 ③ |
| 移動平均 | `AVG(close) OVER (... ROWS BETWEEN 4 PRECEDING AND CURRENT ROW)` | 前 4 列是熱身期，驗證時用 `WHERE ma20 IS NOT NULL` 濾掉 | [SC] 手冊15 ③ |
| 每組取一筆（去重） | `ROW_NUMBER() OVER (PARTITION BY k ORDER BY ...) = 1` | 先去重再算指標，不然 MA 會算錯 | [SC] `example/vw_stock_price_daily.sql` |
| 累計轉增量 | `sold_num - LAG(sold_num) OVER (...)` | 快照資料的必備招 | [HH] transform |

- 視窗函數和 GROUP BY 的差別：`PARTITION BY` 會分組但**不壓縮列數**，每一列都看得到自己組內的鄰居（[SC] 手冊15）。
- **範例 B：視窗函數（MA5／MA20）**（[SC] 手冊15 Step 4；BigQuery 語法）

```sql
CREATE OR REPLACE TABLE app.stock_trend_analysis AS
SELECT stock_id, trade_date, close, volume,
  LAG(close) OVER (PARTITION BY stock_id ORDER BY trade_date)                                        AS prev_close,
  AVG(close) OVER (PARTITION BY stock_id ORDER BY trade_date ROWS BETWEEN 4  PRECEDING AND CURRENT ROW) AS ma5,
  AVG(close) OVER (PARTITION BY stock_id ORDER BY trade_date ROWS BETWEEN 19 PRECEDING AND CURRENT ROW) AS ma20
FROM stage.stock_price_daily;          -- 來源是去重後的 stage，不是 raw

-- 驗證：每支股票最近 3 天，濾掉熱身期
SELECT * FROM app.stock_trend_analysis WHERE ma20 IS NOT NULL
QUALIFY ROW_NUMBER() OVER (PARTITION BY stock_id ORDER BY trade_date DESC) <= 3;
```

- 練習鉤子：自己加一條 `ma10`（`9 PRECEDING`）；寫一個只掃 2024-06 分區的 WHERE（[SC] 手冊15 練習 2、3）。
- **產出**：2–3 段存檔的 SQL（`sql/*.sql`），每段開頭有一行註解寫它回答哪個問題。

### Step 4：VIEW 還是實體表？做出 BI 讀的那一層（15 min）

- **目標**：BI 和 API 只讀乾淨、穩定的那一層。
- **做法**

| | VIEW | 實體表（CTAS） | Materialized View（BigQuery） |
|---|---|---|---|
| 本質 | 存了名字的 SELECT，不存資料 | 結果實際存下來 | 存結果，來源變動時背景增量更新 |
| 新鮮度 | 即時 | 要排程重算（`CREATE OR REPLACE`，冪等） | 自動 |
| 速度、成本 | 每次查都重算 | 快，省掃描量 | 快；要算查詢＋維護＋儲存三筆費用 |
| 適合 | stage：邏輯常改（去重、改名） | app：報表每次開都會查 | 彙總固定、想省排程；有語法限制 |
| 出處 | [SC] `補充C - MySQL深入_主鍵索引外鍵與分區.md` §4；手冊15 Step 3 | 手冊15 Step 4；[HH] `mysql.py` `create_table_from_view` | `補充K - BigQuery深入_亮點實驗與Studio功能.md` 第三部分 |

- **範例 C：建 VIEW ＋唯讀帳號**（[SC] `example/vw_stock_price_daily.sql`、手冊08 Step 3.5）

```sql
CREATE OR REPLACE VIEW vw_stock_price_daily AS
SELECT stock_id, date AS trade_date, open, max, min, close, spread, Trading_Volume
FROM (SELECT s.*, ROW_NUMBER() OVER (PARTITION BY s.stock_id, s.date ORDER BY s.Trading_Volume DESC) AS rn
      FROM TaiwanStockPrice s) t
WHERE t.rn = 1;                                   -- 不動原始資料，在查詢層清理

CREATE USER 'metabase_ro'@'%' IDENTIFIED BY 'metabase';   -- 本機預設值；_ro = read-only
GRANT SELECT ON mydb.* TO 'metabase_ro'@'%';               -- 只給查詢；DELETE 會被拒絕
-- 實體表版（MySQL，[HH]）：DROP TABLE IF EXISTS t; CREATE TABLE t AS SELECT * FROM vw_...;
```

- **產出**：一個 VIEW（stage 或去重用）、一張 app 實體表、一個唯讀帳號。

### Step 5：選圖，組出 Dashboard（40 min）

- **目標**：一面看得懂的 Dashboard，每張圖回答一個問題。
- **做法 5-1：問題 → 圖**

| 你想回答的問題 | 用這種圖 | repo 例子 | 避免 |
|---|---|---|---|
| 隨時間怎麼變？ | 折線、時間序列 | 台積電收盤價走勢；收盤＋MA5＋MA20 疊線 | 多支股票混在一起算 MA |
| 誰排第一？ | 橫條（由大到小排） | 商品營收 Top 10；各股成交量比較 | 類別名稱很長卻用直條 |
| 現在是多少？ | 數字卡、計分卡 | 最新收盤價；大盤總成交量 | 平均值被預設 SUM 加總 |
| 分佈長什麼樣？ | 直方圖、箱型圖 | 「選縣市 → 看總價萬分佈」（[MT] topic_6 Section 9） | 只報平均，不看中位數 |
| 組成和佔比？ | 堆疊長條；類別 ≤ 5 才用圓餅 | 各國銷售佔比 | 十幾類的圓餅 |
| 兩個變數有關嗎？ | 散佈圖 | 行程距離對金額（[MT] topic_4） | 把相關講成因果 |
| 時段×類別哪裡最熱？ | 熱力圖 | 上車地區 × 小時（[MT] topic_4） | 顏色沒有圖例 |
| 量和價一起看 | 折線＋長條 | Looker：收盤走勢＋成交量長條 | 雙軸刻度誤導 |
| 要查明細 | 表格卡 | 每日行情明細 | 把明細表當主圖 |
| 在哪裡？ | 地圖 | 房地產 Streamlit 地圖（[DD] 02 情境 8） | 沒有地理欄位硬畫 |

- **做法 5-2：Dashboard 藍圖，先規劃再動手**（[SC] 手冊08 Step 4.5 的五件事）：每個 Question 是一張圖 → 每張圖呈現什麼 → 要哪幾張 → 怎麼排版 → 哪些 Filter 套在哪些圖。排版建議：最上排放數字卡，中間放全寬走勢，下面放比較圖和明細。
- **做法 5-3：兩套 BI 怎麼選**

| | Metabase（本機，手冊08） | Looker Studio（雲端，手冊15 Step 5） |
|---|---|---|
| 部署 | 自己跑一個容器（約 1 GB 記憶體，估計值） | SaaS，開瀏覽器就能用 |
| 資料來源 | MySQL `mydb`，用 `metabase_ro` **唯讀**帳號 | BigQuery **app 層**（BI 只讀 app） |
| 設定存在哪 | 設定庫 `metabasedb`（`metabase_app` 讀寫）；容器無狀態 | Google 帳號 |
| 招牌功能 | Question 點選＋SQL 模式＋`{{變數}}`、X-ray、自動刷新、Collections | 四區塊：2 張計分卡（總成交量、平均收盤）＋MA 疊線＋雙股走勢與成交量長條 |
| 常見坑 | Host 要填服務名 `mysql`，不能填 127.0.0.1；新 VIEW 要「同步 schema」 | avg_close 預設 SUM；週末缺值畫成 0 → 改「線性插值」 |

- 權限有兩層：MySQL 的 `metabase_ro` 管 Metabase 整體拿得到什麼；Metabase 的 Permissions 管哪個使用者看得到什麼（手冊08 功能地圖）。
- **產出**：Dashboard 截圖（含 Filter）、藍圖草圖、BI 連線用的唯讀帳號。

### Step 6：Serve ＋說故事（30 min）

- **目標**：讓別人用得到結果，也讓面試官記住一個數字。
- **做法 6-1：Serve 怎麼選**（[DD] 02 5️⃣ Serve；思考點 3「誰在用」）

| 誰在用 | 形式 | 工具 | repo 例子 |
|---|---|---|---|
| 非技術同事 | Dashboard | Metabase、Looker Studio、Streamlit | 手冊08；手冊15 Step 5；[SC] `example/stock_dashboard.py` |
| 其他系統、前端 | API | FastAPI | [SC] `api/main.py`；[MT] `data/raw/topic_*/api.py` |
| 主管、一次性分析 | 報告 | Notebook ＋ `output/pipeline_doc.md` | [MT] Section 6 |
| 需要即時知道 | 推播 | Email、Slack、LINE；Metabase 訂閱需先設 SMTP | [DD] 02；手冊08 |

- 架構提醒：Streamlit 直連 MySQL「能動」，但等於把 DB 帳密放進前端。走 `MySQL ← API（守門員）← Streamlit` 才是完整分層（[SC] `補充E - MySQL to FastAPI.md` 附錄二）。
- **範例 D：最小 FastAPI 端點**（[SC] `api/main.py` 精簡版）

```python
import os, pandas as pd
from fastapi import FastAPI, HTTPException, Query
from sqlalchemy import create_engine, text
app = FastAPI(title="Stock Price API")
engine = create_engine(os.environ["DB_URL"])          # 模組層級共用連線池；帳密放 .env

@app.get("/stocks/{stock_id}/prices")                  # 路徑放「身分」，查詢字串放「過濾」
def get_prices(stock_id: str, limit: int = Query(30, ge=1, le=1000)):   # 超出範圍自動回 422
    sql = text("SELECT date, close, spread FROM TaiwanStockPrice "
               "WHERE stock_id = :sid ORDER BY date DESC LIMIT :n")       # 參數化，絕不用 f-string 拼
    df = pd.read_sql(sql, engine, params={"sid": stock_id, "n": limit})
    if df.empty:
        raise HTTPException(status_code=404, detail=f"找不到 {stock_id} 的資料")
    return df.to_dict(orient="records")                # SQL → DataFrame → JSON
# 啟動：uv run uvicorn api.main:app --port 8000 → 打開 /docs 看自動文件
```

- 上線前還缺的（概念版，補充E）：CORS 白名單（不要寫 `["*"]`）、cursor 分頁、`/v1` 版本前綴、API key、rate limit（429）、gunicorn 多 worker。
- **做法 6-2：說故事／面試講法**

| 段落 | 講什麼 | 出處 |
|---|---|---|
| 3 分鐘 Demo 結構 | 問題 → 清洗 → LLM → output → 升級方向 | [MT] `_instructor/speaker_notes.md` |
| 用 Medallion 講 | Bronze 存了什麼、Silver 清了什麼、Gold 分析了什麼 | 同上 |
| 一個數字 | 現場打得出來（API／Dashboard 即時查），不是只放在投影片上 | [MT] README「一個能 Demo 講的數字」 |
| 痛點有數字 | X 分鐘／X 元／X% | [DD] `07-demo-day-rubric.md` 問題定義 |
| 誠實 | 「最後悔的 1 個決定」不要說都很好；取樣資料要講是取樣 | [DD] 07 常見追問 4 |

- 一句話範本：「我拿 **2,000 筆** raw 交易，清掉 **X 筆（Y%）** 缺值和負數，發現 **前 3 國貢獻 Z% 營收**。打 `/stats` 就看得到，所以我建議 ＿＿。」（數字現場查；2,000 筆是 [MT] 的取樣量，不能講成全量）
- ❌／✅ 對照：❌「我做了很多分析和圖表」　✅「MA5 在 ＿月＿日上穿 MA20，Dashboard 篩 2330 就看得到，這是我設計 app 層的理由」（日期現場查，不要背）
- **產出**：Dashboard 連結或截圖、API `/docs` 截圖、30 秒講稿（含一個數字）。

---

## 3. 卡關排錯：你遇到的狀況／原因／怎麼解

| 你遇到的狀況 | 原因 | 怎麼解 |
|---|---|---|
| MA 數字怪怪的 | raw 有重複列，「往前推 4 列」推到同一天兩次 | 先去重（stage／VIEW），再算指標（[SC] 手冊15） |
| 前幾天的 MA20 不準 | 熱身期，不滿 20 列也照樣平均 | `WHERE ma20 IS NOT NULL`，或標成熱身期 |
| 計分卡的平均收盤大到不合理 | Looker 預設匯總是 SUM | 匯總改「平均」（手冊15 Step 5 排錯） |
| 走勢線呈鋸齒狀、掉到 0 | 週末沒資料，缺值被畫成 0 | 「缺少資料」改「線性插值」 |
| Metabase 連不到 MySQL | Host 填了 127.0.0.1，或兩個容器不在同一個網路 | Host 填 `mysql`；`docker network connect my_network mysql` |
| 新 VIEW 在 Metabase 找不到 | schema 沒同步 | 管理員 → 資料庫 → 同步資料庫 schema |
| 重啟後 Dashboard 不見了 | `metabasedb` 被 `down -v` 清掉，或連到別台 MySQL | 別對 MySQL 的 compose 下 `down -v`；確認 `MB_DB_*` |
| `df[df.stock_id == "2330"]` 查到 0 筆 | 表把 stock_id 存成整數 | `astype(str)`，或回頭修 schema |
| groupby 依月份排序亂掉 | SQLite 讀回的日期是字串 | `read_sql(..., parse_dates=[...])` |
| 平均值很漂亮，但老闆不信 | 長尾分佈，平均被少數大單拉高 | 同時報中位數，加一張分佈圖 |
| 累計銷量越加越大 | 把快照的累計值拿來加總 | 用 `LAG` 轉成日增量（[HH]） |
| API 被注入或查詢被改寫 | 用 f-string 把輸入拼進 SQL | 用 `:param` 參數化（補充E ④） |
| BigQuery 查詢很貴 | `SELECT *`、沒帶分區條件 | 只選需要的欄、加日期條件；先 `--dry_run` |
| 公開連結洩漏資料 | Metabase 開了公開分享 | 正式環境先評估再開（手冊08 Step 8） |

## 4. 交付前檢查清單（☐）

- ☐ 3–5 張指標定義卡：問題／公式／粒度／時間窗／累計還是增量
- ☐ EDA 有 `describe` 和至少 1 張分佈或排行圖；寫下 3 條觀察
- ☐ 至少 1 段 GROUP BY 和 1 段視窗函數（LAG 或 MA），SQL 存成檔案
- ☐ 指標是從去重後的那層（stage／VIEW）算的，不是直接用 raw
- ☐ VIEW 和實體表的選擇講得出理由（新鮮度 vs 速度）
- ☐ BI 用**唯讀帳號**連；只讀 app 層或 VIEW
- ☐ Dashboard 有藍圖：數字卡＋走勢＋比較＋明細，至少 1 個 Filter
- ☐ 每張圖都是照「問題 → 圖」表選的；類別超過 5 個不用圓餅
- ☐ FastAPI：有 `/health`、1 支參數化查詢端點；查無資料回 404；`/docs` 打得開
- ☐ 前端不直連資料庫；repo 沒有 hardcoded 密碼
- ☐ 30 秒講稿裡有**一個現場查得到的數字**，取樣、估計值都有標出來
- ☐ 能回答：「資料量 ×100 你怎麼改？」（MySQL → BigQuery、加分區、CTAS 排程）

## 5. 給撰寫者的注意（來源核對時發現）

- [MT] topic_1 名稱不一致：`requirements_spec.md` 標題寫「餐飲連鎖 POS 庫存優化」，README、Notebook 寫「零售 POS 銷售分析」。報告檔也有 `output/report.md`（`docs/pipeline.mmd`）和 `output/pipeline_doc.md`（Notebook Section 6、spec）兩種寫法。成品文件以 `pipeline_doc.md` 為準，並在文件裡註明。
- [HH] `metabase/docker-compose-metabase.yml` 用 `metabase/metabase:latest`，沒有鎖版本。手冊08 的按鈕名稱以 Metabase v0.62 為準。教學文件要寫「實測版本」，並提醒介面可能微調。
- 「Metabase 容器約 1 GB 記憶體」出自 [SC] 手冊15 的對照表，屬估計值。BigQuery dry run 的數字依資料量而變，只能引用比例關係。
- Looker Studio 已更名「數據分析」，兩個名字介面上都會看到（手冊15）。
