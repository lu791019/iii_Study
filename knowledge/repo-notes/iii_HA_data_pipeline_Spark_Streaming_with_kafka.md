# iii_HA_data_pipeline_Spark_Streaming_with_kafka

> 一句話：醫院專題的即時預測管線：病患資料經 Kafka 進 Spark Streaming，套用預先訓練的 scikit-learn 模型，算出儀器使用時間再送回 Kafka／HDFS。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/iii_HA_data_pipeline_Spark_Streaming_with_kafka |
| 年份 | 2019-09（模型檔名 0905、0910、20190911；最後 commit 2020-12-13） |
| 情境 | 資策會結業專題的子模組；README 自述「初次將 Spark 應用於實務」 |
| 技術 | HDFS（pyhdfs）、PySpark RDD、Spark SQL、Spark Streaming（DStream）、Kafka（KafkaUtils、KafkaProducer）、`sc.broadcast` 分發模型、scikit-learn／XGBoost pickle、MySQL、Cloudera 環境（推論，依 HDFS 路徑） |

## 做了什麼

README 把改寫過程寫成三步：

1. `hdfs_pre_DF.py`：從 HDFS 讀 CSV → pandas DataFrame ETL → 轉 RDD → 推論。README 自評：沒用到 Spark 的優勢，慢。
2. `hdfs_pre.py`：一開始就用 `textFile` 轉 RDD，整段在 RDD 上做，較快。
3. `SparkStreaming.py` → `PysparkStreaming_Kafka.py` → `Streaming_xgb.py`：先用 socket 當輸入練 DStream，再換成 Kafka，模型用 broadcast 傳給 worker，結果寫回 Kafka 與 HDFS。

README 自述 XGBoost 版 MAPE < 20%（未核實，估計值）。On-going：以 JDBC 存 SQL、存 metastore。

## 可當教學案例的點

- **「先跑通簡化版，再換一件事」的活教材**：pandas 版 → RDD 版 → socket Streaming → Kafka Streaming，每版只換一件事，完全符合 [style-guide.md](../style-guide.md) 的教學設計。
- **為什麼要分散式**：用作者自己的「DF 版慢」結論，講 driver 與 executor 的差別。
- **模型怎麼上管線**：broadcast pre-trained model 是很好懂的「批次訓練、串流推論」範例。
- **反例**：同一支程式在根目錄、`Final program/`、`Spark_stream_realtime/` 各有一份，版本不明。

## 如果今天重做會怎麼升級（建議）

- 建議：DStream 已被淘汰，改用 Structured Streaming（`readStream.format("kafka")`），或評估是否真的需要即時（多數排程情境用每 5 分鐘批次就夠）。
- 建議：模型用 MLflow 管理版本，推論改成 pandas UDF。
- 建議：本機環境用 Docker Compose（Kafka + Spark）取代 Cloudera VM，學員一鍵重現。
- 建議：只保留一份最終程式，舊版用 Git 歷史或 `docs/evolution.md` 說明演進。
