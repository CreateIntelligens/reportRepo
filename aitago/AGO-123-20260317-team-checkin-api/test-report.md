# AGO-123 API 測試報告

| 項目 | 內容 |
|------|------|
| 工單 | [AGO-123](https://ai360c.atlassian.net/browse/AGO-123) |
| 父工單 | [AGO-122](https://ai360c.atlassian.net/browse/AGO-122) \[專案\]新北殘障 |
| 工單標題 | \[專案\]新北殘障-組隊打卡功能開發 |
| 測試人員 | williamsliu |
| 測試日期 | 2026-03-16 |
| 測試環境 | UAT（`line.uat.sport115ntp.aitago.tw`） |
| 測試類型 | 後端 API 測試（自動化） |
| 測試案例 | N/A |
| 測試計畫 | N/A（詳見 HTML 測試報告） |
| HTML 報告 | [test-report-v2.html](./test-report-v2.html) |

---

## 備註

> 後端 API 測試完畢。目前有檢測出部分問題，再麻煩請根據現況以及可行性，針對有需要的問題進行修復，如有對應調整需要驗證，可以再行通知。
>
> 完整互動式報告請下載 `test-report-v2.html` 後開啟查看（需渲染 HTML）。

---

## 測試執行摘要

| 統計項目 | 數值 |
|----------|------|
| 執行時間 | 2026/3/16 13:01:41 |
| 總耗時 | 102.74s |
| 測試套件 | 12 個（7 通過 / 5 套件有缺陷） |
| 總測試案例 | 269 |
| ✅ 通過 | 251 |
| ❌ 失敗 | 18 |
| ⚠️ 缺陷發現 | 18 筆（Critical 3 / Major 6 / Minor 7） |

---

## 測試套件結果

| 測試套件 | ✅ 通過 | ❌ 失敗 | ⚠️ 缺陷 |
|---------|--------|--------|---------|
| Smoke Test — Public Endpoints（無認證） | 2 | - | - |
| Smoke Test — Authenticated GET Endpoints | 29 | - | - |
| Smoke Test — 認證端點驗證 | 2 | - | - |
| Smoke Test — 未認證 GET 應回 401/403 | 19 | - | - |
| Smoke Test — 未認證 POST 應回 401/403 | 4 | - | - |
| Smoke Test — HTTP Method 不允許 | 2 | - | - |
| Conversation — 參數驗證 | 4 | - | - |
| Message — 參數驗證 | 3 | - | - |
| LineUsers — 參數驗證 | 4 | - | - |
| **Metrics — 參數驗證** | 2 | **3** | **3** |
| AI Image Logs — 參數驗證 | 4 | - | - |
| **Token — 參數驗證** | 2 | **6** | **6** |
| Token Refresh — 參數驗證 | 2 | - | - |
| 分頁參數 — 邊界值測試 | 29 | - | - |
| 排序參數測試 | 3 | - | - |
| 篩選參數測試 | 5 | - | - |
| Include 關聯載入測試 | 3 | - | - |
| Coupon Codes Redeem — 參數驗證 | 5 | - | - |
| Coupon Codes Cancel — 參數驗證 | 2 | - | - |
| Coupon Codes Lock — 參數驗證 | 3 | - | - |
| Coupon Codes Show — 路徑參數驗證 | 1 | - | - |
| Coupon Assign — 參數驗證 | 4 | - | - |
| Coupon Categories — 參數驗證 | 2 | - | - |
| Coupon List — 分頁和排序 | 2 | - | - |
| CheckIn Teams Progress — 參數驗證 | 5 | - | - |
| CheckIn Detect — 參數驗證 | 4 | - | - |
| CheckIn Records — 篩選驗證 | 3 | - | - |
| Keyword Answer — 參數驗證 | 4 | - | - |
| Keyword Exists — 參數驗證 | 3 | - | - |
| **Security — IDOR** | 7 | **5** | **5** |
| Security — ID 型態異常 | 16 | - | - |
| **Security — Error Response 資訊洩露檢查** | 2 | **1** | **1** |
| Security — Rate Limiting 基礎檢測 | 1 | - | - |
| Security — Tenant Setting 安全 | 1 | - | - |
| Security — Token 偽造與篡改 | 4 | - | - |
| Security — Token Revoke 後不可用 | 1 | - | - |
| **Security — CORS Headers 檢查** | - | **1** | **1** |
| **Security — HTTP Security Headers** | - | **1** | **1** |
| Security — SQL Injection 防護（Query Params） | 15 | - | - |
| Security — SQL Injection 防護（POST Body） | 5 | - | - |
| Security — XSS 防護 | 10 | - | - |
| Security — Command Injection 防護 | 5 | - | - |
| Security — Path Traversal 防護 | 4 | - | - |
| Security — SSRF 防護 | 4 | - | - |
| HTTP Method 測試 | 4 | - | - |
| **Content-Type 測試** | 2 | **1** | **1** |
| 不存在路由 — 404 處理 | 8 | - | - |
| 空值和特殊字元處理 | 5 | - | - |

---

## 缺陷清單（16 筆）

> 完整重現步驟、預期 vs 實際行為、建議修復方向詳見 [test-report-v2.html](./test-report-v2.html)。

### 🔴 Critical（3 筆）

| ID | 標題 | 影響模組 | 說明 |
|----|------|----------|------|
| **C-01** | Token 認證端點缺乏帳號密碼驗證 | Token（認證系統） | 登入 API 不會檢查帳號密碼是否正確，任何人不需帳密即可取得有效 Token，等同大門沒有鎖。`POST /api/token` |
| **C-02** | 錯誤回應洩露伺服器內部資訊（Stack Trace） | 全系統 | 存取不存在資源時，系統將伺服器檔案路徑、框架版本、PHP class 名稱與完整呼叫堆疊回傳給前端，攻擊者可藉此規劃精準攻擊。`GET /api/{resource}/{不存在的ID}` |
| **C-03** | CORS 設定允許所有來源存取（`Access-Control-Allow-Origin: *`） | 全系統 | 任何網站都可在用戶瀏覽器中向此 API 發送請求，惡意網站可借用已登入用戶身份竊取資料。 |

### 🟡 Major（6 筆）

| ID | 標題 | 影響模組 | 說明 |
|----|------|----------|------|
| **M-01** | Metrics/user_tagging 缺少必填參數回傳 500 | LineMetrics | 查詢用戶標籤統計時，忘記帶日期參數不提示必填，而是直接回傳 500 Internal Server Error。`GET /api/metrics/user_tagging` |
| **M-02** | Metrics/line 帶日期範圍時回應超時（> 15s） | LineMetrics | 查詢 LINE 好友統計帶上日期範圍時，API 超過 15 秒才有回應，導致前端一直顯示載入中。`GET /api/metrics/line` |
| **M-03** | CheckIn 相關端點未要求認證（無 Token 可直接存取） | CheckIn（打卡系統） | 打卡系統的查詢與執行 API 不需要登入就能存取，需確認是否為預期的公開設計並補充文件說明。 |
| **M-04** | POST /token 空 body 時 Server Hang（長時間無回應） | Token | 對認證 API 發送空請求時，伺服器長時間無回應（超過 10 秒），應立即回傳必填欄位錯誤。`POST /api/token` |
| **M-05** | Playground 不存在 ID 回傳 500（應為 404） | Playground（遊戲/抽獎） | 查詢不存在的遊戲 ID 時回傳 500，其他模組（audiences、campaigns 等）同樣情境回傳 404。`GET /api/playground/{id}` |
| **M-06** | POST /audiences 空 body 回 401 而非 422 | Audience | 新增受眾時送出空白內容，系統回傳 401 Unauthorized 而非 422 驗證錯誤，讓帶了 Token 的開發者困惑。`POST /api/audiences` |

### 🔵 Minor（7 筆）

| ID | 標題 | 影響模組 | 說明 |
|----|------|----------|------|
| **m-01** | 缺少 X-Content-Type-Options Header | 全系統 | 所有 API 回應缺少 `X-Content-Type-Options: nosniff`，瀏覽器可能嘗試猜測回傳內容類型，進而執行惡意腳本。 |
| **m-02** | 缺少 X-Frame-Options Header | 全系統 | 缺少此 header 可能讓系統被嵌入惡意網站的 iframe 中，用於點擊劫持（Clickjacking）攻擊。 |
| **m-03** | Tags 接受 XSS Payload 作為名稱 | Tags | 可用 `<script>alert(1)</script>` 作為標籤名稱存入資料庫，若前端未做 HTML 轉義可觸發 XSS。`POST /api/tags` |
| **m-04** | 缺少 Rate Limiting 機制 | 全系統 | 系統沒有限制同一來源短時間內的請求次數，修復 C-01 後暴力破解密碼的風險將顯著提高。 |
| **m-05** | 不存在路由回傳含有內部資訊 | 全系統 | 存取不存在的 API 路徑時，錯誤訊息包含伺服器內部路徑資訊，此問題與 C-02 同源。 |
| **m-06** | Locale 端點無認證可公開存取 | Locales | `/api/locales` 無需認證即可存取，若為預期設計應在文件中明確標注為 public endpoint。`GET /api/locales` |
| **m-07** | Token 空 Body + 缺少 service 欄位行為不一致 | Token | 空 body 時 Server Hang，但只缺少 `service` 時正確回傳 422，兩種缺少欄位的行為不一致。`POST /api/token` |

---

## 總結

| 統計項目 | 數量 |
|----------|------|
| 總測試案例 | 269 |
| ✅ 通過 | 251（93.3%） |
| ❌ 失敗 | 18（6.7%） |
| ⚠️ 缺陷 | 16 筆（Critical 3 / Major 6 / Minor 7） |

**整體評估**：功能性測試（CRUD 參數驗證、分頁排序、Smoke Test）表現良好，大部分端點行為符合預期。主要問題集中在**認證系統（Token）**與**資安防護**兩大區塊，建議優先處理 C-01（Token 認證繞過）、C-02（Stack Trace 洩露）、C-03（CORS 設定）三個 Critical 缺陷。
