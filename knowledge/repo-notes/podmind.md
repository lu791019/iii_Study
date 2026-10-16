# podmind

> **私人 repo**｜2026-10-16 讀取 README、VISION.md、decisions.md、implementation_plan.md、task.md、SESSION-RESUME.md 與 `spike/`
> 狀態：規劃完成、Phase 0 spike 完成，Phase 1 尚未開工。

## 一句話用途

給想追前沿 podcast 但時間或英文不夠的中文科技人：用中英雙語對照逐字稿＋AI 摘要，同時支援「聽得更好」（增強聆聽）和「不聽也能跟上」（替代聆聽）。

## 技術棧

| 層 | 技術（規劃） |
| --- | --- |
| 前端 | Next.js 15 App Router、TypeScript、Tailwind CSS、shadcn/ui |
| 資料 | Supabase（Postgres） |
| 收集 | RSS（spike 用 Python stdlib `xml.etree`；正式版用 `rss-parser`） |
| 轉錄 | mlx-whisper 本地（large-v3-turbo） |
| 翻譯＋摘要 | Claude Sonnet；spike 用 `claude -p` |
| 排程 | 手動觸發（MVP 不上 cron） |

## 架構重點

```
RSS → 下載音檔 → ffmpeg 裁剪 → mlx-whisper（JSON segments＋時間碼）→ Claude 滑動視窗段落翻譯 → Claude 摘要（TL;DR＋金句＋大綱）→ Markdown／三種 view
```

- **3-View**：聽（播放＋逐字稿跟隨）、讀（雙語對照，產品 moat）、掃（TL;DR＋金句＋大綱＋一鍵草稿）。三個 view 共用同一份底層資料，成本是線性不是 3 倍。
- spike 是單檔 Python（`spike.py`），prompt 獨立放 `prompts/*.md`，設定放 `config.json`。

## 可借用的知識精華

1. **產品哲學一句話當過濾器**：「沈浸式聽不可取代，在這層上加 buff」，每個功能先過這關。
2. **把假設寫成可量測的驗證點再 spike**：段長 5–15 秒、翻譯連貫、中文 WER < 10%、單集成本與處理時間上限（皆為 repo 內的目標值）；跑完逐條打勾，結論 5/6 pass → GO。
3. **spike 要小**：0.5 天、只看 Markdown、不做 HTML demo；每檔只跑前幾分鐘。
4. **Spike 修正清單**（中文 Whisper 常見問題）：段落碎片化 → 合併過短段落；專有名詞認錯 → `--initial-prompt` 注入；簡體字 → OpenCC；幻覺 → `temperature 0`＋過濾；並行呼叫斷線 → 改循序＋retry。
5. **ADR 格式**：Context／Decision／Rationale／Alternatives Rejected／Consequences；連「MVP 砍掉的功能」也寫成一條 ADR，避免重複討論。
6. **產品獨立性原則**：可擴散的產品不依賴第三方當資料源或儲存層，即使能省 1–2 週工程時間。
7. **成本導向選型**：本地轉錄零成本、Sonnet 而非 Opus（摘要任務品質夠用）、雲端 Whisper 留到擴散階段。
8. **SESSION-RESUME.md**：一句話狀態、現在在哪、下次第一個動作（「使用者說 → Claude 該做」表）、blockers、文件讀取順序、session 結束時要更新什麼。
9. **語氣訊號字典與反模式紀錄**：分清楚探索模式與結案模式；把 AI 犯過的判斷錯誤（過度解讀動機、過早拍板、為差異化硬找 niche）寫下來。
10. **AI 產出只是素材**：自媒體草稿規定附 source 連結、個人觀點至少一半篇幅、引用不超過 30 秒內容。

## 適合的教學案例／學員專題

- **教學案例**：「怎麼做 spike」與「怎麼寫 ADR」的範本；AI 協作的 session 接手文件；多角色 AI 會議收斂規劃。
- **學員專題**：「英文技術影片／podcast → 雙語逐字稿＋重點摘要」，最小路徑是 spike 版（單檔腳本 → Markdown）。
- **難度**：🟡 中等（Apple Silicon 才能跑 mlx-whisper，其他機器改用雲端 Whisper 或較小模型）。
