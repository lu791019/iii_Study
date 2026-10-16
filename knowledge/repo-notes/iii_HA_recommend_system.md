# iii_HA_recommend_system

> 一句話：醫院專題的健康文章推薦模組：爬健康網站文章、中文斷詞分群，再依使用者點擊紀錄做協同過濾推薦。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/iii_HA_recommend_system |
| 年份 | 2019-08～09（Notebook 檔名 20190812、20190902、20190908；最後 commit 2020-03-31） |
| 情境 | 資策會結業專題的子模組 |
| 技術 | requests、BeautifulSoup、`concurrent.futures` ThreadPool、logging、jieba、scikit-learn（K-Means、Naive Bayes）、Keras（embedding 情緒分析）、MySQL（pymysql）、MongoDB |

## 做了什麼

- `recommend_system/ThreadPool_crawler_final.py`：ThreadPool 併發爬文章，有 log 檔與計時 decorator。
- `jieba_kmeans_naivebayes_*.ipynb`、`kmeans_*.ipynb`：斷詞後分群與分類，替文章貼 tag。
- `recommend_system/recommend_CollaborativeFiltering_final.py`：從 MySQL 讀點擊紀錄，依點擊新舊加權（最新 3 篇 × 1、次新 3 篇 × 0.5、再 3 篇 × 0.2），計算 tag 之間的相似度後推薦。
- `sentiment_analysis_embedding_*.ipynb`：用 embedding 做情緒分析的實驗。

## 可當教學案例的點

- **完整的推薦鏈**：爬蟲 → 斷詞 → 分群貼標 → 行為紀錄 → 相似度 → 推薦，一個 repo 就看得到資料怎麼一路變成功能。
- **爬蟲從一支腳本到併發**：ThreadPool + logging + decorator，是 [../topic-briefs/crawl.md](../topic-briefs/crawl.md) 「單支腳本 → 併發」的好範例。
- **時間衰減加權**：用「越新的點擊越重要」講推薦系統的業務直覺。

## 如果今天重做會怎麼升級（建議）

- 建議：文章 tag 改用 embedding 模型或 LLM 自動分類，分群結果再人工抽查。
- 建議：推薦邏輯包成 API（FastAPI），讓 LINE Bot 呼叫，而不是把推薦程式複製進 Bot。
- 建議：爬蟲加上 robots.txt 檢查、每秒請求上限與重試，結果寫入時做 upsert。
- 建議：加評估指標（點擊率、Precision@K），並保留「熱門文章」當 baseline。

## 注意事項

- `temp/` 下有點擊紀錄 CSV，給學員看時只展示欄位結構。
