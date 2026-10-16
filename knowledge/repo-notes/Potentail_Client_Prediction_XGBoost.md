# Potentail_Client_Prediction_XGBoost

> 一句話：潛在客戶預測的 XGBoost 二元分類，特徵選擇、編碼與資料切分都由 XML 設定檔控制。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Potentail_Client_Prediction_XGBoost |
| 年份 | 2020（最後 commit 2020-08-09） |
| 情境 | 工作（推論：線上教育公司 Big Data Engineer 時期） |
| 技術 | XGBoost（DMatrix）、xgbfir（特徵交互作用分析）、imbalanced-learn RandomUnderSampler、SQLAlchemy、XML 設定（ElementTree）、logging、Graphviz |

## 做了什麼

- `utils.py`：建資料庫連線、讀 SQL 檔撈資料、補缺值、取 log、依設定檔選欄位、修正欄位、one-hot、依日期切訓練與測試集。
- `model.py`：`XGBModeling` 類別，讀 XML 設定 → 轉 DMatrix → 訓練 → 特徵重要性與交互作用。
- 三份 XML：`FixColConfig.xml`、`onehotconfigfixed.xml`、`DataIntegratefix.xml`。

## 可當教學案例的點

- **設定檔驅動**：換一批特徵不用改程式，只改設定；這是從「專題」走向「可維護的工作程式」的關鍵一步。
- **依日期切分**：用時間切訓練與測試，比隨機切更接近上線後的真實情況。
- **類別包裝**：把流程拆成 `utils` 與 `model` 兩層，適合講函式到類別的重構。

## 如果今天重做會怎麼升級（建議）

- 建議：XML 改成 YAML 或 Pydantic 設定，並加 schema 驗證。
- 建議：用 Optuna 調參、MLflow 記錄實驗，SHAP 解釋單筆預測給業務看。
- 建議：加 README：題目、目標定義（多久內成交算正樣本）、評估指標（AUC、Precision@K）與執行指令。
- 建議：Windows 寫死的 Graphviz 路徑改成可設定。
