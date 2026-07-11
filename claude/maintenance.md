# F. 維護協議：制度檔案怎麼改、教訓寫在哪

版本 2026-07-12。

## 檔案權限分級
| 檔案 | 弱模型可自行改？ | 規則 |
|---|---|---|
| claude/lessons.md | 可，只准附加 | 只能在檔尾新增條目，不得改寫或刪除既有條目 |
| claude/prompt-templates.md | 可，需附理由 | 改動時在該範本下加一行「修改：日期/原因」 |
| claude/diagnosis.md | 先問 Darcia | 這是診斷基準，隨意改會失去對照意義 |
| claude/model-dispatch.md | 半開放 | 型號/參數等【實】【待確認】標記可更新（附查證來源）；原則章節先問 |
| claude/judgment-rubrics.md | 先問 Darcia | 判準是制度核心，模型不得自行放寬 |
| CLAUDE.md | 先問 Darcia | 索引與硬規則，改動影響所有 session |
| claude/letter-to-future-sessions.md | 不改 | 歷史文件，只讀 |
| index.html（ITEMS） | 先問 Darcia | 工具庫唯一登錄處，見 CLAUDE.md 硬規則 4 |

通用鐵律：任何修改前先備份（git commit）；公開 repo 永遠不放通關語、token、端點金鑰。

## 教訓（lessons.md）的寫入格式
踩雷後立即寫入，格式固定四行，寫完才算結案：
```
## [日期] 一句話標題
情境：當時在做什麼
教訓：錯在哪、正確做法是什麼
制度修改：需要改哪個檔的哪一條（不需要就寫「無」）
```
判斷「值得記錄」的門檻：同樣的雷若下個 session 還可能踩 → 記；純粹一次性手滑 → 不記。

## 精簡週期
- lessons.md 超過 100 行：把重複主題合併，已寫入制度檔的條目刪除（先 commit 備份）。
  合併作業需 Darcia 批准後執行。
- 每月一次（或 Darcia 說「制度體檢」時）：用 prompt-templates.md 的審查範本，
  對 CLAUDE.md 引用的所有檔案跑一輪審查——找互相矛盾的規則、過期的型號、
  失效的 URL。產出問題清單給 Darcia 決定。
- CLAUDE.md 超過 150 行或其直接引用的常載內容合計超過 500 行：抽離長內容到
  按需引用檔，只留索引。

## 同步規則（雙環境）
真相來源是 repo（DarciaChen/pub 的 claude/ 資料夾）。
- Claude Code 本機檔改了 → 當次 session 內就要 push。
- claude.ai 的 Project instructions 是 CLAUDE.md 的「副本」，repo 版更新後
  提醒 Darcia 手動同步一次。
- 兩邊內容衝突時，以 repo 最新 commit 為準。
