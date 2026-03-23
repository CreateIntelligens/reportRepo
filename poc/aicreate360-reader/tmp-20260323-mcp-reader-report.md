# 測試報告：aicreate360-reader-mcp v1.0.1

> **測試命名**：tmp-20260323-mcp-reader
> **套件**：[aicreate360-reader-mcp v1.0.1](https://www.npmjs.com/package/aicreate360-reader-mcp)
> **執行日期**：2026-03-23
> **報告產出**：2026-03-23
> **報告狀態**：M1–M5 大部分完成；M5-TC05 Copilot CLI 待四月額度恢復後補測

---

## 一、套件基本資訊

| 項目 | 說明 |
|------|------|
| 套件名稱 | `aicreate360-reader-mcp` |
| 版本 | 1.0.1 |
| 授權 | MIT |
| 後端服務 | `https://create360.ai`（同 POC-60 skill，共用後端）|
| 唯一工具 | `aicreate360_fetch_url(url: string)` |
| 回傳格式 | `content[{type:"text", text: markdown}]` + `structuredContent {url, markdown}` |
| 錯誤旗標 | `isError: true` + 錯誤訊息 |
| 啟動方式 | `npx -y aicreate360-reader-mcp`（無需安裝）|
| Runtime 需求 | Node.js >= 24 |
| 傳輸層 | MCP stdio（Newline-delimited JSON-RPC 2.0）|
| Timeout | 20,000ms（hardcoded）|

---

## 二、測試總覽

| 分類 | ✅ 通過 | ❌ 失敗 | ⬜ 待補 | ⚪ 跳過 | 合計 |
|------|:------:|:------:|:------:|:------:|:----:|
| M1 安裝與環境 | 3 | 0 | 0 | 1 | 4 |
| M2 MCP 協議層 | 5 | 1 | 0 | 0 | 6 |
| M3 工具功能 | 5 | 0 | 0 | 0 | 5 |
| M4 錯誤處理 | 4 | 1 | 0 | 0 | 5 |
| M5 多 CLI 整合 | 4 | 0 | 1 | 0 | 5 |
| **合計** | **21** | **2** | **1** | **1** | **25** |

> M4-TC02（Timeout）未執行，不列入計數。

---

## 三、M1 安裝與環境測試

### M1-TC01：git clone + pnpm build ✅

| 步驟 | 結果 |
|------|:----:|
| pnpm install（376 packages）| ✅ |
| pnpm build（tsc）| ✅ |
| dist/ 產出完整 | ✅ |
| node dist/index.js initialize | ✅ |

**新發現**：HTTPS clone 失敗（`Password authentication is not supported`），需改用 SSH。README 未說明 repo 需 org 成員存取。

### M1-TC02：npx 安裝 ✅
`npx -y aicreate360-reader-mcp` 直接啟動，無需 clone + build。README(EN) 有記載，README(zh-TW) 缺少（OBS-4）。

### M1-TC04：Build 產出完整性 ✅
`dist/` 含所有必要模組：`index.js` / `constants.js` / `schemas.js` / `types.js` / `services/source-fetcher.js`

### M1-TC05：vitest 覆蓋率 ✅

| 指標 | 實際 | 門檻 |
|------|:----:|:----:|
| Statements | **100%** | 80% |
| Branches | **100%** | 80% |
| Functions | **100%** | 80% |
| Lines | **100%** | 80% |

14 tests（2 test files）全部通過，覆蓋率滿分。

### M1-TC03：Node.js 版本限制 ⚪（跳過）
測試機為 Node.js v25.7.0，無法在同環境切換 < 24 版本驗證。

---

## 四、M2 MCP 協議層測試

### ✅ 通過（5/6）

| TC | 測試重點 | 結論 |
|----|---------|------|
| M2-TC01 | Server 啟動 | `initialize` 正常，serverInfo 版本一致 |
| M2-TC02 | tools/list | 1 個工具，annotations 全通過（readOnly / notDestructive / idempotent / openWorld）|
| M2-TC03 | 合法 URL 呼叫 | `isError` 不出現，content 非空 |
| M2-TC04 | 非法 URL 攔截 | `"not-a-url"` / `""` / `"ftp://"` 全被 zod schema 攔截 |
| M2-TC05 | structuredContent | `{url, markdown}` 結構正確 |

### ❌ M2-TC06：isError 旗標 — 失敗（→ BUG-1）

**URL**：`https://stackoverflow.com/questions/1732348`

**現象**：後端取回 Cloudflare 驗證頁面，但 `detectBlocked()` 未偵測到，回傳 `isError=false`。

```
Performing security verification
This website uses a security service to protect against malicious bots.
```

**根因**：`BLOCKED_PATTERNS`（11 項）未包含 `"performing security verification"`。

---

## 五、M3 工具功能測試（全部通過）

| TC | 測試重點 | 結果 | 備註 |
|----|---------|:----:|------|
| M3-TC01 | 靜態頁面（Wikipedia EN）| ✅ | 229,628 chars，無 HTML 殘留 |
| M3-TC02 | stripReaderEnvelope | ✅ | "Markdown Content:" 前綴正確剝除 |
| M3-TC03 | SPA 頁面（GitHub Repo）| ✅ | isError=false，5,895 chars 靜態殼（OBS-2）|
| M3-TC04 | 多語系（中文 Wikipedia）| ✅ | 83,176 chars，無亂碼 |
| M3-TC05 | 空路徑 URL | ✅ | isError=true，HTTP 500 |

---

## 六、M4 錯誤處理測試

### ✅ 通過（4/5）

| TC | URL | 實際行為 |
|----|-----|---------|
| M4-TC03 | httpbin 404 / 500 | isError=true，訊息含 status code |
| M4-TC04 | httpbin /bytes/0 | isError=true，HTTP 502（後端攔截）|
| M4-TC05 | 不存在的 domain | isError=true，HTTP 502 |

### ❌ M4-TC01：Blocked Pattern 偵測 — 判定失敗但有補充說明（→ OBS-1）

**URL**：`https://stackoverflow.com`（根域名）

**實際**：`isError=true`，但原因是後端 HTTP 502，非 `detectBlocked()` 觸發。

核心目標（isError=true）已達成，但與 M2-TC06 的差異揭露 Cloudflare 有兩種攔截路徑：

| URL 型態 | 後端行為 | MCP 回傳 |
|---------|---------|---------|
| 根域名（stackoverflow.com）| HTTP 502 | isError=true ✅ |
| 具體頁面（/questions/...）| HTTP 200 + CF 驗證頁 | isError=false ❌ BUG-1 |

---

## 七、M5 多 CLI 整合測試

### M5-TC01：Gemini CLI 設定 ✅
`~/.gemini/settings.json` 加入 `mcpServers`（npx 方式），`gemini mcp list` 顯示 Connected。

### M5-TC02 P-A：Wikipedia 摘要 ✅

| 評估項目 | 結果 |
|---------|:----:|
| MCP tool 呼叫 | ✅ |
| 摘要準確性 | ✅ Pass |
| 幻覺 | ✅ 無 |
| 品質 | ⭐⭐⭐⭐⭐ 5/5 |

### M5-TC02 P-B：HN 文章列表 ✅ / ⚠️

10 篇提取成功，標題正確。**連結僅顯示域名**（非完整 URL），與 POC-60 T5 P-B 觀察一致。

### M5-TC03 P-C：Cloudflare 失敗情境（BUG-1 實境驗證）

| 層次 | 行為 | 評估 |
|------|------|:----:|
| MCP 工具層 | isError=false，CF 驗證頁原文回傳 | ❌ BUG-1 |
| Gemini 模型層 | 自行識別 CF 頁面，正確回報無法讀取 | ✅ 補救 |
| 使用者體驗 | 最終呈現正確 | ✅ |

Gemini 2.5 補救了 MCP 層漏洞，但**較弱模型仍有誤判風險**。

### M5-TC04：Claude Code 整合 ✅

`.mcp.json` 設定成功，tool 正確呼叫，摘要品質 5/5。

**OBS-5（新發現）**：Wikipedia 頁面 232,884 chars 超過 Claude Code token 上限。

| CLI | 超大頁面處理方式 |
|-----|----------------|
| Claude Code | 明確告知截斷，摘要基於前段內容 |
| Gemini CLI | 靜默存暫存檔，摘要完整 |

### M5-TC05：Copilot CLI ⬜（待補測，四月）

`copilot v1.0.10`（獨立 binary）確認支援 MCP（`--additional-mcp-config` 旗標）。測試日額度用盡，P-A/B/C 待四月補測。

---

## 八、重要發現彙整

### 🐛 BUG-1：Cloudflare 驗證頁繞過 Blocked 偵測（嚴重度：中）

| 項目 | 說明 |
|------|------|
| 發現 | M2-TC06 自動化 + M5-TC03 Gemini CLI 實境驗證 |
| 現象 | "Performing security verification" 不在 `BLOCKED_PATTERNS`，`isError=false` |
| 影響 | CF 驗證頁被當作正常 markdown 傳給 AI；強模型可自行補救，弱模型有誤判風險 |
| 建議修正 | `source-fetcher.js` 新增 `"performing security verification"` / `"verify you are human"` |

### 📋 OBS-1：Cloudflare 兩種觸發路徑
根域名 → 後端 502（isError=true ✅）；具體頁面 → CF 驗證頁原文（isError=false ❌ BUG-1）

### 📋 OBS-2：SPA 靜默回傳靜態殼（繼承 POC-60）
GitHub Repo 等 SPA：`isError=false`，回傳無意義框架，AI 無從判斷。屬後端架構限制。

### 📋 OBS-3：後端統一回 HTTP 502
DNS 失敗、空 body、被封鎖等情況統一回 502，錯誤訊息資訊量少，難以區分失敗原因。

### 📋 OBS-4：README zh-TW 缺少 npx 說明 + HTTPS clone 限制未說明

### 📋 OBS-5：超大頁面 Token 上限行為差異（Claude Code vs Gemini CLI）
Claude Code 明確截斷提示；Gemini CLI 靜默處理。非 MCP 問題，屬各 CLI 實作差異。

---

## 九、與 POC-60 skill 差異對比

| 面向 | skill（POC-60）| MCP（本次）|
|------|:-------------:|:----------:|
| 正常頁面擷取 | ✅ | ✅ 等價 |
| 錯誤回傳格式 | HTTP status（原始）| `isError: true` + 訊息（AI 友善）|
| Blocked 偵測 | ❌ 無此層 | ✅ 有偵測層（但 BUG-1）|
| SPA 靜態殼 | ⚠️ 已知 | ⚠️ 同，不 crash |
| Cloudflare WAF | 🔴 被擋 | 🔴 被擋 + BUG-1 部分漏偵測 |
| 多語系 | ✅ | ✅ 等價 |
| robots.txt | ❌ 不遵守 | ❌ 不遵守（後端行為不變）|

---

## 十、測試覆蓋率完整表

| TC | 分類 | P | 結果 | 執行 |
|----|------|:-:|:----:|------|
| M1-TC01 | clone+build | P0 | ✅ | 手動 |
| M1-TC02 | npx 安裝 | P1 | ✅ | 自動 |
| M1-TC03 | Node < 24 | P2 | ⚪ 跳過 | — |
| M1-TC04 | build 產出 | P1 | ✅ | 自動 |
| M1-TC05 | vitest 100% | P1 | ✅ | 手動 |
| M2-TC01 | server 啟動 | P0 | ✅ | 自動 |
| M2-TC02 | tools/list | P0 | ✅ | 自動 |
| M2-TC03 | 合法 URL | P0 | ✅ | 自動 |
| M2-TC04 | 非法 URL 攔截 | P0 | ✅ | 自動 |
| M2-TC05 | structuredContent | P1 | ✅ | 自動 |
| M2-TC06 | isError 旗標 | P0 | ❌ BUG-1 | 自動 |
| M3-TC01 | 靜態頁面 | P0 | ✅ | 自動 |
| M3-TC02 | stripEnvelope | P1 | ✅ | 自動 |
| M3-TC03 | SPA 頁面 | P1 | ✅ OBS-2 | 自動 |
| M3-TC04 | 多語系 URL | P1 | ✅ | 自動 |
| M3-TC05 | 空路徑 | P1 | ✅ | 自動 |
| M4-TC01 | Blocked 偵測 | P0 | ❌ OBS-1 | 自動 |
| M4-TC02 | Timeout 20s | P1 | — 未執行 | — |
| M4-TC03 | HTTP 4xx/5xx | P1 | ✅ | 自動 |
| M4-TC04 | 空 body | P2 | ✅ | 自動 |
| M4-TC05 | DNS 失敗 | P2 | ✅ | 自動 |
| M5-TC01 | Gemini 安裝 | P0 | ✅ | 手動 |
| M5-TC02 | Gemini P-A/B | P0 | ✅ | 手動 |
| M5-TC03 | Gemini P-C | P1 | ❌ BUG-1 / ✅ 補救 | 手動 |
| M5-TC04 | Claude Code | P1 | ✅ OBS-5 | 手動 |
| M5-TC05 | Copilot CLI | P1 | ⬜ 待補測（四月）| 手動 |

---

## 十一、後續行動建議

| 優先 | 行動 | 負責 |
|:----:|------|------|
| 🔴 P0 | BUG-1：`BLOCKED_PATTERNS` 補充 `"performing security verification"` | RD |
| 🟡 P1 | README zh-TW 補充 npx 說明 + SSH clone 限制說明 | RD |
| 🟡 P1 | M5-TC05 Copilot CLI 補測（四月額度恢復）| QA |
| 🟢 P2 | OBS-3：後端 502 通用錯誤碼改善（提供更具體錯誤分類）| RD（評估）|
