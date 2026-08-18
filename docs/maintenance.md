# 維運說明（給本 repo 維護者）

## 金鑰洩漏處理

使用者在 issue 或 PR 內貼出自己的金鑰（`inv_` 開頭）是**可預期的常見狀況**，不是意外。看到就照以下順序處理，不要拖：

1. **立刻讓該金鑰失效** —— 該使用者的金鑰在 App 內由本人管理，因此需請對方立即自行刪除；同時評估是否需要後端強制撤銷。
2. **編輯掉留言內容**，把金鑰改成 `inv_REDACTED`。GitHub 會保留編輯歷史，因此撤銷必須先做、不能只靠編輯。
3. **回覆對方**說明金鑰已失效，請在 App 內重新建立一把。

## 支援清單同步規則

「支援哪些 client」這件事同時出現在兩個地方：

- 本 repo `README.md` 的 [設定你的 AI 工具](../README.md#2-設定你的-ai-工具)
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

不上 registry 的理由：本服務需要台灣「發票存摺」App 帳號，registry 上看到它的絕大多數是裝不了的國際使用者；發布 registry 需要建立簽章金鑰並變更 production DNS，成本與收益不成比例。

### ⚠️ 「靠 topics 讓目錄站收錄」是錯的

2026-08-18 查證：本節原本寫「目錄站（glama、mcp.so 等）主要靠自動爬 GitHub、讀 repo topics 收錄」，**這個機制不存在**。

- **Glama** 的 [收錄方法頁](https://glama.ai/mcp/methodology)寫明：上架靠 **maintainer 主動提交**，且提交者需通過 GitHub OAuth 驗證他對該 repo 有 write/admin 權限。沒有自動爬 GitHub，也不讀 topics。
- **mcp.so／mcpservers.org** 這類則是靠 submit，或定期 re-scan 上游的 `awesome-mcp-servers` 清單。要被自動收錄，路徑是先進 awesome list，不是設 topics。
- 更麻煩的是 Glama 的排序訊號 **TDQS** 評的是 **tool 的 JSON Schema**、不是 README；遠端 connector 要被 introspect，需 maintainer 另外提供一組 sandbox 憑證。**本服務沒有金鑰連 `tools/list` 都回 401，所以即使送上去也抓不到 tool 定義、評不了分。**

因此：repo topics（`mcp` `mcp-server` `taiwan` `einvoice` `invoice` `claude-code`）**對目錄站收錄大概沒有作用**，留著無害但不要當成收錄手段。目錄站上架是主動動作，且前置條件是先備好 sandbox 金鑰與測試帳號 —— 屬通路／行銷範圍，不在本 repo 的工作範圍內。

日後若要發布，namespace 為 `tw.com.invos/invoice-mcp`（DNS 驗證），需在 `invos.com.tw` 的 **apex** 加 TXT record（不可放在 selector 下），並以 `mcp-publisher` CLI 發布。詳見 workspace 內的 `docs/superpowers/specs/2026-08-17-invoice-mcp-public-repo-design.md` §7。

## 連結規範

`.github/` 底下的連結**一律用絕對 URL**。issue 與 PR 模板會被渲染在 repo 之外的頁面（issue／PR 本文），相對路徑在那裡是相對 `https://github.com/mdata-group/invoice-mcp/` 解析的，`../../docs/x.md` 會爬到 `https://github.com/mdata-group/docs/x.md`。

2026-08-18 的教訓：`ISSUE_TEMPLATE/install-problem.md` 的「我已經跑過疑難排解的自檢指令」曾用 `../../docs/troubleshooting.md`，導致**每一張用該模板開出來的 issue，那個連結都是死的** —— 而它正是使用者自助分流的唯一入口。同目錄的 `config.yml` 一直是對的（絕對 blob URL），所以同一個目錄裡並存過兩種連結基準。

repo 內的 `README.md`、`docs/`、`skills/` 之間用相對路徑即可，那些檔案只在 repo 內被瀏覽。

## 刻意不做的事

2026-08-18 對標 34 個熱門專案（高星 MCP server、需自備憑證的 MCP server、面向消費者的開發者工具、非英語專案四層各約 9 個）的實測普及率。下面每一項都**低於 50%**，也就是不是慣例，因此刻意不做。

**看到這節請不要「順手補齊」。** 空目錄、只有 header 的 CHANGELOG、罐頭 CODE_OF_CONDUCT 會讓專案看起來像抄模板；說得出「知道有這些慣例、也知道普及率、並且說得出為什麼不做」才是有人在管。

| 項目 | 普及率 | 不做的理由 |
| --- | --- | --- |
| `CODE_OF_CONDUCT.md` | 8/34 = 24% | 維護者只有內部團隊，且不收外部 PR |
| `CHANGELOG.md` | 12/34 = 35% | 改用 GitHub Release notes |
| `examples/` 目錄 | 3/34 = 9% | 只有兩個工具，範例已全在[範例 Prompt](prompts.md) |
| demo GIF | 7/34 = 21% | 沒有金鑰連 `tools/list` 都回 401，無法錄不含真實發票的公開試玩 |
| 教學影片 | 4/34 = 12% | 同上，且截圖已足夠說明 App 內取金鑰流程 |
| FAQ 章節 | 3/34 = 9% | 內容已落在[疑難排解](troubleshooting.md) |
| `docs/` index | 9/20 = 45% | 只有 4 個檔，README 已直連全部 |
| 上官方 registry／目錄站 | —— | 見上方「關於 `server.json`」 |

同一批統計裡**確實過半、而我們照做**的：LICENSE 88%、標題層級無跳級 85%、badge 79%、issue template 53%。

反過來，我們有兩項是慣例外的自訂風格，留著但知道自己在偏離：**章節編號** 1/34 = 3%（安裝指南用序號幫讀者定位，但交叉引用一律走 anchor、不綁號碼，見上方「連結規範」）、**疑難排解拆檔** 1/34 = 3%（README 維持索引角色）。
