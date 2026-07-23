# 踩雷教訓紀錄（只准附加，格式見 maintenance.md）

## [2026-07-12] Apps Script 部署端點回 UNAUTHORIZED
情境：依 Claude 記憶中的通關語與端點 POST 部署制度檔，連續兩次被拒。
教訓：記憶中的憑證與規格會無聲過期；連續兩次同類錯誤即停止重試、換路（改 git push）並回報，不得繼續嘗試消耗額度或觸發鎖定。
制度修改：無（judgment-rubrics.md 第一部分第 4 條已涵蓋此模式，本條為其首個實例）。

## [2026-07-11] Claude Code 全螢幕渲染器在傳統 PowerShell 主控台閃退
情境：初次設定時啟用 fullscreen renderer（研究預覽功能），程式閃退且設定被保存，直接重啟會再閃退。
教訓：解法是設環境變數 CLAUDE_CODE_DISABLE_ALTERNATE_SCREEN=1 強制傳統渲染器啟動，進去後跑 /tui default 永久改回。研究預覽功能在老舊終端機上先別開；建議改用 Windows Terminal。
制度修改：無。

## [2026-07-12] UNAUTHORIZED 根因結案：欄位名是 secret 不是 passphrase
情境：承上條。取得「部署工具庫」實際規格後比對，端點 URL 從未變更，失敗原因是記憶中的通關語欄位名記成 passphrase，實際為 secret。
教訓：規格漂移可以只差一個欄位名。整合外部端點時，第一步先打健康檢查（action=ping），分離「憑證錯」與「格式錯」兩種失敗；驗證部署一律認回傳 commit SHA，不認 Pages/raw URL（CDN 快取會誤判）。
制度修改：CLAUDE.md 硬規則 3 與高頻任務節、diagnosis.md 漏洞 3 修法 1，均已改為 commit SHA 驗證原則（2026-07-12 完成）。

## [2026-07-23] GCP 專案變更導致 Web App 授權失效

情境：在另一個對話測試 Apps Script API，將 sales-visit-system 的 GCP 從預設換成自訂專案（starlit-surge-451203-i3）。換完後 Web App 完全無法存取 Sheets，錯誤：「你沒有呼叫 SpreadsheetApp.getActiveSpreadsheet 的權限」。

教訓：
1. GCP 從預設換成自訂是**單向操作，無法還原**。換之前必須確認對線上系統的影響。
2. 換 GCP 後舊 OAuth token 失效，但不會自動跳授權視窗（token 還在，只是對錯的 GCP）。修復方法：myaccount.google.com/permissions → 撤銷 → 重新執行函數 → 允許。
3. 跨對話操作同一個系統的基礎設定（GCP、OAuth、部署）前，必須先確認另一個對話的狀況。

制度修改：sales-visit-system 事故紀錄.md 已記錄完整修復步驟。

## [2026-07-23] clasp push --force 覆蓋 GAS 線上程式碼

情境：repo 與 GAS 長期脫節（GAS 上直接修改過程式碼，未同步回 repo），在未確認的情況下執行 clasp push --force，用舊副本覆蓋了線上正在運作的程式碼。

教訓：
1. push 前必須先確認 repo 與 GAS 是否同步。若不確定，先 clasp pull 到暫存目錄比對。
2. GAS 上直接修改的程式碼必須立刻 clasp pull 回 repo，否則兩邊會脫節。
3. 還原方法：clasp pull --versionNumber N 找回特定版本。

制度修改：sales-visit-system 交接摘要已更新部署流程說明。

## [2026-07-23] GAS Web App 前後端都需要新版本部署

情境：誤以為 Index.html（前端）改動只要 clasp push 就能生效，不需要建新版本部署。

教訓：GAS Web App 的前後端都跟著部署版本走，clasp push 只是更新程式碼，Web App 要吃到新版必須建新版本部署。沒有例外。

制度修改：sales-visit-system 交接摘要已更新此規則。
