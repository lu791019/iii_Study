# channel-learn（私人 repo）

> **一句話用途**：Dex 的多租戶 cohort 教學平台的程式碼與內容。學員從練基本功（題庫、瀏覽器內跑碼、真人批改）一路走到帶完專題，最後做出求職作品集；Discord 社群和職涯工具都接在平台上。

⚠️ 這是私人 repo。下面列的路徑只供有權限的人對照，內容不要整段搬進公開 repo。

## 結構

```
api/          FastAPI 純 JSON API（不 render HTML）
  app/        業務邏輯：題庫、批改、XP 與任務、AI 提示教練、職涯工具、Discord 整合、新手任務
  routers/    各模組端點
  migrations/ 依序編號的 SQL migration（40+ 支）
  seed/       題庫（Python、SQL、pandas、演算法、DE、申論）與關卡地圖 JSON
  scripts/    排程用的 Job（社群貼文、收割、週報、匯入、清理）
  tests/      pytest
web/          Astro 靜態輸出＋islands
  src/content/  四個內容集合：coursebook（DE 課程手冊）、handbook（專題手冊＋8 情境）、setup（環境安裝 S0–S10）、workshops（工作坊四站）
  src/pages/    學員端、教師端、職涯區、公開頁
  e2e/          Playwright 端到端測試
dev/          部署腳本、效能與稽核用 e2e
PRODUCT.md    產品定位、使用者、原則、品牌承諾
walkthrough.md  每批功能的「改了什麼／怎麼驗證」
task.md       任務清單
```

架構是「架構分離、部署合體」：單一 VM 或單一容器同時跑 db、api 和靜態前端，目標是零固定成本。

## 可借用的知識精華

1. **成功定義分兩層**：北極星是轉職成功，可量測的 proxy 是完課率加作品集完成率，寫在 `PRODUCT.md`。
2. **產品原則五條**：XP 只獎勵真實學習、不懲罰；敘事就是教學；工程是本體、電影是皮膚；先驗證再擴張；核心服務不稀缺、不操縱。
3. **不杜撰的紀律**：還沒有成效數據或見證之前，任何介面都不准寫出數字或見證。
4. **內容單一來源**：手冊類內容由同步腳本從教材 repo 拉進來，檔頭寫明「別手改內文」，frontmatter 帶 `source` 連回原文。
5. **瀏覽器內跑碼的限制要回頭影響出題**：Pyodide 不支援 `input()`，出題時就要避開。
6. **AI 提示教練**有三層防線（系統指令、伺服器端輸出檢查、漸進式提示），還有每日額度、月預算上限，以及「用過教練才答對，XP 打八折」。
7. **新手任務**：每個角色一條有終點的任務線，每一步自動偵測或自己勾，完成就給獎勵；每步記 `done_at`，之後拿來觀察「幾天做完」。
8. **walkthrough 格式**：每一批都寫 Changes Implemented → Verification Results（pytest 數、e2e 數、prod 冒煙）→ Next Steps，可以直接套用到課程改版紀錄。
9. **CI 守底線**：CI 只跑後端全量測試加前端可建置；比較重的 e2e 留在本機跑。
10. **公開頁（SEO）**：只公開平台才有的內容（職位指南、題目試閱、規則），build 時就渲染好，並附 sitemap、canonical。

## 值得參考的檔案（repo 為私人）

| 路徑 | 看什麼 |
| --- | --- |
| `PRODUCT.md` | 定位、使用者、成功定義、產品原則的寫法 |
| `walkthrough.md` | 功能批次的變更與驗證紀錄格式 |
| `web/src/content.config.ts` | 內容集合怎麼定義、同步來源怎麼註記 |
| `web/src/content/workshops/0-prologue.md` | 工作坊站頁範例（開課前準備、這一站在做什麼、流程） |
| `api/app/career_guides.py` | 五個職位指南的資料結構（同一份模板） |
| `api/app/career_practice.py` | 行為面試口試練習（題庫加範本回饋） |
| `api/app/coach_prompt.py`、`coach_guard.py` | 不給答案的教練 prompt 與輸出檢查 |
| `api/app/onboarding.py` | 三角色新手任務線 |
| `.github/workflows/ci.yml` | 最小 CI |

## 相關

- 萃取後的手冊：[../playbooks/content-channel.md](../playbooks/content-channel.md)
- 規劃與決策紀錄在另一個 repo：[channel-learn-planning.md](channel-learn-planning.md)
