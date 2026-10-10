# 轉職諮詢首談範本：Intake 問卷與差距分析

> 第一次諮詢用。前半段是訪談問卷，後半段是差距分析表，諮詢後一週內填完並和學員確認。
> 欄位標示：**【模型】** 取自 job-hunter-v2 的資料模型，可以直接請學員在工具裡填；**【建議】** 是轉職諮詢額外需要、模型裡沒有的欄位。
> 使用方式與後續流程見 [../playbooks/career-coaching.md](../playbooks/career-coaching.md)。

## 基本資料

| 項目 | 填寫 | 來源 |
| --- | --- | --- |
| 姓名 |  | 【模型】User.full_name |
| Email |  | 【模型】User.email |
| 居住地與可通勤範圍 |  | 【模型】UserProfile.location |
| 可接受的工作型態（到班／混合／遠端） |  | 【建議】 |
| LinkedIn |  | 【模型】UserProfile.linkedin_url |
| GitHub |  | 【模型】UserProfile.github_url |
| 個人網站或作品集網址 |  | 【模型】UserProfile.website_url |

> 電話等聯絡方式只存在約定的私人位置，不寫進共用文件。

## 一、定位與動機

| # | 問題 | 對應欄位 |
| --- | --- | --- |
| 1 | 用一句話介紹你自己：現在是誰、想轉去哪裡？ | 【模型】UserProfile.headline |
| 2 | 為什麼想轉職？是什麼事件讓你下定決心？ | 【模型】UserProfile.bio |
| 3 | 目標職位是什麼？有第二志願嗎？ | 【建議】PROJECT.md 有規劃「期望職位」，模型尚未實作 |
| 4 | 期望薪資範圍？最低可接受多少？ | 【建議】同上，規劃中的「薪資設定」 |
| 5 | 如果一年後轉職成功，你的一天長什麼樣子？ | 【建議】 |
| 6 | 家人或經濟上有什麼限制，會影響轉職時程？ | 【建議】 |

## 二、經歷盤點

每段工作填一列。【模型】Experience：company、title、location、start_date、end_date、is_current、description。

| 公司 | 職稱 | 期間 | 目前在職 | 主要工作內容 | 可轉移的經驗【建議】 |
| --- | --- | --- | --- | --- | --- |
|  |  |  | 是／否 |  |  |
|  |  |  | 是／否 |  |  |

追問清單【建議】：

- 有沒有處理過資料、報表、Excel 公式或自動化？規模多大？
- 有沒有改善過流程？改善前後差多少（時間、錯誤率、成本）？
- 有沒有跨部門溝通或帶人的經驗？
- 哪一段經驗最能說明「你為什麼適合新職位」？

## 三、學歷與進修

【模型】Education：school、degree、field_of_study、start_date、end_date、description。

| 學校或機構 | 學位或課程 | 科系或主題 | 期間 | 備註（專題、證照） |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |

補充【建議】：目前正在上的課程（例如資策會、TibaMe 班別）、預計結訓日。

## 四、技能自評

【模型】Skill：name、level（自由文字）。建議統一用下面四級，並要求每項附證據【建議】。

| 等級 | 定義 |
| --- | --- |
| 1 聽過 | 知道用途，沒有動手 |
| 2 跟著做 | 照教材能跑出結果 |
| 3 獨立做 | 能在自己的專題中使用並排錯 |
| 4 能解釋取捨 | 能說明為什麼選它、不選替代方案 |

| 技能 | 自評等級 | 證據（作品、連結、證照） |
| --- | --- | --- |
| Python |  |  |
| SQL |  |  |
| Docker |  |  |
| Git／GitHub |  |  |
| 排程與編排（Airflow 等） |  |  |
| 雲端（GCP 等） |  |  |
|  |  |  |

## 五、作品集

【模型】PortfolioItem：title、description、project_url、image_url、tags。

| 作品名稱 | 一句話說明 | 連結 | 技術標籤 | 目前等級【建議】 |
| --- | --- | --- | --- | --- |
|  |  |  |  | 作業／作品／MVP |
|  |  |  |  | 作業／作品／MVP |

等級定義見 [../style-guide.md](../style-guide.md) 的專題三級。

## 六、求職現況與資源

| 項目 | 填寫 | 來源 |
| --- | --- | --- |
| 已投遞或已收藏的職缺數 |  | 【模型】Job.status（saved、applied） |
| 進入面試的次數 |  | 【模型】Job.status（interviewing） |
| 收到的 offer 或拒絕 |  | 【模型】Job.status（offer、rejected） |
| 每週可投入的學習時數 |  | 【建議】 |
| 希望在什麼時候開始新工作 |  | 【建議】 |
| 曾經參加過的面試題型與卡住的地方 |  | 【建議】 |

## 差距分析表（諮詢後由顧問填寫）

### 1. 目標職缺需求整理

收集 3–5 個目標職缺。【模型】Job：title、company、location、description、requirements、url、source。

| 職缺 | 公司 | 連結 | 必備技能 | 加分技能 |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |
|  |  |  |  |  |

### 2. 技能差距

把上表出現兩次以上的技能列進來。【建議】job-hunter-v2 規劃中的「匹配分析」會自動做這一步，目前先人工對照。

| 技能 | 出現次數 | 現況（有證據／有但沒證據／沒有） | 補法 | 預計完成 |
| --- | --- | --- | --- | --- |
|  |  |  |  |  |
|  |  |  |  |  |

### 3. 說服力差距

| 面試官可能的疑問 | 目前的回答素材 | 要補什麼 |
| --- | --- | --- |
| 為什麼轉職？ |  |  |
| 沒有相關經驗，憑什麼能勝任？ |  |  |
| 你的作品和其他學員有什麼不同？ |  |  |
| 前一份工作學到什麼可以帶過來？ |  |  |

### 4. 首談結論

| 項目 | 內容 |
| --- | --- |
| 目標職位（第一、第二志願） |  |
| 最大的三個差距 |  |
| 下次諮詢前要完成的事（最多三項） |  |
| 下次諮詢日期 |  |
| 第一次模擬面試的題庫類別（general／technical／leadership） |  |

## 來源

- 個人檔案模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/profile/models.py
- 作品集模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/portfolio/models.py
- 職缺模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/jobs/models.py
- 使用者模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/auth/models.py
- 期望職位與薪資（規劃中）、匹配分析（規劃中）：https://github.com/lu791019/job-hunter-v2/blob/main/PROJECT.md
- 技能等級對應的課程內容：https://github.com/lu791019/stock-crawler-de-course-materials
