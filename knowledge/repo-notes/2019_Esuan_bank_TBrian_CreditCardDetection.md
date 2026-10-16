# 2019_Esuan_bank_TBrian_CreditCardDetection

> 一句話：玉山銀行 × T-Brain 2019 信用卡盜刷偵測競賽的分析筆記與 Notebook。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/2019_Esuan_bank_TBrian_CreditCardDetection |
| 年份 | 2019 秋（Notebook 檔名 0925；最後 commit 2020-03-08） |
| 情境 | 比賽（推論：團隊參賽，README 提到隊友建議；時間在資策會結訓後、第一份資料工作前） |
| 技術 | pandas、視覺化、衍生變數、IsolationForest、RandomForest、DecisionTree、LightGBM、F1 score、cross validation |

## 做了什麼

- 題目：用前 90 天的授權交易紀錄，預測第 91～120 天的每筆交易是否為盜刷。
- README 是一份協作筆記：逐欄分析、長條圖、盜刷率分析、衍生變數想法、各演算法的程式與結果。
- README 自述 IsolationForest 的 F1 約 0.5（未核實，估計值），之後嘗試監督式模型與 LightGBM。

## 可當教學案例的點

- **不平衡資料的評估指標**：盜刷占比很低，accuracy 會騙人，這題是講 F1、Precision、Recall 的最佳例子。
- **欄位分析表**：「欄位 ／ 說明 ／ 特性 ／ 分析」的表格寫法，可以直接給學員當 EDA 範本。
- **非監督 vs 監督**：先用 IsolationForest 不看標籤找異常，再用有標籤的模型，適合講兩者差別。

## 如果今天重做會怎麼升級（建議）

- 建議：README 改成「題目 → 做法 → 最佳分數 → 學到什麼」，筆記移到 `docs/`。
- 建議：用時間切分驗證（前 60 天訓練、後 30 天驗證），避免隨機切分造成資訊洩漏。
- 建議：加入以卡號或使用者為單位的聚合特徵（過去 N 筆的金額、頻率），並記錄每個特徵的貢獻。

## 注意事項

- README 提到其他隊友的名字，引用時不要轉貼。
