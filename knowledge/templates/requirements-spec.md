# 需求規格書範本（requirements_spec.md）

> 給工作坊「每題一份」的需求規格書。學員選題後先讀這份，就知道情境、資料、今天要做到哪。
> 由 midterm-mvp-template 的 6 份 `data/raw/topic_N/requirements_spec.md` 歸納而成。

## 使用說明

- 一題一份，放在該題資料夾最上層：`data/raw/topic_N/requirements_spec.md`。
- 長度約 80–90 行，學員 5 分鐘讀完。
- 每個 `{...}` 都要換成具體內容；欄位陷阱、成功指標一定要寫。
- 「交流引導」兩段每題都一樣，直接保留，讓工作坊的交流時段有題可問。

## 寫作規則

| 段落 | 規則 |
|------|------|
| 情境 | 用「老闆／主管說：『……』」一段話開場，裡面帶出 3–4 個要回答的問題 |
| 你的角色 | `{產業}資料顧問`，一行 |
| 使用者 | 誰 + 多久看一次 + 用來決定什麼 |
| 資料來源 | 檔案表（檔案、筆數、說明、來源連結）+ 欄位表（欄位、型別、說明）；欄位多時寫「共 N 欄，列出關鍵欄位」 |
| 欄位陷阱 | 直接寫在欄位說明裡：「注意：含逗號，需轉數字」「民國年格式」「可能有空值」 |
| MVP Scope | 分 ETL 與 LLM 兩塊；ETL 寫 3–5 個具體清洗與統計動作 + 產出檔名；LLM 寫分類（列出標籤）+ 1–2 種生成文字 |
| 預期產出 | 檔名 + 一句說明 |
| 成功指標 | 一句可口說的數字句 + 一句「建議有數據支撐（不是空話）」 |

---

## 範本

````markdown
# 題目 {N}：{題目名稱}

## 情境

你是一家{公司類型}的資料顧問。{老闆／營運主管／產品經理}說：「{痛點一句}。我想知道{問題 1}、{問題 2}、{問題 3}，這樣我們可以{要做的決策}。」

## 你的角色

{產業}資料顧問

## 使用者

{使用者職稱}——{多久}看一次{什麼報告}，決定{哪些決策}。

## 資料來源

| 檔案 | 筆數 | 說明 | 來源 |
|------|------|------|------|
| `{file}.csv` | {2,000} | {一句話說明，含時間範圍} | [{來源名稱}]({URL}) |

### 欄位說明

| 欄位 | 型別 | 說明 |
|------|------|------|
| `{col_1}` | {int/float/str/datetime} | {說明} |
| `{col_2}` | str | {說明}（注意：{陷阱}） |
| … | | |

## 今日 MVP Scope

### ETL（pandas 清洗 + 統計）

- {型別轉換：日期轉 datetime、去符號轉數字……}
- {缺漏值／異常值處理}
- {衍生欄位：時間特徵、差值、單位換算……}
- 統計：{排行 Top N}、{分組比較}、{時間趨勢}
- 產出 `data/processed/{topic}_stats.csv`

### LLM（AI 加值分析）

- 對 `{文字欄位}` 做**{分類任務}**（{標籤1}/{標籤2}/{標籤3}/{標籤4}/其他）
- 生成**{洞察或建議類文字}**：「{示範問句}」
- {選做：跨群組比較摘要}

## 預期產出

- `data/processed/{topic}_stats.csv`：{統計內容}
- `data/processed/{topic}_analyzed.csv`：{每筆 + LLM 欄位}（選做）
- `output/pipeline_doc.md`：{報告主題}

## 成功指標

- 能說出「{最重要的 3 個 X 和 3 個 Y}」
- {建議／洞察}有數據支撐（不是空話）

---

## 交流引導

### 第一次交流（選題後，5 min）

組員會問你以下問題，看著上面的情境和欄位回答：

1. 你的資料有哪些欄位？哪個欄位你覺得最有分析價值？
2. 你打算怎麼清洗這份資料？預期會遇到什麼問題？
3. 如果只能做一個統計圖表給老闆看，你會做什麼？
4. 這份資料最大的限制是什麼？

### 第二次交流 — 閃電秀預演（準備 Demo 前）

每組派 2~3 人，每人 3 分鐘，向組員說明你等等要 Demo 的內容：

- 我的題目解決什麼問題？
- 我做了哪些清洗和分析？
- 最有趣的發現是什麼？（用數字說）
- LLM 在 pipeline 中做了什麼？

> 交流時可以比正式 Demo 更詳盡，組員給回饋幫你精煉重點。
````

---

## 填寫範例：公共自行車站點調度

> 示範用的新題目（不在原 repo 內），展示範本如何套到新情境。資料來源為臺北市資料大平臺公開資料；欄位名稱依實際下載檔為準，上課前請講師下載取樣後再核對一次。

````markdown
# 題目 7：公共自行車站點調度分析

## 情境

你是一家城市交通營運商的資料顧問。調度主管說：「我們每天都在被投訴『借不到車』和『沒位置還車』，但調度車只有幾台。我想知道哪些站最常空站或滿站、哪些行政區問題最嚴重、一天中哪幾個時段最需要補車，這樣我們可以排出更有效率的調度路線。」

## 你的角色

交通服務資料顧問

## 使用者

調度主管——每天早上看前一天的站點報告，決定調度車今天先跑哪些站。

## 資料來源

| 檔案 | 筆數 | 說明 | 來源 |
|------|------|------|------|
| `stations.csv` | 2,000 | 講師每 30 分鐘抓一次站點即時資料，累積數天後取樣 | [臺北市資料大平臺：YouBike2.0 臺北市公共自行車即時資訊](https://data.taipei/) |

### 欄位說明

| 欄位 | 型別 | 說明 |
|------|------|------|
| `sno` | str | 站點代號 |
| `sna` | str | 站點名稱（中文，前綴有「YouBike2.0_」需去除） |
| `sarea` | str | 行政區 |
| `total` | int | 總車位數 |
| `available_rent_bikes` | int | 可借車數 |
| `available_return_bikes` | int | 可還空位數 |
| `latitude` / `longitude` | float | 經緯度 |
| `updateTime` | str | 資料更新時間（注意：字串，需轉 datetime） |
| `act` | str | 站點是否營運（"1" = 營運中） |

## 今日 MVP Scope

### ETL（pandas 清洗 + 統計）

- `updateTime` 轉 datetime，提取日期、小時、星期幾
- 過濾未營運站點（`act != "1"`）與 `total = 0` 的異常列
- 新增欄位：`rent_ratio = available_rent_bikes / total`；`is_empty`（可借 = 0）、`is_full`（可還 = 0）
- 統計：空站次數 Top 10 站、滿站次數 Top 10 站、各行政區空站率、各小時空站率
- 產出 `data/processed/station_stats.csv`

### LLM（AI 加值分析）

- 對站名做**站點類型分類**（捷運站周邊/學校/商圈/住宅區/公園/其他）
- 生成**調度建議**：「早上 8 點前應先補哪 3 個站？理由是什麼？」
- 選做：比較兩個行政區的問題型態（缺車 vs 缺位）

## 預期產出

- `data/processed/station_stats.csv`：站點 × 時段統計
- `data/processed/stations_analyzed.csv`：每站 + LLM 站點類型
- `output/pipeline_doc.md`：站點問題分析 + 調度建議

## 成功指標

- 能說出「最常空站的 3 個站、最常滿站的 3 個站，以及最需要補車的 3 個時段」
- 調度建議有數據支撐（不是空話）

---

## 交流引導

（同範本，原文保留）
````

---

## 參考來源

- [data/raw/topic_1/requirements_spec.md](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_1/requirements_spec.md)（零售 POS）
- [topic_2](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_2/requirements_spec.md)（電商比價）、[topic_3](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_3/requirements_spec.md)（音樂串流）、[topic_4](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_4/requirements_spec.md)（叫車熱點，兩檔 merge）
- [topic_5](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_5/requirements_spec.md)（求職薪資）、[topic_6](https://github.com/lu791019/midterm-mvp-template/blob/main/data/raw/topic_6/requirements_spec.md)（不動產，民國年）
- [docs/topic_catalog.md](https://github.com/lu791019/midterm-mvp-template/blob/main/docs/topic_catalog.md)、[docs/data_sources.md](https://github.com/lu791019/midterm-mvp-template/blob/main/docs/data_sources.md)
