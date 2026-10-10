# 教學手冊：生產級 DE 專案課（台股爬蟲系統）

> 這份文件讓之後協助 Dex 的 AI 或助教，快速掌握 stock-crawler 課程怎麼教：章節順序、架構怎麼一步步長大、每章手冊的固定骨架、學員最常卡在哪，以及「業界等級」在這門課具體指什麼。
> 整理自 `stock-crawler-de-course-materials`（2026-10-10 讀取，main 分支）。

## 一句話定位

用一個「台股資料爬蟲」當唯一的貫穿專案，從一支 Python script 走到「Airflow 排程 → RabbitMQ → Celery worker → MySQL 與 BigQuery 雙寫 → BI 儀表板」，最後整套搬上 GCP。專案改編自 TibameSam/crawler，Dex 加上整合版 Docker Compose、Airflow／Metabase／BigQuery 模組、bug 修復與完整手冊。

## 貫穿全課的資料流

```
Airflow / APScheduler（排程編排）
  → Producer 發任務 → RabbitMQ 佇列 → Celery Worker 呼叫 FinMind API
  → 雙寫：MySQL（OLTP，營運）＋ BigQuery raw（OLAP，分析）
  → Metabase（本機）／ Looker Studio（雲端）
```

雙寫的開關只有一個環境變數：`GCP_PROJECT_ID` 有值就寫 BigQuery，沒值就只寫 MySQL，並印出「BQ 未設定，略過雲端寫入」。所以第 1–13 章在本機完全不需要 GCP。

## 章節總表

### 主線（17 章，分四個 Phase）

| 章 | 主題 | 技術 | 產出 |
| --- | --- | --- | --- |
| 01 | Celery 基礎：Producer／Broker／Worker | Celery、RabbitMQ、Flower、uv、Docker Compose | 100 個假任務被多個 worker 平行消化 |
| 02 | 真實爬蟲（只印出） | FinMind API、requests、pandas | worker 印出 5 支股票的股價 DataFrame |
| 03 | 多佇列分流＋水平擴充 | `apply_async(queue=)`、`-Q`、`--scale` | twse／tpex 兩條佇列各自有專屬 worker |
| 04 | 失敗處理 | `self.retry()`、`Reject(requeue=True)`、`acks_late` | 四種失敗情境的訊息命運觀察紀錄 |
| 05 | 寫入 MySQL | SQLAlchemy、pymysql、phpMyAdmin | `TaiwanStockPrice` 表有資料，另存 CSV |
| 06 | 去重與冪等 | 複合主鍵、`on_duplicate_key_update`（upsert） | 重跑三次筆數不變 |
| 07 | Web 管理介面與排錯 SOP | RabbitMQ UI、Flower、phpMyAdmin、Portainer | 一套由外而內的五步排錯順序 |
| 08 | Metabase BI 視覺化 | Metabase、MySQL VIEW、唯讀帳號 | 第一張 Dashboard |
| 09 | 定時排程 | cron、APScheduler（Background／Blocking） | pipeline 到點自動發任務 |
| 10 | Airflow 基礎 | Airflow 2.10、DAG、LocalExecutor | 第一個 DAG 在 UI 跑綠 |
| 11 | Airflow 進階 Operator | Branch、XCom、TriggerDagRun、DockerOperator、Dummy | 五塊積木各一個範例 DAG |
| 12 | Airflow 接上爬蟲 | 三種串法、ETL DAG、CeleryExecutor | Airflow 觸發爬蟲並建出 VIEW 與實體表 |
| 13 | 完整系統整合 | `docker-compose-all.yml`（13 個服務） | 一鍵啟動＋七步驟端到端驗證 |
| 14 | GCP 開通與雲端部署 | GCE、防火牆、IAM、服務帳戶、預算警告 | 整套系統跑在一台 VM，雙寫開始運轉 |
| 15 | BigQuery 資料倉儲 | OLTP vs OLAP、raw／stage／app 三層、Looker Studio | 分區表、MA5／MA20 趨勢表、四區塊儀表板 |
| 16 | 系統搬家：多台 VM 與 Cloud SQL | Cloud SQL、Secret Manager、內外部 IP、Spanner（Bonus） | VM1 發任務、VM2 做事、寫入託管 MySQL |
| 17 | 每日資料線 | Airflow 排程、Cloud Composer | 每個交易日自動爬取＋重算 app 層 |

Phase 劃分：A＝第 1–7 章（任務怎麼可靠地跑）、B＝第 8–9 章（資料出口與自動化）、C＝第 10–13 章（編排與整合）、D＝第 14–17 章（上雲）。

### 補充章（字母順序即建議閱讀順序）

| 篇 | 主題 | 建議時點 |
| --- | --- | --- |
| A | 用 draw.io 畫本地端與雲端架構圖（含 Mermaid） | 隨時；13 章後畫本地、14 章後畫雲端 |
| B | 同步／非同步、多執行緒、多行程、分散式 | 01–02 章後 |
| C | MySQL 深入：索引、約束、外鍵、交易、權限、分區 | 05 章後 |
| D | ACID 與 CAP，同一支爬蟲改寫 MongoDB | 05–06 章後 |
| E | MySQL to FastAPI（資料的 API 出口） | 05 章後 |
| F | Unit Test 與整合測試（pytest、mock、冪等驗證） | 06 章與補充 E 後 |
| G | .env 與環境變數：`${}` 替換 vs `env_file` 注入 | 10–13 章 |
| H | API 上雲：Artifact Registry、Cloud Run | 16–17 章後 |
| I | CI／CD：GitHub Actions | 17 章與補充 H 後 |
| J | 從一支爬蟲到分散式：七步拆解（`example/evolution/`） | 開課前導覽或全課後回顧 |
| K | BigQuery 深入：dry run 掃描量、Time Travel、BQML | 15 章後 |
| L | 雲地混合：本機資料橋接上雲（金鑰、sync 回填） | 15 章後 |

## 架構演進：本機 → 分散式 → 雲端

| 階段 | 章 | 架構樣貌 | 這一步換掉了什麼 |
| --- | --- | --- | --- |
| 0. 單體 script | 補充 J Step 0–4 | 一支檔案抓、整理、存 | 只整理程式形狀，不加元件 |
| 1. 本機分散式 | 01–07 | Producer → RabbitMQ → 多個 worker → MySQL | 換掉「誰來執行」（`.delay()`） |
| 2. 資料出口 | 08、補充 E | MySQL → Metabase／FastAPI | 加上給人看、給程式用的出口 |
| 3. 自動觸發 | 09–12 | APScheduler → Airflow DAG 發任務 | 換掉「誰來按下 producer」 |
| 4. 本機全整合 | 13 | 13 個容器一鍵起、七步驟驗證 | 把所有零件放進同一份 compose |
| 5. 單台雲端 VM | 14 | 同一份 compose 搬上 GCE，雙寫開啟 | 換掉「機器在哪」 |
| 6. 倉儲分層 | 15 | BigQuery raw → stage（view）→ app（CTAS） | 分析從 MySQL 移到倉儲 |
| 7. 多機＋託管 | 16 | VM1（Airflow、RabbitMQ）、VM2（worker）、Cloud SQL | 只改 `MYSQL_HOST`，BigQuery 線零改動 |
| 8. 每日資料線 | 17 | 排程觸發雙寫＋重算 app；可換 Composer | 從自架 Airflow 到託管 Airflow |

教學上要強調的規律（補充 J 原話的精神）：前四步只整理程式形狀；之後每加一個元件，只換掉一件事。Step 5 換「誰來執行」，Step 6 換「誰來觸發」。

補充 J 的兩條判準，適合直接拿來教學員拆自己的專題：

1. **拆的依據是「會為了不同理由改變」**：範圍變了動 producer；來源變了動 client；資料形狀變了動 transformer；儲存目標變了動 repository；環境變了動 config；觸發時機變了動 DAG。
2. **每一步都要能執行**：改完必須能跑、輸出與前一步相同。所以 Celery 一定排在後面，因為它需要「任務」這個邊界先存在。

`example/evolution/` 提供可 diff 的證據：`diff step4_task/task.py step5_celery/task.py` 只差裝飾器與 import。

## 每章手冊的固定骨架

手冊有兩種版型，寫新章節時要跟著對應的版型走。

### 第 1–13 章（本機段）

| 段落 | 做什麼 |
| --- | --- |
| 標題＋引言 | 「第 N 章：主題 — 副標」，引言一段說明本章目標與為什麼重要 |
| 做完這一章，你會做到 | 4–6 條可驗證的能力，用動詞開頭 |
| 先搞懂（觀念） | 先用生活比喻（餐廳、電梯按鈕、鬧鐘），再講術語 |
| 這一章會用到的檔案 | 檔案、角色、說明表 |
| 一行一行讀懂 | 逐段讀關鍵程式，註解寫在程式碼裡 |
| 一步一步跟著做 | 指令＋預期輸出＋「✅ …就過關」 |
| 檢查你是不是真的做到了 | 「你應該看到／它證明了什麼」表 |
| 想再深入一點 | 選讀延伸 |
| 卡住了？常見錯誤這樣排 | 「你遇到的狀況／原因／怎麼解」三欄表 |
| 想一想（確認你懂了） | Q1–Q5，附完整答案，提醒「先自己想過再看答案」 |
| 換你試試看 | 2–3 個小練習，常有「觀察並解釋」 |
| 這一章你學到了／下一章要做什麼 | 條列重點，並預告下一章「只換了一個東西」 |

### 第 14–17 章（雲端段）

骨架改為：本章用到的工具與服務（表）→ 做完這一章你會 → 先搞懂 → 一步一步（Part A、B、C…）→ **團體專案上雲：本章設定的團隊版** → 收工（停機省額度）→ 檢查 → 想一想 → 練習 → 排錯 → 本章總結。

雲端段多出兩個固定段落：

- **團體專案上雲**：把一個人的流程轉成三到五人的期末專題版本，例如誰開專案、IAM 角色、docker 群組、全組 IP 白名單、強密碼、磁碟快照。
- **收工**：每章都教怎麼停 VM、停 Cloud SQL、刪 Composer，因為額度是學員自己的。

另有一份 `crawler_實作操作手冊.md`，是「不講觀念、只給步驟」的指令速查，給已學過、要重建環境的人。

## 學員常見卡點與教材的處理方式

| 卡點 | 出現在 | 教材怎麼處理 |
| --- | --- | --- |
| `ModuleNotFoundError: crawler` | 01 起 | Step 0 就提醒「指令一定要在專案根目錄」，先打 `pwd` |
| 把 `crawler(x)` 當成非同步 | 01 | Q1 對照 `.delay()` 與直接呼叫，用時間軸圖說明 |
| 以為調高 concurrency 會照順序 | 01 | Q5 加練習 3：用 concurrency=1 與 8 對照，結論「併發下的順序永遠不該被依賴」 |
| 任務送出卻沒人做 | 03、07、12 | 佇列名稱要兩邊一致，worker 要加 `-Q`；排錯 SOP 先看 RabbitMQ 的 Ready 有沒有堆積 |
| retry 與 requeue 分不清 | 04 | 用獨立的 `worker_demo.py` 故意失敗、故意殺 worker，觀察訊息命運 |
| 容器內連 `127.0.0.1` | 05、08、12 | 反覆強調容器間要用服務名（`mysql`），並用環境變數覆蓋 |
| 重跑資料重複 | 06 | 先走一遍「append → 報錯 → upsert」，再做反向驗證 |
| 改了 `.env` 密碼卻登不進去 | 補充 G | 三個必踩的坑之一：MySQL 密碼只在資料卷第一次初始化時寫入 |
| 排程時間差 8 小時 | 09 | 一律寫 `timezone="Asia/Taipei"`；搭配 `coalesce=True` |
| Airflow 8080 跟 phpMyAdmin 撞 port | 10、13 | 單獨跑擇一啟動；all 版把 Airflow 改到 8081 |
| DAG 觸發了卻沒動 | 10、11 | 先 unpause；子 DAG 也要 unpause |
| 分支後 end 被 skipped | 12 | `trigger_rule="none_failed_min_one_success"` 是分支 DAG 的固定搭配 |
| 「容器都 Up」就以為系統會動 | 13 | 七步驟驗證：服務狀態 → Web 介面 → log → 發任務 → worker 執行 → DB → Airflow |
| 換地方後雲端介面連不上 | 14、17 | 對外 IP 變了，防火牆 source-ranges 要更新；停機再開外部 IP 也會換 |
| 磁碟滿 `No space left on device` | 14、15 | 專段教清 build cache 與擴充磁碟 |
| 密碼三邊不一致 | 16 | 用 Secret Manager 的值閉環驗證；`.env` 一律用 `$(gcloud …)` 取值，不手抄 |
| `.env` 抄進佔位符 `{IP}` | 16 | 錯誤訊息出現 `%7B…%7D` 就是這個；用指令產生取代手抄 |
| BigQuery 分區修改配額 403 | 15、17 | 說明是同一天反覆全量重跑造成，隔天重置，MySQL 那半邊不受影響 |
| Composer 選到 Airflow 3 | 17 | 截圖標出下拉選單陷阱；大版本不能原地換 |

處理方式的共同點：

- **先讓學員故意遇到錯**，再給解法（第 4 章故意失敗、第 6 章先 append 報錯、第 16 章先用舊密碼被拒）。
- **錯誤訊息原文進教材**，學員可以直接用訊息搜尋手冊。
- **區分「暫時現象」和「真的錯」**：啟動初期的 `Connection refused` 會自動重連，不算錯；Flower 上的舊 FAILURE 是歷史紀錄，要認時間戳。

## 「業界等級」在這門課指什麼

Dex 的課程把「業界等級」落在可驗證的性質上，而不是技術名詞的數量：

| 性質 | 在課程中怎麼做到 | 章 |
| --- | --- | --- |
| 可水平擴充 | 任務與執行者分離，加 worker 不改程式；`--scale`、跨機器 VM2 | 01、03、16 |
| 容錯 | `acks_late`、retry、requeue；BigQuery 寫入失敗不擋 MySQL | 04、15 |
| 冪等 | 複合主鍵＋upsert；stage／app 可重建；DAG 重跑無害 | 06、15、17 |
| 可觀測 | Flower、RabbitMQ UI、Airflow Grid／Logs、排錯 SOP | 07、10 |
| 可編排、可補跑 | Airflow 的依賴、分支、Clear 單一 task 重跑 | 10–12 |
| 設定外部化 | `config.py` 讀環境變數；搬家只改 `.env` 的幾行值 | 04、補充 G、16 |
| OLTP／OLAP 分工 | 雙寫；raw 只 append、stage 清理、app 給報表 | 15 |
| 機密管理 | `.env` 不進 git；團體專案換強密碼；Secret Manager 版本輪替 | 14、16 |
| 成本意識 | 開通先設預算警告；每章教停機；dry run 看掃描量 | 14–17、補充 K |
| 託管 vs 自架的判斷 | 「用錢買維運」；資料庫最先換託管；自架 Airflow vs Composer | 16、17 |
| 驗證，而不是相信 | 七步驟端到端驗證；pytest 單元＋整合測試；GitHub Actions CI | 13、補充 F、I |

可以這樣對學員說：面試官要聽的不是「我用了 Airflow」，而是「worker 掛了任務會怎樣」「重跑會不會重複」「MySQL 換成 Cloud SQL 要改哪裡」。這門課每一章都在幫學員準備其中一題的答案。

## 本機預設值與安全提醒

- 教材中的 MySQL `root / 1234`、RabbitMQ `worker / worker`、Airflow `admin / admin` 都是**本機預設值，勿用於正式環境**。第 14 章〈團體專案上雲〉明確教學員換強密碼並放進 `.env`。
- 協助學員時，不要把學員的 `.env`、服務帳戶 JSON 金鑰或專案 ID 貼進聊天或文件。

## 協助 Dex 時的使用建議（建議）

以下是根據教材整理的使用方式，不是 repo 內的規定：

- **學員問題分流**：先問卡在第幾章，再對照該章的「卡住了」或「排錯」表。大多數問題原文就在表裡。
- **專題輔導**：用補充 J 的兩條判準檢查學員自己的專題，問「這段程式會為了什麼理由改變」。
- **面試準備**：把上方「業界等級」表的每一列，轉成一題「請說明你的系統如何做到 X」。
- **新增章節**：沿用對應版型的骨架；觀念先用比喻，錯誤訊息照原文寫進排錯表。

## 已知的不一致（維護時注意）

- README 的資料夾說明寫「17 章＋補充 A~K」，實際已有補充 L。
- README 一處寫 `docker-compose-all.yml` 為 13 個服務，另一處寫「共 11 容器」。
- `crawler_實作操作手冊.md` 開頭仍寫「14 章＋補充 B/B/C」，是舊版說法。
- `task.md` 提到投影片（EP12／17／18／20）落後於手冊，尚未更新。

## 來源

- README：https://github.com/lu791019/stock-crawler-de-course-materials/blob/main/README.md
- 課程手冊目錄：https://github.com/lu791019/stock-crawler-de-course-materials/tree/main/課程手冊
- 第 1 章（骨架、Q5 concurrency）：https://github.com/lu791019/stock-crawler-de-course-materials/blob/main/課程手冊/課程手冊01%20-%20Celery基礎.md
- 第 7 章（排錯 SOP）、第 13 章（七步驟驗證）、第 14–17 章（雲端段）：同上目錄
- 補充 G（三個必踩的坑）、補充 J（兩條判準）：同上目錄
- 七步拆解程式：https://github.com/lu791019/stock-crawler-de-course-materials/tree/main/example/evolution
- 14–17 章雙寫改版計畫：https://github.com/lu791019/stock-crawler-de-course-materials/blob/main/implementation_plan.md 、 https://github.com/lu791019/stock-crawler-de-course-materials/blob/main/task.md
- 指令速查：https://github.com/lu791019/stock-crawler-de-course-materials/blob/main/crawler_實作操作手冊.md
