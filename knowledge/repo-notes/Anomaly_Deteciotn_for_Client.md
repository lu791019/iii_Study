# Anomaly_Deteciotn_for_Client

> 一句話：客戶合約的異常（風險）偵測：從 SQL Server 撈資料、做編碼與特徵，產出每日風險名單。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Anomaly_Deteciotn_for_Client |
| 年份 | 2020-01～02（Notebook 檔名 0117、0122；結果資料夾 2020-02-21；最後 commit 2020-03-09） |
| 情境 | 工作（推論：線上教育公司 Big Data Engineer 時期，程式中出現該產業的品牌代號） |
| 技術 | pandas、SQLAlchemy（SQL Server）、category_encoders TargetEncoder、scikit-learn One-Class SVM、SMOTE + SVC（舊版腳本）、joblib、logging、ConfigParser |

## 做了什麼

- 多份 Notebook：資料整合、前處理、分段特徵、TargetEncoder、模型訓練（含百萬筆資料版本）。
- `model_package/` 存 One-Class SVM 模型；`Final_Script_py_and_result/Karma.py` 是可執行腳本，讀 CSV（SQL 查詢結果），輸出風險合約清單。
- README 收錄一支舊版正式腳本：分 DEV／STG／PROD／LOCAL、每日 log、讀設定檔、多品牌各自一個 SMOTE + SVC 模型（推論：作者接手並延伸既有的 Python 2 腳本）。

## 可當教學案例的點

- **Notebook 到正式腳本的落差**：同一個 repo 同時有探索用 Notebook 和要每天跑的腳本，適合講「環境切換、log、設定檔」這些作業不會教的事。
- **接手舊系統**：Python 2 語法（`except Exception, e`）與新 Notebook 並存，是轉職後第一份工作常見的真實情境。
- **非監督異常偵測**：沒有足夠標籤時用 One-Class SVM 找出「不像正常客戶」的合約。

## 如果今天重做會怎麼升級（建議）

- 建議：先跟業務定義「異常」並收集回饋標籤，再比較 IsolationForest、One-Class SVM 與監督式模型。
- 建議：腳本改成 Airflow DAG，連線資訊放環境變數，結果寫回資料庫而不是 CSV。
- 建議：輸出加上「為什麼被判異常」（特徵貢獻），讓業務能採取行動。
- 建議：工作專案公開前先去識別化，品牌代號、資料庫名稱、結果檔都要拿掉。

## 注意事項

