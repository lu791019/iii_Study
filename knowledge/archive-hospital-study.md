# archive：資策會醫院資料專題

`archive/iii_report_HospitalStudy/` 是 Dex 在資策會（iii）轉職培訓時的專題，與公開 repo [iii_DB102_Project_Hospital_assitant](https://github.com/lu791019/iii_DB102_Project_Hospital_assitant) 同一個專題。

## 內容

| 部分 | 做了什麼 | 主要技術 |
| --- | --- | --- |
| 資料整理 | 醫院檢查單（CT 等）時間欄位處理、欄位改名、Tool Summary 統計 | pandas、matplotlib、seaborn |
| 建模 | 以檢查開始與結束時間差（ETIME − BTIME）為目標的迴歸預測，比較多種模型與調參 | scikit-learn（Lasso、Ridge、SVR、RandomForest、GradientBoosting、AdaBoost）、GridSearchCV、vecstack stacking |
| 資料庫 | DataFrame 與 CSV 寫入 MySQL、Elasticsearch、MongoDB | SQLAlchemy、elasticsearch、pymongo |
| 爬蟲 | 爬取資料寫入 MongoDB | requests、BeautifulSoup |

## 可以怎麼用

- **轉職案例**：作為「講師本人當年的專題」，對照 [style-guide.md](style-guide.md) 的三級（作業／作品／MVP），示範一個專題怎麼從作業升級到 MVP。
- **教學反例**：Notebook 檔名與流程混亂、資料與程式混放、帳密與內網位址寫死在程式中，適合當「專題常見問題」的教材。
- **資料為假資料**：檔案中的病歷號、醫師代碼等為示範用假資料（依 Dex 說明）。

## 安全注意

- 2026-10-10 已將 Notebook 中的 MySQL 密碼替換為 `<PASSWORD>`、內網 IP 替換為 `<HOST>`。
- 舊版本仍留在 git 歷史中；若該密碼仍在其他地方使用，應更換，或清除 git 歷史。
