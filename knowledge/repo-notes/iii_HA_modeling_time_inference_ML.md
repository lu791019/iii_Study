# iii_HA_modeling_time_inference_ML

> 一句話：醫院專題的「檢查時間預測」建模紀錄，用 XGBoost、Bagging、Stacking 搭配兩種類別編碼做比較。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/iii_HA_modeling_time_inference_ML |
| 年份 | 2019（最後 commit 2019-09-29） |
| 情境 | 資策會結業專題的子模組 |
| 技術 | Python：pandas、XGBoost、GridSearchCV、BaggingRegressor、LinearRegression；R：3-fold Stacking（第一層多元迴歸） |

## 做了什麼

`time_predict/` 下 5 份 Word 檔，內容是程式碼：

- `XGBOOST.docx`：XGBoost + GridSearchCV。
- `bagging_one_hot_encoding.docx`、`bagging_mean_encoding.docx`：Bagging，比較 One-Hot 與 Mean Encoding。
- `stacking_one_hot_encoding.docx`、`stacking_mean_encoding.docx`：用 R 手刻 Stacking（訓練集切 3 份、留 20% 測試）。
- 目標是檢查開始到結束的時間差，先排除極端值（只留 1 到 90 分鐘之間的紀錄）。

## 可當教學案例的點

- **同一題做系統化比較**：模型 × 編碼方式的 2 × 2 實驗設計，適合教「實驗要一次只換一件事」。
- **手刻 Stacking**：用 R 把 K-fold 產生 meta feature 的邏輯攤開來，比直接呼叫套件更好理解。
- **反例**：程式碼存成 .docx，無法執行、無法 diff，是「為什麼要用 Git 管程式」最直觀的例子。

## 如果今天重做會怎麼升級（建議）

- 建議：程式改成 `.py` 或 Notebook，實驗結果用一張表記錄（模型、編碼、MAE／MAPE、訓練時間）。
- 建議：用 scikit-learn 的 `Pipeline` + `StackingRegressor`，或 MLflow 記錄每次實驗。
- 建議：極端值門檻寫成設定，並說明為什麼是 1 到 90 分鐘（業務規則還是資料分布）。
- 建議：加上「用平均值預測」的 baseline，才看得出模型到底省了多少誤差。
