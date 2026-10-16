# daydreamdex

> 來源：https://github.com/lu791019/daydreamdex（2026-10-16 讀取 README、GUIDE、`knowledge/` 下的 cmo、clo、edu、cto、cpo、cos-playbook）。未讀 `knowledge/cfo/`（財務）與 `state/`（營運狀態）。

## 一句話用途

建在 Claude Code 上的「一人公司 Agent OS」：Dex 當 CEO，主 session 當幕僚長（CoS），調度 8 位 CXO（CMO、CPO、CLO 顧問總監、EDU 教學總監、CTO、COO、CFO、TechConsult），每位有自己的知識庫、persona 和短期記憶。

## 技術棧

Claude Code（Skill + Subagent 混合架構）、Superpowers skill 框架、Markdown 知識庫、PostToolUse hook 提醒更新記憶。CTO 角色整合 [datadevdex](datadevdex.md)，CXO 可呼叫 [dex-agent-os](dex-agent-os.md) 的 CLI。

## 對教學與諮詢可借用的知識

1. **五個教學原則**（`knowledge/edu/teaching-methodology.md`）：先寫再問、提示不給答案（Scaffolding）、追問單一概念、手動模擬表格、選擇適合素材；每條附學習理論與一個真實課堂案例。
2. **鷹架式提示四步**（`knowledge/edu/scaffolding-prompts.md`）：確認目標 → 拆小步驟、每步一個提問 → 提問指向概念不指向語法 → 給驗證方式。另有「學員說不知道怎麼開始」「問這樣寫對不對」等情境話術，以及 AI 助教的回覆範本。
3. **初學者常見誤區庫**（`knowledge/edu/common-mistakes.md`）：每個誤區固定「錯誤寫法 → 為什麼錯 → 正確寫法 → 教學策略 → 延伸誤區」，可直接當作新誤區的範本。
4. **選資料集看學員程度**：剛學完語法的學員選 500 筆而不是 15,000 筆，因為能肉眼檢查、欄位少、一堂課做得完（teaching-methodology 原則 5）。
5. **初學者的卡點在「為什麼這樣寫」，不在語法**；「x = x + 1」是認知斷點，要用「手動模擬 → 對照輸出 → 理解程式碼」這座橋（teaching-methodology「關鍵洞察」）。
6. **諮詢準備清單**（`knowledge/clo/consulting-framework.md`）：客戶背景、本次目標（職涯／技術／產品）、預設提問、上次 follow-up、可推薦資源；諮詢紀錄固定六段，最後一段是「內容萃取潛力」。
7. **品牌語氣與平台規則**（`knowledge/cmo/brand-voice.md`、`style-dna.md`、`platform-rules.md`）：寫作七條鐵律（重新詮釋不是翻譯、開場先共感再翻轉、引導式優於判定式、收尾只留一句能截圖的話等），跨平台禁止「不假裝做過沒做的事、不雞湯」。
8. **決策權限分級**：低風險（整理、摘要、草稿）自動做；發布、刪除、動帳號金流一定問 Dex（brand-voice「決策權限」）。替 Dex 產出對外內容時照這條。
9. **工程原則**（`knowledge/cto/tech-standards.md`）：先規劃再動手（implementation_plan.md + task.md）、AI 產碼要 schema-aware、Template + AI 混合、LLM 輸出的 parser 綁格式標記；可用在專題業師的 code review。
10. **「CEO 掌握決策權」的設計**：AI 負責調度與草稿，人做最後決定。可當作教學員設計 AI 協作流程的範例。

## 值得引用的段落

| 位置 | 內容 |
| --- | --- |
| `README.md`「設計原則」 | CoS 是輕量路由器、不觸發不介入、知識庫精煉而非複製 |
| `GUIDE.md`「三種使用模式」 | `/edu 設計一個 6 小時的 Claude Code 實戰工作坊` 這類用法示範 |
| `knowledge/edu/teaching-methodology.md` 原則 4 | 找零問題的手算表格範本 |
| `knowledge/edu/scaffolding-prompts.md`「常見引導情境」 | 可直接拿來當 TA 話術 |
| `knowledge/edu/course-catalog.md` | Tibame 黑客松工作坊（3.5 小時、三星制度）、顧問團挑戰賽 v3.0 的摘要 |
| `knowledge/cmo/style-dna.md` | 七條寫作鐵律與五種開場 Hook |
