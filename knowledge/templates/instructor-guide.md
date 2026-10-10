# 講師引導手冊範本（Instructor Guide）

> 給主講 + TA 的「照著跑就能完成活動」手冊。由 midterm-mvp-template 的 `_instructor/` 歸納而成。
> 一場工作坊建議拆成三份：**講師引導手冊**（本範本）、**講稿**（依投影片）、**TA brief**（5 分鐘讀完）。後兩份的骨架附在最後。

## 寫作規則

- 開頭一句話：「給主講師 + TA 用：照著這份跑就能完成 {時段} {時長} 活動。」
- 所有清單都寫成 checkbox，所有對照都寫成表格。
- 時間表要有「講師做什麼」與「學員做什麼」兩欄，實作段加粗。
- 引導句用問句，不直接給答案。
- 講稿用「」直接寫出要說的話，可以照念。
- 時間、檔名、Demo 長度要跟學員手冊、README、投影片完全一致（改一處就全改）。

---

## 範本

````markdown
# {活動名稱}｜講師引導手冊

> 給主講師 + TA 用：照著這份跑就能完成 {時段} {N} 小時活動。

---

## 活動概覽

| 項目 | 內容 |
|------|------|
| 名稱 | {活動名稱} |
| 時長 | {HH:MM-HH:MM}（{N} 小時） |
| 規模 | {人數} 人 × {組數} 組 |
| 講師人力 | 主講 1 + TA {1-2} |
| 形式 | {個人實作 + 組內互助} |
| 前後活動 | {同日其他時段的安排，若無寫「無」} |

## 活動目標

每位學員完成：

```
{一行流程，例如 CSV → pandas → SQLite → SQL → LLM → FastAPI}
```

{回家作業是哪些 Section}

## 事前準備清單

### T-1 週
- [ ] 確認場地 Wi-Fi 可支援 {人數+2} 台以上裝置
- [ ] 確認共用 API Key 額度（設用量上限）
- [ ] 發學員事前通知（帳號、環境測試、要準備的東西）
- [ ] 列印題目卡 × {題數}
- [ ] 跑自動化測試，確認每題 solution 端到端可跑

### 當天出門前
- [ ] 確認 repo 最新版可 clone
- [ ] 準備備用 API Key
- [ ] 帶延長線、計時器

### 到場後
- [ ] 測投影、測 Wi-Fi
- [ ] 確認分組，每組桌上放題目卡
- [ ] 投影切到工作坊簡報

---

## 時間表（詳細版）

| 時段 | 分鐘 | 內容 | 講師做什麼 | 學員做什麼 | Notebook |
|------|------|------|-----------|-----------|---------|
| {..} | 20 | 開場講解 | 投影 demo 完整流程 + 題目導覽 | 聽、看 | |
| **{..}** | **25** | **實作 1** | 巡場、解環境問題 | {環境 + 選題} | Section 0 |
| {..} | 5 | 交流 1 | 走動聽 | 依 spec 的交流引導互問 | |
| {..} | 10 | 講解 | {核心技能重點 + 各題難點} | 聽 | |
| **{..}** | **45** | **實作 2** | 巡場 | {核心技能} | Section {..} |
| {..} | 10 | 講解 | {進階主題 demo + fallback} | 聽 | |
| **{..}** | **30** | **實作 3** | 巡場 | {進階 + 打包} | Section {..} |
| {..} | 10 | 交流 2：預演 | 走動給回饋 | 組內預演 Demo | |
| **{..}** | **15** | **Demo** | 計時 + 點評 | 每組 1 人 × 3 min | |
| {..} | 10 | 收尾 | 升級路線 + Q&A | | |

---

## 開場逐字稿（{N} min，可精簡）

> 「歡迎大家。今天{做什麼}。
>
> 目標：{一句話產出}。
>
> 我先 demo 一遍——（投影跑 solution：{流程}）
>
> 你等等要做的就是這件事。Notebook 裡有步驟提示，前面幾格附了參考程式碼。
>
> 規則：
> 1. 每人選一題，整條流程自己做
> 2. 同組可以互相討論、debug
> 3. 可以用 AI，但 Demo 時要能說明
> 4. 做不完沒關係，repo 帶走課後繼續
>
> 現在選題、開環境。」

---

## 巡場引導重點

### 實作 {N}（{主題}，{分鐘} min — Section {..}）

**T+10**
- 觀察：{例如還卡在環境、讀不到資料}
- 介入時機：{例如超過 10 分鐘還沒讀到資料 → 幫看路徑}

**T+25**
- 觀察：{進度指標}
- 介入時機：{指著 Notebook 提示問「這段你看到了嗎？」}

**T+40**
- 觀察：{是否完成最低產出}
- 介入時機：{還沒做到 → 建議跳過細節，先把最低產出做完}

### 常見卡關 + 引導句

| 卡關 | 引導句 |
|------|--------|
| {路徑錯} | 「你現在在哪個資料夾？用 `os.getcwd()` 看看」 |
| {型別錯} | 「先用 `df.dtypes` 看看欄位是什麼型別」 |
| {外部服務失敗} | 「先跑 fallback 版確認流程對，API 之後再接」 |
| {做不完} | 「沒關係，先把 Section {..} 跑完，{..} 回家做也行」 |

---

## Demo 環節（{N} min）

### 格式
- 每組推派 1 人，3 分鐘，嚴格計時；2:30 舉手提醒
- 結構：問題 → 資料與清洗 → {AI/核心技術} → output → 升級方向

### 講師點評模板（每人挑 1 句）
- 「你的 {..} 很清楚，{..} 這個發現很有意思」
- 「如果資料量變 10 倍，你的 pipeline 會怎麼調整？」
- 「這個 {LLM 結果} 你同意嗎？有沒有分錯的？」

### 評分（內部，不公開）

| 維度 | 權重 |
|------|------|
| {流程完成度} | 30% |
| {AI / 核心技術應用品質} | 25% |
| {作品打包：README + 升級計畫} | 20% |
| {洞察品質：有數字支撐} | 15% |
| {Demo 表達} | 10% |

---

## 收尾（{N} min）

> 「今天做的 {作品}，不要做完就丟。後續學 {技術A}、{技術B} 的時候，都會回來升級它。
>
> 三件事帶走：
> 1. {產出 1}
> 2. {產出 2}
> 3. 課後 push 到你自己的 GitHub
>
> 這就是你履歷上的第一個 {..} 專案。」

---

## 備案

| 風險 | 應對 |
|------|------|
| Wi-Fi 不穩 | 資料都在 repo 裡，離線也能跑 |
| API Key 額度用完 | 全部切 fallback |
| 有人完全做不出來 | 開 solution 帶著跑，確保至少有產出 |
| 時間不夠 Demo | 每組縮成 2 分鐘 |
| {雲端環境}打不開 | 本地 {替代方案} |

---

## 物資清單

| 物資 | 數量 | 用途 |
|------|------|------|
| 題目卡（A4） | {題數} | 各題情境 + 資料說明 |
| 延長線 | {組數} | 每組 1 條 |
| 計時器 | 1 | Demo 計時 |
| 備用 API Key | 1 組 | 備案 |

---

## 檔案對照

| 用途 | 檔案 |
|------|------|
| 學員手冊 | `_instructor/student_handbook.md` |
| 講師手冊 | 本文件 |
| 講稿 | `_instructor/speaker_notes.md` |
| TA brief | `_instructor/ta_briefing.md` |
| 簡報 | `_instructor/{slides}.html` |
| 每題答案 | `_instructor/solutions/topic_N_solution.ipynb` |
| 學員 Notebook | `data/raw/topic_N/pipeline_starter.ipynb` |
````

---

## 附：講稿骨架（speaker_notes.md）

```markdown
# {活動名稱} — 講稿
> {時段} 版。{N} 段實作 + Demo。

## Part 1：開場講解（{時段}，{N} 分鐘）
### [Slide 1] 封面 (30s)
「{要說的話}」
### [Slide 2] 今日目標 (30s)
### [Slide 3] 做到哪算完成 (45s)
### [Slide N] 實作 1 Time (15s)
「現在開始實作 1，{N} 分鐘。{範圍}。基礎完成是 {..}，進階是 {..}。{N} 分鐘後交流：{交流題在哪}。開始。」

## Part 2：{主題}講解（{時段}）
## Part 3：…
## Part 4：預演 + Demo + 收尾
```

規則：每張 slide 標秒數；「實作 N Time」那張固定講「時間、範圍、基礎完成、進階、開始」。

## 附：TA brief 骨架（ta_briefing.md）

```markdown
# {活動名稱}｜TA 行前 Brief
> 活動前一天或當天早上給 TA 看。5 分鐘讀完就能上場。

## 你的角色
你不用講課，你的工作是：巡場、觀察、引導、計時。

## 時間軸 + 你要做什麼        ← 時段 | 模式 | 你做什麼
## 巡場 SOP
### 什麼時候介入             ← 訊號 | 行動（盯螢幕 3 分鐘不動、一直 Google、整組卡住→叫講師）
### 什麼時候不介入           ← 正在打字、在跟組員討論、在用 AI
### 巡場節奏                 ← 每 10 分鐘一圈；T+5 / T+15 / T+25 / T+35 觀察什麼
## 常見問題 QA               ← 環境類 / 程式碼類 / AI 類 / API 類 / 心態類，各一張「問題 | 回答」表
## Demo 計時
## 你不需要做的事            ← ❌ 講課 ❌ 幫學員打程式 ❌ 評分 ❌ 處理課程以外的問題
```

---

## 參考來源

- [_instructor/facilitator_guide.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/facilitator_guide.md)
- [_instructor/speaker_notes.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/speaker_notes.md)
- [_instructor/ta_briefing.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/ta_briefing.md)
- [_instructor/student_handbook.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/student_handbook.md)
- [_instructor/project_report_rubric.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/project_report_rubric.md)
- [_instructor/midterm_workshop_proposal.md](https://github.com/lu791019/midterm-mvp-template/blob/main/_instructor/midterm_workshop_proposal.md)
