# daydreamdex-astro

> 來源：https://github.com/lu791019/daydreamdex-astro（線上：https://daydreamdex.com）。2026-10-16 讀取 `src/consts.ts`、`src/data/profile.ts`、`src/pages/`（index、about、coaching、instructor、consultant）、`src/content/blog/` 數篇、`docs-v2-page-spec.md`。README 是 Astro 範本原文，沒有內容。

## 一句話用途

Dex 個人品牌「Dex 的塵世哲學」的官網與部落格：14 篇長文、三個服務頁（職涯諮詢、顧問、講座）、影評。是 Dex 公開自我介紹最完整、最新的來源。

## 技術棧

Astro 6（blog 範本起家）、MDX、Tailwind CSS v4、astro-icon、sitemap／RSS；測試用 Playwright（e2e、RWD）與 Vitest；`scripts/` 有 GA、Search Console、PageSpeed、Cloudflare 的週報腳本。部署在 Cloudflare（推論自 `public/_headers`、`_redirects` 與 `scripts/cf.py`）。

## 對教學與諮詢可借用的知識

1. **五個學員痛點 → 解方**（`about.astro`、`coaching.astro`）：路徑不清、作品集不知從哪做、技術轉不成職場能力、履歷沒回應、技術卡關。可直接當諮詢初談的分類。
2. **兩頂帽子**（`coaching.astro`）：職涯諮詢師（把興趣、能力、市場需求放在同一張地圖）與人生教練（「我到底想要什麼」），先聽再判斷。
3. **教練風格四點**：結構化引導、為什麼而學、技術 × 職涯 × 生涯、明確下一步行動。
4. **通用講座兩種剪法**（`instructor.astro`）：A 版 AI 實戰版（兩段現場示範）、B 版職涯規劃版（地圖 → 羅盤 → 路線與行李 → 90 天第一步），各 2 小時、時程切到分鐘。可當講座範本。
5. **五大講座主題**：AI 時代職涯戰略、資料工程入門與轉職路徑、AI 技能與工具應用、職涯個人品牌、AI Coding 導入；每題有「適合誰／要點／學員收穫」。
6. **三幕劇轉型法**（`consultant.astro`）：前期製作（需求與架構）→ 拍攝製作（MVP 與迭代）→ 後製與上映（部署、交接、自主維運），每幕列交付物。也適合拿來講專題的階段。
7. **顧問案例的寫法**：挑戰 → 做法 → 成果 → 一句話（例：「一套沒有人維護得了的系統，做得再漂亮都是負債。」）。學員寫專題 README 可以仿這個結構。
8. **作品集文章**（`content/blog/2024-08-04-*.md`）：作品集三個關鍵點（搭時事、做大而全面並對齊想轉的方向、「埋梗」），以及履歷 8 大項目。
9. **DE 成長文章**（`content/blog/2025-03-06-*.md`、`2025-03-20-*.md`）：從「資料搬運工」到架構師的能力（可維護、懂數據、監控、雲端與 DevOps、商業思維），以及四種 Stakeholder 的需求。
10. **單一事實來源**：`src/data/profile.ts` 註明「Dex 的事實 single source」，網站的學員數、職稱都從這裡讀；`docs-v2-page-spec.md` 規定學員數一律 50+、職稱 AI 技術總監。寫 Dex 的介紹時以這兩個檔為準。

## 值得引用的段落

| 位置 | 內容 |
| --- | --- |
| `src/pages/index.astro`「A LETTER TO YOU」 | 共感開場的範例：「我懂你的猶豫。30 多歲還能轉職嗎……」 |
| `src/pages/index.astro`「PART ONE · 起點」 | Dex 自述轉職歷程，含「工程師轉行銷被說走偏了」那段 |
| `src/pages/about.astro`「我的故事」 | 為什麼開始做職涯輔導 |
| `src/pages/consultant.astro`「我跟其他顧問的差別」 | 「業界最佳實踐」vs「以你規模做得起的做法」對照表 |
| `src/content/blog/2026-10-09-career-switch-portfolio-like-filmmaking.md` | 作品集像電影；最新、最短、最能代表現在的語氣 |
| `src/content/blog/2025-03-04-30s-career-change-5-questions-to-ask.md` | 轉職前問自己的五個問題 |
| `src/content/blog/2024-08-04-python-career-transition-guide-and-experience.md`「作品集的關鍵點 3」 | 「面試就是一個說故事和聽故事的過程」 |

## 注意

`src/data/profile.ts` 與頁面裡有個人聯絡方式與諮詢費用，不要抄進本 repo。
