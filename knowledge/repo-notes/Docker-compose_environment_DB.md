# Docker-compose_environment_DB

> 一句話：資策會大數據班用的 Docker Compose 環境集合，一份檔案起 ELK、Kafka、Hadoop、Spark、MySQL、MongoDB。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Docker-compose_environment_DB |
| 年份 | 2019（最後 commit 2019-09-29） |
| 情境 | 資策會課程與專題環境（推論：部分 image 為課程提供） |
| 技術 | Docker Compose；Elasticsearch／Kibana（6.2.4、6.7.2、7.1.1 三種版本）、Confluent Kafka + Zookeeper、jupyter/all-spark-notebook、MySQL 8、MongoDB 4.1、Hadoop master／slave、bde2020 Hadoop + Spark + Hue |

## 做了什麼

6 份 compose 檔：

| 檔案 | 內容 |
| --- | --- |
| `docker-compose_iii_classall.yml` | 課程全套：ES、Kibana、Kafka、Jupyter、MySQL、Hadoop、MongoDB |
| `docker-compose_orign.yml`、`docker-compose.yml` | 同上的不同版本組合 |
| `docker-compose_HDP_cluster.yml` | Hadoop 叢集 + Kafka + Jupyter |
| `docker-compose_HDP_Spark_cluster.yml` | namenode、datanode、spark-master、spark-worker、notebook、Hue |
| `docker-compose_AIclothes.yml` | 另一個專題的環境（Flask、Redis、人臉與文字服務、Kafka、Hadoop）（推論：同班其他組或合作專題） |

## 可當教學案例的點

- **環境即程式碼**：用 compose 檔記錄「這個專題需要哪些服務」，是 [../playbooks/course-design.md](../playbooks/course-design.md) 環境建置教學的前身。
- **版本地獄**：同一 repo 有三種 Elasticsearch 版本，適合講「為什麼要固定版本、寫在 README」。

## 如果今天重做會怎麼升級（建議）

- 建議：每份 compose 檔附 README：用途、需要多少記憶體、啟動與驗證指令。
- 建議：密碼改用 `.env` 檔與 `env_file`，`.env.example` 進版控。
- 建議：加 `healthcheck` 與 `depends_on: condition: service_healthy`，Kafka 改用不需 Zookeeper 的 KRaft 模式。
- 建議：按需求拆 profile（`docker compose --profile kafka up`），學員不必一次起十幾個容器。
