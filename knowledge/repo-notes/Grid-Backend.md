# Grid-Backend

> 一句話：智慧電網的雲端後端：訂閱邊緣端與氣象站的 MQTT 資料存進 InfluxDB，用 PostgreSQL 管設備設定，並提供查詢 API 與告警信。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Grid-Backend |
| 年份 | 2024-07～11（migration 時間戳 20240705 → 20241111；最後 commit 2024-11-15） |
| 情境 | 工作（推論：資料工程與後端時期的能源／智慧電網專案） |
| 技術 | TypeScript、Hono、MQTT、InfluxDB（時序資料）、PostgreSQL + Sequelize（migration）、node-cron（氣象資料排程）、nodemailer（告警信）、Azure IoT SDK、xlsx、winston、Jest + ts-jest + sequelize-mock、Docker |
| 相關 | [Grid-Edge](Grid-Edge.md)（邊緣端） |

## 做了什麼

- `initController` 啟動時訂閱 Edge 與氣象資料；`messageHandlerServices` 分 weather、device 兩類訊息處理後寫 InfluxDB。
- API：給 Edge 取設備訂閱設定、給使用者查 InfluxDB 的設備資料、抓取全部氣象資料。
- 15 支 migration 記錄 schema 演進：設備、設備設定、RBAC、流程、上傳、事件、告警設定、告警信。
- `test/` 有 MQTT 訊息處理、API controller、工具函式的測試。

## 可當教學案例的點

- **兩種資料庫的分工**：量測值（每秒或每分鐘進來）放時序庫，設定與權限放關聯式資料庫，是講 OLTP 與時序資料最具體的例子。
- **migration 就是 schema 的 Git**：從檔名時間戳就能讀出系統四個月怎麼長大，適合教「為什麼不手改資料表」。
- **有測試的後端**：用 mock 隔開資料庫寫單元測試，對照學員專題「沒有測試」的常態。

## 如果今天重做會怎麼升級（建議）

- 建議：加 CI（GitHub Actions 跑 `npm test` 與 lint），測試覆蓋率寫進 README。
- 建議：同時用 Sequelize 與 drizzle-orm（package.json 都有）容易混亂，選一個。
- 建議：告警規則抽成設定並加上去重與靜默時間，避免告警信洪水。
- 建議：用 Grafana 接 InfluxDB 做監控儀表板，README 附截圖。

## 注意事項

