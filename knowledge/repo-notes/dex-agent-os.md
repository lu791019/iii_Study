# dex-agent-os

> 來源：https://github.com/lu791019/dex-agent-os（2026-10-16 讀取 README、AGENTS.md、PLAN.md 標題與規則章節、`canonical/rules/00-core.md`、`10-writing-style.md`、`800_System/templates/`、`800_System/references/style-dna/threads-dna.md`，以及 `800_System/references/examples/threads/` 中與職涯、AI、資料工程有關的公開貼文）。
> **刻意不讀**：`100_Journal/`、`600_Life/`、`200_Work/`（含諮詢紀錄）、`800_System/knowledge/`（從日記萃取的反思）。這些是私人內容。

## 一句話用途

Dex 的個人 AI 代理人作業系統：把每天的工作紀錄、學習輸入、會議與諮詢逐字稿，用 `claude --print` 加工成日記、洞察，再轉成 Threads、Facebook、部落格、電子報、短影音等多頻道草稿，最後由 Dex 審稿發布。

## 技術棧

Python 3 + Bash CLI（`./bin/agent <command>`）、Claude Code CLI（走訂閱額度，不用 API 費）、純 Markdown 檔案系統 + Git（無資料庫）；整合 Readwise、Threads API、Google（Gmail、Classroom、Docs）、Fireflies.ai、Notion、Anybox、Dayflow。`canonical/` 是規則唯一來源，`bin/sync` 同步到 Claude Code、Cursor、Antigravity。

## 對教學與諮詢可借用的知識

1. **IPO 協作原則**（`canonical/rules/00-core.md`）：Input 必須來自 Dex 的真實紀錄、Process 由 AI 整理與提供選項、Output 由 Dex 最後校正。替 Dex 寫東西時的底線。
2. **寫作禁止事項**（`AGENTS.md`、`10-writing-style.md`）：不假裝做過、不過度行銷、不空泛開場、不堆 emoji、不雞湯。
3. **各頻道格式**（`10-writing-style.md`）：電子報（開場反差 → 3–5 主題 → 推薦與行動）、Threads（Hook → 3–5 點 → 金句，150–250 字）、Blog（SEO 標題、H2/H3、教學步驟感）。
4. **Threads 風格 DNA**（`threads-dna.md`，從 78 篇真實貼文萃取）：五種結構、五種 Hook、高互動特徵（反差斷言、觸及職涯焦慮、實用清單），以及「缺乏立場或情緒的文章表現最差」。
5. **諮詢紀錄範本**（`800_System/templates/consultation-notes-template.md`）：核心問題、背景脈絡、討論要點、建議、行動項（who／what／when）、後續追蹤、內容萃取潛力。可以和 [templates/consultation-intake.md](../templates/consultation-intake.md) 搭配；實際紀錄不要放進公開 repo。
6. **一個主題，多種輸出**（PLAN.md「內容生產管線」）：洞察 → TOPIC.md → 各頻道草稿。教學員經營個人品牌時可以示範這條管線。
7. **Google Classroom 同步**（`classroom-sync`）：公告、作業追蹤、學生提交狀態。講師端追蹤可參考，但學生資料不要 commit。
8. **Style DNA 會長大**：範例越多，風格萃取越準。教學員「用 AI 寫出自己的風格」時可以用這個做法。
9. **Token 管理**（PLAN.md 第 9 節）：「永遠載入的越少越好」，規則分層、需要時再讀。

## 值得引用的段落

| 位置 | 內容 |
| --- | --- |
| `README.md`「資料流」 | 輸入 → 收集器 → 產生器 → 主題企劃 → 多頻道草稿 → 人工審稿 |
| `canonical/rules/00-core.md`「決策權限」 | 低風險自動、高風險問 Dex |
| `800_System/references/style-dna/threads-dna.md`「一句話風格概括」 | 「用工程師的邏輯腦 + 朋友聊天的口吻……」 |
| `800_System/references/examples/threads/065-*.md` | 最高互動貼文：奉勸新手不要用 AI 寫程式（731❤️） |
| `800_System/references/examples/threads/043-*.md` | 當 AI 都能寫 pipeline，資料工程師的價值 |
| `800_System/references/examples/threads/059-*.md` | 中高齡轉職：經歷是「信任基礎」 |

## 注意

這個 repo 是公開的，但含個人日記與諮詢資料夾；[repo-catalog.md](../repo-catalog.md) 已提醒諮詢紀錄不要 commit 進公開 repo。引用時只用上面列的系統文件與已公開的 Threads 貼文。
