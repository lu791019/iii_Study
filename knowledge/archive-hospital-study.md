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
- **不要引用原始資料**：資料檔含疑似真實病患紀錄。

## 安全注意

- `iii_project_ML_predict/Step 1_df and csv can export in Mysql in sure.ipynb` 含一組 MySQL 帳號密碼。
- 多個 Notebook 寫死內網 IP（Elasticsearch、MongoDB）。
- 本 repo 為公開 repo；建議把 archive 移出公開 repo，或把 repo 改為私人並清除 git 歷史。若該密碼仍在其他地方使用，應更換。
