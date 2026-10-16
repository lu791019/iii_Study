# castflow

> 公開 repo｜[github.com/lu791019/castflow](https://github.com/lu791019/castflow)｜2026-10-16 讀取 README、PLAN.md、implementation_plan.md、task.md 與 `src/` 結構

## 一句話用途

從 Podcast 音訊到社群貼文的 AI 內容管線：上傳音訊 → AI 轉錄 → 依個人風格生成 6 平台文案 → 排程直發 Threads 與 Facebook。

## 技術棧

| 層 | 技術 |
| --- | --- |
| 前端＋API | Next.js 15 App Router、Tailwind CSS、shadcn/ui、API Routes |
| 資料＋儲存 | Supabase（PostgreSQL、Storage、pg_cron） |
| 音訊處理 | ffmpeg.wasm（瀏覽器端壓縮） |
| 轉錄 | Groq Whisper（whisper-large-v3-turbo） |
| 文案生成 | Claude CLI（`claude --print`）或 Anthropic／OpenAI／Gemini API，可切換 |
| 發布 | Meta Graph API、Threads API |
| 部署 | Vercel（規劃） |

## 架構重點

```
上傳音訊 → ffmpeg.wasm 壓縮 → Supabase Storage → Groq Whisper → transcripts 表 → Claude（逐字稿＋風格 DNA）→ contents 表 → 編輯 → Meta API 發布／一鍵複製
```

- 9 張表：episodes、transcripts、writing_rules、style_dnas、style_examples、platform_templates、contents、publish_logs、settings。
- `lib/` 按外部服務分目錄：`anthropic/`、`openai/`（Whisper）、`meta/`、`supabase/`、`audio/`、`prompts/`。
- 所有 API Route 用 `x-api-key` header 保護（`lib/api-guard.ts`）。
- AI 設定先讀 DB，再 fallback 到環境變數；`auto` 模式依序找 API Key，都沒有就走 CLI。

## 可借用的知識精華

1. **閉環才是差異化**：市面工具只做排程或只做轉錄，把「音訊 → 文案 → 發布」串起來就是賣點。
2. **風格 DNA 7 維度**：結構模式、開場 Hook、語氣、CTA、長度格式、高互動特徵、禁忌。從至少 3 篇歷史貼文＋互動數據提取，生成時注入 prompt。
3. **Prompt 分層**：平台模板 → 寫作規則 → 風格 DNA → 範例庫，每層一張表，可分開調整。
4. **在瀏覽器端解決檔案大小限制**：Whisper API 單檔 25MB，前端先壓成 64kbps mono 16kHz，避開 serverless 的上傳限制。
5. **CLI／API 雙模式**：本機用 `claude --print` 吃訂閱額度，部署到 Vercel 改走 API，同一份程式。
6. **排程選型看最小間隔**：Vercel Hobby Cron 一天只能一次，改用 Supabase pg_cron 做分鐘級。
7. **先跑通再疊加**：風格 DNA 從早期 Phase 移到 Phase 5，先讓「上傳 → 轉錄 → 生成 → 發布」通。
8. **外部審核提早做**：Meta Developer App 放 Phase 1，避免做到發布才卡住。
9. **發布狀態機**：draft → scheduled → published → failed，失敗可手動 retry，每次寫 publish_logs。
10. **一個教訓**：PLAN.md 仍寫 OpenAI Whisper＋GPT-4o，README 已改成 Groq＋Claude。改選型要同步改計畫（單一事實來源）。

## 適合的教學案例／學員專題

- **教學案例**：LLM 應用全端實作（Next.js＋Supabase）、Prompt 分層設計、第三方 API 串接與排程。
- **學員專題**：「課程錄音 → 重點筆記 → 社群貼文草稿」，最小路徑是不做自動發布、只輸出 Markdown；進階版加風格 DNA。
- **難度**：🟡 中等（外部帳號申請與 token 管理是主要卡點）。
