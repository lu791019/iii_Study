# 教學手冊：轉職諮詢與面試輔導（取材自 job-hunter-v2）

> 這份文件讓之後協助 Dex 做轉職諮詢的 AI 或助教，知道 job-hunter-v2 這個工具收集什麼資料、怎麼追蹤職缺、怎麼練行為面試、題庫有哪些題，以及 AI 回饋規劃成什麼樣子。最後一節提出一套諮詢流程。
> 整理自 `job-hunter-v2`（2026-10-10 讀取）。標示規則：**【repo】** 是 repo 裡實際有的內容；**【建議】** 是本文件根據 repo 推導出的提案，repo 裡沒有。

## 工具定位【repo】

job-hunter-v2 是 AI 求職助手 SaaS 的雛形，目標用戶與痛點如下（PROJECT.md）：

| 用戶類型 | 需求痛點 |
| --- | --- |
| 應屆畢業生 | 無經驗、不會寫履歷、面試緊張 |
| 轉職者 | 需要重新包裝技能、跨領域說服力 |
| 新鮮人 | 優化履歷、準備行為面試 |

三個核心功能：AI 客製化履歷、行為面試錄影練習＋AI 評估、職缺匹配分析。

## 實作現況：哪些真的能用【repo】

諮詢時不要把「規劃中」的功能當成現有功能介紹給學員。

| 功能 | 後端 | 前端 | 說明 |
| --- | --- | --- | --- |
| 註冊登入（JWT） | 完成 | 完成 | Email＋密碼 |
| 個人檔案（學歷、經歷、技能） | 完成 | 完成 | `/profile` |
| 作品集（檔案、連結） | 完成 | 有頁面 | `PortfolioPage` |
| 職缺 CRUD | 完成 | 無專屬頁面 | 只能經 API 新增 |
| 題庫模式面試練習 | 完成 | 完成 | 30 題、三類 |
| JD 模式面試練習 | 完成（LLM 出題） | 按鈕停用，標「coming soon」 | 需要職缺有 description |
| 錄影與 48 小時回放 | 完成 | 完成 | 歷史頁可回看 |
| 職缺 URL 解析、匹配分析 | 未做 | 未做 | Phase 3 規劃 |
| AI 履歷產生 | 未做 | 未做 | Phase 4 規劃 |
| AI 面試評估（語音轉文字、STAR） | 未做 | 未做 | Phase 7 規劃 |
| 付費（Stripe、綠界） | 未做 | 未做 | Phase 6 規劃 |

## 個人檔案：資料模型【repo】

| 模型 | 欄位 | 備註 |
| --- | --- | --- |
| User | email、full_name、provider、picture | provider 預留 Google 登入 |
| UserProfile | headline、bio、phone、location、linkedin_url、github_url、website_url | 每人一筆 |
| Education | school（必填）、degree、field_of_study、start_date、end_date、description | 多筆 |
| Experience | company、title（必填）、location、start_date、end_date、is_current、description | 多筆 |
| Skill | name（必填）、level | level 為自由文字，例如 Expert、Intermediate |
| PortfolioItem | title（必填）、description、project_url、image_url、tags（JSON） | 多筆 |

PROJECT.md 的 US-1.3 另外列了「期望職位／薪資設定」，但模型裡還沒有這兩個欄位。

### 對轉職者特別值得收集的資訊【建議】

repo 的欄位是一般求職者的通用版。轉職諮詢需要多問「從哪裡來、要去哪裡、為什麼」：

| 資訊 | 對應 repo 欄位 | 為什麼對轉職者重要 |
| --- | --- | --- |
| 一句話定位 | headline | 檢查能否講清楚「我是誰、要轉去哪」 |
| 轉職動機與故事 | bio | 面試必問「為什麼轉職」，要先有版本 |
| 前職的可轉移經驗 | Experience.description | 要從舊工作找出數據、流程、溝通類的素材 |
| 是否在職 | Experience.is_current | 決定每週可投入時數與求職節奏 |
| 技能與自評等級 | Skill.name、level | 自評常偏高或偏低，要用作品驗證 |
| 作品與 GitHub | PortfolioItem、github_url | 轉職者最主要的說服力來源 |
| 目標職位與薪資 | 規劃中，尚無欄位 | 沒有目標就做不了差距分析 |
| 每週可投入時數、預計求職時間 | 無 | 決定計畫長短（style-guide 預設短版 1–1.5 月、完整版 2–2.5 月） |
| 地點與工作型態 | location | 遠端、混合或通勤範圍會影響職缺池 |

## 職缺追蹤【repo】

Job 模型欄位：title、company、location、description、requirements、url、source、status，以及 created_at、updated_at。

狀態值寫在模型註解裡，預設 `saved`：

```
saved → applied → interviewing → offer
                             └→ rejected
```

注意事項：

- 狀態只寫在註解，後端沒有驗證，也沒有記錄狀態變更的時間點。
- 職缺可以關聯到面試練習（`InterviewSession.job_id`），這是 JD 模式出題的來源。
- 規劃中的 Phase 3：貼上 104、LinkedIn、Indeed 網址由 AI 解析，並顯示匹配百分比與「符合／缺少的技能」。

## 行為面試練習流程【repo】

| 步驟 | 內容 | 設定值 |
| --- | --- | --- |
| 1. 選模式 | 題庫模式，或 JD 模式（依職缺描述由 LLM 出題） | `question_bank`、`jd_based` |
| 2. 選類別（題庫模式） | general → behavioral、technical → technical、leadership → situational | 不選就從全部 30 題抽 |
| 3. 抽題 | 隨機抽 5 題，同一場不重複出題 | `MAX_QUESTIONS = 5` |
| 4. 讀題 | 全螢幕攝影機畫面，題目以玻璃卡片疊在上面，倒數後淡出 | 10 秒 |
| 5. 錄影 | 手動按開始；不可跳題、不可先看下一題 | 每題最長 5 分鐘 |
| 6. 預覽 | 回放後選擇重錄或提交 | WebM 格式 |
| 7. 結束 | 場次標為 completed 或 abandoned；離開頁面有確認提示 | 歷史頁列出所有場次 |
| 8. 回看 | 48 小時內可回放；過期回傳 403 | `is_permanent` 預留給付費永久保存 |

這套設計模擬真實面試的壓力：讀題時間短、必須照順序、不能重看題目再準備。

## 題庫（30 題，三類各 10 題）【repo】

原文為英文，中文為本文件的意譯。

### General／Behavioral（一般行為題）

| # | 原文 | 中文意譯 |
| --- | --- | --- |
| 1 | Tell me about a time you faced a significant challenge at work. How did you handle it? | 工作上遇過的重大挑戰，你怎麼處理 |
| 2 | Describe a situation where you had to work with a difficult team member. | 和難相處的隊友共事的經驗 |
| 3 | Give an example of a goal you set and how you achieved it. | 你設定過的目標，以及怎麼達成 |
| 4 | Tell me about a time you failed. What did you learn? | 一次失敗經驗與學到的事 |
| 5 | Describe a situation where you had to adapt to a major change. | 必須適應重大改變的情境 |
| 6 | Tell me about a time you went above and beyond your responsibilities. | 做到超出職責範圍的經驗 |
| 7 | Give an example of when you had to make a decision with incomplete information. | 資訊不完整時做決策的例子 |
| 8 | Describe a time when you received constructive criticism. How did you respond? | 收到建設性批評時的反應 |
| 9 | Tell me about a project you're most proud of and why. | 最自豪的專案與原因 |
| 10 | Describe a situation where you had to prioritize multiple tasks with tight deadlines. | 多項緊急任務時怎麼排優先序 |

### Technical（技術情境題）

| # | 原文 | 中文意譯 |
| --- | --- | --- |
| 11 | Describe a technical problem you solved recently. Walk me through your approach. | 最近解決的技術問題與思路 |
| 12 | Tell me about a time you had to learn a new technology quickly for a project. | 為專案快速學新技術的經驗 |
| 13 | Describe a situation where you had to debug a complex issue in production. | 在正式環境除錯複雜問題 |
| 14 | Tell me about a time you improved the performance or reliability of a system. | 提升系統效能或穩定性的經驗 |
| 15 | Describe your approach to code review. Give an example of feedback you gave or received. | 你的 code review 方式與一次回饋例子 |
| 16 | Tell me about a technical decision you made that you later had to revisit. | 一個事後得回頭修正的技術決策 |
| 17 | Describe a time when you had to balance technical debt against delivering new features. | 技術債與新功能之間的取捨 |
| 18 | Tell me about your experience with testing. How do you decide what to test? | 測試經驗，如何決定測什麼 |
| 19 | Describe a situation where you had to explain a complex technical concept to a non-technical audience. | 向非技術聽眾解釋複雜概念 |
| 20 | Tell me about a time you contributed to improving your team's development process. | 改善團隊開發流程的貢獻 |

### Leadership／Situational（領導與情境題）

| # | 原文 | 中文意譯 |
| --- | --- | --- |
| 21 | Describe a time you took the lead on a project. What was the outcome? | 主導專案的經驗與結果 |
| 22 | Tell me about a time you had to persuade others to accept your idea. | 說服他人接受你想法的經驗 |
| 23 | Describe a situation where you mentored or helped develop a colleague. | 帶人或協助同事成長 |
| 24 | Tell me about a time you had to resolve a conflict between team members. | 化解隊友之間衝突 |
| 25 | Describe a situation where you had to make an unpopular decision. | 做出不受歡迎決定的情境 |
| 26 | Tell me about a time you identified a problem before anyone else noticed. | 比別人更早發現問題 |
| 27 | Describe how you handle a situation where you disagree with your manager's direction. | 和主管方向不同時怎麼處理 |
| 28 | Tell me about a time you had to delegate tasks effectively. | 有效分派任務的經驗 |
| 29 | Describe a situation where you had to influence without direct authority. | 沒有職權時如何影響他人 |
| 30 | Tell me about a time you helped your team through a crisis or high-pressure situation. | 帶團隊度過危機或高壓情境 |

## AI 回饋的框架

### 已實作：JD 出題【repo】

`generate_questions_from_llm()` 的 prompt 要求 LLM 依職缺描述產生剛好 5 題行為面試題，回傳 JSON 陣列，每題有 `text` 與 `category`（behavioral、technical、situational 三選一），temperature 0.7。LLM 透過 Factory 切換 OpenAI 或 Anthropic，由 `DEFAULT_LLM_PROVIDER` 決定。

### 規劃中：面試評估（Phase 7）【repo】

PROJECT.md 的 US-5.4 只列三項，尚無 prompt 或評分表：

1. 語音轉文字（規劃用 Whisper API）
2. STAR 結構分析
3. 改進建議

商業設定：MVP 階段免費，之後一次評估 3 Credits；Pro 訂閱無限次。

### 回饋框架提案【建議】

repo 只寫了「STAR 結構分析」。以下是可以直接用在諮詢或日後 prompt 的評估維度：

| 維度 | 看什麼 | 回饋句型 |
| --- | --- | --- |
| S 情境 | 有沒有交代時間、角色、規模 | 「情境少了團隊人數與你的角色」 |
| T 任務 | 你負責的目標是否明確 | 「聽不出這件事為什麼歸你負責」 |
| A 行動 | 用「我」而不是「我們」；有具體步驟與取捨 | 「第二步可以補為什麼選 A 不選 B」 |
| R 結果 | 有數字或可驗證的結果，以及學到什麼 | 「結果補一個數字，例如處理時間從 X 降到 Y」 |
| 轉職連結 | 舊經驗是否連到新職位需要的能力 | 「這段客訴處理可以連到資料品質的敏感度」 |
| 長度與節奏 | 1.5–3 分鐘；開頭 15 秒內講到重點 | 「前 40 秒都在講背景」 |

回饋原則：先講做得好的一點，再給最多兩個改進，每個改進附一句改寫範例。這延續 style-guide「給數字、工具名，不寫空泛形容詞」的精神。

## 如何用於諮詢

以下流程整體是【建議】，每一步標出 repo 能支援的部分。

| 階段 | 目標 | 做法 | 工具支援 | 產出 |
| --- | --- | --- | --- | --- |
| 1. Intake（首談） | 了解背景、動機與限制 | 用 `templates/consultation-intake.md` 訪談 | 【repo】Profile、Experience、Skill、Portfolio 可讓學員自填 | 填好的 intake 表 |
| 2. 差距分析 | 目標職位要求 vs 現況 | 收集 3–5 個目標職缺，把 requirements 拆成技能清單，逐項對照 | 【repo】Job 模型可存職缺；匹配分析尚未實作，先人工對照 | 技能差距表（有／部分／缺） |
| 3. 計畫 | 補差距、做作品、排投遞節奏 | 依差距選專題（例如 DE 方向走 stock-crawler 課程），訂每週檢核點 | 【建議】沿用 `templates/project-plan.md` 的時程與風險表 | 1–2.5 個月行動計畫 |
| 4. 模擬面試 | 練習講出經驗與專題 | 先題庫模式三類各一場，再針對目標職缺出題 | 【repo】題庫模式＋錄影＋48 小時回放；JD 模式後端可用、前端未開放 | 錄影＋STAR 回饋紀錄 |
| 5. 追蹤 | 維持節奏、調整策略 | 每 1–2 週看職缺狀態分布與面試回饋 | 【repo】Job.status 五種狀態；【建議】補狀態變更時間以計算轉換率 | 追蹤週報 |

### 每階段的操作細節【建議】

**1. Intake**
- 第一次見面先聽故事，不急著給建議；結束前確認目標職位與每週可投入時數。
- 請學員課後自己在工具填 Profile，下次見面直接看資料。

**2. 差距分析**
- 差距分三級：「有，且有作品證明」「有，但沒有證據」「沒有」。第二級最常見，也最容易補，做一個作品就能升級。
- 對照 style-guide 的專題三級（作業、作品、MVP），標出學員目前作品在哪一級。

**3. 計畫**
- 每週都要有可交付的檢核點，不只寫工作內容。
- 投遞與學習並行；不要等作品全部完成才開始投。

**4. 模擬面試**
- 第一輪用題庫 general 類，練「為什麼轉職」與失敗經驗（第 4 題）。
- 第二輪用 technical 類，要求學員用自己的專題回答第 11、13、14、16 題。DE 學員可直接引用課程的 retry、冪等、雙寫設計。
- 回放時和學員一起看，用上方 STAR 表給回饋；48 小時內要看完。

**5. 追蹤**
- 看各狀態的數量：saved 很多、applied 很少代表投遞卡住；applied 多、interviewing 少代表履歷或職缺匹配有問題；interviewing 多、offer 少代表面試表現要加強。
- 每次追蹤都更新差距表，並決定下一輪模擬面試的類別。

## 隱私與安全

- 學員的電話、履歷、錄影都是個人資料。諮詢紀錄只寫在約定的位置，不貼到公開 repo 或聊天群組。
- 部署 job-hunter-v2 時，`SECRET_KEY` 與 LLM API key 只放在 `.env`，不進 git。

## 來源

- 功能與開發進度：https://github.com/lu791019/job-hunter-v2/blob/main/README.md
- 目標用戶、User Stories、付費模式：https://github.com/lu791019/job-hunter-v2/blob/main/PROJECT.md
- 開發者指南：https://github.com/lu791019/job-hunter-v2/blob/main/GUIDE.md
- 題庫：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/interview/question_bank.py
- 出題與錄影邏輯：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/interview/service.py
- 個人檔案模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/profile/models.py
- 職缺模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/jobs/models.py
- 作品集模型：https://github.com/lu791019/job-hunter-v2/blob/main/backend/app/portfolio/models.py
- 面試練習設計：https://github.com/lu791019/job-hunter-v2/blob/main/docs/plans/2026-02-19-interview-practice-design.md
- 面試 UI 調整（讀題倒數、強制循序）：https://github.com/lu791019/job-hunter-v2/blob/main/task.md 、 https://github.com/lu791019/job-hunter-v2/blob/main/frontend/src/pages/interview/InterviewSessionPage.tsx
