# 測試報告 — AGO-216 Aitago API 效能基線量測

## 基本資訊

| 項目 | 內容 |
|------|------|
| 報告類型 | Feature 環境 API 效能基線測試 |
| 測試日期 | 2026-04-22 ~ 2026-04-24 |
| 測試環境 | `https://feature-line-crm.aitago.tw/api`（develop 分支）|
| 測試人員 | williamsliu |
| 測試工具 | **k6 v1.7.1**（單 VU 線性量測）+ jq（raw JSON 分析）|
| 執行模式 | vus=1, iterations=30/endpoint, think_time=400ms, timeout=30s, max_retries=3 |
| Epic | [AGO-216](https://ai360c.atlassian.net/browse/AGO-216) |

---

## 總覽

| Phase | 子工單 | 內容 | 請求數 | 失敗率 | 結果 |
|-------|-------|------|-------:|-------:|:----:|
| Phase 1 | [AGO-220](https://ai360c.atlassian.net/browse/AGO-220) | 盤點 104 條 API | — | — | ✅ Done |
| Phase 2 | [AGO-221](https://ai360c.atlassian.net/browse/AGO-221) | k6 + fixture 準備 | — | — | ✅ Done |
| Phase 3 | [AGO-222](https://ai360c.atlassian.net/browse/AGO-222) | 撰寫 A/B/C 腳本 | — | — | ✅ Done |
| Phase 4 | [AGO-223](https://ai360c.atlassian.net/browse/AGO-223) | 執行測試 | — | — | ✅ Done |
| Phase 5 | [AGO-224](https://ai360c.atlassian.net/browse/AGO-224) | 彙整報告 | — | — | ✅ Done |
| 失敗拆解 | [AGO-227](https://ai360c.atlassian.net/browse/AGO-227) | per-endpoint 分析 | — | — | ✅ Done |
| **總計** | | | **1586**（+rerun ~5332 次）| **Phase A 12.9% → 9.67% / Phase B 0% / Phase C 34% → 25.5% → 扣真 Bug 僅 12%** | ⚠️ 5 Bug 待 RD |

### 執行輪次

| Run | 日期時間 | Phase | 耗時 | 輸出 |
|-----|---------|-------|------|------|
| Phase A initial | 2026-04-22 | A | ~30m | `phase-a-clean-dashboard.html` |
| Phase B | 2026-04-22 | B | ~5m | `phase-b-summary.json`（0% fail）|
| Phase C | 2026-04-22 | C | ~30m | `phase-c-clean2-dashboard.html` |
| **Phase A raw** | **2026-04-23 21:12** | **A** | **35m37s** | **`phase-a-raw-20260423-211202.json` 5.9MB** |
| **Phase C raw** | **2026-04-23 21:59** | **C** | **37m53s** | **`phase-c-raw-20260423-215947.json` 5.8MB** |
| **Phase A re-run**（script 修 keywords/check_binding）| **2026-04-24 09:32** | **A** | **35m39s** | **`phase-a-raw-20260424-093213.json` 5.9MB** |
| **Phase C re-run Stage 1A**（script 結構修）| **2026-04-24 15:45** | **C** | **36m38s** | **`phase-c-raw-20260424-154501.json` 5.6MB** |
| **Phase C re-run Stage 1B**（source.id / ULID / is_unique_code 修）| **2026-04-24 16:27** | **C** | **36m24s** | **`phase-c-raw-20260424-162721.json` 5.7MB** |

---

## Phase A — GET 純讀取（重跑 2026-04-24 為準）

### 🔴 嚴重慢端點（p95 > 1 秒）

> 單位：毫秒（ms）— 1000ms = 1 秒

| Endpoint | 樣本數 | p50 (ms) | **p95 (ms)** | p99 (ms) | max (ms) | 狀態 |
|----------|:--:|----:|--------:|----:|----:|:----:|
| `GET /line_users/coupons` | 30 | 5145 (≈5.1s) | **7469 (≈7.5s)** | 8983 (≈9s) | 8983 (≈9s) | ✅ 200（**慢但零失敗**） |
| `GET /coupons` | 30 | 5163 (≈5.2s) | **7260 (≈7.3s)** | 8738 (≈8.7s) | 8738 (≈8.7s) | ✅ 200（**慢但零失敗**） |
| `GET /materials` | 30 | 874 | **2038 (≈2s)** | 2105 | 2105 | ⚠️ 兩輪 p95 落差大（前次 423ms） |
| `GET /campaigns/{id}/insights` | 30 | 240 | **1264 (≈1.3s)** | 1448 | 1448 | ⚠️ 聚合運算未 cache? |

### 🟡 偏慢（p95 500ms~1s）

| Endpoint | p95 (ms) |
|----------|----:|
| `GET /metrics/line` | 799 |
| `GET /audiences` | 592 |

### ❌ 失敗端點（重跑後）

| Endpoint | Failed | Status | 處理 |
|----------|:------:|:------:|------|
| `GET /metrics/user_tagging` | 120 (30×4 retry) | **500** | 🚨 [AGO-228](https://ai360c.atlassian.net/browse/AGO-228) |
| `GET /playground/get_lottery` | 30 | 400 | fixture 不足（待補）|
| `GET /keywords/answer` | 30 | 422 | 需 LINE reply_token 無法 mock（排除）|

**Phase A 失敗率：9.67%（180/1861）** — 扣除已知限制後，**真 server error = 120/1861 = 6.45%**，全源於 AGO-228。

### Script 修復驗證 ✅（2026-04-24 re-run 前/後對比）

| Endpoint | 修復前 | 修復後 |
|----------|:------:|:-----:|
| `GET /keywords/exists?keyword=`（原參數名 `name` → `keyword`）| 30/30 422 | **30/30 200** |
| `GET /line_users/check_binding_status?email=&line_user_id=`（補必填）| 30/30 422 | **30/30 200** |

---

## Phase B — 認證（/token /refresh /revoke）

| 指標 | 結果 |
|------|------|
| 總請求 | 60 |
| 失敗 | **0（0%）** ✅ |
| p95 | < 500ms |

Auth 模組**完全健康**，無異常端點。

---

## Phase C — 寫入（POST / PUT / DELETE）

### ❌ 失敗端點（以 Stage 1B rerun 為準，2026-04-24 16:27）

| Endpoint | Failed | Status | 分類 |
|----------|:------:|:------:|------|
| `PUT /templates/{id}` | 120 (30×4) | **500** | 🚨 [AGO-229](https://ai360c.atlassian.net/browse/AGO-229) **真 Bug** |
| `PUT /coupons/categories/{id}` | 120 (30×4) | **500** | 🚨 [AGO-230](https://ai360c.atlassian.net/browse/AGO-230) **真 Bug** |
| `PUT /line_users/tagging_tag (restore-batch)` | 90 | 422 | script 空 tags 陣列（非 bug）|
| `PUT /line_users/tagging_tag (restore)` | 30 | 422 | 同上 |
| `PUT /coupons/codes/lock` | 30 | 422 | Stage 2 未修（副作用風險）|
| `POST /coupons/codes/cancel` | 30 | 422 | Stage 2 未修（副作用風險）|
| `POST /prizes` | 30 | 422 | Stage 2 未修（需 fixture）|
| 零星（mbti cache / materials convert）| 2 | 422 | 偶發 |

**Phase C 最終失敗率：25.5%（452/1775 含 retry）**，扣除真 Bug 240 筆後，剩 **212 筆（12%）** 皆為 script/fixture 議題，非後端 bug。

### 歷史輪次比較

| 輪次 | 失敗數 | 比例 |
|------|------:|----:|
| 原 2026-04-23 | 604 | 34% |
| Stage 1A | 572 | 32% |
| **Stage 1B**（最終）| **452** | **25.5%** |

### 偏慢寫入端點（200 OK，但 p95 > 500ms）

| Endpoint | p50 (ms) | p95 (ms) | max (ms) |
|----------|----:|----:|----:|
| `POST /templates` | 434 | 587 | 611 |
| `DELETE /templates/{id}` | 469 | 581 | 599 |
| `POST /short_urls` | 282 | 360 | 453 |

其他 25+ Phase C 端點 p95 < 200ms，整體**健康**。

### 🔁 Phase C Script 修復歷程（2026-04-24 Stage 1A + 1B）

OpenAPI 對照糾正原假設：405/422 大多是 **script 結構錯誤**，非後端 bug。兩輪修復 + rerun 驗證：

| 輪次 | 失敗數 | 比例 | 主要修復 |
|------|------:|----:|---------|
| 原 2026-04-23 | 604 | 34% | 基線 |
| Stage 1A | 572 | 32% | 刪 `PUT /tags/{id}`（API 無此動詞）、`/coupons/{id}` 改 POST、tagging_tag/batch 結構修 |
| **Stage 1B** | **452** | **25.5%** | 加 `source.id`、`line_user_ids`、更新 coupon 移 `is_unique_code`、notes 改 ULID |

**四類 script 修復實證**（curl + OpenAPI 驗證）：
1. `PUT /line_users/tagging_tag`：`source` 需 `{id, type}` 兩欄位
2. `POST /line_users/batch_tagging_tag`：`line_users.line_user_ids`（非 `.ids`）
3. `POST /coupons/{id}` 更新：**移除 `is_unique_code`**（API 訊息「is unique code 必須缺少」）
4. `POST /notes`：`user_id` 必須是 LineUser 內部 ULID（`01jq...`），非 LINE UID

**扣除真 Bug 240 筆後**：剩 212 筆（12%）全為 script/fixture 議題，非後端 bug。真正穩定重現的後端 500 僅兩條（AGO-229/230）。

### 🆕 Phase C 新 Finding

| # | Finding | 類型 |
|---|---------|:----:|
| 1 | `PUT /line_users/tagging_tag` 空 tags array (`tags: []`) 仍 422 | script 限制（非 bug，實際 UI 不送空）|
| 2 | `POST /line_users/batch_tagging_tag` 帶 `source:{id}` 缺 `type` → **後端 500 `Undefined array key "type"`**（`LineUsersService.php:105`）| **後端 edge case bug 候選**（validation 未擋，service 層 NPE）|
| 3 | Stage 2 未修：`PUT /coupons/codes/lock` / `POST /coupons/codes/cancel` / `POST /prizes` | 有副作用或需 fixture，保留待下輪 |

---

## 異常端點總結（5 個新 Bug 單）

| 工單 | Endpoint | 現象 | Priority |
|------|---------|------|:--:|
| [AGO-225](https://ai360c.atlassian.net/browse/AGO-225) | `GET /coupons` | p95 7.2s（慢） | High |
| [AGO-226](https://ai360c.atlassian.net/browse/AGO-226) | `GET /line_users/coupons` | p95 7.5s + 公開 + 無 cache + DoS 風險 | **Highest** |
| [AGO-228](https://ai360c.atlassian.net/browse/AGO-228) | `GET /metrics/user_tagging` | 100% 回 500 | High |
| [AGO-229](https://ai360c.atlassian.net/browse/AGO-229) | `PUT /templates/{id}` | 100% 回 500 | High |
| [AGO-230](https://ai360c.atlassian.net/browse/AGO-230) | `PUT /coupons/categories/{id}` | 100% 回 500 | High |

---

## 關鍵發現

### 1. 失敗與慢延遲**無關**
顛覆原始假設「慢 endpoint 容易 timeout 失敗」：
- AGO-225 `/coupons`（7.2s）：**零失敗**（30/30 OK）
- AGO-226 `/line_users/coupons`（7.5s）：**零失敗**（30/30 OK）
- 失敗完全集中在另外 5（Phase A）+ 15（Phase C）個 endpoint

### 2. 三個 100% 失敗的 500 Bug
- AGO-228（metrics/user_tagging）：同 token 打其他 metrics 皆 200，**確認 endpoint 本身壞**
- AGO-229 / AGO-230：同模組 POST/DELETE/GET 皆 200，**只有 PUT 壞**

### 3. 一條 Bug 修復效率（AGO-213 / AGO-214 衍生）
開發 AGO-198 時發現前端時區 workaround（AGO-213）與後端 `MarketingTriggerService` time-parse bug（AGO-214，**尚未修**）— 效能基線測試間接驗證到這條鏈路。

---

## SLO 建議（基於本輪基線）

| 類別 | p95 目標 | p99 目標 | 現狀 |
|------|:--------:|:--------:|:----:|
| Read（GET）| < 200ms | < 500ms | 4 條超標（AGO-225/226 + /materials + insights，部分 p95 達 7 秒）|
| Write | < 300ms | < 800ms | 3 條超標（templates 系列 p95 ≈ 0.58 秒）|
| Auth（/token*）| < 150ms | < 300ms | ✅ 已達成 |
| 公開 endpoint | < 500ms | < 1 秒 | AGO-226 嚴重超標（p95 ≈ 7.5 秒）|
| 整體 5xx rate | < 0.1% | — | AGO-228 單條 100% 超標 |
| Write 端點失敗率 | < 1% | — | 待 RD 修完 AGO-229/230 |

> **SLO 目標格式說明**：所有 p95/p99 目標皆為**毫秒（ms）**；「< 200ms」意即「95% 的請求須在 200 毫秒內回應完成」。

---

## 測試範圍（對照 Epic §1.2）

### ✅ In Scope
- 104 條 JWT 認證 API（A GET / B Auth / C Write）
- feature 環境 + develop 分支
- k6 單 VU 線性量測

### ❌ Out of Scope
- Partner API（無 API Key）
- D 類非同步任務 / E 類檔案上傳
- Webhook / 管理類 API
- 壓測 / 併發
- 多租戶測試情境

---

## 下一步建議

### 立即（本週）
1. RD 接手 **AGO-228 / AGO-229 / AGO-230**（3 個 500 Bug）
2. RD 接手 **AGO-225 / AGO-226**（2 個慢 endpoint，共用 CouponRepository）
3. QA 修 script：`PUT /tags/{id}` + `PUT /coupons/{id}` 改 PATCH，重跑 Phase C

### 下輪（兩週內）
1. `/materials` p95 2038 vs 423 差異 — 跑第三輪 confirm
2. `/campaigns/{id}/insights` 1.3s 聚合 cache 評估
3. 8 個 422 endpoint 規格釐清補 script

### 長期
1. D / E 類 API 測試
2. Partner API（需 API Key）
3. CI 週期化（k6 + InfluxDB + Grafana / GitHub Actions）
4. 壓測（vus=50, duration=5m）— 需 RD 確認環境承受度

---

## 附錄

### 相關文件（於 aitago_test repo）
- 測試計劃：`docs/api-performance-test-plan.md`
- 端點清單：`docs/api-performance-targets.md`
- **完整 Findings**：`docs/api-performance-findings-2026-04-23.md`
- **Phase 5 正式報告**：`docs/api-performance-report-2026-04-22.md`
- k6 腳本：`api-perf/scripts/phase-{a,b,c}.js`
- Raw JSON：`api-perf/results/phase-{a,c}-raw-*.json`（> 10MB 未進本 repo）
- Dashboard HTML：`api-perf/results/phase-{a,c}-*dashboard.html`

### Jira Epic 結構
```
AGO-216 Epic（進行中，待 Bug 關閉後收尾）
├─ Phase 1 AGO-220 ✅ 完成
├─ Phase 2 AGO-221 ✅ 完成
├─ Phase 3 AGO-222 ✅ 完成
├─ Phase 4 AGO-223 ✅ 完成
├─ Phase 5 AGO-224 ✅ 完成
├─ 失敗拆解 AGO-227 ✅ 完成
├─ AGO-225 Bug / High — 待 RD
├─ AGO-226 Bug / Highest — 待 RD
├─ AGO-228 Bug / High — 待 RD
├─ AGO-229 Bug / High — 待 RD
└─ AGO-230 Bug / High — 待 RD
```

### QA 工具環境
- k6 v1.7.1（darwin/arm64）
- jq 1.7.1
- Python 3.12（orchestrator scripts）
- ai360-4f WiFi（MeterSphere）

---

**報告交付**：williamsliu → 2026-04-24
