# datadevdex

> 來源：https://github.com/lu791019/datadevdex（2026-10-16 讀取 README、`.claude/agents/datadevdex.md`、`.claude/agents/datadevdex-knowledge.md`、`memory/tech-lead-discussions.md`）。

## 一句話用途

給 Claude Code 用的「會自我進化的 Tech Lead 人格」：有技術立場、會記住你分享過的做法，並把具體經驗蒸餾成可被推翻的技術原則。在 [daydreamdex](daydreamdex.md) 裡擔任 CTO 的大腦。

## 技術棧

Claude Code custom agent（subagent）＋ slash command `/tech-lead`；三層 Markdown 記憶：Layer A 知識庫（具體案例）、Layer B 討論紀錄（跨 session 結論）、Layer C 技術哲學（嵌在 agent prompt，會進化）。四種模式：consult、challenge、review、learn。

## 對教學與諮詢可借用的知識

1. **歸納 → 蒸餾 → 演繹 → 修正**：同一主題累積 ≥3 筆經驗才萃取成原則，新證據可以推翻原則。可拿來教學員「怎麼從專題經驗長出自己的技術觀點」。
2. **「做」vs「做好」**：做 = 功能可運作；做好 = 有測試、有監控、不會再發同樣問題。可直接放進專題完成標準。
3. **成果追蹤**：「開發了但沒累積出成果，從結果來看與沒開發無異」；建議每個 Phase 結束寫 Retrospective（使用頻率／解決的痛點／重來會怎麼做）。和專題復盤一致。
4. **規劃文件要寫「不做什麼」**：implementation_plan.md 加 `Non-goals` 與 `Alternatives Considered`。學員的計畫書可以照這樣補「為什麼不選 B」，對應面試會問的問題。
5. **ADR 只給架構級決策**：跨模組、引入新依賴、改變資料流才寫，「不要用重砲打蚊子」。
6. **無聊是自動化的入口**：「先手動做到痛，再自動化」。可當教 Airflow／排程前的動機。
7. **Review 四維度**：可維護性、可測試性、效能、安全；區分「必須改」與「建議改」。業師 code review 可以照用。
8. **基礎工程原則**：先規劃再動手（schema-aware）、YAGNI、Pragmatic over dogmatic、寫完就驗證、質疑不必要的抽象。
9. **回應風格**：「有自己的觀點，不和稀泥」「不確定時承認不確定」。這也是 Dex 希望 AI 協作者具備的態度。

## 值得引用的段落

| 位置 | 內容 |
| --- | --- |
| `README.md`「核心概念」 | stateless AI assistant 的四個問題 |
| `README.md`「進化範例」 | monorepo vs polyrepo 從兩個案例萃取成一條原則 |
| `README.md`「設計決策」 | 「因為教條是好工程的敵人」 |
| `.claude/agents/datadevdex.md`「萃取的原則」 | 三條已萃取的原則與來源標註格式 |
| `.claude/agents/datadevdex-knowledge.md`「[觀念]「做」vs「做好」」 | 完成的定義升級 |
