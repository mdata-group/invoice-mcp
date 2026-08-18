# 維運說明（給本 repo 維護者）

## 金鑰洩漏處理

使用者在 issue 或 PR 內貼出自己的金鑰（`inv_` 開頭）是**可預期的常見狀況**，不是意外。看到就照以下順序處理，不要拖：

1. **立刻讓該金鑰失效** —— 該使用者的金鑰在 App 內由本人管理，因此需請對方立即自行刪除；同時評估是否需要後端強制撤銷。
2. **編輯掉留言內容**，把金鑰改成 `inv_REDACTED`。GitHub 會保留編輯歷史，因此撤銷必須先做、不能只靠編輯。
3. **回覆對方**說明金鑰已失效，請在 App 內重新建立一把。

## 支援清單同步規則

「支援哪些 client」這件事同時出現在兩個地方：

- 本 repo 的 `README.md` §2
- App 內設定教學頁（`mdata-web/apps/mcp-tokens/src/screens/TutorialPage.tsx`）

**兩者必須同一個 sprint 內一起改。** 這是本 repo 與 App 內容唯一的重疊項，也是唯一會漂移的地方。

### ⚠️ 不要寫「不支援清單」

2026-08-18 的教訓：README 原本列了一張「不支援的工具」表，其中「ChatGPT 不支援自訂 MCP server」**在文件還沒公開前就已經是錯的** —— ChatGPT Desktop 的「連接到自訂 MCP → 可串流 HTTP」已提供 URL、持有者權杖環境變數與自訂標頭欄位。那句話源自 2026-06 的內部文件，且把「網頁版/App」擴大解釋成整個 ChatGPT。

各家 client 的 MCP 支援每個月都在變，**任何寫死的清單都會過期，而過期的清單就是誤導**。

→ 改為**給判斷條件**：能用 streamable HTTP + 能帶自訂 `Authorization` header，就能連。清單只保留「我們實測過的」與「已知確實連不了的」，並註明以讀者手上版本為準。

**App 內 `TutorialPage` 目前仍寫著「ChatGPT⋯尚未支援」，同樣需要更正** —— 那句話面向的使用者更多，優先度不低於 README。改動屬 `mdata-web`，需與 PM 確認文案。

## 內容 SoT 規則

- 本 repo 的 `README.md` 是安裝說明的**唯一 SoT**。
- 行銷 blog **不得抄錄安裝指令原文**，一律連回本 repo。抄過去就一定會漂移。
- `microservice/invoice_mcp/CLIENT-QUICKSTART.md` 已收斂為一行連結，不要在那裡重新長出內容。

## endpoint 變更

`.github/workflows/endpoint-liveness.yml` 每日檢查 endpoint 是否回 401。CI 轉紅時代表 endpoint URL 變更或服務異常 —— 先確認 URL，再同步更新 `README.md`、`docs/troubleshooting.md`、`server.json` 三處。

## 關於 `server.json`

`server.json` 是官方 MCP Registry 的發布用 metadata，**目前尚未發布**，純屬預留。

不上 registry 的理由：本服務需要台灣「發票存摺」App 帳號，registry 上看到它的絕大多數是裝不了的國際使用者；而目錄站（glama、mcp.so 等）主要靠**自動爬 GitHub、讀 repo topics** 收錄，不經過 registry。發布 registry 需要建立簽章金鑰並變更 production DNS，成本與收益不成比例。

日後若要發布，namespace 為 `tw.com.invos/invoice-mcp`（DNS 驗證），需在 `invos.com.tw` 的 **apex** 加 TXT record（不可放在 selector 下），並以 `mcp-publisher` CLI 發布。詳見 workspace 內的 `docs/superpowers/specs/2026-08-17-invoice-mcp-public-repo-design.md` §7。
