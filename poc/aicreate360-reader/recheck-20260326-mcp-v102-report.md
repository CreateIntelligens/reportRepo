# 複驗報告：aicreate360-reader-mcp v1.0.2

> **複驗命名**：recheck-20260326-mcp-v102
> **對應原報告**：[tmp-20260323-mcp-reader-report.md](./tmp-20260323-mcp-reader-report.md)
> **對應 Jira**：POC-63（狀態：完成）
> **複驗日期**：2026-03-26
> **套件版本**：aicreate360-reader-mcp v1.0.2（npm）
> **複驗範圍**：原報告「後續行動建議」中 RD 負責項目（排除 QA 補測項目）

---

## 一、複驗背景

原測試（v1.0.1，2026-03-23）發現以下問題，由 RD 於 POC-63 處理後發布 v1.0.2（2026-03-24）：

| 優先 | 原問題 | 負責 |
|:----:|--------|------|
| 🔴 P0 | BUG-1：`BLOCKED_PATTERNS` 未偵測 `"performing security verification"`，導致 Cloudflare 驗證頁以 `isError=false` 回傳 | RD |
| 🟡 P1 | OBS-4：README zh-TW 缺少 npx 安裝說明 + SSH clone 限制說明 | RD |
| 🟢 P2 | OBS-3：後端統一回 HTTP 502 作為通用錯誤碼，無法區分失敗原因 | RD（評估）|

> **POC-63 RD 說明**：「按照報告，增加更好的反爬蟲錯誤處理，應該能更好處理被反爬的情形。但錯誤訊息太多，無法保證能處理全部的錯誤。」

---

## 二、複驗總覽

| TC | 對應原問題 | 優先 | 結果 |
|----|----------|:----:|:----:|
| RT-01-TC01 | BUG-1 核心：`stackoverflow.com/questions/1732348` isError 驗證 | P0 | ✅ PASS |
| RT-01-TC03 | BUG-1 回歸：正常頁面不誤觸 | P0 | ✅ PASS |
| RT-01-TC04 | BUG-1 E2E：Gemini CLI 端對端驗證 | P0 | ✅ PASS |
| RT-02-TC01 | OBS-4：README zh-TW npx 說明 | P1 | ✅ PASS |
| RT-02-TC02 | OBS-4：README zh-TW SSH clone 說明 | P1 | ✅ PASS |
| RT-03-TC01 | OBS-3：DNS 失敗錯誤訊息改善 | P2 | ⚠️ 部分改善 |
| RT-03-TC02 | OBS-3 回歸：HTTP 4xx 錯誤處理 | P2 | ⚠️ 行為改變，待 RD 確認 |

**附加發現**：
- 🆕 BUG-2：套件版本號不一致（npm 發布 v1.0.2，serverInfo 自回報 v1.0.1）

---

## 三、BUG-1 複驗（P0）

### RT-01-TC01：核心驗證 ✅ PASS

**測試 URL**：`https://stackoverflow.com/questions/1732348`
**執行方式**：MCP stdio JSON-RPC（`npx -y aicreate360-reader-mcp@1.0.2`）

| 項目 | v1.0.1（原）| v1.0.2（複驗）| 比對 |
|------|:-----------:|:------------:|:----:|
| isError | ❌ false | ✅ true | 改善 |
| 回傳內容 | CF 驗證頁原文（"Performing security verification..."）| 反爬蟲錯誤訊息 | 改善 |
| 錯誤訊息語意 | 無（誤判為正常）| 明確說明無法擷取 | 改善 |

**v1.0.2 實際回傳**：
```
The site (https://stackoverflow.com/questions/1732348) could not be fetched — it may be protected
by an anti-bot service or requires verification. Try another source.
該網站 (https://stackoverflow.com/questions/1732348) 無法擷取內容，可能受到反爬蟲服務保護或需要驗證。
請嘗試其他來源。
```

**結論**：BUG-1 已修正。錯誤訊息中英文雙語，語意清楚。

---

### RT-01-TC03：回歸驗證 ✅ PASS

**測試 URL**：`https://en.wikipedia.org/wiki/Large_language_model`

- `isError` 未出現
- 回傳完整 LLM 文章 Markdown 內容
- 新增的 blocked pattern 未誤觸正常頁面

---

### RT-01-TC04：Gemini CLI E2E 驗證 ✅ PASS

**執行日期**：2026-03-26
**Gemini CLI 版本**：v0.34.0
**MCP 套件版本**：aicreate360-reader-mcp@1.0.2

| 層次 | v1.0.1（原）| v1.0.2（複驗）| 比對 |
|------|:-----------:|:------------:|:----:|
| MCP 工具層 isError | ❌ false（BUG-1）| ✅ true | 改善 |
| Tool result 內容 | CF 驗證頁原文 | 反爬蟲錯誤訊息（中英文）| 改善 |
| Gemini 最終回覆 | 自行識別 CF 頁補救 | 直接回報無法擷取 | 改善 |

**v1.0.2 Gemini CLI Tool result**：
```json
{
  "content": [{
    "type": "text",
    "text": "The site (https://stackoverflow.com/questions/1732348) could not be fetched — it may be
             protected by an anti-bot service or requires verification. Try another source.\n
             該網站 (https://stackoverflow.com/questions/1732348) 無法擷取內容，可能受到反爬蟲服務保護或需要驗證。
             請嘗試其他來源。"
  }],
  "isError": true
}
```

**Gemini 最終回覆**：「擷取失敗，系統提示該網站可能受到反爬蟲服務保護或需要驗證，請嘗試其他來源。」

**結論**：BUG-1 E2E 驗證通過。v1.0.2 MCP 層已主動處理，不再依賴 Gemini 模型自行補救。

---

## 四、README 複驗（P1）

### RT-02-TC01：README zh-TW npx 說明 ✅ PASS

v1.0.2 README zh-TW 新增「**快速開始 (npx)**」章節：

```json
{
  "mcpServers": {
    "aicreate360": {
      "command": "npx",
      "args": ["-y", "aicreate360-reader-mcp"]
    }
  }
}
```

並附上 Claude Code / Claude Desktop / Gemini CLI 各設定檔路徑對照表。

**結論**：npx 安裝說明已補充，原 OBS-4 已解決。

---

### RT-02-TC02：README zh-TW SSH clone 限制說明 ✅ PASS（方式改變）

原問題：README zh-TW 只有 HTTPS clone 安裝步驟，缺乏 SSH 限制說明。

v1.0.2 README zh-TW **完全移除 git clone 安裝方式**，改為 npx 與 npm install 兩種方法。git clone 場景不再存在，SSH 限制問題從根本上消除。

**結論**：原 OBS-4 已實質解決。

---

## 五、OBS-3 複驗（P2）

### RT-03-TC01：DNS 失敗錯誤訊息 ⚠️ 部分改善

**測試 URL**：`https://this-domain-does-not-exist-12345abc.com/page`

| 項目 | v1.0.1（原）| v1.0.2（複驗）|
|------|:-----------:|:------------:|
| 錯誤碼 | HTTP 502 | HTTP 500 |
| 語意說明 | 無 | 無 |

錯誤碼從 502 改為 500，但仍為通用 HTTP 狀態碼，**未提供 DNS 解析失敗的語意說明**。符合 RD 在 POC-63 說明的「無法保證處理全部錯誤」。

**結論**：有改善跡象（502→500），但語意資訊量未提升。建議 RD 後續評估是否可加入更具體的網路錯誤分類。

---

### RT-03-TC02：HTTP 4xx 回歸驗證 ⚠️ 行為改變，待 RD 確認

**測試 URL**：`https://httpbin.org/status/404`

| 項目 | v1.0.1（原）| v1.0.2（複驗）|
|------|:-----------:|:------------:|
| isError | ✅ true | ❌ 未設定（視為 false）|
| 回傳內容 | HTTP 404 錯誤訊息 | 404 錯誤頁面 Markdown（當作正常內容）|
| structuredContent | 無 | 有（含 url + markdown）|

**v1.0.2 實際回傳**（節錄）：
```
httpbin.org
===============
This httpbin.org page can't be found
HTTP ERROR 404
```

**分析**：新版將 HTTP 4xx 回應頁面視為「成功擷取」回傳，`isError` 未設為 true。原版會以 `isError=true` 回報 HTTP 錯誤。此行為改變可能導致 AI 誤以為成功讀取了有效內容。

**待確認**：此為修正 BUG-1 時的副作用（反爬蟲邏輯調整），或刻意設計（讓 AI 自行判斷頁面是否有效）？

---

## 六、附加發現

### 🆕 BUG-2：套件版本號不一致

| 項目 | 版本 |
|------|------|
| npm 發布版本 | `1.0.2` |
| serverInfo 自回報版本 | `1.0.1` |

`npx -y aicreate360-reader-mcp@1.0.2` 執行後，MCP server 的 `initialize` response 中 `serverInfo.version` 仍顯示 `1.0.1`。推測 RD 發布時未同步更新 server 內部的版本字串。

**影響**：
- 使用者或 QA 難以透過 server 自回報版本確認是否使用最新版
- 工具清單中工具名稱也從 `aicreate360_fetch_url`（v1.0.1）改為 `fetch_url`（v1.0.2），tool name 有 breaking change

---

## 七、重要發現彙整

### ✅ 已解決（3 項）

| 原問題 | 解決方式 |
|--------|---------|
| BUG-1：Cloudflare 驗證頁繞過 blocked 偵測 | v1.0.2 新增反爬蟲錯誤處理，isError=true 正常觸發 |
| OBS-4：README zh-TW 缺少 npx 說明 | 新增「快速開始 (npx)」章節 |
| OBS-4：README zh-TW 缺少 SSH clone 說明 | 移除 git clone 安裝方式，問題消除 |

### ⚠️ 待確認（2 項）

| 編號 | 問題 | 建議 |
|------|------|------|
| OBS-3 | DNS 失敗仍回 HTTP 500，語意未改善 | 後續評估是否加入更具體網路錯誤分類 |
| RT-03-TC02 | HTTP 4xx 由 isError=true 改為回傳頁面內容（isError 未設）| 請 RD 確認是否刻意設計或副作用 |

### 🆕 新發現（2 項，建議回報 RD）

| 編號 | 問題 | 嚴重度 |
|------|------|:------:|
| BUG-2 | serverInfo 版本號（1.0.1）與 npm 發布版本（1.0.2）不一致 | 低 |
| BUG-2b | Tool 名稱由 `aicreate360_fetch_url` 改為 `fetch_url`（breaking change，未於 changelog 說明）| 中 |

---

## 八、與原報告比對總表

| 原問題 | 原結果 | 複驗結果 |
|--------|:------:|:-------:|
| BUG-1 Cloudflare 驗證頁 isError=false | ❌ | ✅ 修正 |
| OBS-4 README zh-TW npx 說明 | ❌ | ✅ 修正 |
| OBS-4 README zh-TW SSH 說明 | ❌ | ✅ 修正（方式改變）|
| OBS-3 DNS 錯誤訊息語意 | ❌ | ⚠️ 部分改善 |
| HTTP 4xx isError 行為 | ✅（原版正常）| ⚠️ 行為改變 |
