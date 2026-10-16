# iii_DB102_Project_Hospital_assitant

> 一句話：資策會大數據班結業專題「Hospital Assistant」的總 repo，用醫院檢查資料預測檢查時間、用 LINE Bot 推播，加上 CT 影像辨識與健康文章推薦。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/iii_DB102_Project_Hospital_assitant |
| 年份 | 2019（最後 commit 2019-08-10；資料檔名有 0715） |
| 情境 | 資策會 × 緯育 TibaMe「大數據與資料分析實體班」結業專題，6 人小組 |
| 技術 | pandas、matplotlib、seaborn、MySQL、Elasticsearch、MongoDB、SQLAlchemy、scikit-learn、XGBoost、vecstack、Keras、LINE Bot（Flask）、jieba |
| 相關 | 本 repo 的 [archive-hospital-study.md](../archive-hospital-study.md)（同一專題的 Notebook 封存）；子 repo：[iii_HA_modeling_time_inference_ML](iii_HA_modeling_time_inference_ML.md)、[iii_HA_recommend_system](iii_HA_recommend_system.md)、[iii_HA_linechatbot](iii_HA_linechatbot.md)、[iii_HA_Image_Recognition_DL](iii_HA_Image_Recognition_DL.md)、[iii_HA_data_pipeline_Spark_Streaming_with_kafka](iii_HA_data_pipeline_Spark_Streaming_with_kafka.md) |

## 做了什麼

README 用 Why／Who／Where／What 寫提案，三個重點：

1. **醫療檢查時間分析**：預測病患檢查時間，用來優化排程、減少機台空檔。
2. **文字辨識與醫療保健推薦**：LINE Chatbot 判讀病患輸入，推薦相關健康資訊。
3. **CT 影像初步判斷**：用公開 CT 影像做深度學習辨識。

資料夾依技術分：`ETL by PYTHON`、`Database(Mysql ELS Mongodb)`、`Machine_leraning_Preduct_utime`、`Deep_learning_ Image recognition__by_CT`、`Charbot_Crawlering_recommender systems`。

## 可當教學案例的點

- **轉職者第一個專題的真實樣貌**：題目從痛點出發（排程滿檔但機台使用率不平均），這點做得很好；可對照 [style-guide.md](../style-guide.md) 的「作業 → 作品 → MVP」三級，讓學員自己評這個專題在哪一級。
- **README 的 5W 提案法**：Why／Who／Where／What 可以直接給學員當專題提案骨架。
- **專題常見問題反例**：資料夾名有錯字與空格、Notebook 的 `-checkpoint` 檔被 commit、資料與程式混放、聊天機器人資料夾的 README 是複製來的教學文。

## 如果今天重做會怎麼升級（建議）

- 建議：README 第一屏放架構圖與一行箭頭流程（`醫院 CSV → pandas → MySQL → 模型 → LINE 推播`），再接各子模組連結。
- 建議：把五個子 repo 收成 monorepo，每個模組有自己的 README 與執行指令。
- 建議：資料改用公開資料集或明確標示的假資料，並在 README 寫清楚來源。
- 建議：模型結果寫明評估指標與 baseline（例如「平均檢查時間」當 baseline），讓預測的價值可以被量化。

## 注意事項

- `Resources/` 下有醫院檢查紀錄格式的資料檔；依 Dex 說明為假資料，但格式像真實紀錄，給學員看時建議只看欄位結構、不展示內容。
- README 列有組員姓名，引用時不要轉貼。
