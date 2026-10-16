# filmer

> **私人 repo**｜2026-10-16 讀取 README、CLAUDE.md、product-strategy.md、product-thinking-log.md、implementation_plan.md、task.md 與 `app/` 結構
> 狀態：產品策略三個 Phase 完成、專案初始化完成；後端與主要功能尚未實作。

## 一句話用途

給看完電影有很多想法、卻覺得自己寫不好而從不發表的人：用錄音或文字留下觀影感受，系統用語意配對讓你遇見感受相近的人。

## 技術棧

| 層 | 技術（規劃） |
| --- | --- |
| App | Flutter 3.x（iOS／Android／Web）、Riverpod、GoRouter |
| SEO 站 | Next.js 或 Astro（獨立於 App） |
| 後端 | Supabase（Auth、Postgres＋pgvector、Storage、Edge Functions） |
| 語音轉文字 | OpenAI Whisper API |
| 向量化 | OpenAI text-embedding-3-small |
| 電影資料 | TMDB API |

## 架構重點

```
錄音／打字 → Whisper 轉文字 → embedding → pgvector → Feed、語意推薦、相性推薦、情感知識圖譜
```

- 三種推薦路徑：語意推薦（你對 A 片的感受與別人對 B 片相似）、相性推薦（taste vector 相近的人看了什麼）、圖譜探索。
- **v0.1 就靜默存 embedding**，v0.3 才上推薦：先累積資料資產。
- Flutter 目錄：`core/`（config、theme、router、services）、`features/`（auth、feed、post…）、`shared/`。
- Supabase migration 以數字前綴排序；每張表都要 RLS。

## 可借用的知識精華

1. **定位聲明句型**：「給（誰），（產品）是（什麼），不同於（競品），我們（怎麼做）」。
2. **Proto-persona＋JTBD**：「當我…，我想要…，這樣我可以…」，主要用戶與次要用戶分開寫。
3. **價值觀寫成「A > B」**：真實 > 完美、感受 > 評價、遇見 > 追蹤、說 > 寫、一句有共鳴的話 > 一百個讚。寫進 CLAUDE.md，開發時回頭檢驗。
4. **「不做什麼」也是決策**：不做星星評分、追蹤制、留言串、短影片、即時語音、MVP 私訊，每條都寫理由與未來啟動條件。
5. **暫緩清單附啟動時機**：例如「有 100+ 活躍創作者時」才做訂閱，把想法存起來而不是刪掉。
6. **洞察紀錄**：例如「表達衝動有保鮮期」→ 產品要在衝動最強時攔截用戶（「剛看完」模式）。
7. **護城河是資料不是 UI**：別人能抄介面，抄不走累積的情感資料。
8. **SDD → BDD → TDD → SDD Review**：每個端點至少 3 個 Given/When/Then（happy path、驗證失敗、權限），測試先紅再綠，最後比對 spec。
9. **Subagent 分工**：主 session 寫 spec 與測試；實作、spec review、code review 各派 subagent。
10. **未回答的問題要列清單**：錄音時限、內容審核、語音資料隱私等，後來在 plan 的 User Review Required 逐條確認。

## 適合的教學案例／學員專題

- **教學案例**：產品策略文件怎麼寫（定位、persona、JTBD、決策紀錄）；SDD／BDD／TDD 方法論；向量資料庫入門。
- **學員專題**：「心得／評論語意配對」，最小路徑是純文字＋embedding＋pgvector 相似搜尋，不做錄音與 App。
- **難度**：🔴 較難（跨平台 App＋向量推薦）；只做推薦核心可降到 🟡。
