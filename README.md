# line-clock

寵物美容店的 LINE LIFF 工具集。三個獨立的靜態 HTML 頁面，後端都是 Google Apps Script（GAS）+ Google Sheets。

## 頁面與後端對應

| 頁面 | 功能 | LIFF ID | GAS 部署 |
|---|---|---|---|
| `index.html` | 員工打卡（上班/下班，`?type=上班` 或 `?type=下班`） | `2009523185-nTwexRny` | 打卡 GAS（`AKfycbzXUJ7...`，與請假共用） |
| `leave.html` | 請假申請（病假/事假/其他） | `2009523185-BBsMXrnO` | 打卡 GAS（同上，`action: "leave"` 分支） |
| `customer.html` | 客戶查詢 + 結單（散客/儲值/包月，公用平板） | `2009523185-49rQ33n5` | 客戶 GAS（`AKfycby99prBN...`） |

GAS URL 直接寫在各 HTML 的 `<script>` 開頭常數區。GAS 重新部署產生新 URL 時，要同步更新對應檔案的 `GAS_URL` / `APPS_SCRIPT_URL`。

## 前端慣例

- 純靜態頁、無建置流程，CSS/JS 全部內嵌。
- POST 一律用 `text/plain` body 裝 JSON 字串，避免 CORS preflight（GAS 端自行 `JSON.parse(e.postData.contents)`）。
- `customer.html` 的美容師清單（`OPERATORS`）與服務項目（`SERVICES`）是頁內常數陣列，改名字/加項目直接改陣列即可。

## ⚠️ 後端（GAS）待辦

前端已在 payload 加入 LIFF access token，但要 GAS 端配合才真正生效：

1. **打卡/請假驗證 token**：`index.html` 與 `leave.html` 的 POST payload 現在多了 `token` 欄位（LIFF access token）。打卡 GAS 的 `doPost` 應呼叫 LINE 的 `https://api.line.me/oauth2/v2.1/verify?access_token=...` 與 `https://api.line.me/v2/profile`（帶 Bearer token）確認 token 有效、且 profile 的 userId 與 payload 的 `userId` 一致，否則拒絕寫入。沒驗證前，任何拿到 GAS URL 的人都能偽造打卡/請假。
2. **客戶資料改 POST 讀取**：客戶 GAS 的 `doPost` 加上 `action === "customers"` 分支（回傳格式同現行 `doGet`），並對未知 action 回 `{ok:false}`、不可預設當成結單寫入。完成後把 `customer.html` 的 `CUSTOMERS_VIA_POST` 改成 `true`，token 就不會再出現在 URL/伺服器 log。
3. **回應格式**：打卡/請假前端現在會讀回應——若回 JSON 且 `ok === false` 會顯示錯誤訊息（`message` 欄位）；回非 JSON 則維持舊行為視為成功。建議 GAS 統一回 `{ok: true/false, message: "..."}`。
