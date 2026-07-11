# 部署工具庫（DarciaChen/pub）完整規格

## 端點
POST https://script.google.com/macros/s/AKfycbyB17GVVPNeipEggfPyW7Kvni_uTmo2IuaaKjF-2VRuEwdeNoVMfwO8PnZGmQV4bcWqeg/exec
Content-Type: application/json

## 請求格式
```json
{
  "secret": "<通關語>",
  "action": "deploy",
  "path": "檔名.html",
  "content": "<完整檔案內容字串>",
  "message": "commit 訊息",
  "entry": {
    "type": "...", "name": "...",
    "file": "..." 或 "url": "...",
    "date": "...", "desc": "..."
  }
}
```
- 通關語欄位名是 `secret`（實測 passphrase/pass/key 等均回 UNAUTHORIZED）。**通關語本身不寫在這份公開文件裡**，由擁有者在對話中提供
- `entry` 選填；僅新工具需要登錄 index.html 時才帶。省略時只更新檔案內容，不動 index.html（回應顯示 `skipped: "no entry provided"`）

## 健康檢查
`{"secret":"<通關語>","action":"ping"}` → `{"ok":true,"pong":"<ISO時間戳>"}`

## 成功回應
```json
{
  "ok": true,
  "upload": {
    "ok": true,
    "path": "piano-guitar.html",
    "commit": "22d49042d2094e331d74507799011be992170d34",
    "created": false
  },
  "register": { "ok": true, "skipped": "no entry provided" },
  "pagesUrl": "https://darciachen.github.io/pub/piano-guitar.html"
}
```
- 以 `upload.commit` SHA 為部署成功的**唯一依據**
- `created`：false=更新既有檔、true=新建檔
- `pagesUrl` 僅供參考，**勿用於驗證**（CDN 快取會誤判）

## 失敗回應
`{"ok":false,"error":"UNAUTHORIZED"}` ← secret 錯誤或欄位名不對

## curl 注意事項（Apps Script 特性）
- Apps Script 會 302 轉址到 script.googleusercontent.com；curl 加 `-L` 會把 POST 轉成 GET 導致失敗
- 正確做法：POST 不帶 `-L`，從回應 header 取 Location，再對該網址發 GET 拿 JSON 結果：
```bash
LOC=$(curl -s -D - -o /dev/null -X POST -H "Content-Type: application/json" \
  --data-binary @payload.json "<端點URL>" | grep -i '^location:' | sed 's/^location: //I' | tr -d '\r\n')
curl -s "$LOC"
```
- 大檔（實測 1.36MB payload）可正常部署
- 驗證原則：只認回傳的 commit SHA；不要用 GitHub Pages URL 或 raw.githubusercontent.com 驗證（CDN 快取）
