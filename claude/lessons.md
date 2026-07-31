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

## [2026-07-28] Gemini grounding 與強制 JSON 模式不可並用
情境：開發「開發系統」工具時，依規格要求同時啟用 googleSearch grounding 與 responseMimeType: application/json，實際呼叫直接回 HTTP 400。
教訓：Gemini API 的 Google Search grounding 與 JSON/YAML/XML 強制輸出模式互斥，官方論壇與文件皆證實。正確做法是拆成兩段呼叫：第一段開 grounding 取得有依據的原始資料（不強制格式），第二段關閉 grounding、加 responseMimeType 做「只整理不新增」的收斂。這樣同時符合零幻覺原則（查證與格式化分離）。兩段呼叫都必須設 maxOutputTokens，否則長名單會被截斷、導致第二段 JSON.parse 失敗，錯誤訊息會誤導成「模型不聽話」。
制度修改：無（屬工具實作層，已落實於 tools/prospect-leadgen-v3.html 與 API_System_Prompt.txt）。

## [2026-07-28] AI Studio 新建金鑰一律繫結服務帳戶，與 HTTP referrer 限制互斥
情境：為降低前端金鑰外洩風險，規劃為開發系統的金鑰加上「網站限制」（限定 darciachen.github.io/*）。實際操作發現 AI Studio 新建金鑰的應用程式限制中，「網站」選項被灰掉並提示「這個選項不適用於透過服務帳戶驗證的 API 金鑰」；改從 Cloud Console 建立亦然，Gemini API 選項提示「必須使用繫結至服務帳戶的 API 金鑰，才能驗證這個 API」。
教訓：現行政策下，能呼叫 Gemini API 的金鑰強制繫結服務帳戶，而服務帳戶驗證與 browserKeyRestrictions（HTTP referrer）互斥，兩者無法並存。前端直接呼叫 Gemini 的架構「無法」用網站白名單防護，這是政策層限制，不是設定方式問題，不要再嘗試各種建立路徑繞它。替代風險控管：金鑰僅限 Gemini API（本來就是預設）、免費層無帳單風險、設定帳單預算快訊當預警、定期輪替金鑰、一個工具一把金鑰以縮小影響範圍。
制度修改：無。

## [2026-07-28] Standard API key 九月全面失效，判讀看 Bound account 欄位
情境：查證金鑰狀態時發現 Google 分兩階段淘汰舊制金鑰：2026/6/19 起拒絕未設限制的 standard key，2026/9 起拒絕所有 standard key，須改用 auth key。
教訓：判讀方法最快是看 Cloud Console「憑證」清單的 Bound account 欄位——有繫結服務帳戶者為 auth key（不受影響），顯示「—」者為 standard key（九月會失效）。注意：standard key 就算補設限制也擋不過九月那關，唯一解是改用新建的 auth key。另注意六月那關的實際執行有寬限落差（本案舊金鑰無限制卻仍可用），不可因「現在還能用」而推論安全。
制度修改：無。

## [2026-07-28] 自動化 workflow 只做 clasp push，未做 deploy，教訓未被制度化
情境：sales-visit-system 的 GitHub Actions（clasp-push.yml）長期只執行 clasp push --force，沒有 clasp deploy。推送備用金鑰功能後 Actions 顯示成功，但 Web App 實際仍跑舊版程式碼，需人工進編輯器建版本。
教訓：「clasp push ≠ 新版本部署」這條教訓早在 2026-07-23 就已記錄，但只寫進文件、沒有落實到自動化流程裡，等於教訓沒有被制度化，同一個坑再踩一次。凡是文件上的規則，都要回頭檢查對應的自動化腳本是否已實作；否則規則只在人記得時有效。修法：workflow 補上 clasp deploy -i <既有部署ID>，指定 ID 才會更新同一個網址，不指定會另建新部署產生新網址。
制度修改：clasp-push.yml 已補上 deploy 步驟（commit 6d1d4e1）。

## [2026-07-28] 部署 ID 就是 Web App 網址中的 AKfycby 那串
情境：要為 workflow 補上 clasp deploy -i 時，向 Darcia 索取部署 ID，並誤稱網址裡的 AKfycby... 不是部署 ID、應另有一組短碼。
教訓：clasp 的 deploymentId 與 Web App 網址中的識別字串就是同一個值（官方與社群設定檔範例皆為 deploymentId=AKfycb...）。此為憑印象作答導致的錯誤指路，害使用者多跑一趟。凡涉及識別碼、端點、規格格式的斷言，查證後再說；不確定時直接說不確定，比給一個看似合理的錯誤方向成本低。
制度修改：無（judgment-rubrics 既有「不得憑記憶斷言規格」原則之實例）。

## [2026-07-28] 金鑰格式 AIza→AQ.，且備援判斷漏掉 401
情境：為使用者整理待辦清單時，寫「新金鑰開頭 AIza」，被使用者指出實際是 AQ 開頭。查證後確認：舊制 Standard key 開頭 `AIza`，新制 Auth key 開頭 `AQ.Ab`，AI Studio 現在核發一律為後者。
教訓：
1. 判讀金鑰類型看開頭最快（AIza=standard、AQ.Ab=auth），比翻 Cloud Console 的 Bound account 欄位省事。
2. **連帶發現今天寫的備援邏輯有漏洞**：`isKeyInvalidError` / `_isGeminiKeyInvalid` 只判斷 400/403，但已有回報 AQ. 金鑰在原生端點回 **401 ACCESS_TOKEN_TYPE_UNSUPPORTED**。判斷條件須加入 401，否則九月主鑰失效若回 401，備援不會觸發。錯誤碼屬於外部規格，同樣不得憑常見情況推測。
3. AQ. 金鑰在原生端點（generativelanguage.googleapis.com + `?key=`）可正常運作，在 OpenAI 相容路徑會回 401。三個系統均走原生端點，不受影響。
4. 更深層的教訓（社群共識）：不要在程式裡驗證金鑰的「形狀」（如 regex 檢查 AIza 開頭），憑證格式是供應商可隨時更換的不透明字串，寫死格式等於埋下未來的故障點。本次僅將格式寫在 placeholder 提示文字（不參與驗證），屬可接受用法。
制度修改：待辦清單已列為第 3-0 項待修；tools/prospect-leadgen-v3.html 的 placeholder 已改（commit cf76a937）。

## [2026-07-31] 「顯示文字」與「實際運作值」是兩個獨立寫死的字串，改一個不會連動另一個
情境：把開發系統模型從 gemini-2.5-flash-lite 改成 gemini-3.5-flash-lite 時，只改了程式邏輯用的 `const MODEL` 常數，頁尾給人看的說明文字是另一行單獨寫死的字串，沒有同步改。使用者反覆換 Wi-Fi、清瀏覽器快取、加網址亂數參數試了五六輪，都被誤導成「快取沒更新」，實際上是我漏改一處、部署內容從頭到尾都是對的。
教訓：
1. 任何「顯示給人看的狀態文字」（頁尾版本號、標籤、說明），若不是直接讀取程式碼裡真正在用的變數，而是另外手寫一份字串，改動時必須同時搜尋兩者、逐一確認，不能只改邏輯層。優先做法：讓顯示文字直接用模板字面值插入實際變數（如 `模型 ${MODEL}`），從架構上杜絕兩者不同步的可能，而不是每次手動同步兩處。
2. 診斷快取問題時，先查「部署真的到位了嗎」要用對 API：GitHub Pages 若走 GitHub Actions 部署（workflow 名稱通常是「pages build and deployment」），舊式 `GET /repos/{owner}/{repo}/pages/builds/latest` 端點不會反映最新狀態，會一直卡在舊 commit、造成「看起來還沒建置」的假象。正確做法是查 `GET /repos/{owner}/{repo}/deployments` 搭配 `.../statuses`，這才是 Actions 型 Pages 部署的權威來源。本次因為先查錯 API，多繞了一輪才找到真正原因。
3. 使用者說「這跟你換什麼裝置、清什麼快取都沒關係，你去檢查你自己部署的內容」時，這通常是對的信號——當同一個現象在使用者已經排除多個外部變數後仍重現，優先假設是自己這邊的內容或部署有遺漏，而不是繼續要求使用者做更多環境排除。
制度修改：tools/prospect-leadgen-v3.html 頁尾文字已改為直接引用 MODEL 常數（commit 待補），避免未來再度不同步。

## [2026-07-31] Gemini API 開通計費會完全取消免費層，不是「超過免費額度才收費」
情境：評估是否為三個系統的專案開通 Google Cloud 計費，原先設想是「確認用量在免費額度內就不會花錢」，先前查證的二手部落格文章也支持這個猜測，但缺乏官方原文佐證。使用者親自到 AI Studio/Cloud Console 官方頁面確認：開通計費後，免費層會被完全取消。
教訓：Gemini API 的計費與免費層是互斥關係，不是疊加關係——不是「免費額度用完才開始算錢」，而是「一開通計費，所有用量從第一次呼叫起都是付費用量」。若使用者目的是「解鎖更高額度但不想付錢」，開通計費这条路不成立，只要是「不想付費」的前提，就應直接排除，不必再評估或折衷。
制度修改：無（屬外部規格認知修正，供未來評估額度方案時直接引用，不必重新查證）。
