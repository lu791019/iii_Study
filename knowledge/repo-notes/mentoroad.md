# mentoroad

> 來源：https://github.com/lu791019/mentoroad（2026-10-16 讀取 README、package.json、`prisma/schema.prisma` 的 model 清單、`src/` 結構）。最後 commit 2024-10。README 快照另存於 [../sources/mentoroad.README.md](../sources/mentoroad.README.md)。

## 一句話用途

「導生門路」：仿 Topmate／ADPList 的導師預約平台雛形，願景是讓導師與門生互相串連、角色可以隨時互換，並把生涯教練的引導理論做成生成式 AI 引導機器人。網站把它列為實作成績「Mentoroad 導生門路 AI 平台」（`daydreamdex-astro/src/pages/consultant.astro`）。

## 技術棧

Next.js 13（T3 stack）、tRPC、Prisma（User、Service、Product、Subscription、Booking 等 model）、NextAuth、Stripe、Google Calendar API、Tailwind CSS、TypeScript。目前是預約與付款的骨架，README 描述的 AI 引導功能在程式碼裡還看不到（推論自 `src/` 結構）。

## 對教學與諮詢可借用的知識

1. **服務定位的差異**：Topmate、ADPList 的用戶是「確定要往哪個領域前進」的人；mentoroad 想服務「迷茫不知道該找誰和該怎麼辦」的人。這是 Dex 諮詢服務的核心對象。
2. **「每個人都是導師，也都是門生」**：教學相長的理念，和網站「教別人就是把自己學過的路再走一次」一致。
3. **諮詢 = 生涯理論 + 助人技巧 + 傳授分享**：README 的開發方式明列三者，可當設計諮詢流程或 AI 諮詢機器人時的三根支柱。
4. **導師平台的基本資料模型**：服務（Service）、方案（Product）、訂閱、預約（Booking）、行事曆串接。學員做「預約／媒合」類專題時可參考。
5. **可當 Full-stack 專題範例**：T3 stack + 金流 + 行事曆，規模適合講「從願景到 MVP」。

## 值得引用的段落

| 位置 | 內容 |
| --- | --- |
| `README.md`「願景理念」 | 「每個人都是導師，都可以傳授專業; 每個人同時也是門生」 |
| `README.md`「參考依據」 | 和 Topmate、ADPList 的用戶差異 |
