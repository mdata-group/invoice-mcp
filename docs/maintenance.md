# 維運說明（給本 repo 維護者）

## 金鑰洩漏處理

使用者在 issue 或 PR 內貼出自己的金鑰（`inv_` 開頭）是**可預期的常見狀況**，不是意外。看到就照以下順序處理，不要拖：

1. **立刻讓該金鑰失效** —— 該使用者的金鑰在 App 內由本人管理，因此需請對方立即自行刪除；同時評估是否需要後端強制撤銷。
2. **編輯掉留言內容**，把金鑰改成 `inv_REDACTED`。GitHub 會保留編輯歷史，因此撤銷必須先做、不能只靠編輯。
3. **回覆對方**說明金鑰已失效，請在 App 內重新建立一把。

## 支援清單同步規則

「支援哪些 client」這件事同時出現在兩個地方：

- 本 repo `README.md` 的 [設定你的 AI 工具](../README.md#2-設定你的-ai-工具)
- App 內的設定教學頁（實作在內部 repo，追蹤見下）

**兩者必須同一個 sprint 內一起改。** 這是本 repo 與 App 內容唯一的重疊項，也是唯一會漂移的地方。

### ⚠️ 不要寫「不支援清單」

2026-08-18 的教訓：README 原本列了一張「不支援的工具」表，其中「ChatGPT 不支援自訂 MCP server」**在文件還沒公開前就已經是錯的** —— ChatGPT Desktop 的「連接到自訂 MCP → 可串流 HTTP」已提供 URL、持有者權杖環境變數與自訂標頭欄位。那句話源自 2026-06 的內部文件，且把「網頁版/App」擴大解釋成整個 ChatGPT。

各家 client 的 MCP 支援每個月都在變，**任何寫死的清單都會過期，而過期的清單就是誤導**。

→ 改為**給判斷條件**：能用 streamable HTTP + 能帶自訂 `Authorization` header，就能連。清單只保留「我們實測過的」，並註明以讀者手上版本為準。

**2026-08-19 更新：連「已知確實連不了的」那張表也砍了。** 原本留的兩列後來都被證實過期 —— Claude Desktop 的自訂 connector 已有 **Request headers** 欄位（[官方文件](https://claude.com/docs/connectors/custom/remote-mcp)，Anthropic 標為 beta、分批放行），Gemini 消費者版也有了 Spark 自訂 app（需 Spark 資格且限美國）。**負面斷言的維護成本最高、危害也最大**，因為它會直接叫使用者不要嘗試。往後只列實測過的，不列連不了的。

**App 內的設定教學頁仍寫著「ChatGPT⋯尚未支援」，這句話已由實測證明為錯**（2026-08-18 實測 ChatGPT Desktop 可用，設定方式已寫入 `README.md`）。**正確處置是把那句話刪掉，不是改成「已支援」** —— 改成後者等於承諾要永久追蹤第三方的支援狀態。該改動不在本 repo，需與 PM 確認文案（追蹤卡在內部 issue tracker）。

### ⚠️ 非必要不列舉 AI 工具名

上一條的另一面，同一個邏輯。判斷標準：**把工具名拿掉之後，讀者還知道這句適用於他嗎？**

- **還知道 → 具名是多餘的，拿掉。** 例：README 開頭原本寫「讓 Claude / Codex / Gemini 直接讀你的發票資料回答」，但同一句前面已經說「用你自己的 AI 工具」—— 列舉純屬重複，而且 Gemini CLI 停服時逼我們回來改了一次。
- **不知道了 → 具名在承載資訊，留著。** 例：「多半是 Claude Code 的資料夾 scope 問題」—— 拿掉會讓其他工具的使用者去找一個不存在的問題。

具名只該出現在兩種地方：**安裝指令**，以及**某工具特有的行為或坑**。其餘一律寫「你的 AI 工具」。

**封閉列舉改成舉例也算。** 隱私條款原本寫「（Anthropic／OpenAI／Google）」，已改為「（例如 Anthropic、OpenAI、Google）」—— 讀者可能用清單外的工具。

## 稱謂規則

對使用者說話一律用「**發票存摺**」或「**我們**」／「**本服務**」，**不要用「麻布」或法人名**。整份文件的讀者是發票存摺的使用者，突然出現另一個品牌名或法人名，讀者連不起來。

2026-08-19 的教訓：`README.md` 的隱私條款原本寫「麻布不經手、也不負擔這段推理費用」，是全 repo 唯一用「麻布」的地方（其餘一律「我們」／「本服務」），已改為「發票存摺不經手」。

例外只有 `LICENSE` 的 Copyright 行 —— 那一行的讀者是要轉載的人與法務，功能是指認權利人，必須寫**法人正式名稱**，不是品牌名。

## 內容 SoT 規則

- 本 repo 的 `README.md` 是安裝說明的**唯一 SoT**。
- 行銷 blog **不得抄錄安裝指令原文**，一律連回本 repo。抄過去就一定會漂移。
- `microservice/invoice_mcp/CLIENT-QUICKSTART.md` 已收斂為一行連結，不要在那裡重新長出內容。

## endpoint 變更

`.github/workflows/endpoint-liveness.yml` 每日檢查 endpoint 是否回 401。CI 轉紅時代表 endpoint URL 變更或服務異常 —— 先確認 URL，再同步更新 `README.md`、`docs/troubleshooting.md`、`server.json` 三處。

### ⚠️ 不要讓 endpoint 廣告 OAuth

2026-08-18 實測：三個 `/.well-known/oauth-*` 路徑都回 **404**，401 回應也**不帶** `WWW-Authenticate`。**這是靜態金鑰體驗正常運作的前提，不要「為了符合 MCP 規格」去補上。**

一旦補了任何一項，client 就會開始探測 OAuth —— Claude Code 會把明明連得上的 server 顯示成「需要認證」並跳出用不到的 OAuth 流程（[claude-code#17152](https://github.com/anthropics/claude-code/issues/17152)），使用者體驗直接退化。

## 關於 `server.json`

`server.json` 是官方 MCP Registry 的發布用 metadata，**目前尚未發布**，純屬預留。

不上 registry 的理由：本服務需要台灣「發票存摺」App 帳號，registry 上看到它的絕大多數是裝不了的國際使用者；發布 registry 需要建立簽章金鑰並變更 production DNS，成本與收益不成比例。

### ⚠️ 「靠 topics 讓目錄站收錄」是錯的

2026-08-18 查證：本節原本寫「目錄站（glama、mcp.so 等）主要靠自動爬 GitHub、讀 repo topics 收錄」，**這個機制不存在**。

- **Glama** 的 [收錄方法頁](https://glama.ai/mcp/methodology)寫明：上架靠 **maintainer 主動提交**，且提交者需通過 GitHub OAuth 驗證他對該 repo 有 write/admin 權限。沒有自動爬 GitHub，也不讀 topics。
- **mcp.so／mcpservers.org** 這類則是靠 submit，或定期 re-scan 上游的 `awesome-mcp-servers` 清單。要被自動收錄，路徑是先進 awesome list，不是設 topics。
- 更麻煩的是 Glama 的排序訊號 **TDQS** 評的是 **tool 的 JSON Schema**、不是 README；遠端 connector 要被 introspect，需 maintainer 另外提供一組 sandbox 憑證。**本服務沒有金鑰連 `tools/list` 都回 401，所以即使送上去也抓不到 tool 定義、評不了分。**

因此：repo topics（`mcp` `mcp-server` `taiwan` `einvoice` `invoice` `claude-code`）**對目錄站收錄大概沒有作用**，留著無害但不要當成收錄手段。目錄站上架是主動動作，且前置條件是先備好 sandbox 金鑰與測試帳號 —— 屬通路／行銷範圍，不在本 repo 的工作範圍內。

日後若要發布，namespace 為 `tw.com.invos/invoice-mcp`（DNS 驗證），需在 `invos.com.tw` 的 **apex** 加 TXT record（不可放在 selector 下），並以 `mcp-publisher` CLI 發布。上面兩段已包含發布所需的關鍵資訊。

### ⚠️ 別把 Anthropic Directory 和 MCP Registry 混為一談

上面「要建簽章金鑰 + 改 production DNS」的成本**只適用於 open MCP Registry**。[Anthropic 官方文件](https://claude.com/docs/connectors/directory)明寫：「No domain-ownership proof (DNS or `.well-known`) is required—that requirement applies only to the open MCP Registry, not the Anthropic Directory.」

上 Anthropic Connectors Directory 只需要 Team/Enterprise org 與 directory 管理權限，**不需要 DNS 驗證**。

不過**結論不變（兩個都不做）**，理由是：Directory connector 與 custom connector「run on the same infrastructure」，進 Directory 不會解決認證問題；且受眾同樣是絕大多數裝不了的國際使用者。**理由要寫對，別讓下一個人以為「因為 DNS 成本高所以不能上 Directory」。**

## 連結規範

`.github/` 底下的連結**一律用絕對 URL**。issue 與 PR 模板會被渲染在 repo 之外的頁面（issue／PR 本文），相對路徑在那裡是相對 `https://github.com/mdata-group/invoice-mcp/` 解析的，`../../docs/x.md` 會爬到 `https://github.com/mdata-group/docs/x.md`。

2026-08-18 的教訓：`ISSUE_TEMPLATE/install-problem.md` 的「我已經跑過疑難排解的自檢指令」曾用 `../../docs/troubleshooting.md`，導致**每一張用該模板開出來的 issue，那個連結都是死的** —— 而它正是使用者自助分流的唯一入口。同目錄的 `config.yml` 一直是對的（絕對 blob URL），所以同一個目錄裡並存過兩種連結基準。

repo 內的 `README.md`、`docs/`、`skills/` 之間用相對路徑即可，那些檔案只在 repo 內被瀏覽。

## 截圖規格

換圖時照這五條，否則會出現三張尺寸不一的圖：

1. **來源檔寬度 600px**（`sips --resampleWidth 600`）。iPhone @3x 原檔是 1206px，直接放會超出需求三倍。
2. **顯示寬度一律鎖 300px**，用 `<img src="..." alt="..." width="300">`，不要用 `![]()` 語法。300px 是單張的寬度，兩欄表格內並排合計約 600px。
3. **不鎖寬度的後果**：`![]()` 的獨立圖片會撐滿 README 內容寬（約 830px），1:2.17 的手機截圖會渲染成約 1800px 高 —— 一張圖佔兩個螢幕，把安裝指令推到很後面。2026-08-18 的 `03-save-key.png` 就是這樣。
4. **檔名 `NN-用途.png`**，兩位數序號照閱讀順序。
5. **alt 用中文描述操作路徑**（例如「個人設定 › 隱私與通知」），不要用檔名或「screenshot」。

換圖後同步更新「1. 拿金鑰」末尾的版本註記（目前為 iOS 2026-08 / App v7.43.2）。

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

### `CONTRIBUTING.md`：唯一過半而我們刻意不做的

普及率 18/34 = 53%，過半。但我們**不收外部 PR** —— 做一份 `CONTRIBUTING.md` 會直接暗示歡迎貢獻，是誤導。

所以只取「協作政策要前置可見」這一半、不取「新增檔案」那一半：政策一行寫在 `README.md` 的「問題回報」章節。

2026-08-19 之前這條政策只寫在 `.github/pull_request_template.md` 的 HTML 註解裡 —— 那只有**已經 fork 完、改完、正要按下開 PR** 的人才看得到，等於把政策放在成本已經付出之後。兩處措辭要一致，改一邊記得改另一邊。

GitHub 的 community profile 會因此永遠顯示 `contributing: ✗`，那是預期的，不要為了那個勾去補檔案。（同一份 profile 的 `issue_template: ✗` 是誤判 —— 它只認舊式的單一 `.github/ISSUE_TEMPLATE.md`，不認我們用的目錄式。）

反過來，我們有兩項是慣例外的自訂風格，留著但知道自己在偏離：**章節編號** 1/34 = 3%（安裝指南用序號幫讀者定位，但交叉引用一律走 anchor、不綁號碼，見上方「連結規範」）、**疑難排解拆檔** 1/34 = 3%（README 維持索引角色）。
