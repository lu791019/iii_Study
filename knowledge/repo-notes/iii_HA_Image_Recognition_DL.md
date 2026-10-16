# iii_HA_Image_Recognition_DL

> 一句話：醫院專題的 CT 影像辨識模組，用 keras-frcnn（Faster R-CNN）做遷移學習，並用 Kafka 把上傳的影像送去辨識、結果回傳 Flask 網頁。

| 項目 | 內容 |
| --- | --- |
| 連結 | https://github.com/lu791019/iii_HA_Image_Recognition_DL |
| 年份 | 2019（最後 commit 2019-09-29） |
| 情境 | 資策會結業專題的子模組 |
| 技術 | Keras、TensorFlow、Faster R-CNN（fork 自開源 keras-frcnn）、遷移學習、Kafka producer／consumer、Flask、Windows `.bat` 批次執行 |

## 做了什麼

- 以開源 keras-frcnn 為底，用公開 CT 影像訓練（權重檔太大，放在雲端硬碟）。
- `Consumer_img.py` 常駐等 Kafka 訊息，`demo/producer.py` 上傳影像，`recognition.bat` 串起辨識與回傳。
- 訓練與測試步驟寫在 `Steps_for_training_model.txt`、`Steps_by_testing.txt`，另有一份 Word 說明。

## 可當教學案例的點

- **站在開源肩膀上**：轉職專題不必從零訓練模型，fork 成熟實作再換資料，是務實的選擇；但 README 要清楚標出哪些是自己做的。
- **模型 + 訊息佇列**：用 Kafka 把「上傳」和「辨識」解耦，是把 DL 接進系統的好例子。
- **反例**：`__pycache__`、大量原始套件程式一起 commit，只支援 Windows 批次檔（README 自述 shell 版 on going）。

## 如果今天重做會怎麼升級（建議）

- 建議：改用 PyTorch／Ultralytics YOLO 等現行框架，或直接用醫療影像預訓練模型微調。
- 建議：推論包成 FastAPI 服務放進 Docker，Kafka 只在真的需要非同步時才用。
- 建議：權重放 Hugging Face Hub 或 Git LFS，README 附下載指令。
- 建議：寫清楚評估方式（mAP、各類別 Recall）與資料授權，醫療影像要特別說明是公開資料。
