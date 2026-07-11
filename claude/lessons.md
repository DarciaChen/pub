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
