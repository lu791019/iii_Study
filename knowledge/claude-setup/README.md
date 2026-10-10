# Claude 設定

讓本機的 Claude Code 在任何專案中都知道你的身分與風格，並在需要時讀取這個 repo 的知識庫。

## 三層設定

| 層級 | 檔案 | 作用範圍 | 狀態 |
| --- | --- | --- | --- |
| 個人全域 | `~/.claude/CLAUDE.md` | 本機所有專案 | 需手動安裝，範本為 [user-CLAUDE.md](user-CLAUDE.md) |
| 專案 | repo 根目錄 `CLAUDE.md` | 只在本 repo | 已設定 |
| 技能 | `.claude/skills/advisor-docs/SKILL.md` | 只在本 repo，撰寫教材與計畫書時自動載入 | 已設定 |

## 安裝個人全域設定（macOS / Linux）

先 clone 本 repo，再執行（把路徑換成你的實際位置）：

```bash
REPO=~/code/classroom_advisor          # 本 repo 的本機路徑
mkdir -p ~/.claude
# 若已有 ~/.claude/CLAUDE.md，先備份
[ -f ~/.claude/CLAUDE.md ] && cp ~/.claude/CLAUDE.md ~/.claude/CLAUDE.md.bak
sed "s#<CLASSROOM_ADVISOR_PATH>#$REPO#" "$REPO/knowledge/claude-setup/user-CLAUDE.md" >> ~/.claude/CLAUDE.md
```

使用 `>>` 是附加在既有內容後面，不會覆蓋你原本的設定。安裝後開新的 Claude Code 工作階段即生效，可輸入 `/memory` 檢查載入了哪些 CLAUDE.md。

## 讓個人技能在所有專案可用（選用）

若希望在其他專案也能用「撰寫教材與計畫書」技能：

```bash
mkdir -p ~/.claude/skills
ln -s "$REPO/.claude/skills/advisor-docs" ~/.claude/skills/advisor-docs
```

用符號連結的好處是 repo 更新後技能也會跟著更新。
