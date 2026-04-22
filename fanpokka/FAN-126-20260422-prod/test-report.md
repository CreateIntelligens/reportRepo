# FAN-126 超紅新聲代｜Prod 自動化 Smoke 測試報告

> **工單**：[FAN-126](https://ai360c.atlassian.net/browse/FAN-126) 超紅新聲代｜Prod 自動化 Smoke 補測
> **父單**：[FAN-112](https://ai360c.atlassian.net/browse/FAN-112) 超紅新聲代 LIFF 報名功能
> **關聯**：[FAN-113](https://ai360c.atlassian.net/browse/FAN-113) stg 手動完整測試（已完成）
> **環境**：**PROD**（使用者授權，活動尚未推廣）
> **日期**：2026-04-22
> **測試方式**：chrome-devtools MCP 自動化（Vercel Agent Browser）
> **MeterSphere**：[FAN-126 超紅 prod 自動化 smoke 測試](http://10.9.0.11:8081/test-plan/41088405932679168)

---

## 測試摘要

| 項目 | 數值 |
|------|------|
| 總案例數 | 32 |
| ✅ PASSED | 26 |
| 🔴 FAILED | 2（非阻斷） |
| ⚠️ BLOCKED / N/A | 4 |
| 通過率 | 81.25%（若排除 N/A：26/28 = 92.9%） |
| 主流程是否受影響 | ❌ **無**（三校報名 happy path 全部走得完） |
| Bug 單 | **不開**（使用者指示，以 comment 形式記錄於 FAN-126） |

---

## 測試範圍

三校 LIFF 報名表：

| 代號 | 學校 | LIFF URL |
|------|------|---------|
| NHU | 南華大學 | https://liff.line.me/2007057346-4vR9x09N/2026TS-NHU |
| FCU | 逢甲大學 | https://liff.line.me/2007057346-4vR9x09N/2026TS-FCU |
| CUFA | 崇右影藝科技大學 | https://liff.line.me/2007057346-4vR9x09N/2026TS-CUFA |

---

## Prod 操作授權與足跡

- **授權**：活動尚未推廣，使用者明確同意少量資料新增
- **上限**：≤6 筆，**實際產生 4 筆**（NHU×1 / FCU×2 含 1 筆覆蓋 / CUFA×1）
- **姓名前綴**：`自動FAN126`
- **lineUserId**：`U3e1c1326c22c3880e2fcc34bbad9d91c`
- **已比對 Google Sheet**：三校分頁 `2026TS-NHU/FCU/CUFA` 均可依姓名快速定位清理

---

## 測試結果總覽

### A. 功能 smoke（14 PASSED / 1 N/A）

| # | 案例 | 結果 | 備註 |
|---|------|------|------|
| A-01 | NHU LIFF 載入 + landing 顯示 | ✅ PASSED | title=南華大學、landing=`/nhu/landing.jpg` |
| A-02 | FCU LIFF 載入 + landing 顯示 | ✅ PASSED | title=逢甲大學、landing=`/fcu/landing.jpg` |
| A-03 | CUFA LIFF 載入 + landing 顯示 | ✅ PASSED | title=崇右影藝科技大學、landing=`/cufa/landing.jpg` |
| A-04 | NHU 點 landing 進入表單 | ✅ PASSED | 7 欄位渲染正常 |
| A-05 | FCU 點 landing 進入表單 | ✅ PASSED | 同上 |
| A-06 | CUFA 點 landing 進入表單 | ✅ PASSED | 同上 |
| A-07 | NHU 成功送出 | ✅ PASSED | `POST /api/registration-campaign/2026TS-NHU` 200 |
| A-08 | FCU 成功送出 | ✅ PASSED | reqid=59, 200 |
| A-09 | CUFA 成功送出 | ✅ PASSED | reqid=118, 200 |
| A-10 | NHU 感謝頁 baseline | ✅ PASSED | 純圖 `/nhu/thanks.jpg` |
| A-11 | FCU 感謝頁 baseline | ✅ PASSED | 純圖 `/fcu/thanks.jpg` |
| A-12 | CUFA 感謝頁 baseline | ✅ PASSED | 純圖 `/cufa/thanks.jpg` |
| A-13 | NHU 同 UID 二次進入 | ✅ PASSED | `GET /api/registration-campaign?lineUserId=...` 200 回填 |
| A-14 | NHU 修改後覆蓋同列 | ✅ PASSED | FCU 覆蓋送出成功，Sheet 維持單列 |
| A-15 | 三校 LINE Tag 加標 | ⚠️ N/A | 需 Messaging API channel token，自動化範圍外 |

### B. 視覺 / OG baseline（4 PASSED / 2 N/A-INFO）

| # | 案例 | 結果 | 備註 |
|---|------|------|------|
| B-01 | 三校 landing 截圖 baseline | ✅ PASSED | 存 `screenshots/` |
| B-02 | landing src 比對 | ✅ PASSED | 每校獨立圖 `/{nhu,fcu,cufa}/landing.jpg` |
| B-03 | 三校 OG meta | ✅ PASSED | title/desc per-school；og:image **三校共用**（G1） |
| B-04 | favicon / manifest 比對 | ⚠️ INFO | 未宣告 `<link rel=icon>`，走 server 預設（G2） |
| B-05 | Retina 2x 解析度 | ✅ PASSED | NHU landing 1179×2406，渲染 480×900 = 2.46× |
| B-06 | RWD 斷點 500/768/1024/1440 | ✅ PASSED | 無破版 |

### C. 手動易漏邊角（8 PASSED / 2 FAILED / 2 N/A-INFO）

| # | 案例 | 結果 | 備註 |
|---|------|------|------|
| C-01 | Console errors/warnings | ⚠️ INFO | 3× `ERR_CONNECTION_REFUSED` 為 GA (本機 adblock)；CUFA 首次 prefetch 404 (G6) |
| C-02 | Network domain 稽核 | ✅ PASSED | 外連 domain 合理，無可疑連線 |
| C-03 | Security headers | 🔴 **FAILED** | 缺 HSTS/CSP/Referrer-Policy；`X-Powered-By: PHP/8.4.20`、`Server: nginx/1.18.0` 暴露（G3） |
| C-04 | LIFF SDK init timing | ✅ PASSED | NHU FCP 240ms / FCU 120ms / CUFA 168ms，皆 <1s |
| C-05 | 鍵盤 Tab / Enter 送出 | 🔴 **FAILED** | 提交按鈕為 `<div>` 非 `<button>`，鍵盤使用者無法送出（G7，WCAG 2.1.1） |
| C-06 | 跨校切換 state 隔離 | ✅ PASSED | 每校獨立狀態（同 UID 切 CUFA 未送過 → 空白；NHU/FCU 送過 → 回填） |
| C-07 | viewport meta / charset | ✅ PASSED | `width=375, initial-scale=1`；UTF-8 |
| C-08 | F5 重新整理後狀態 | ✅ PASSED | 丟棄暫存編輯，從 server 重新 prefill |
| C-09 | Submit 連點 2 次 | ✅ PASSED | 雙擊後 network 僅 1 筆 POST (reqid=194) |
| C-10 | 三校 performance 比對 | ✅ PASSED | TTFB 44-48ms / FCP 120-240ms / Load 587-650ms，三校一致 |
| C-11 | 感謝頁關閉按鈕 | ⚠️ N/A | 感謝頁純圖無按鈕設計（G8，PM 已確認以最新圖為準） |

---

## 🚨 8 項發現（全部記錄，**不開 bug**，僅作優化參考）

### P2 — 建議後續優化（不阻擋上線）

#### G3 ｜ Security Headers 缺失 + 伺服器版本暴露
- **測試**：C-03 FAILED
- **實測 `/api/registration-campaign` 回應標頭**：
  - ✅ 有：`X-Content-Type-Options: nosniff`, `X-Frame-Options: SAMEORIGIN`, `X-XSS-Protection: 1; mode=block`
  - ❌ 缺：`Strict-Transport-Security`, `Content-Security-Policy`, `Referrer-Policy`, `Permissions-Policy`
  - ⚠️ 暴露：`Server: nginx/1.18.0 (Ubuntu)`、`X-Powered-By: PHP/8.4.20`
- **影響**：無 HSTS 可被 HTTPS 降級；版本資訊便於針對性漏洞掃描。**不影響使用者操作**
- **建議**：nginx 加 `server_tokens off`、`fastcgi_hide_header X-Powered-By`；加 HSTS 與 CSP

#### G7 ｜ 提交按鈕非語意化 `<button>`，鍵盤使用者無法送出
- **測試**：C-05 FAILED
- **實測**：`<div class="button-text">提交表單</div>` 包在 `<div class="register-button">`，無 `tabindex`、無 `role="button"`；Enter-in-input 亦不觸發 submit
- **影響**：違反 WCAG 2.1 SC 2.1.1 Keyboard；**滑鼠/觸控送出正常**；LIFF 主場景為手機觸控，實務影響極小
- **建議**：改為 `<button type="button" @click="submit">提交表單</button>`

### P3 — 設計討論項（非缺陷）

#### G1 ｜ 三校 og:image 共用
- 社群分享預覽圖三校相同（`/top_singer_2026/og_image.jpg`）；僅 title / description per-school
- 若 PM 期望每校獨立預覽圖需補 `/{nhu,fcu,cufa}/og_image.jpg`

#### G5 ｜ payload 含 UI 未呈現的 `note.nickname`
- 前端自動帶入 `liff.getProfile().displayName` 送後端
- **已比對三校 Sheet 確認未落盤**；推測僅進後端 log 或 DB 其他欄位
- 建議確認用途，並視情況在同意條款揭露

### P4 — 資訊性（可忽略）

#### G2 ｜ HTML 無 `<link rel="icon">` / PWA manifest
- `/favicon.ico` 走 server 預設猜測；加到桌面無 manifest 體驗

#### G4 ｜ API 回應 message 文案與操作不符
- 寫入成功回 `{"message": "Resource get successfully"}`（get vs create 語意錯）
- DX 與 log 除錯混淆，建議改 `"Resource created successfully"` 或 `"Registration saved"`

#### G6 ｜ CUFA 無資料時 prefetch 回 404
- NHU/FCU 有資料 → `GET /api/registration-campaign?lineUserId=...` 200
- CUFA 無資料 → 同 endpoint **404** + console error
- 建議改 200+null 或 204，避免 console pollution

#### G8 ｜ 感謝頁純圖、無 close/back UI
- 送出後使用者僅能手動關閉 LIFF WebView
- PM 2026-04-21 確認以最新圖為準，**非缺陷**

---

## 效能數據（C-10 三校比對）

| 指標 | NHU | FCU | CUFA |
|------|-----|-----|------|
| TTFB | 45ms | 48ms | 44ms |
| FCP | 240ms | 120ms | 168ms |
| DOM Interactive | 202ms | 73ms | 113ms |
| Load Event | 639ms | 650ms | 587ms |
| Transfer Size | 6144B | 6145B | ~6KB |

三校效能一致，皆遠低於 CWV 目標（LCP < 2.5s）。

---

## 排除項（自動化做不到 / Prod 不宜）

- M1 LINE Login（已手動登入後由 MCP 接手）
- 網路中斷 / API 500 模擬（需 stg）
- 截止狀態（需後端時間操作）
- LINE WebView 實機三平台（iOS / Android / Desktop）
- LINE 分享預覽實測（需真實 OA）
- M3 15 欄位空值驗證（MCP fill 空字串不觸發 Vue v-model）

---

## 產出檔案

- 測試計畫原檔：`fanpokka_test` repo → `test-plans/automation/FAN-126-prod-automation.md`
- Baseline 截圖 9 張：`test-plans/automation/screenshots/FAN-126-baseline/`
  - `{NHU,FCU,CUFA}-landing.png`
  - `{NHU,FCU,CUFA}-thanks.png`
  - `FCU-form-prefilled.png`
  - `NHU-rwd-{500,768,1024,1440}.png`
- MS 結果 mapping：32/32 已寫入

---

## 結論

**無影響主流程走完的嚴重問題，可正常交付上線。**

8 項發現皆為優化建議或設計討論項，依使用者指示不開 bug，已於 FAN-126 comment 記錄供後續參考。
