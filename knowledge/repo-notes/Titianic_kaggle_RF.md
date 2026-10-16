# Titianic_kaggle_RF

> 一句話：Kaggle 鐵達尼號存活預測的隨機森林練習。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/Titianic_kaggle_RF |
| 年份 | 2019（最後 commit 2019-09-22） |
| 情境 | 自學（Kaggle 入門） |
| 技術 | pandas、numpy、scikit-learn（RandomForestClassifier、LabelEncoder、OneHotEncoder、Imputer） |

## 做了什麼

兩份 Notebook（demo 與 final）加一支整理好的 `.py`，從讀資料、補缺值、編碼到隨機森林預測。README 只有標題。

## 可當教學案例的點

- **「學習歷程」可以放上 GitHub**：轉職初期的練習 repo 是時間軸的證據，不必完美。
- **Notebook → `.py` 的第一步**：作者已經把 Notebook 整理成一支腳本，可以當「從探索到可重跑」的入門示範。
- **想一想**：`Imputer` 已從 scikit-learn 移除，請學員找出替代（`SimpleImputer`），練習讀錯誤訊息與文件。

## 如果今天重做會怎麼升級（建議）

- 建議：用 `ColumnTransformer` + `Pipeline` 包前處理，避免訓練與測試前處理不一致。
- 建議：README 寫 Kaggle 分數、特徵工程想法與下一步，符合 [templates/project-plan.md](../templates/project-plan.md) 的章節。
- 建議：入門題的下一步可接 [docs-index.md](../docs-index.md) 的「機器學習流程」文件推薦的中階題（Bank Churn 等）。
