# Sails-backend

> 一句話：能源場域的資料接收後端（aisails）：接儲能系統（ESS）、Skysails 風機、光達風場、雷擊、氣象等設備資料，存進 InfluxDB 與 PostgreSQL。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Sails-backend |
| 年份 | 2024-01～04（migration 時間戳 20240105 → 20240425；最後 commit 2024-11-15） |
| 情境 | 工作（推論：資料工程與後端時期的能源專案；README 提到光達資料來自與研究機構的合作） |
| 技術 | Node.js、Express、JWT（express-jwt）、Swagger、MQTT、promise-ftp、csv-parser、InfluxDB、PostgreSQL + Sequelize migration、node-cron、winston、Jest + supertest、Docker |

## 做了什麼

- ESS 資料用 REST API 推進來；Skysails 風機資料量大，改用 MQTT 訂閱（README 自述每秒約 30 筆，估計值）。
- 光達：從 FTP 下載廠商特規的 ZPH 檔，用廠商提供的轉檔程式包成 Docker image 轉成 CSV 再入庫（README 註明只支援 amd64）。
- 其他控制器：雷擊、氣象與氣象風險、柴油發電機、PCS、通知、使用者與 JWT。
- `Test/` 下各服務都有 Jest 測試，README 說明 ES Module 環境下怎麼 mock。

## 可當教學案例的點

- **依資料量選協定**：低頻用 REST、高頻用 MQTT，README 有寫原因，可以當「為什麼選 A 不選 B」的面試講法範例。
- **髒活也是工作**：廠商特規格式、只能用廠商 exe 轉檔，用 Docker 包起來，是真實資料工程最常見的情境。
- **README 的「特別說明」段**：把踩過的坑（ID 用完要清理、ES6 mock 寫法）寫下來，是值得學員學的習慣。

## 如果今天重做會怎麼升級（建議）

- 建議：高頻單筆寫入改成批次寫入（buffer 後每秒一批），減少資料庫壓力與 ID 消耗。
- 建議：各設備的接收邏輯抽成共用介面（adapter pattern），新增設備只加一個檔案。
- 建議：Swagger 文件與實際路由用程式產生，避免不同步。
- 建議：加 CI 與健康檢查端點，部署後能自動驗證。
