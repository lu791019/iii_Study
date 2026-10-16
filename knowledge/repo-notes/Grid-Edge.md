# Grid-Edge

> 一句話：智慧電網的邊緣端程式：依設定定時用 Modbus、SNMP、API 讀取設備資料，經 MQTT 送上雲端，並接收雲端下發的 Modbus 控制指令。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Grid-Edge |
| 年份 | 2024（最後 commit 2024-11-15，搭配 Grid-Backend 同日上傳） |
| 情境 | 工作（推論：資料工程與後端時期的能源／智慧電網專案） |
| 技術 | TypeScript、Node.js、Hono、modbus-serial、net-snmp、MQTT、BullMQ + Redis（Modbus 讀寫佇列）、PouchDB（本地設定）、node-cron、winston（每日輪替 log）、Docker、私有 registry |
| 相關 | [Grid-Backend](Grid-Backend.md)（雲端端） |

## 做了什麼

README 用 Mermaid 畫了架構，流程：

```
啟動 → 向伺服器取設備設定 → 存進 PouchDB → 依頻率建立 API／Modbus／SNMP 排程 → 讀值發布到 MQTT
雲端控制指令（MQTT control/modbus）→ 訊息處理 → Modbus 佇列（read／write）→ 結果再發布到 MQTT
```

## 可當教學案例的點

- **邊緣運算的三個關鍵**：設定從雲端拉、存在本地（斷網也能跑）、資料用輕量協定（MQTT）上傳。
- **為什麼寫入要排隊**：Modbus 設備一次只能處理一個請求，用 BullMQ 佇列序列化讀寫，是講「佇列解決什麼問題」的實例。
- **工控協定入門**：Modbus（暫存器讀寫）、SNMP（網路設備監控）與一般 HTTP API 並存，適合製造或能源背景的轉職者。
- **README 寫法**：Mermaid 架構圖 + 每個資料夾一段說明，是值得學員模仿的 README。

## 如果今天重做會怎麼升級（建議）

- 建議：加單元測試（目前只有 Grid-Backend 有），特別是資料轉換與佇列重試。
- 建議：設備設定加 schema 驗證（Zod），錯誤設定在啟動時就擋下。
- 建議：加上斷線時的本地緩衝與補送機制，並在 README 說明遺失資料的處理策略。
- 建議：`dist/`、本地資料庫檔案不進版控。

## 注意事項

