# 疑難排解

## 先跑這行自檢

```bash
curl -s -o /dev/null -w "%{http_code}\n" -X POST \
  https://mdata-mcp.invos.com.tw/invoice/mcp \
  -H "Content-Type: application/json" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
```

**預期輸出 `401`。** 這代表服務活著、而且金鑰驗證正常運作（因為這行沒帶金鑰）。

- 收到 `401` → 服務正常，問題出在你的 client 設定或金鑰，往下看。
- 收到連線錯誤或逾時 → 網路問題，或服務異常，請[開一張 issue](https://github.com/mdata-group/invoice-mcp/issues/new/choose)。

接著帶上你的金鑰再測一次：

```bash
curl -s -o /dev/null -w "%{http_code}\n" -X POST \
  https://mdata-mcp.invos.com.tw/invoice/mcp \
  -H "Authorization: Bearer inv_你的金鑰" \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-06-18","capabilities":{},"clientInfo":{"name":"curl","version":"1.0"}}}'
```

- 仍是 `401` → **金鑰的問題**（打錯、複製不全、已在 App 內刪除）。
- 變成 `200` → **金鑰沒問題，是 client 設定的問題**，看下一節。

## `/mcp` 看不到 invoice-mcp（Claude Code）

最常見的原因：`claude mcp add` 預設是 **local scope**，設定綁在你**執行指令當下的資料夾**。在家目錄執行、卻從專案資料夾開 Claude Code，設定就不會載入。

自查它到底掛在哪個資料夾、有沒有帶到金鑰：

```bash
python3 - <<'EOF'
import json, os
d = json.load(open(os.path.expanduser("~/.claude.json")))
for path, proj in (d.get("projects") or {}).items():
    m = (proj or {}).get("mcpServers") or {}
    if "invoice-mcp" in m:
        print(path, "-> 有帶 Authorization:", "Authorization" in m["invoice-mcp"].get("headers", {}))
EOF
```

輸出的路徑必須等於你開 Claude Code 的資料夾。不對的話，在**正確的資料夾**重跑一次 `claude mcp add`，或改用 `--scope user` 一勞永逸。

## 401

1. 金鑰打錯或沒複製完整。金鑰**大小寫敏感、逐字比對**，不要改動任何字元。
2. **金鑰過期** —— 建立時選的有效期限（30／90／180 天）已到。到 App 的「我的金鑰」看該筆的「有效期限」欄位；過期就重新建立一把。
3. 金鑰已在 App 內被刪除（刪除立即生效、無法復原）。
4. **金鑰被系統自動停用** —— 若該金鑰曾被公開外洩，發票存摺可能主動停用它。
5. 注意**整個服務都需要金鑰** —— 沒有有效金鑰時，連工具清單都列不出來。所以「列不出工具」第一個要懷疑的就是金鑰。

> 在 App 的「我的金鑰」列表裡，每一筆會顯示狀態標籤與「有效期限」，可以直接看出是過期、停用還是正常。

**金鑰本身沒問題，但 header 寫錯了也會回 401**，手動填 GUI 欄位時特別容易中：

6. **`Bearer` 後面少了空格。** header 值必須是 `Bearer inv_xxx` —— `Bearer` + **一個半形空格** + 金鑰。
7. **header 名稱沒填。** 有些工具的「標頭」是左右兩格，左邊要填 `Authorization`，右邊才是值。只填值那格，header 送不出去。
8. **設定裡用了 `${環境變數}`，但變數其實沒設。** 實測 Claude Code 在這種情況下**不會報錯、也不會省略 header**，而是把 `${VAR}` 這串**原文當成金鑰送出**，於是回 401。其他工具的處理方式可能不同，但症狀一樣難認 —— 先確認該變數在你啟動工具的環境裡真的有值。

## 期別被拒絕

月份只能是偶數月：`…02 / 04 / 06 / 08 / 10 / 12`。

`11505` 無效，`11506` 才對（民國 115 年 5–6 月）。詳見[MCP 工具詳表](tools.md)。

## 查詢結果金額偏少

兩個常見原因：

1. **沒翻完分頁** —— `paging.total_pages` 大於 1 時要逐頁取。
2. **只查了一個期別** —— 「今年」涵蓋 6 個期別。

明確在 prompt 裡要求「每個期別都查、每一頁都翻完」。

## App 的「我的金鑰」頁顯示「登入已過期，請重新登入後再試」

金鑰管理頁用的是一組**短效（約 30 分鐘）的管理工作階段**，而且**沒有自動續期**。頁面開著放久了、或從背景切回來，就會出現這個訊息，此時連建立金鑰都會失敗。

**解法**：退出「我的金鑰」頁，從**個人設定 › 隱私與通知 › 我的金鑰**重新進入即可（重新進入會帶入新的工作階段）。不需要重新登入 App。

> 這只影響 App 內的金鑰管理頁。**已經建立好的金鑰不受影響** —— 它的有效期限是你建立時選的 30／90／180 天。

## 資料好像不是最新的

發票資料同步有延遲。要最新資料，先在發票存摺 App 內下拉更新，再回來查詢。

## 還是不行

[開一張 issue](https://github.com/mdata-group/invoice-mcp/issues/new/choose)。

⚠️ **回報時絕對不要貼上你的金鑰。** 需要貼設定檔時，請先把 `inv_` 開頭那串換成 `inv_REDACTED`。
