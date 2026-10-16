# iii_HA_linechatbot

> 一句話：醫院專題的病患端 LINE 聊天機器人，可以查預約檢查、依身體部位建議掛號科別，並推薦健康文章。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/iii_HA_linechatbot |
| 年份 | 2019（最後 commit 2019-09-29） |
| 情境 | 資策會結業專題的子模組 |
| 技術 | Flask、line-bot-sdk（Buttons／Confirm Template、圖文選單）、jieba、MariaDB、Docker Compose（db + jupyter + serveo 三個服務）、serveo／autossh tunnel |

## 做了什麼

- `code/app.py`（約 600 行）：webhook 接收訊息；病患綁定編碼後查詢預約檢查並提醒檢查前注意事項；點選身體部位回覆建議科別；內含協同過濾推薦邏輯。
- `docker-compose.yml`：MariaDB 存資料、Jupyter 容器跑 Flask、serveo 容器把本機 port 開到公網給 LINE webhook。README 只有一句「docker-compose up 即可」。

## 可當教學案例的點

- **最短的「結果送到使用者手上」路徑**：模型或資料做完，用 LINE Bot 推播，是轉職專題最容易 Demo 的出口。
- **規則式 Bot 對照 LLM Bot**：2019 年用字典比對回覆科別；可以讓學員想一想「今天用 LLM 會好在哪、風險在哪（醫療建議）」。

## 如果今天重做會怎麼升級（建議）

- 建議：tunnel 改用 ngrok 或直接部署到 Cloud Run 等平台，webhook 有固定網址。
- 建議：把推薦、查預約拆成獨立 API，Bot 只負責對話。
- 建議：科別建議若改用 LLM，要加免責聲明、限制回答範圍，並記錄對話以便檢查。
