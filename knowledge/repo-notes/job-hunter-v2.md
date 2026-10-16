# job-hunter-v2

> 公開 repo｜[github.com/lu791019/job-hunter-v2](https://github.com/lu791019/job-hunter-v2)｜2026-10-16 讀取 README、GUIDE.md、PROJECT.md、implementation_plan.md、docs/plans 與 backend 結構
> 諮詢面的整理（資料模型、題庫、諮詢流程）已在 [playbooks/career-coaching.md](../playbooks/career-coaching.md)，這裡只寫產品與工程面。

## 一句話用途

AI 求職助手的雛形：管理個人檔案與作品集、追蹤職缺、用題庫或 JD 出題做行為面試錄影練習並回放；AI 履歷與 AI 面試評估仍在規劃中。

## 技術棧

| 層 | 技術 |
| --- | --- |
| 前端 | React 19、Vite、TypeScript、Tailwind CSS v4、Zustand、React Hook Form＋Zod、Lucide |
| 後端 | FastAPI、SQLAlchemy 2.0（async）、Alembic、Pydantic v2 |
| 認證 | JWT（access 30 分鐘＋refresh 7 天）、passlib bcrypt |
| 資料庫 | SQLite（開發）／PostgreSQL 15（正式，docker-compose） |
| AI | OpenAI＋Anthropic，Factory Pattern 抽象層 |
| 儲存 | 本地檔案（開發）／S3（正式），StorageBackend 抽象 |
| 錄影 | 瀏覽器 MediaRecorder API |

## 架構重點

```
React（AuthGuard 保護路由）→ Axios（自動帶 Bearer token）→ FastAPI /api/v1/* → service → SQLAlchemy／StorageBackend／LLM Factory
```

- 每個後端模組固定四檔：`models.py`、`schemas.py`、`service.py`、`router.py`。
- `llm/base.py` 定義 `LLMProvider`（`generate()`、`generate_json()`），`providers/` 放 Claude 與 OpenAI 實作。
- 面試流程：讀題 10 秒 → 準備 → 錄影 → 預覽 → 送出 → 下一題；不可跳題、不可預覽下一題；48 小時內可回放。
- PROJECT.md 有 8 個 Phase、Epic／User Story、付費模式（Freemium＋Credits＋訂閱）規劃。

## 可借用的知識精華

1. **模組四檔結構**：學員一看就懂「資料長怎樣、驗證在哪、邏輯在哪、路由在哪」。
2. **LLM 抽象層從第一天就做**：`generate_json()` 預留 schema 參數，換供應商不改業務邏輯。
3. **儲存抽象**：本地開發與雲端正式環境只差一個實作。
4. **For Claude 計畫範本**：開頭 `For Claude: REQUIRED SUB-SKILL: Use superpowers:executing-plans`，接著「Codebase Conventions (Read First)」，每個 Task 列 Create／Modify（含行號）／Test。
5. **Handoff 摘要**：`implementation_plan.md` 寫已完成、下一步、**已修改但未 commit 的檔案**，下一個 session 不會漏。
6. **Phase 可以跳**：先做最有感的面試練習（Phase 5），再回頭做 URL 解析、履歷。
7. **設計系統寫成表**：主色、CTA、背景、卡片、字體、icon 一張表定下來，UI polish 才有依據。
8. **功能狀態誠實標**：README 對未完成功能標「規劃中」，前端按鈕停用並標 coming soon。
9. **一個教訓**：PROJECT.md 的 Phase 表寫面試練習 0%，GUIDE.md 寫已完成；進度只該在 task.md 維護一份。

## 適合的教學案例／學員專題

- **教學案例**：FastAPI 分層架構、JWT 認證流程、async ORM＋Alembic 遷移、寫給 AI 的 TDD 實作計畫。
- **學員專題**：「面試練習工具」或「職缺追蹤＋JD 匹配分析」，最小路徑是題庫＋錄影＋回放，不做 AI 評估。
- **諮詢用途**：見 [career-coaching.md](../playbooks/career-coaching.md)。
- **難度**：🟡 中等。
