# social-scraper

> 公開 repo｜[github.com/lu791019/social-scraper](https://github.com/lu791019/social-scraper)｜2026-10-16 讀取 README、GUIDE.md、planning.md、PLAN.md、implementation_plan.md、task.md 與程式結構

## 一句話用途

在 LINE 傳一個連結，自動爬取內容、用 Claude 產摘要，寫回 Google Sheet 或 Notion，再推播結果；主打「零 API 成本」。

## 技術棧

| 元件 | 技術 |
| --- | --- |
| IG／Threads 爬蟲 | Patchright（修補版 Playwright，反偵測） |
| GitHub | httpx＋GitHub REST API |
| 一般網頁 | httpx＋readability-lxml＋markdownify |
| OCR＋摘要 | `claude --print`（訂閱額度） |
| 語音轉文字 | ffmpeg＋mlx-whisper（Apple Silicon 本地） |
| 介面 | FastAPI＋line-bot-sdk，本機用 ngrok 暴露 webhook |
| 落地 | gspread（Service Account）、notion-client |
| 工具鏈 | uv、pytest、pytest-asyncio |

## 架構重點

```
LINE 傳連結
  ├── IG／Threads → Patchright → 圖片 OCR／影片 STT → 摘要 → Sheet1
  ├── GitHub → REST API → README 中文摘要 → GitHub 工作表
  └── 其他網頁 → httpx＋readability → 全文 Markdown → Notion Database
                         ↓
               寫回後 LINE 推播結果
```

- 分層目錄：`scraper/`（各平台）、`media/`（OCR、轉錄）、`services/`（Sheet、摘要、Notion）、`line_webhook/`。
- 兩種入口：LINE Bot（日常）與批次模式（Sheet A 欄貼網址、B 欄空白代表待處理）。
- 測試附 `tests/fixtures/*.json`（IG、Threads 的真實回應結構樣本），整合測試用 marker 分開。

## 可借用的知識精華

1. **爬蟲路線三選一**：官方 API（最穩、資料最少）→ 無頭瀏覽器模擬 → 攔截 Mobile API（快但常變）。決策是「先用 API 拿能拿的，不夠才上爬蟲」。
2. **防封四層**：IP（住宅代理、隨機間隔）、瀏覽器指紋、行為（像人一樣瀏覽、單帳號每日上限）、帳號（輪替、養號、存 session）。其中行為層最關鍵。
3. **不登入就沒有封號風險**：只匿名讀公開內容，私人帳號的貼文就放棄。
4. **先 spike 再寫解析器**：IG 新版 JSON 路徑、Threads 的 `thread_items[].post` 都是 spike 找出來，寫在 task.md。
5. **合併 LLM 呼叫**：摘要和關鍵點合成一次呼叫；多張圖合併成一次 OCR，省時間也省額度。
6. **佇列與即時回饋**：同時最多處理 2 筆，其餘排隊；先回「已收到」，完成或失敗再推播；輸入「進度」可查佇列。
7. **用 B 欄當狀態旗標**：B 欄空白 = 待處理，重跑只要清空 B 欄，批次模式天然可重試。
8. **把環境限制寫進文件**：Patchright 與 Chrome 不能共存、ngrok 重啟換網址、首次下載模型約 1.5GB、GitHub 未帶 token 每小時 60 次。
9. **成本估算對照**：完整反爬方案（代理＋多帳號＋API）月費約 50–200 美元以上（planning.md 估計值），對比本地方案的零 API 費，說明為什麼選後者。
10. **法律與 ToS 風險要先講**：planning.md 明寫「爬公開資料通常可接受，但違反 ToS 是確定的」。

## 適合的教學案例／學員專題

- **教學案例**：爬蟲倫理與路線選擇、LINE Bot webhook、`claude --print` 當本地 LLM、用 fixtures 測爬蟲。可接 [topic-briefs/crawl.md](../topic-briefs/crawl.md)。
- **學員專題**：「連結收藏摘要 Bot」，最小路徑是只做一般網頁＋GitHub（不碰 IG／Threads 的反爬），落地到 Google Sheet。
- **難度**：🟢 簡單（只做網頁）→ 🔴 較難（加上社群平台爬蟲與影片轉錄）。
