# airflow_heroku_jupyter

> 一句話：把 Airflow（加 JupyterLab）部署到 Heroku 的設定範本，另附一張工作上影像辨識結果入倉的流程圖。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/airflow_heroku_jupyter |
| 年份 | 2021（最後 commit 2021-01-27） |
| 情境 | 自學（以開源 heroku-airflow 範本為底）；流程圖是工作上的入倉流程（Dex 確認） |
| 技術 | Apache Airflow（webserver、worker、scheduler）、Celery + Redis、Heroku Postgres、Procfile、JupyterLab；流程圖：Jupyter Hub on GPU server、HDFS、Hive Table、Hive Partition Table、MQ |

## 做了什麼

- `Procfile` 起 Airflow webserver、worker、scheduler 三個 process；`requirements.txt` 加了 pandas、jupyterlab。README 是原範本的說明。
- 流程圖（`Copy of flowchart_consultant_specificity.png`）：教學影像經 MQ → GPU 伺服器上做人臉辨識 → 結果存成 CSV → `hdfs dfs -put` → Hive Table → Hive Partition Table，並清掉暫存檔。

## 可當教學案例的點

- **排程工具的三個角色**：webserver、scheduler、worker 分開跑，是 [../topic-briefs/ops.md](../topic-briefs/ops.md) 講 Airflow 架構的好圖解。
- **泳道圖**：流程圖按系統分泳道（GPU 伺服器、Hadoop、產品端），可以直接當學員畫架構圖的範例。
- **AI 結果也要入倉**：模型輸出不是終點，要進資料倉儲才能被分析，這是資料工程師在 AI 專案裡的角色。

## 如果今天重做會怎麼升級（建議）

- 建議：Heroku 已無免費方案，教學改用 Docker Compose 本機跑 Airflow，或用 Cloud Composer／Astronomer。
- 建議：升級到 Airflow 2.x（TaskFlow API），DAG 與設定放進同一個 repo 並附範例 DAG。
- 建議：流程圖的 Hive 部分今天可換成 BigQuery 或 Iceberg 表，並加上資料品質檢查。
- 建議：`airflow.cfg` 中的帳密與 key 一律改用環境變數。
