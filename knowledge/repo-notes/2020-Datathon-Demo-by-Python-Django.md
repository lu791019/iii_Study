# 2020-Datathon-Demo-by-Python-Django

> 一句話：2020 Datathon 的 Demo 網站：用發票與天氣資料做 RFM 分析，依天氣推薦適合推播飲品的會員，包成 Django 儀表板。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/2020-Datathon-Demo-by-Python-Django |
| 年份 | 2020 下半（套件版本為 2020 年 9～10 月；最後 commit 2024-01-03，為整理上傳日） |
| 情境 | 比賽（主辦單位與名次 repo 未註明） |
| 技術 | Django 2.1、AppSeed DattaAble 開源後台模板、pyodbc／django-pyodbc-azure（Azure SQL）、pandas、seaborn、Docker、gunicorn、nginx、Heroku Procfile |

## 做了什麼

- `Analysis/`：發票、天氣資料前處理與 RFM 分析 Notebook（README：Data Preprocessing and Data Analyzing to RFM result）。
- `datathon/demo_views.py`：使用者選商品與天氣（晴／雨），從對應的 RFM 表撈會員依消費頻率排序，回傳推薦名單。
- 後台 UI 用開源模板，部署檔（Dockerfile、docker-compose、gunicorn、nginx）一應俱全。

## 可當教學案例的點

- **比賽也能交「產品」**：多數隊伍交簡報與 Notebook，做成可點的網頁 Demo 更容易讓評審記住。
- **用模板省時間**：UI 用現成模板，力氣放在資料與推薦邏輯，對轉職者是正確的取捨。
- **RFM 是最好教的分群**：Recency、Frequency、Monetary 三個欄位就能講清楚會員分群。

## 如果今天重做會怎麼升級（建議）

- 建議：Demo 改用 Streamlit 或 FastAPI + 簡單前端，部署到免費平台，README 放線上連結與截圖。
- 建議：「天氣影響飲品消費」先用統計檢定或簡單模型驗證，再做推薦，簡報才講得出「為什麼」。
- 建議：README 換成自己的內容（目前仍是模板原文），寫題目、資料、做法、架構圖。
- 建議：Notebook 檔名用主題命名而不是組員名字。
