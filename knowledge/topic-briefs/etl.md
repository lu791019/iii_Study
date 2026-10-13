# Brief：資料清理與儲存（ETL）

> 用途：之後要寫成教學文件的素材。結構照「開頭一句 → 流程總覽 → 編號步驟（目標／做法表／範例／產出）→ 交付前檢查清單」排好。
> 來源代號：**[SC]** stock-crawler-de-course-materials｜**[HH]** hahow-crawler-de-course-materials｜**[DD]** DataDreamers-decrosscareer-student-project-handbook｜**[MT]** midterm-mvp-template
> 帳密只出現課程的**本機預設值**（root / 1234、metabase_ro / metabase），正式環境一律走 `.env`（[SC] 補充G）。

---

## 0. 開頭一句與心法（候選）

- 開頭一句：這份給已經拿得到資料、但還沒辦法「放心重跑、放心查」的轉職學員。讀完你會做出 raw → cleaned → analyzed 三層，重跑三次筆數都不變。
- 心法候選：
  - 「**Raw 永遠保留**，只在 Silver / Gold 層做清洗。多一份備份，安心很多。」（[DD] `02-pipeline-blueprint.md` 3️⃣ Storage）
  - 「**表的設計和寫入的方式，永遠成對出現。**」（[SC] `課程手冊/課程手冊06 - 去重與冪等.md` 反向驗證）
  - 類比收束：冪等就像電梯的「5 樓」按鈕，按五次還是去 5 樓（[SC] 手冊06）。

## 1. 流程總覽（一行箭頭）

```
來源(CSV/API) → raw（原樣落地，只 append）→ cleaned（型別/缺值/重複/單位/日期）→ analyzed（指標/彙總）→ 品質測試 → 交給分析與 Serve
```

- 期中版：`CSV → pandas → SQLite (raw/cleaned/analyzed) → SQL → LLM → FastAPI`（[MT] `README.md`、`docs/pipeline.mmd`）
- 雲端版：`爬蟲 → MySQL（OLTP）＋ BigQuery raw → stage → app → Looker Studio`（[SC] `課程手冊15 - BigQuery資料倉儲.md`）

**做完這一章，你會做到**（可驗證動詞句）
1. 把原始資料原樣寫進 raw 表，並說得出為什麼 raw 不能覆蓋。
2. 用 pandas 跑完清理 checklist，印出「raw N 筆 → cleaned M 筆（-X%）」。
3. 依資料量和查詢方式選 SQLite / MySQL / BigQuery / MongoDB，並講得出理由。
4. 幫表設主鍵（必要時用複合主鍵）和索引，用 `EXPLAIN` 證明有走索引。
5. 用 upsert 寫入，重跑三次筆數不變。
6. 寫 4 條以上的資料品質斷言，壞資料會讓 pipeline 停下來。

## 2. 名詞對照：三層＝Medallion

| 本課叫法（[MT]） | Medallion | 倉儲叫法（[SC] 手冊15） | [DD] 02 Storage | 職責 | 規矩 |
|---|---|---|---|---|---|
| `raw_*` | Bronze | raw | 原始層 Raw | 原樣落地，一個欄位都不改 | 只寫入、不修改；它是「發生過什麼」的證據 |
| `cleaned_*` | Silver | stage | 清洗層 Cleaned | 去重、改欄名、轉型別、補／刪缺值 | 從 raw 算出來，隨時可重建 |
| `analyzed_*` | Gold | app | 分析層 Curated | 給人和 BI 用的成品（指標、彙總、LLM 標籤） | 從 Silver 算出來，BI 只讀這層 |

- 排錯時照層往下追：報表怪 → 查 app；app 沒錯 → 查 stage；stage 沒錯 → 回 raw（[SC] 手冊15「倉儲的三層習慣」）。
- Demo 時用 Medallion 術語講：Bronze 存了什麼、Silver 清了什麼、Gold 分析了什麼（[MT] `_instructor/speaker_notes.md` Demo 段）。

## 3. ETL vs ELT

| | ETL（傳統） | ELT（雲端主流） |
|---|---|---|
| 順序 | E → T → L：先轉換，再進倉儲 | E → L → T：先全部進倉儲，再用倉儲的算力轉換 |
| T 在哪裡做 | pandas／Python（倉儲外） | SQL（BigQuery、Snowflake、dbt） |
| 原始資料 | 常在轉換時就丟掉 | 保留在 raw，算錯了重跑就好 |
| 適合 | 嚴格 schema、小資料、單機（期中 SQLite） | 雲端倉儲、資料量大、邏輯常改 |
| 本課對應 | [MT] Section 1–3（pandas 清洗後寫 cleaned） | [SC] 手冊15：雙寫 append 進 raw，stage／app 用 SQL |

- 出處：[DD] `02-pipeline-blueprint.md`「ETL vs ELT」；[SC] 手冊15 Q3。
- 架構視角（Storage ⇄ Process）跟行為視角（ETL／ELT）是同一條 pipeline 的兩種描述（[DD] 02「兩種視角」）。

---

## 4. 步驟

### Step 1：先定義「一筆資料是什麼」（15 min）

- **目標**：寫下資料粒度（grain）和業務主鍵。後面的去重、主鍵、upsert 全靠它。
- **做法**

| 動作 | 怎麼做 | 例子 |
|---|---|---|
| 寫粒度 | 「一列代表＿＿」 | 一列＝某股某日的日線（[SC]）；一列＝某課程某次抓取的快照（[HH] `hahow_course_sales`） |
| 選業務主鍵 | 哪幾欄能唯一辨識一列 | `(stock_id, date)`；簽到用 `(user_id, date)`；訂單明細用 `(order_id, product_id)`（[SC] 手冊06「主鍵怎麼選」） |
| 分主檔和歷史 | 會被更新的放主檔（upsert），要留軌跡的放快照（append＋時間戳） | [HH] `data_ingestion/mysql.py`：`hahow_course` 以 `id` 為 PK 做 upsert；`hahow_course_sales` 用自增 id，只 insert，帶 `captured_at` |
| 盤點欄位 | `dtypes`／`isnull().sum()`／`describe()` 三件事 | [MT] Notebook Step 1-2 |

- **產出**：一段粒度宣告，加上 `docs/` 裡的 Data Dictionary 草稿（[MT] `docs/README_TEMPLATE.md` §3）。

### Step 2：Extract → raw 層（Bronze）（20 min）

- **目標**：原始資料原樣落地，可以重算。
- **做法**

| 規矩 | 為什麼 | 出處 |
|---|---|---|
| 不改欄位、不清洗 | 分析算錯時，還有機會回去重算 | [DD] 02 Storage 小提醒 |
| raw 只 append，不 replace | `if_exists="replace"` 會刪表重建，舊資料全沒 | [SC] 手冊05 to_sql 參數表 |
| 加上 `ingested_at`／`uploaded_at` | 追得到哪一批、什麼時候進來 | [HH] `hahow_crawler_course_optimized_sales.py` `df['uploaded_at']` |
| 雲端用 load job，不用 streaming | 批次 load 免費（只算儲存），streaming 依寫入量計費 | [SC] 手冊15 ① |
| 日期分區 | 查一個月就只掃那個月 | [SC] 手冊15 ②、[HH] `hahow_sync_mysql_to_bigquery.py` `partition_key` |

- **範例**：期中版是 `df_raw.to_sql("raw_orders", conn, if_exists="replace")`，教學上可以接受，因為來源 CSV 一直都在。正式排程要改成 append。這點要在文件裡講清楚。
- **產出**：`raw_*` 表；筆數跟來源一致（[MT] 預期「raw_orders: 2000 筆」）。

### Step 3：Transform → cleaned 層（Silver）（30 min）

- **目標**：一份型別正確、沒有重複、單位一致的表。
- **做法：清理 checklist**

| 項目 | 檢查 | 處方 | 出處 |
|---|---|---|---|
| 型別 | `df.dtypes` | `pd.to_numeric(errors="coerce")`；ID 一律轉字串（`0050` 不能變 `50`） | [SC] 手冊06「型別注意」；[MT] topic_6 Step 2-3 |
| 缺值 | `isnull().sum()` | 關鍵欄 `dropna(subset=...)`；非關鍵欄 `fillna`。爬蟲端就先做防呆 | [MT] Step 2-2；[SC] 手冊06「空值處理的原則」 |
| 重複 | 依業務主鍵 `duplicated(subset=...)` | `drop_duplicates(subset=[...], keep="last")`，去重前先統一型別 | [SC] 手冊06「把去重搬到 DataFrame 層」 |
| 異常值 | `describe()` 看 min／max | 數量、單價、距離 ≤ 0 的過濾掉；哨兵值（如 `price < 999999`）排除 | [MT] Step 2-4；[HH] `hahow_bigquery_data_transform.py` |
| 單位 | 欄名帶單位 | 元 → 萬（÷10000）；平方公尺 → 坪（÷3.30579）；新欄位名寫明單位 | [MT] topic_6 Step 2-3 |
| 編碼 | 中文亂碼 | 讀寫 CSV 用 `encoding="utf-8-sig"`（Excel 開才不亂）；政府資料常見 Big5／cp950（常見情況，依實際檔案確認） | [SC] 手冊05 排錯表 |
| 日期 | 字串？時區？民國年？ | `pd.to_datetime`；ISO 字串去掉 `T`／`Z`；民國年 +1911；統一用 UTC 存 | [HH] `hahow_crawler_common.py` `to_datetime`；[MT] topic_6 |
| 寫 DB 前 | NaN／NaT 寫不進去 | `df.replace({pd.NaT: None, np.nan: None})` | [HH] `hahow_crawler_course_optimized_sales.py` |
| 累計值 | 欄位是累積數嗎？ | 不要直接加總；到分析層用 `LAG` 轉成日增量 | [HH] `sold_num - LAG(sold_num)` |

- **範例 A：pandas 清理**（改寫自 [MT] topic_1／topic_6 Section 2）

```python
import pandas as pd, numpy as np
df = pd.read_sql("SELECT * FROM raw_orders", conn)            # 從 raw 讀，不從 CSV 讀
before = len(df)
df = df.dropna(subset=["description", "customer_id"])          # 缺值：關鍵欄不能空
df["customer_id"] = df["customer_id"].astype("int64").astype(str)  # 12345.0 → "12345"
df["invoice_date"] = pd.to_datetime(df["invoice_date"], errors="coerce")
df["quantity"] = pd.to_numeric(df["quantity"], errors="coerce")
df = df[(df["quantity"] > 0) & (df["unit_price"] > 0)]         # 異常值
df["total_amount"] = df["quantity"] * df["unit_price"]
df = df.drop_duplicates(subset=["invoice_id", "stock_code"], keep="last")
# 民國年（topic_6）：1130101 → 2024-01-01
# s = df["交易年月日"].astype(str).str.zfill(7)
# df["交易日"] = pd.to_datetime((s.str[:3].astype(int) + 1911).astype(str) + s.str[3:], format="%Y%m%d", errors="coerce")
print(f"raw {before} → cleaned {len(df)}（-{before - len(df)}，{(before - len(df)) / before:.1%}）")
```

- **產出**：`cleaned_*` 表（筆數 ≤ raw），以及 Transform Logic 表：步驟／程式碼／理由／影響筆數（[MT] `README_TEMPLATE.md` §4）。

### Step 4：選儲存＋設計 schema（30 min）

- **目標**：選對落地位置，讓資料庫本身擋掉重複、加快查詢。
- **做法 4-1：選儲存**

| 階段 | 選 | 什麼時候 | 出處 |
|---|---|---|---|
| 期中、單機、2,000 筆 | CSV ＋ SQLite（`pipeline.db` 一個檔案） | 一個人跑、不用架服務 | [MT] |
| 欄位固定、要多人或多 worker 寫入、要 JOIN | MySQL（OLTP） | 每天資料 < 1 GB；需要 ACID、主鍵、upsert | [SC] 手冊05；[DD] 02 思考點 1 |
| 掃全歷史、做聚合、給 BI 用 | BigQuery（OLAP） | 1–100 GB 以上；按掃描量計費，欄式儲存 | [SC] 手冊15；補充K |
| 欄位常變、半結構化（爬蟲 dict、log） | MongoDB | 免 schema、水平擴展；髒資料要靠程式自己檢查 | [SC] `補充D - ACID與CAP_從MySQL到MongoDB.md` §6 |
| 交易、金流、庫存 | MySQL | 錢的事沒有「最終一致」 | [SC] 補充D §6 |

- 一句話對比：MySQL 像收銀機，BigQuery 像財報分析室；兩邊是分工，不是誰取代誰（[SC] 手冊15）。
- **做法 4-2：schema 設計**

| 工具 | 管什麼 | 擋重複？ | 加速？ | 口訣／提醒 | 出處 |
|---|---|:-:|:-:|---|---|
| PRIMARY KEY | 每列的身分證 | ✅ | ✅ | 一表一個；複合主鍵 `(stock_id, date)` | [SC] `補充C - MySQL深入_主鍵索引外鍵與分區.md` §2 |
| UNIQUE | 這欄不准重複 | ✅ | ✅ | 混搭：代理鍵當 PK，自然鍵另設 UNIQUE | 補充C §2 |
| INDEX | 只加速 | ❌ | ✅ | 照 WHERE 建；最左前綴：像電話簿（姓、名），只知道名字查不了 | 補充C §1 |
| NOT NULL／DEFAULT | 入場檢查 | ❌ | ❌ | `date NOT NULL`；`created_at DEFAULT CURRENT_TIMESTAMP` | 補充C §2 |
| FOREIGN KEY | 表與表掛鉤 | ❌ | — | 交易系統用；落地表常常不設，把驗證放在 pipeline | 補充C §3 |
| 分區 | 大表切抽屜 | ❌ | ✅ | 千萬筆以上才值得；分區鍵必須在主鍵裡 | 補充C §7 |

- 自然鍵和代理鍵：號碼牌（`AUTO_INCREMENT`）防不了重複排隊。同股同日塞兩次，會各拿到一個 id（補充C）。
- 驗證：`EXPLAIN SELECT ... WHERE stock_id='2330'` 從 `type=ALL rows=320` 變成 `type=ref rows=32`（補充C §1，假資料 320 筆）。
- **範例 B：to_sql ＋ SQLAlchemy，自己掌握型別**（[SC] 手冊05 參數表）

```python
from sqlalchemy import create_engine, types
engine = create_engine("sqlite:///pipeline.db")                     # 期中
# engine = create_engine(os.environ["DB_URL"])                      # MySQL：帳密放 .env；本機預設值 root/1234
df.to_sql("cleaned_orders", engine, if_exists="replace", index=False,  # cleaned 可整表重建
          dtype={"customer_id": types.String(20), "invoice_date": types.DateTime()},
          chunksize=2000, method="multi")                            # 十萬筆以上建議 chunk 1000–5000
```

- 提醒：`to_sql` 自動建表**不會設主鍵**，型別也是猜的。需要 upsert 的表要用 `Table(...)` 或 `CREATE TABLE` 明確宣告（[SC] 手冊06「想再深入一點」）。
- **產出**：DDL（或 `Table(...)` 定義）、索引清單、一張「為什麼選這個儲存」的 ADR 草稿（[DD] `07-demo-day-rubric.md` 架構設計：至少 2 個選型講得出原因）。

### Step 5：冪等載入（upsert）（30 min）

- **目標**：同一批資料寫幾次，結果都一樣。
- **做法：故意失敗再修好**（[SC] 手冊06 Step 1–7）

| | 表**沒有**主鍵 | 表**有**主鍵 |
|---|---|---|
| append（普通 INSERT） | ✅ 能跑，但重跑筆數翻倍 | 第一輪 ✅；第二輪 ❌ `IntegrityError (1062, "Duplicate entry ...")` |
| upsert（ON DUPLICATE KEY UPDATE） | ✅ 能跑，但備案永遠不會觸發（等於 append） | ✅ 撞到主鍵就改成更新，重跑任意次都正確 |

- **範例 C：MySQL 批次 upsert**（改寫自 [SC] `crawler/tasks_crawler_finmind_duplicate.py`、[HH] `mysql.py`；課程原版用 `iterrows` 逐列，這裡改成一次送多列）

```python
from sqlalchemy import MetaData, Table, Column, String, Date, Float, BigInteger
from sqlalchemy.dialects.mysql import insert           # MySQL 專用；PostgreSQL 改 ON CONFLICT
metadata = MetaData()
price = Table("TaiwanStockPrice_duplicate", metadata,
    Column("stock_id", String(50), primary_key=True),
    Column("date", Date, primary_key=True),            # 複合主鍵＝同股同日只有一筆
    Column("close", Float), Column("Trading_Volume", BigInteger))
metadata.create_all(engine)                            # 表不存在才建
rows = df.replace({np.nan: None}).to_dict(orient="records")
stmt = insert(price).values(rows)
stmt = stmt.on_duplicate_key_update({c.name: stmt.inserted[c.name] for c in price.columns if not c.primary_key})
with engine.begin() as conn:                           # 一個交易：全成功才 COMMIT
    conn.execute(stmt)
```

- **驗證**（重跑三次筆數不變）

```sql
SELECT COUNT(*) FROM TaiwanStockPrice_duplicate;                       -- 跑 3 次，3 個數字相同
SELECT stock_id, date, COUNT(*) FROM TaiwanStockPrice_duplicate
GROUP BY stock_id, date HAVING COUNT(*) > 1;                           -- 必須是空結果
```

- DB 層和 DataFrame 層去重怎麼選（[SC] 手冊06 對照表）：多 worker、資料一直長大 → DB 層 upsert；單機一次性整理 → `drop_duplicates` 後整表重寫。
- 自動化證明：[SC] `tests/test_upsert_integration.py`，寫兩次筆數 `first == second == 2`；同主鍵改值 → 更新而不是新增。
- **產出**：重跑三次的筆數截圖或 log，加上一支 integration test。

### Step 6：資料品質測試＋analyzed 層（20 min）

- **目標**：壞資料進不了 Gold；三層筆數講得出故事。
- **做法**

| 維度 | 斷言 | 出處 |
|---|---|---|
| 完整性 | 關鍵欄 `IS NULL` 筆數 = 0 | [MT] `README_TEMPLATE.md` §5、清洗檢查點 `assert df.isnull().sum().sum() == 0` |
| 唯一性 | 主鍵 `GROUP BY ... HAVING COUNT(*)>1` 為空 | [SC] 手冊06 練習 2 |
| 有效性 | 數量／金額 > 0；日期落在合理區間 | [MT] topic_1 檢查點 `(df["quantity"] > 0).all()` |
| 一致性 | `dtypes` 符合 Data Dictionary；ID 是字串 | [MT] §5 |
| 筆數守恆 | cleaned ≤ raw；stage ≤ raw；差額要解釋得出來 | [MT] 三表驗證 `UNION ALL`；[SC] 手冊15 4-1 |
| 新鮮度 | `MAX(date)` 是最近一個交易日 | [SC] 手冊08 藍圖「最新資料日期」卡 |

- **範例 D：BigQuery raw → stage → app**（[SC] 手冊15 Step 3–4）

```sql
CREATE OR REPLACE VIEW stage.stock_price_daily AS           -- Silver：去重＋統一欄名，用 view 隨時可改
SELECT stock_id, date AS trade_date, open, max, min, close, spread,
       Trading_Volume AS volume, Trading_money AS amount
FROM (SELECT s.*, ROW_NUMBER() OVER (PARTITION BY stock_id, date ORDER BY Trading_Volume DESC) AS rn
      FROM raw.TaiwanStockPrice s) WHERE rn = 1;

CREATE OR REPLACE TABLE app.market_daily_summary AS         -- Gold：實體表，BI 只讀這層；OR REPLACE＝冪等重算
SELECT trade_date, COUNT(DISTINCT stock_id) AS active_stocks, SUM(volume) AS total_volume,
       ROUND(AVG(close), 2) AS avg_close, COUNTIF(spread > 0) AS up_count
FROM stage.stock_price_daily GROUP BY trade_date;

SELECT "raw" AS layer, COUNT(*) n FROM raw.TaiwanStockPrice     -- 4-1：raw 比 stage 多的就是重複列
UNION ALL SELECT "stage", COUNT(*) FROM stage.stock_price_daily
UNION ALL SELECT "app.summary", COUNT(*) FROM app.market_daily_summary;
```

- 業界工具地圖（先認識名字就好）：dbt tests、Great Expectations、Soda（[SC] 手冊05「更大的世界」；[DD] 02 6️⃣ Observe）。
- **產出**：`analyzed_*`／app 表、`check_quality()` 或 pytest、筆數追蹤句「raw N → cleaned M（-X%）→ analyzed K」（[MT] `README_TEMPLATE.md` §4）。

---

## 5. 卡關排錯：你遇到的狀況／原因／怎麼解

| 你遇到的狀況 | 原因 | 怎麼解 |
|---|---|---|
| 重跑後筆數翻倍 | `to_sql(if_exists="append")`＋表沒主鍵 | 宣告複合主鍵，改用 upsert（[SC] 手冊05 Step 8 → 手冊06） |
| `IntegrityError (1062, "Duplicate entry ... for key 'PRIMARY'")` | 表有主鍵，寫入端還在用 append | 表設計和寫入方式要成對：換 upsert；搬舊資料用 `INSERT IGNORE` |
| `0050` 變成 `50`，字串比對查到 0 筆 | `to_sql` 自動建表把 `stock_id` 猜成 BIGINT | `dtype=` 指定 String，或自己建表；比對前先 `astype(str)` |
| `drop_duplicates` 沒去掉重複 | 一邊是 `"2330"`，另一邊是 `2330` | 去重前先統一主鍵欄的型別（[SC] 手冊06 ③） |
| `customer_id` 顯示 `12345.0` | 有 NaN 的整數欄被 pandas 轉成 float | 先 dropna 再轉 int → str，或用 `Int64` |
| 舊資料整批消失 | 對 raw 用了 `if_exists="replace"` | raw 只 append；replace 只用在可重建的 cleaned／analyzed |
| 寫 DB 報 NaN／NaT 錯 | MySQL 不吃 numpy NaN | `df.replace({pd.NaT: None, np.nan: None})`（[HH]） |
| CSV 中文亂碼 | 編碼 | `utf-8-sig`；政府資料試 `cp950` |
| 坪數、萬元算出來怪怪的 | 單位混用，或常數不一致 | 欄名帶單位；常數只定義一次（見 §7 注意） |
| upsert 很慢 | `iterrows()` 逐列送 SQL | 批次 `values(rows)`，或分 chunk |
| 多 worker 同時寫，資料被蓋掉 | DataFrame 層「讀回 → 合併 → 清空 → 寫回」不是原子操作 | 改用 DB 層 upsert，讓資料庫裁決（[SC] 手冊06） |
| BigQuery `Quota exceeded: ... partition modifications` | 一次重抓全歷史，同一天反覆重跑 | 正式環境每天只 append 當日分區（[SC] 手冊15 排錯） |
| BigQuery 帳單變高 | `SELECT *`、沒帶分區條件 | 只選需要的欄、加分區條件；先 `--dry_run`（[SC] 補充K 實驗一） |
| raw 一直長出重複列 | 雙寫 append 的天性 | 不是 bug；分析走 stage（已去重） |

## 6. 交付前檢查清單（☐）

- ☐ README 有一行箭頭流程，和 raw／cleaned／analyzed 三層的名字
- ☐ raw 表筆數 = 來源筆數；raw 沒有被 replace 或清洗過
- ☐ Transform Logic 表有寫：每一步的程式碼／理由／影響筆數
- ☐ 筆數追蹤句：raw N → cleaned M（-X%）→ analyzed K
- ☐ 主鍵（或複合主鍵）有宣告，Data Dictionary 有寫粒度
- ☐ 至少一個依 WHERE 建的索引，並有 `EXPLAIN` 證據（MySQL 版）
- ☐ upsert 重跑三次筆數不變；`HAVING COUNT(*)>1` 為空
- ☐ 資料品質斷言 ≥ 4 條（完整、唯一、有效、筆數守恆），失敗時 pipeline 會停
- ☐ 儲存選型有 1 句理由（為什麼不是另一個），寫進 ADR 或 README
- ☐ 沒有 hardcoded 密碼；`.env.example` 完整（[DD] 07 實作品質）
- ☐ 面試 5 秒講法：「我把 raw 永遠留著，清理在 Silver 做；用 (stock_id, date) 複合主鍵加 upsert，重跑三次筆數不變。」

## 7. 給撰寫者的注意（來源核對時發現）

- [MT] topic_6 單位常數不一致：Step 2-3 說明寫 `÷3.30579`，程式骨架寫 `/ 3.306`。成品文件要統一成一個常數，並且只定義一次。
- [MT] topic_6 說明列出「民國年轉西元年」是難點，但 starter 程式格沒有這段。§4 範例 A 補了一個寫法，實測後才寫進教材。
- [MT] `docs/pipeline.mmd` 寫「三表筆數一致 ✓」，但 analyzed 只跑 `n=50` 筆（`run_batch_analysis`）。三表不會一致，應該寫成「筆數守恆、差額可解釋」。
- [DD] `07-demo-day-rubric.md` 沒有獨立的「資料品質」評分項。品質落在「3. 實作品質」（測試、CI、無 hardcoded secret）和 02 的 6️⃣ Observe（dbt tests／Great Expectations）。§4 Step 6 的品質維度改用 [MT] `README_TEMPLATE.md` §5。
- EXPLAIN 的 320 → 32、BigQuery dry run 的 2.97 MB → 310 KB → 0 B 都來自課程假資料或當時的實測截圖，引用時標「約略值，依資料量不同」。
