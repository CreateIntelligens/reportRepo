# POC-60 整合測試報告：aicreate360-reader-skill

> **工單**：[POC-60](https://ai360c.atlassian.net/jira/software/c/projects/POC/boards/34?selectedIssue=POC-60)
> **執行日期**：2026-03-19 ～ 2026-03-20
> **報告產出**：2026-03-20
> **測試範圍**：T1 服務層 / T2 URL 多樣性 / T4 存取保護 / T5 多 Agent CLI 比對 / T6 多模型比對

---

## 一、Skill 基本資訊

| 項目 | 說明 |
|------|------|
| 服務端點 | `GET https://create360.ai/{target-url}` |
| 回傳格式 | `text/plain; charset=utf-8`（Markdown） |
| 標準輸出結構 | `Title / URL Source / Published Time（可選）/ Markdown Content` |
| 認證需求 | 無 |
| 後端 IP（觀察到） | `34.80.213.246` |

---

## 二、T1 服務層測試結論

| 測試項目 | 結果 | 說明 |
|---------|:----:|------|
| 正常公開頁面 | ✅ | 200 OK，標準 Markdown 輸出 |
| Content-Type | ✅ | `text/plain; charset=utf-8`，符合文件 |
| 空路徑 `/` | ✅ | 400 Bad Request，不 crash |
| 無效 URL 格式 | ✅ | 回傳 "Invalid URL" |
| GitHub Repo（SPA） | ⚠️ | 只取到靜態 HTML 框架，README 未取得 |
| 目標 404（SPA） | ⚠️ | 無法與正常頁面區分，回傳相同靜態框架 |

---

## 三、T2 URL 類型多樣性測試結論

**已測 21 個案例（跨 9 大類型）**

### 成功率統計

| 類型 | ✅ 成功 | ⚠️ 部分 | 🔒 被擋 | 🔴 機器人頁 |
|------|:------:|:------:|:------:|:----------:|
| 靜態 HTML（Wiki/新聞） | 3 | 2 | 0 | 0 |
| 技術內容（GitHub/文件） | 0 | 1 | 1 | 1 |
| 影片 | 0 | 1 | 0 | 1 |
| 圖片直連 | 0 | 0 | 0 | 1 |
| 學術文件 | 0 | 1 | 0 | 0 |
| 結構化資料（JSON/CSV/RSS） | 0 | 0 | 0 | 3 |
| 社群論壇 | 1 | 1 | 0 | 0 |
| 電商 | 0 | 0 | 0 | 1 |
| 邊界類 | 2 | 1 | 0 | 0 |
| **合計** | **6（29%）** | **7（33%）** | **1（5%）** | **7（33%）** |

### 主要發現

| 發現 | 說明 |
|------|------|
| ✅ 多語系支援 | EN/ZH/JA 均無亂碼，中文 URL 正確 percent-encode |
| ✅ Markdown 格式品質佳 | 靜態頁面表格、連結、粗體均正確轉換 |
| ✅ meta 資訊擷取 | Published Time 從 `<meta>` 擷取 |
| ⚠️ SPA 頁面只有殼 | BBC、MDN、GitHub Repo 主頁只有靜態框架 |
| ⚠️ `#fragment` 被忽略 | 錨點 URL 回傳整頁，不跳至章節 |
| ⚠️ 日文含 HTML 殘留 | `<table>` 標籤未完全轉為 Markdown |
| 🔴 Google 機器人攔截 | Raw 資源（圖片/MP4/CSV/JSON/RSS）被重定向至反機器人頁 |
| 🔴 Cloudflare WAF | Stack Overflow 等網站完全被擋 |

---

## 四、T4 存取保護行為測試結論

| 防護類型 | 測試對象 | create360.ai 行為 | 是否遵守 |
|---------|---------|-----------------|:-------:|
| robots.txt（嚴格禁止 AI） | NYTimes | **成功抓取內容** | ❌ |
| Cloudflare WAF | Stack Overflow | 顯示 "Just a moment..." | 被擋 |
| 反爬蟲 | Amazon | 觸發 Google 機器人頁 | 被擋 |
| 登入牆 | Twitter/X | 只取得登入提示文字 | 部分 |
| 付費牆 + Cloudflare | WSJ | 觸發 Google 機器人頁 | 被擋 |

> ⚠️ **法律/倫理提醒**：create360.ai 不遵守 robots.txt，使用者需自行確認目標網站的 ToS。

---

## 五、T5 多 Agent CLI 比對

**測試設計**：3 個 CLI × 3 個標準 Prompt（P-A / P-B / P-C）

| Prompt | 說明 | 目標 URL |
|--------|------|---------|
| P-A | Wikipedia LLM 閱讀摘要 | `en.wikipedia.org/wiki/Large_language_model` |
| P-B | HN 文章列表提取 | `news.ycombinator.com` |
| P-C | Cloudflare 失敗處理觀察 | `stackoverflow.com/...` |

### 5.1 P-A：閱讀摘要

| 評估項目 | Copilot CLI | Claude CLI | Gemini CLI |
|---------|:-----------:|:----------:|:----------:|
| URL 建構正確 | ✅ | ✅ | ✅ |
| fallback 誤判 | ✅ 無 | ✅ 無 | ✅ 無 |
| 摘要準確性 | Pass | Pass | Pass |
| 品質（1-5） | **5** | **5** | **5** |
| 幻覺 | 無 | 無 | 無 |

### 5.2 P-B：文章列表提取

| 評估項目 | Copilot CLI | Claude CLI | Gemini CLI |
|---------|:-----------:|:----------:|:----------:|
| URL 建構正確 | ✅ | ✅ | ✅ |
| 提取篇數 | **30** | **10** | **5** |
| 準確性 | Partial（連結僅域名）| Pass | Pass |
| 品質（1-5） | **4** | **5** | **5** |

> 備註：Copilot CLI 雖提取最多（30篇），但連結欄位僅顯示域名而非完整 URL。Claude / Gemini 篇數較少但資料完整度較高。

### 5.3 P-C：失敗處理（Cloudflare 攔截）

| 評估項目 | Copilot CLI | Claude CLI | Gemini CLI |
|---------|:-----------:|:----------:|:----------:|
| 被攔截 | ✅ 是 | ✅ 是 | ✅ 是 |
| 攔截類型辨識 | Cloudflare | Cloudflare | Cloudflare |
| 說明清晰度 | ✅ | ✅ | ✅ |
| 嘗試繞過 | ❌ 否 | ❌ 否 | ❌ 否 |
| 失敗處理品質 | **5** | **5** | **5** |

### 5.4 T5 整體結論

| 比對項目 | Copilot CLI | Claude CLI | Gemini CLI |
|---------|:-----------:|:----------:|:----------:|
| Skill URL 建構 | ✅ | ✅ | ✅ |
| fallback 不誤報 | ✅ | ✅ | ✅ |
| 閱讀摘要品質 | 5 | 5 | 5 |
| 文章列表提取量 | 30（最多）| 10 | 5（最少）|
| 失敗處理品質 | 5 | 5 | 5 |
| **整體表現** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐ |

> **結論**：三個 CLI 皆能正確使用 skill，Cloudflare 失敗處理表現一致優異。Claude CLI 在文章提取的完整度最佳；Copilot CLI 提取量最多但連結品質略遜；Gemini CLI 提取量最保守。

---

## 六、T6 多模型比對

**測試設計**：6 個模型 × 3 個 Task（在 Copilot CLI 中切換 `/model`）

| Task | 說明 | 目標 URL |
|------|------|---------|
| Task-α | Wikipedia LLM 閱讀摘要 | `en.wikipedia.org/wiki/Large_language_model` |
| Task-β | HN 文章列表提取 | `news.ycombinator.com` |
| Task-γ | 論點提取（Anthropic 文章） | `anthropic.com/engineering/demystifying-evals-for-ai-agents` |

### 6.1 Task-α：閱讀摘要

| 模型 | URL 建構 | fallback | 準確性 | 品質（1-5） | 幻覺 |
|------|:-------:|:--------:|:------:|:-----------:|:----:|
| claude-opus-4.6 | ✅ | 有（正常行為） | Pass | **4** | 無 |
| claude-sonnet-4.6 | ✅ | 無 | Pass | **5** | 無 |
| gemini-3-pro | ✅ | 有（正常行為） | Pass | **5** | 無 |
| gpt-5-mini | ✅ | 無 | Pass | **4** | 無 |
| gpt-5.3-codex | ✅ | 有（正常行為） | Pass | **5** | 無 |
| gpt-5.4 | ✅ | 無 | Pass | **5** | 無 |

### 6.2 Task-β：文章列表提取

| 模型 | URL 建構 | 提取篇數 | 準確性 | 品質（1-5） |
|------|:-------:|:-------:|:------:|:-----------:|
| claude-opus-4.6 | ✅ | **30** | Pass | **5** |
| claude-sonnet-4.6 | ✅ | **15** | Pass | **5** |
| gemini-3-pro | ✅ | **10** | Pass | **5** |
| gpt-5-mini | ✅ | **5** | Pass | **4** |
| gpt-5.3-codex | ✅ | **7** | Pass | **5** |
| gpt-5.4 | ✅ | **10** | Pass | **5** |

### 6.3 Task-γ：論點提取

| 模型 | URL 建構 | 論點正確 | 論據來自原文 | 品質（1-5） | 幻覺 |
|------|:-------:|:-------:|:-----------:|:-----------:|:----:|
| claude-opus-4.6 | ✅ | Pass | Pass | **5** | 無 |
| claude-sonnet-4.6 | ✅ | Pass | Pass | **5** | 無 |
| gemini-3-pro | ✅ | Pass | Pass | **5** | 無 |
| gpt-5-mini | ✅ | Pass | Pass | **4** | — |
| gpt-5.3-codex | ✅ | Pass | Pass | **5** | 無 |
| gpt-5.4 | ✅ | Pass | Pass | **5** | 無 |

### 6.4 T6 模型綜合評比

| 模型 | Task-α | Task-β | Task-γ | 平均品質 | fallback 觸發 | 整體 |
|------|:------:|:------:|:------:|:--------:|:------------:|:----:|
| claude-opus-4.6 | 4 | 5 | 5 | **4.67** | 有 | ⭐⭐⭐⭐⭐ |
| claude-sonnet-4.6 | 5 | 5 | 5 | **5.00** | 無 | ⭐⭐⭐⭐⭐ |
| gemini-3-pro | 5 | 5 | 5 | **5.00** | 有 | ⭐⭐⭐⭐⭐ |
| gpt-5-mini | 4 | 4 | 4 | **4.00** | 無 | ⭐⭐⭐⭐ |
| gpt-5.3-codex | 5 | 5 | 5 | **5.00** | 有 | ⭐⭐⭐⭐⭐ |
| gpt-5.4 | 5 | 5 | 5 | **5.00** | 無 | ⭐⭐⭐⭐⭐ |

> **結論**：所有模型皆能正確呼叫 skill 並建構 URL，無一出現 fallback 誤判。claude-sonnet-4.6、gemini-3-pro、gpt-5.3-codex、gpt-5.4 平均品質滿分 5/5，表現最優。gpt-5-mini 整體較保守（提取篇數最少、品質稍低），但仍通過所有準確性測試。claude-opus-4.6 在 Task-α 略遜一籌（4/5），但深度分析能力最強。

---

## 七、Known Limitations 彙整

| 限制 | 影響範圍 | 嚴重度 |
|------|---------|:------:|
| SPA / JS 動態渲染 | GitHub repo 主頁、BBC、MDN 等 | 🔴 高 |
| create360.ai IP 被 Google 反機器人偵測 | 圖片/影片/CSV/JSON/RSS 及部分強保護網站 | 🔴 高 |
| Cloudflare WAF 阻擋 | Stack Overflow 等有 CF 保護的網站 | 🟡 中 |
| 不遵守 robots.txt | 所有網站（法律/倫理風險） | 🟡 中（倫理） |
| `#fragment` 不支援 | 任何含錨點 URL | 🟢 低 |
| 日文頁面含 HTML 表格殘留 | 日文 Wikipedia 等含 `<table>` 頁面 | 🟢 低 |

---

## 八、建議使用情境

### ✅ 最適合
- Wikipedia（任何語言）
- 純 HTML 靜態技術文件
- Hacker News 等純 HTML 論壇
- 學術論文 HTML 摘要頁（arXiv）
- 含查詢參數 `?` 的 URL

### ⚠️ 可用但有限制
- YouTube（只有標題，無內文）
- 新聞網站（靜態渲染有效，動態渲染失效）
- 中日韓文頁面（無亂碼，但可能有 HTML 殘留）

### ❌ 不建議使用
- GitHub repo 主頁（改用 raw URL）
- 圖片 / 影片 / PDF 等二進制直連 URL
- JSON API 端點、CSV、RSS XML 直連
- Cloudflare 保護的網站（Stack Overflow 等）
- 強力反爬蟲電商平台（Amazon 等）

---

## 九、報告文件索引

| 文件 | 說明 |
|------|------|
| `test-plans/poc-60/POC-60-test-plan.md` | 完整測試計畫 |
| `test-plans/poc-60/T1-service-api/T1-results.md` | T1 服務層詳細結果 |
| `test-plans/poc-60/T2-url-types/T2-T4-results.md` | T2 URL 多樣性 + T4 存取保護詳細結果 |
| `test-plans/poc-60/T2-url-types/結果摘要表.md` | T2 所有 TC 摘要表 |
| `tests/T5-multi-agent/` | T5 各 CLI 原始結果（P-A/B/C） |
| `tests/T6-multi-model/` | T6 各模型原始結果（Task-α/β/γ） |
| `reports/POC-60-skill-integrated-report.md` | **本報告（整合版）** |
