# VH-141 輪椅版功能端點測試報告

> **Jira**：[VH-141](https://ai360c.atlassian.net/browse/VH-141) — 輪椅版功能端點測試（父單 VH-48 [護聯專案]前端KIOSK切版串接-輪椅版）
> **MeterSphere**：http://10.9.0.11:8081/test-plan/43722835792830464
> **測試環境**：Production — https://youngforehospital.5gao.ai/chat（密碼 `111111`）
> **測試日期**：2026-04-24
> **執行**：williamsliu（QA Agent 協同）
> **Sprint**：VH Sprint 2
> **Label**：`VH-141`

---

## 執行摘要

| 項目 | 數值 |
|------|------|
| 總案例 | 33 |
| AUTO 自動化 | 15 筆 |
| SEMI 半自動 | 12 筆 |
| MANUAL 手動 | 6 筆 |
| **SUCCESS** | **28 / 33（84.8%）** |
| **ERROR** | **5 / 33（15.2%）** |
| 新發現 Bug | **2 張**（VH-142 / VH-143）|
| Console Error | 0 |

### 結果分佈

| 群組 | 總數 | SUCCESS | ERROR |
|------|------|---------|-------|
| W 切換機制 | 6 | 6 | 0 |
| X 情境切換 | 10 | 8 | 2（X-06 / X-10）|
| Y 持久性 | 4 | 2 | 2（Y-02 / Y-03）|
| Z UI 規格 | 9 | 9 | 0 |
| U 基礎功能 | 3 | 2 | 1（U-03）|
| VH140 品牌字 | 1 | 1 | 0 |

---

## 背景

VH-48 護聯 KIOSK 輪椅版前端切版串接已完成，RD 推版至 prod https://youngforehospital.5gao.ai/。本次測試專注於：

1. **輪椅版切換機制**（W）— logo 下方 hotspot 點擊觸發 ButtonPositionDialog
2. **情境切換**（X）— 對話中 / AI 回答中 / 錄音中 / FAQ 開啟中 / Dialog 堆疊 / 斷線下切換
3. **持久性**（Y）— 連續切換 / reload / auto-reload / 模式切換 position 保留
4. **UI 規格**（Z）— Figma 座標、按鈕尺寸、漸層遮罩、對話歷史佈局
5. **基礎功能覆蓋**（U）— 輪椅版模式下完整流程 / 語系 / auto-reload
6. **VH-140 品牌字移除**驗證

沿用 VH-137 prod 測試基礎，新增輪椅版專項案例。

## 環境

| 項目 | 值 |
|------|-----|
| 前端 | https://youngforehospital.5gao.ai（HTTPS） |
| API 虛擬人配置 | :8020 |
| API HCIoT 對話 | :9880 |
| WebRTC whep | :1986/rtc/v1/whep/ |
| 前端分支 | feat/john/hc commit 8cbf48f |
| Playwright | 1.59.1 / Chromium 147 / viewport 1440×2560（AUTO）/ 540×960（SEMI headed） |

---

## AUTO 測試結果（15 筆，13 SUCCESS / 2 ERROR，其中 2 BLOCKED 由 SEMI X-03 覆蓋改 SUCCESS）

### W 切換機制（4 AUTO）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| W-01 | Hotspot 存在 + Dialog 開啟含 2 選項 | ✅ PASS | box 186×261 H/W=1.403 |
| W-02 | 預設 pendingPosition 為相反 | ✅ PASS | top 時 bottom active；反之亦然 |
| W-03 | 當前位置選項 disabled | ✅ PASS | cursor: not-allowed |
| W-04 | 切換後 Dialog 關閉 + position 變更 | ✅ PASS | .standby-bottom-btn 可見 |

### Y 持久性（2 AUTO）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| Y-01 | top↔bottom 連切 5 次無錯誤 | ✅ PASS | console error/pageerror = 0 |
| Y-02 | Reload 後 position 是否重置 | ❌ **ERROR** | **RESET_TO_TOP 確認 → 開 Bug VH-142** |

### Z UI 規格（9 AUTO）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| Z-01 | standby bar 尺寸 1008×100 / #5593B5 | ✅ PASS | 寬高比 10:1 / rgb(85,147,181) |
| Z-02 | standby QR 位置 | ✅ PASS | .standby-qr-bottom 可見 |
| Z-03 | 半透明白底背板 | ✅ PASS | rgba(255,255,255,0.5) |
| Z-04 | ready 三層按鈕可見 | ✅ PASS | FAQ/restart 按鈕可見 |
| Z-05 | conversation 底部主按鈕容器 | ✅ PASS | [class*='bottom'] 存在 |
| Z-06 | bottom-recording-button.svg 資源 | ✅ PASS | HTTP 200 |
| Z-07 | conversation bottom viewport class | ✅ PASS（SEMI X-03 覆蓋）| 原 AUTO 因 WebRTC+FAQ 觸發不穩 BLOCKED → SEMI 實測通過 |
| Z-08 | bottom-anchored 佈局 | ✅ PASS | flex column |
| Z-09 | 漸層遮罩 mask-image | ✅ PASS（SEMI X-03 覆蓋）| 同 Z-07；CSS source code 確認 + 實機視覺驗證 |

### VH140 品牌字（1 AUTO）

| ID | 名稱 | 結果 |
|----|------|------|
| VH140-01 | 前端無「此服務由創造智能提供技術支持」 | ✅ PASS |

---

## SEMI 測試結果（12 筆，11 SUCCESS / 1 ERROR）

### W 切換機制（2 SEMI）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| W-05 | Dialog overlay 外側取消 | ✅ PASS | Dialog 關閉 + position 不變 |
| W-06 | 中英雙語 title | ✅ PASS | 中「是否變更按鈕位置？」/ 英「Change button position?」 |

### X 情境切換（6 SEMI）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| X-01 | standby top→bottom 佈局 | ✅ PASS | 底部 bar + 左上 disclaimer + 右側 QR |
| X-02 | ready top→bottom 三層堆疊 | ✅ PASS | start-chat bar + restart/FAQ row + 訊息泡泡 |
| X-03 | **conversation 中切換** | ✅ PASS | 對話氣泡保留 / viewport 移至肩膀線 / 漸層淡出實測確認（覆蓋 Z-07/Z-09）|
| X-04 | **AI 回答中切換** | ✅ PASS | TTS 不中斷 / 虛擬人影片持續 / UI 更新 / 氣泡保留 |
| X-07 | **RestartDialog 開啟中切換** | ✅ PASS | hotspot 被 overlay(z-index:55) 遮住；點該區會關閉 RestartDialog（overlay click handler）但不觸發 ButtonPositionDialog |
| X-09 | 斷線下切換 | ✅ PASS | ConnectionWarning 紅條顯示中切換功能正常 |

### Y 持久性（2 SEMI）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| Y-03 | 閒置 120s auto-reload 後 position | ❌ **ERROR** | **閒置 125s 觸發 reload 後回 top → Bug VH-142** |
| Y-04 | 模式切換保持 position | ✅ PASS | standby/ready/conversation 三模式皆 bottom |

### U 基礎功能（2 SEMI）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| U-01 | 輪椅版完整流程 | ✅ PASS | 登入→bottom→consult→start→FAQ→PRP「什麼是 PRP?」→AI 回覆含圖片→重啟 |
| U-02 | 輪椅版中英切換走過 | ✅ PASS | lang-toggle → RestartDialog → 英文介面底部佈局保留 |

---

## MANUAL 測試結果（6 筆，4 SUCCESS / 2 ERROR）

| ID | 名稱 | 結果 | 備註 |
|----|------|------|------|
| X-05 | 錄音中切換 | ✅ PASS | 使用者手動測試通過 |
| X-06 | **FAQ panel 開啟中切換** | ❌ **ERROR** | **首次進入未對話 + FAQ 開啟中切換 → 底部按鈕消失必須關 FAQ 才出現；已對話過再開 FAQ 切換則正常 → Bug VH-143** |
| X-08 | 語言 RestartDialog 開啟中切換 | ✅ PASS | |
| X-10 | **斷線中切 bottom → 恢復網路後切回 top** | ❌ **ERROR** | **進入對話後，斷線回連都會變回一般版 → Bug VH-142（同類）** |
| U-03 | 閒置 auto-reload 後 position | ❌ **ERROR** | 等同 Y-03 → Bug VH-142 |

（*U-03 雖歸在 U 群組，此處補列以利對照*）

---

## 🐛 新發現 Bug（2 張，已建單）

### [VH-142](https://ai360c.atlassian.net/browse/VH-142) P1 輪椅版 position 未持久化

**覆蓋案例**：Y-02 / Y-03 / U-03 / X-10（4 筆 ERROR）

**問題**：`actionButtonPosition` 為 `shallowRef<"top">` 預設值，未持久化到 localStorage。以下三種情境會讓輪椅版設定被重置為一般版：
- 手動 F5 reload
- 閒置 120s auto-reload
- 斷線回連

**對輪椅使用者影響**：每次 reset 都需重新點擊高處 hotspot（canvas 座標 36,172 共 186×261 px）才能切回輪椅版。在網路不穩的醫院 KIOSK 場景（斷線回連頻繁），體驗極差。

**建議修正**：`actionButtonPosition` 持久化到 localStorage（建議 key: `button_position`），初始化時讀取 fallback `"top"`。

**截圖**：[screenshots/Y-03-after-auto-reload.png](screenshots/Y-03-after-auto-reload.png)

---

### [VH-143](https://ai360c.atlassian.net/browse/VH-143) P2 首次進入未對話時 FAQ 開啟中切換底部按鈕消失

**覆蓋案例**：X-06

**問題**：特定前置條件（首次進入 + 虛擬人自我介紹結束後 + 未對話 + FAQ 開啟中）切換輪椅版，底部按鈕在 FAQ 模式下消失，必須關閉 FAQ 才出現。

**對比**：已對話過再開 FAQ 切換 → 正常（下方區塊功能正確顯示）。

**建議修正**：檢查 `ChatActionControls.vue` / `ChatReadyOverlay.vue` 在 FAQ panel open + dialogHistory 為空時 bottom layout 的 render 條件。

---

## 重要發現

### 1. X-04 — AI 說話中切換不中斷 TTS / 虛擬人影片

輪椅版切換為純前端 state，不影響 WebRTC stream 與 TTS 播放。切換時機彈性高，這是正向設計。

### 2. X-07 — Dialog 堆疊行為正確

RestartDialog（z-index:55）正確遮蓋 hotspot；點擊 overlay 會先關閉當前 Dialog。符合預期的 Dialog 堆疊管理。

### 3. X-03 — 對話歷史在 bottom 模式下 viewport 正確移至肩膀線

`.dialog-history-viewport-bottom` + `.dialog-history-viewport-gradient` class 套用後 viewport 從 Y≈1350 開始，上方 CSS mask `linear-gradient(to bottom, transparent 0%, black 120px)` 實測視覺淡出正確。

### 4. X-09 — 斷線狀態下切換仍可運作

`actionButtonPosition` 為純前端 shallowRef，不依賴 API。斷線紅條出現時仍能正常切換輪椅版。這是正向設計（降低故障面）。

### 5. W-02 — pendingPosition 預設為相反

ButtonPositionDialog 開啟時自動預選「當前之外」的選項，並將當前位置設為 disabled（cursor: not-allowed）。UI/UX 設計妥當，減少誤點。

---

## 腳本與資料

| 檔案 | 用途 |
|------|------|
| `vman護聯/e2e-auto-tests/tests-prod/vh141-prod.spec.js` | 15 筆 AUTO Playwright 腳本 |
| `vman護聯/e2e-auto-tests/tests-prod/vh141-semi-runner.js` | SEMI headed runner（pause/resume 式觀察） |
| `vman護聯/e2e-auto-tests/tests-prod/vh141-explore.spec.js` | Smoke 探索腳本（5 筆） |
| `vman/scripts/vh141_create_cases.py` | MS 案例批量建立（33 筆） |
| `vman/scripts/vh141_ms_mapping.json` | AUTO 結果回寫 mapping（16 筆含 Y-02 ERROR）|
| `vman/scripts/vh141_ms_semi_mapping.json` | SEMI 結果回寫 mapping（14 筆）|
| `vman/scripts/vh141_create_bugs.py` | Bug 批次建立腳本（VH-142 / VH-143）|
| `vman/scripts/vh141_case_ids.json` | 33 筆 MS case ID |
| `vman/scripts/vh141_assoc_ids.json` | plan 關聯 ID |
| `vman護聯/explore-prod-wheelchair/semi-shots/` | SEMI 觀察截圖（15 張）|

## 執行時序

| 時間 | 階段 | 耗時 |
|---|---|---|
| 10:30 | Pull RD 18 commits（含輪椅版 feat）+ 讀 AGENTS.md + 代碼 | 15 min |
| 10:40 | Prod smoke 探索（5 筆）| 5 min |
| 10:50 | 計畫草擬 + 使用者 review 定稿 33 筆 | 15 min |
| 11:10 | MS 計畫建立 + 33 案例建 + 關聯 plan | 5 min |
| 11:15 | Jira VH-141 comment 寫入 | 1 min |
| 11:20 | AUTO 15 筆跑 + 回寫 MS | 20 min |
| 11:40 | SEMI 12 筆帶跑（使用者協同觀察）| 1.5 hr |
| 14:00 | MANUAL 6 筆使用者手動測試 + MS 回寫 | 30 min |
| 14:30 | Bug 草稿 + VH-142 / VH-143 建立 | 10 min |
| 14:45 | 報告產出 | 15 min |

## 後續建議

1. **VH-142 為 P1 優先修復**：輪椅使用者無障礙關鍵體驗 Bug。建議 localStorage 持久化 button_position。
2. **VH-143 為 P2**：前置條件特殊但可重現；RD 檢查 FAQ panel + 空 dialogHistory + bottom layout 的 render 邏輯。
3. **Prod 監控**：本次 H 群組（console / HTTPS / 關鍵 API）由 VH-137 已建立 smoke，本輪未重複驗證，可繼續沿用。
4. **Figma hotspot 視覺提示**：工單原描述「logo 下方虛線區塊」，但實作為透明 `opacity:0` hotspot。使用者確認「就是用 figma 提示人類但實際不會顯示出來」——規格確認無需加視覺提示。
5. **AUTO 佔比**：15/33 = 45.5%，低於 VH-137 的 75.7%，因情境切換類（X）多為時序 / 視覺觀察難自動化。
