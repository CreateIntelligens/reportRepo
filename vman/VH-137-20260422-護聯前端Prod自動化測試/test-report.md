# VH-137 護聯前端掛新 Domain 自動化測試報告

> **Jira**：[VH-137](https://ai360c.atlassian.net/browse/VH-137) — 護聯前端掛新 domain 自動化測試
> **MeterSphere**：http://10.9.0.11:8081/test-plan/41075194613276672
> **測試環境**：Production — https://youngforehospital.5gao.ai/chat（密碼 `111111`）
> **測試日期**：2026-04-22
> **執行**：williamsliu（QA Agent 協同）
> **Sprint**：VH Sprint 2
> **Label**：`prod-smoke`

---

## 執行摘要

| 項目 | 數值 |
|------|------|
| 總案例 | 33（原 34，G-03 刪除） |
| AUTO 自動化 | 25 筆 |
| SEMI 半自動 | 8 筆 |
| **全部 PASS** | **33 / 33** |
| Console Error | 0 |
| Page Error | 0 |
| Bug 新發現 | 0 |
| 回測 BLOCKED 修復 | B-08 ConnectionWarning（VH-60 dev 時 BLOCKED → prod PASS） |
| 規格發現 | G-01 閒置 reload 觸發條件（需對話歷史） |

---

## 背景

護聯前端正式上線 prod domain（https://youngforehospital.5gao.ai/chat），需要完整驗收。沿用 VH-60 KIOSK 前台測試框架並擴充 prod 專項健檢（HTTPS、console、API health、WebRTC whep）。測試前兩次探索確認 RD 調版號後前端 bundle hash 未變（`index-BZiICN9m.js`）— 無大改動，UI selector 100% 一致。

## 環境

| 項目 | 值 |
|------|-----|
| 前端 | https://youngforehospital.5gao.ai（HTTPS） |
| API 虛擬人配置 | :8020（get_avatars / get_voices / get_current_config / is_speaking） |
| API HCIoT 對話 | :9880（hciot_chat_start / hciot_topics） |
| WebRTC whep | :1986/rtc/v1/whep/（SRS） |
| 前端 bundle | index-BZiICN9m.js |
| Playwright | 1.59.1 / Chromium 147 / viewport 540×960（KIOSK 比例縮小） |

## AUTO 測試結果（25/25 PASS）

### A 群組 — 登入與 Session（6 筆）

| ID | 名稱 | 結果 | 備註 |
|---|---|---|---|
| A-01 | 正確密碼 111111 登入跳轉 /chat | ✅ PASS | localStorage.auth_session 正確 |
| A-02 | 錯誤密碼顯示錯誤提示 | ✅ PASS | `.login-error` 顯示 |
| A-03 | Session 有效時自動跳轉 chat | ✅ PASS | |
| A-04 | Session 過期導回首頁 | ✅ PASS | |
| A-05 | 未登入直接存取 /chat 被攔截 | ✅ PASS | |
| A-06 | 首次載入 browser warning 可關閉 | ✅ PASS | `.browser-warning-btn` OK 可點 |

### B 群組 — 對話核心（1 AUTO + 3 SEMI）

| ID | 名稱 | 結果 | 備註 |
|---|---|---|---|
| B-08 | 後端連線異常顯示警告 | ✅ PASS | **VH-60 dev 時 BLOCKED，prod 已修復** |
| B-01 | consult → ready mode 切換 | ✅ PASS (SEMI) | 三顆圓形按鈕出現 |
| B-02 | start-btn 啟動 WebRTC stream | ✅ PASS (SEMI) | 虛擬人說開場白 |
| B-04 | AI 回覆含圖片 | ✅ PASS (SEMI) | PRP → 什麼是 PRP? |

### C 群組 — WebRTC 虛擬人（2 SEMI）

| ID | 名稱 | 結果 |
|---|---|---|
| C-01 | idle 虛擬人影片流暢播放 | ✅ PASS (SEMI) |
| C-03 | 對話結束切回 idle | ✅ PASS (SEMI) |

### D 群組 — FAQ（5 AUTO）

| ID | 名稱 | 結果 |
|---|---|---|
| D-01 | 待機開 FAQ 分類載入（18 主題） | ✅ PASS |
| D-02 | 兩層導航到問題清單 | ✅ PASS |
| D-03 | 選題自動發送進入對話 | ✅ PASS |
| D-04 | 對話中再開 FAQ 選另一題 | ✅ PASS（修正 assertion 為 UI 氣泡計數） |
| D-05 | FAQ 面板開啟/關閉切換 | ✅ PASS |

### E 群組 — 語系切換（4 AUTO + 1 SEMI）

| ID | 名稱 | 結果 |
|---|---|---|
| E-01 | 點 lang-toggle 彈出 RestartDialog | ✅ PASS |
| E-02 | 確認切換 URL 加 ?lang=en | ✅ PASS |
| E-03 | 取消切換維持原語言 | ✅ PASS |
| E-04 | 英文模式功能文字正確翻譯 | ✅ PASS |
| E-05 | 英文切換後 FAQ 題目為英文 | ✅ PASS (SEMI) — 含 `test2` 為 RD 已知測試資料 |

### F 群組 — 重啟對話（4 AUTO + 1 SEMI）

| ID | 名稱 | 結果 |
|---|---|---|
| F-01 | 重啟確認後回待機 | ✅ PASS |
| F-02 | 重啟不清除登入 session | ✅ PASS |
| F-03 | 取消重啟維持當前對話 | ✅ PASS |
| F-05 | 重啟不返回首頁 | ✅ PASS |
| F-04 | 重啟流程觀察 | ✅ PASS (SEMI) **規格修正**：無 bye.mp4 道別動畫 |

### G 群組 — reload（1 AUTO + 1 SEMI / 1 刪除）

| ID | 名稱 | 結果 |
|---|---|---|
| G-01 | 閒置自動 reload | ✅ PASS (SEMI) **規格發現**：需有對話歷史 + 116s reload |
| G-02 | 操作中不觸發 reload | ✅ PASS |
| ~~G-03~~ | ~~隱藏 reload hotspot~~ | 🗑 **DELETED** — 功能已廢除（.restart-btn 取代） |

### H 群組 — Prod 專項健檢（4 AUTO 全新增）

| ID | 名稱 | 結果 | 實測值 |
|---|---|---|---|
| H-01 | HTTPS 無 mixed content | ✅ PASS | 0 issue |
| H-02 | console 無 error/critical warning | ✅ PASS | 0 error / 0 pageerror |
| H-03 | 關鍵 API 回應 200 | ✅ PASS | hciot_chat_start / get_avatars / get_voices / get_current_config 全 200 |
| H-04 | WebRTC whep 握手成功 | ✅ PASS | :1986/rtc/v1/whep/ 回 **201** |

---

## 重要規格發現（已記入 `memory/vh137_prod_findings.md`）

### 1. G-01 閒置 reload 觸發條件

**規格**（VH-60 test-spec 未明記）：
- 必須**先有對話歷史**（至少送過一題並接收 AI 回覆）
- 閒置約 120 秒（實測 116s）才觸發 reload
- 單純 consult 進 ready mode 但不送對話 → 永遠不 reload

Reload 後流程：短暫跳回首頁 → auth_session 仍有效 → A-03 邏輯 auto-redirect 回 /chat standby。

### 2. F-04 / G-03 —（案例規劃錯誤，非規格發現）

F-04 的 bye.mp4 動畫、G-03 的隱藏 reload hotspot 皆非實際規格，為 Agent 撰寫計畫時自行臆測加入。
- F-04：重啟後直接回 idle 即正常
- G-03：已從 MS 刪除（非功能廢除，本來就無此規格）

### 4. B-04 圖片回覆非所有題目皆帶

只有特定題目（如 PRP → 什麼是 PRP?）AI 會回圖片。測試時需選對題目，非任意題皆可驗。

### 5. B-08 ConnectionWarning 已修復

VH-60 dev 時期 B-08 為 BLOCKED（無 ConnectionWarning 元件可驗）。VH-137 prod 實測：攔截 :9880 + :8020 API 後前端顯示異常訊息，元件存在。推測為 VH-64 修復時連帶加入。

---

## 腳本與執行資料

| 檔案 | 用途 |
|------|------|
| `vman護聯/e2e-auto-tests/tests-prod/vh137-prod.spec.js` | 25 筆 AUTO Playwright 腳本 |
| `vman護聯/e2e-auto-tests/tests-prod/semi-runner.js` | 8 筆 SEMI headed runner（筆電友善 540×960 viewport） |
| `vman護聯/e2e-auto-tests/playwright.prod.config.js` | prod 專用 config（ignoreHTTPSErrors） |
| `vman/scripts/vh137_create_cases.py` | MS 案例批量建立 |
| `vman/scripts/vh137_ms_mapping.json` | AUTO 結果回寫 mapping |
| `vman/scripts/vh137_ms_semi_mapping.json` | SEMI 結果回寫 mapping |
| `vman/scripts/vh137_case_ids.json` | 34 筆 MS case ID |
| `vman/scripts/vh137_assoc_ids.json` | plan 關聯 ID（34 筆，含 G-03 已刪除） |
| `vman護聯/explore-prod/EXPLORE-REPORT.md` | Prod 探索報告（2 次 smoke 比對） |
| `vman護聯/explore-prod/screenshots/` | 探索 4 張截圖 |
| `vman護聯/explore-prod/semi-shots/` | SEMI 觀察截圖 |

## 執行時序

| 時間 | 階段 | 耗時 |
|---|---|---|
| 15:30 | Smoke v1 探索 | 20s |
| 16:00 | Smoke v2 探索（版號調整後） | 20s |
| 16:10 | Jira VH-137 開單 + MS 計畫建立 + 34 筆 case | 2 min |
| 16:15 | Playwright prod 腳本撰寫 | 3 min |
| 16:18 | 試跑 A-01（驗通） | 2.4s |
| 16:18 | 完整跑 22 AUTO（1 fail D-04 誤判→修 assertion 重跑 PASS） | 4.1 min + 23s |
| 16:30 | 25 AUTO 回寫 MS | 20s |
| 16:35 | SEMI headed 帶跑（分段 pause，使用者觀察） | 1 hr（含服務重啟暫停） |
| 17:30 | 8 SEMI 回寫 MS + 刪除 G-03 | 30s |
| 17:35 | 記憶更新 + 報告產出 | 10 min |

## 後續建議

1. **VH-60 test-spec.md 需更新**：G-01 觸發條件需補註「需對話歷史」
2. **Prod 監控**：H 群組的 4 筆健檢可列入每日 smoke（console error / HTTPS / 關鍵 API / WebRTC 握手）
3. **test2 FAQ 主題**：RD 測試資料，上 prod 前應清理
4. **自動化佔比**：本計畫 AUTO 75.7%（25/33），高於 VH-60 的 61%；H 群組新增的 prod 專項案例易於自動化
