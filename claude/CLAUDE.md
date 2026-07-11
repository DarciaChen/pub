# CLAUDE.md — Darcia 作業制度索引

本檔上限 150 行，只做索引與硬規則。長內容一律在引用檔，按需讀取，不要全部載入。
版本 2026-07-12。

## 使用者與環境
- Darcia，台灣，琦勝企業（Conch）業務與技術支援：溫控錶、計數器、PID 控制器、類比 I/O 模組。
- 回覆一律使用繁體中文。
- 個人工具庫：https://darciachen.github.io/pub （repo: DarciaChen/pub）。
  index.html 的 ITEMS 陣列是工具的唯一登錄處，格式 {type, name, file|url, date, desc}。
- 作業環境雙軌：claude.ai（含手機、平板）與 Claude Code（Windows，工作目錄 C:\dev\pub-work）。

## 硬規則（任何模型、任何情況都適用）
1. 改既有檔前先備份：git commit 或另存 .bak。新內容寫新檔，不覆蓋。
2. 型號、暫存器位址、API 參數、費率、法規：先查證，查不到寫「待確認」，禁止憑記憶填。
3. 部署後必驗證：端點部署認回傳 commit SHA（可用 GitHub API 覆核該 commit）；
   git push 認 push 成功與 commit hash。Pages/raw URL 有 CDN 快取，不作為驗證依據。
4. 高風險操作先停下來問 Darcia：刪除檔案、修改 index.html 的 ITEMS、對外發送內容、
   任何涉及現場設備接線或參數下載的建議。
5. 交付訊息必含三件事：做了什麼、怎麼驗證的、還剩什麼沒做。
6. 部署端點與通關語屬機密，不得寫入任何公開 repo 檔案。以 Darcia 的記憶或本機私檔為準。

## 制度文件路由（按需讀取）
| 情境 | 讀這個檔 |
|---|---|
| 開始任何任務前的自檢 | claude/diagnosis.md |
| 要派 subagent 或選模型 | claude/model-dispatch.md |
| 判斷完成度、該不該問、該不該升級 | claude/judgment-rubrics.md |
| 要交辦任務（搜尋/實作/重構/研究/審查） | claude/prompt-templates.md |
| 要修改制度檔或記錄教訓 | claude/maintenance.md |
| 新 session 接手 | claude/letter-to-future-sessions.md |
| 動手前先看有沒有人踩過同樣的雷 | claude/lessons.md |

線上位置：https://darciachen.github.io/pub/claude/[檔名]
Claude Code 本機找不到檔案時，改 fetch 上述 URL。

## 高頻任務速查
- 工具庫部署：主要走 Claude Code 的 git commit + push（有版本歷史）；
  行動場景走「部署工具庫」Apps Script 端點（通關語欄位名為 secret；完整規格存於
  Claude 記憶與各 Project instructions）。部署成功的唯一依據是回傳的 commit SHA；
  Pages URL 與 raw URL 有 CDN 快取，不作為驗證依據。ITEMS 登錄用選填 entry 欄位。
- Conch 技術支援：P50 校正用 Modbus 暫存器 0x201 / 0x203（Pt100 兩點線性校正）。
  所有暫存器值與接線建議必須引用手冊或規格檔，不得背誦。
- 行銷文案：工程師痛點導向、短句、繁體中文。品質判準見 judgment-rubrics.md 文案節。
- 查證類研究（保險、法規、產品規格）：必先搜尋，來源優先序與不確定標註規則
  見 judgment-rubrics.md 研究節。

## 「交接」指令
Darcia 說「交接」時：產出結構化交接摘要，含當前目標、已完成項、關鍵決策與理由、
未解決問題、相關檔案與連結。摘要必須自足，新對話不需回讀舊對話即可接續。

## 安裝說明（給 Darcia，模型可忽略本節）
- Claude Code 專案層：複製本檔到 C:\dev\pub-work\CLAUDE.md（作用於該專案所有 session）。
- Claude Code 全域層：想跨專案通用，放 C:\Users\<你>\.claude\CLAUDE.md。
- claude.ai：內容貼入 Project instructions；或在對話開頭請模型 fetch 本檔線上 URL。
- 制度檔更新後，兩邊都要同步（本機檔與線上檔），以 repo 為準。
