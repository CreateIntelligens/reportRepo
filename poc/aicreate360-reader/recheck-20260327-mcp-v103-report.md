# 複驗報告：aicreate360-reader-mcp v1.0.3

> **複驗命名**：recheck-20260327-mcp-v103
> **對應原報告**：[recheck-20260326-mcp-v102-report.md](./recheck-20260326-mcp-v102-report.md)
> **複驗日期**：2026-03-27
> **套件版本**：aicreate360-reader-mcp v1.0.3（npm）
> **複驗範圍**：v1.0.2 報告中新發現的 BUG-2 / BUG-2b

---

## 一、複驗背景

v1.0.2 複驗（2026-03-26）發現兩個新問題，回報 RD 後發布 v1.0.3：

| 編號 | 原問題 | 嚴重度 |
|------|--------|:------:|
| BUG-2 | `serverInfo.version` 回報 `1.0.1`，與 npm 發布版本 `1.0.2` 不一致 | 低 |
| BUG-2b | Tool 名稱由 `aicreate360_fetch_url` → `fetch_url`，未於 changelog 說明（breaking change）| 中 |

**本次不驗範圍**：
- RT-03-TC02（HTTP 4xx isError 行為改變）→ 確認為刻意設計，關閉

---

## 二、複驗總覽

| TC | 對應問題 | 驗證項目 | 結果 |
|----|---------|---------|:----:|
| B2-TC01 | BUG-2 | `serverInfo.version` === npm 版本（`1.0.3`）| ✅ PASS |
| B2-TC02 | BUG-2 | 版本格式符合 semver（X.Y.Z）| ✅ PASS |
| B2b-TC01 | BUG-2b | `tools/list` 只有 `fetch_url`，無舊名稱 | ✅ PASS |
| B2b-TC02 | BUG-2b | `fetch_url` 可正常呼叫（正常 URL）| ✅ PASS |
| B2b-TC03 | BUG-2b | README / CHANGELOG 說明 tool name 變更 | ❌ FAIL |

**整體結果**：✅ 4 / ❌ 1

---

## 三、BUG-2 複驗：serverInfo 版本一致性

### B2-TC01 + B2-TC02 ✅ PASS

**執行方式**：JSON-RPC `initialize`（Claude Code 代執行）

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"qa-test","version":"1.0"}}}' \
  | npx -y aicreate360-reader-mcp@1.0.3
```

**實際回傳**：
```json
{
  "result": {
    "protocolVersion": "2024-11-05",
    "capabilities": {"tools": {"listChanged": true}},
    "serverInfo": {
      "name": "aicreate360-reader-mcp",
      "version": "1.0.3"
    }
  },
  "jsonrpc": "2.0",
  "id": 1
}
```

| 項目 | v1.0.2（原）| v1.0.3（複驗）| 比對 |
|------|:-----------:|:------------:|:----:|
| serverInfo.version | `"1.0.1"`（不一致）| `"1.0.3"`（一致）| ✅ 修正 |
| 版本格式 | semver ✅ | semver ✅ | 維持 |

**結論**：BUG-2 已修正。`serverInfo.version` 現已與 npm 發布版本同步。

---

## 四、BUG-2b 複驗：Tool 名稱 Breaking Change

### B2b-TC01 ✅ PASS — tools/list 工具名稱

**執行方式**：JSON-RPC `tools/list`

**實際回傳**（節錄）：
```json
{
  "result": {
    "tools": [
      {
        "name": "fetch_url",
        "title": "Fetch URL as Markdown",
        "description": "Fetch any public URL and return its content as clean markdown via create360.ai."
      }
    ]
  }
}
```

| 項目 | 預期 | 實際 | 結果 |
|------|------|------|:----:|
| 清單含 `fetch_url` | 是 | ✅ | ✅ PASS |
| 清單不含 `aicreate360_fetch_url` | 是 | ✅ 不存在 | ✅ PASS |

---

### B2b-TC02 ✅ PASS — fetch_url 功能正常

**測試 URL**：`https://en.wikipedia.org/wiki/Large_language_model`

| 項目 | 預期 | 實際 | 結果 |
|------|------|------|:----:|
| `isError` 未出現 | 是 | ✅ | ✅ PASS |
| 回傳 Markdown 內容 | 是 | ✅ Wikipedia LLM 文章 | ✅ PASS |

---

### B2b-TC03 ❌ FAIL — 文件化 Breaking Change

**查閱來源**：npm registry README（`npm view aicreate360-reader-mcp@1.0.3 readme`）

README 僅列出現行工具名稱，**未記載任何版本異動資訊**：

```
此 server 提供一個工具：

### `fetch_url`
擷取任意公開 URL，回傳該網頁的 Markdown 內容。
```

| 項目 | 預期 | 實際 | 結果 |
|------|------|------|:----:|
| README 說明舊名 → 新名變更 | 有 | ❌ 無 | ❌ FAIL |
| CHANGELOG / Release Notes | 有 | ❌ 無此章節 | ❌ FAIL |

**影響**：既有整合使用者（以 `aicreate360_fetch_url` 撰寫 prompt 或設定 MCP config 的使用者）無法從文件得知需更新設定，可能造成靜默失效。

---

## 五、重要發現彙整

### ✅ 已解決（1 項）

| 原問題 | 解決方式 |
|--------|---------|
| BUG-2：serverInfo 版本號不一致 | v1.0.3 已同步更新版本字串 |

### ⚠️ 未完整解決（1 項）

| 編號 | 問題 | 修正狀況 | 建議 |
|------|------|---------|------|
| BUG-2b | Tool 名稱 breaking change 未文件化 | 功能本身正常（TC01/TC02 PASS），但文件未補充（TC03 FAIL）| 在 README 新增「版本異動記錄」或「Migration Guide」，說明 v1.0.2 起 tool name 由 `aicreate360_fetch_url` 改為 `fetch_url` |

### ⬛ 關閉（1 項）

| 編號 | 問題 | 關閉原因 |
|------|------|---------|
| RT-03-TC02 | HTTP 4xx isError 行為改變 | 確認為刻意設計（v1.0.2 起改為回傳頁面內容供 AI 自行判斷），非缺陷 |

---

## 六、與前次報告比對總表

| 問題 | v1.0.2 狀態 | v1.0.3 複驗結果 |
|------|:-----------:|:--------------:|
| BUG-2 serverInfo 版本不一致 | ❌ | ✅ 修正 |
| BUG-2b Tool 名稱功能（fetch_url 可用）| ✅（已運作）| ✅ 維持 |
| BUG-2b Tool 名稱文件化 | ❌ | ❌ 仍未補充 |
| RT-03-TC02 HTTP 4xx 行為 | ⚠️ 待確認 | ⬛ 關閉（刻意設計）|
