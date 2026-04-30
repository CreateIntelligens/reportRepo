# FAN-95 / FAN-99 / FAN-129 表單選項編輯功能驗證報告

> **工單**：[FAN-95](https://ai360c.atlassian.net/browse/FAN-95) / [FAN-99](https://ai360c.atlassian.net/browse/FAN-99) / [FAN-129](https://ai360c.atlassian.net/browse/FAN-129)
> **QA 子單**：[FAN-136](https://ai360c.atlassian.net/browse/FAN-136) / [FAN-133](https://ai360c.atlassian.net/browse/FAN-133) / [FAN-134](https://ai360c.atlassian.net/browse/FAN-134)
> **環境**：UAT（https://uat.fanpokka.ai/）
> **日期**：2026-04-30
> **測試方式**：chrome-devtools MCP 自動化 + 人工輔助
> **MeterSphere**：[[FAN-95/99/129] 表單選項編輯功能驗證](http://10.9.0.11:8081/#/test-plan/testPlanIndexDetail?id=2242969361006592&orgId=100001&pId=1102140147769344)

---

## 測試摘要

| 項目 | 數值 |
|------|------|
| 總案例數 | 23 |
| ✅ SUCCESS | 22 |
| 🔴 ERROR | 0 |
| ⚠️ BLOCKED | 1（FAN-137 阻擋） |
| 通過率 | **95.7%**（22/23） |
| 主要 Bug 修復驗證 | ✅ **全部通過** |
| 發現新 Bug | 1 件（FAN-137，既存未測） |

---

## 三張工單修復驗證結論

| 工單 | 問題描述 | 修復驗收 |
|------|---------|---------|
| **FAN-95** | 編輯 radio/checkbox/select 欄位時，選項（TagsInput）顯示為空 | ✅ PASS — M1-01/02/03 全通過 |
| **FAN-99** | 後台類型下拉選單 IMAGE_RADIO / IMAGE_CHECKBOX 顯示英文 | ✅ PASS — M3-01/02 全通過 |
| **FAN-129** | 官網表單全 5 種有選項類型重新編輯時選項消失 | ✅ PASS — M1-01~03 + M2-01/02 全通過 |

---

## 根因確認

| 工單 | 根因 | 修正位置 |
|------|------|---------|
| FAN-95 / FAN-129 | `EditAction::mutateRecordDataUsing` 無條件執行 `unset($data['options'])`，TagsInput 開啟編輯時拿不到既有資料 | `FormFieldsRelationManager.php` |
| FAN-99 | `CanSelectable::forSelect()` 使用 `__($case->name)` 翻譯，`lang/zh_TW.json` 缺少 `IMAGE_RADIO` / `IMAGE_CHECKBOX` 大寫 key | `lang/zh_TW.json` |

---

## 測試範圍與分類

| 模組 | 案例數 | 分類 | 關聯工單 |
|------|--------|------|---------|
| M1 文字選項資料保留 | 5 | SEMI | FAN-95、FAN-129 |
| M2 圖片選項資料保留 | 3 | MANUAL | FAN-129 |
| M3 後台類型下拉語言 | 4 | AUTO/SEMI | FAN-99 |
| M4 邊界值與例外 | 6 | SEMI | FAN-95、FAN-129 |
| M5 回歸測試 | 5 | AUTO/MANUAL | 全部 |
| **合計** | **23** | — | — |

---

## 詳細測試結果

### M1：文字選項類型資料保留（FAN-95 / FAN-129 核心）

| # | 案例 | 結果 | 說明 |
|---|------|------|------|
| M1-01 | radio 欄位選項編輯後預填 | ✅ SUCCESS | 紅色/藍色/綠色 3 個 tag 正確預填 |
| M1-02 | checkbox 欄位選項編輯後預填 | ✅ SUCCESS | 閱讀/運動/音樂/旅遊 4 個 tag 正確預填 |
| M1-03 | select 欄位選項編輯後預填 | ✅ SUCCESS | 選單1/選單2 正確預填 |
| M1-04 | 修改選項後前台顯示正確 | ✅ SUCCESS | 刪選單2加選單3後前台同步更新 |
| M1-05 | 連續 3 次 edit-save cycle | ✅ SUCCESS | 每輪選項均正確保留，無遺失 |

### M2：圖片選項類型資料保留（FAN-129）

| # | 案例 | 結果 | 說明 |
|---|------|------|------|
| M2-01 | image_radio 編輯後文字+圖片預填 | ✅ SUCCESS | 圖單一/圖單二，文字+圖片縮圖均正確 |
| M2-02 | image_checkbox 編輯後文字+圖片預填 | ✅ SUCCESS | 圖多一/圖多二，文字+圖片縮圖均正確 |
| M2-03 | 前台 image 類型選項文字中文 | ✅ SUCCESS | 圖單一/圖單二/圖多一/圖多二 正確顯示 |

### M3：後台類型下拉語言（FAN-99 核心）

| # | 案例 | 結果 | 說明 |
|---|------|------|------|
| M3-01 | 新增欄位類型下拉 IMAGE_RADIO/CHECKBOX 顯示中文 | ✅ SUCCESS | 顯示「圖片單選」「圖片多選」，無英文殘留 |
| M3-02 | 編輯既有 image 欄位類型欄顯示中文 | ✅ SUCCESS | 類型欄正確顯示中文 |
| M3-03 | 欄位清單 type 欄 image 類型顯示中文 | ✅ SUCCESS | 清單欄位類型全部中文 |
| M3-04 | 前台 image 類型選項文字語言 | ✅ SUCCESS | 前台圖片選項中文標籤正確 |

### M4：邊界值與例外

| # | 案例 | 結果 | 說明 |
|---|------|------|------|
| M4-01 | 單一選項邊界 | ✅ SUCCESS | 1 個 tag「是」正確保留 |
| M4-02 | 10 個選項邊界 | ✅ SUCCESS | 選項1～10 全數保留，無遺漏 |
| M4-03 | 特殊字元選項 | ✅ SUCCESS | `選項A & B`、`C/D（混合）`、`E「引號」` 均無跳脫 |
| M4-04 | 同族切換 radio→checkbox | ✅ SUCCESS | 切換後選項保留 |
| M4-05 | 跨族切換 radio→image_radio | ✅ SUCCESS | 切換後選項正確清空，顯示空 Repeater |
| M4-06 | 其他選項 toggle 狀態保留 | ✅ SUCCESS | toggle 開啟狀態及標籤文字均正確保留 |

### M5：回歸測試

| # | 案例 | 結果 | 說明 |
|---|------|------|------|
| M5-01 | 全 7 種類型欄位正常新增 | ✅ SUCCESS | 文字/多行文字/下拉/單選/多選/圖片單選/圖片多選 均正常建立 |
| M5-02 | 欄位刪除正常 | ✅ SUCCESS | 刪除後欄位從清單消失，DB 乾淨 |
| M5-03 | 欄位 drag-to-reorder 排序正常 | ✅ SUCCESS | 拖曳後前台順序同步正確 |
| M5-04 | 前台表單提交含選項類型欄位 | ✅ SUCCESS | 選「選單1」送出，後台 Submission 正確記錄 `qatest: 選單1` |
| M5-05 | 前台「其他」選項自訂輸入 | ⚠️ BLOCKED | 測試時發現 FAN-137（checkbox 多選+其他連動異常），Bug 阻擋送出，待修復後重測 |

---

## 發現新 Bug

### FAN-137：checkbox 多選+其他 全選項連動異常＋送出驗證失敗

| 項目 | 內容 |
|------|------|
| 工單 | [FAN-137](https://ai360c.atlassian.net/browse/FAN-137) |
| 父單 | FAN-67（表單添加「其他」選項） |
| 優先級 | High |
| 關聯 | FAN-94（FAN-67 QA 子單，原從未執行） |
| 根因 | `form-detail.blade.php` 第 112 行：「其他」checkbox 使用 `wire:model.live`，一般選項使用 `wire:model`，混用同一 array 屬性導致 Livewire re-render 覆蓋未同步狀態 |
| 影響 | 1. 勾選/取消任一選項時全部連動；2. 送出時出現「欄位必須是陣列」 |
| 歸屬 | FAN-67 原始實作遺留，非本次三張工單修正引入 |

---

## 測試環境設定

| 項目 | 內容 |
|------|------|
| 後台 URL | https://uat.fanpokka.ai/admin/ |
| 前台 URL | https://uat.fanpokka.ai/GDiN2eOb9x（0430qatest表單） |
| 測試帳號 | super_admin3@example.com（後台）、Williams（前台） |
| 測試表單 | Content ID 129 — 0430qatest表單 |

---

## 截圖存證

存放於 `fanpokka/test-plans/manual/screenshots/`：

| 截圖 | 說明 |
|------|------|
| FAN-95-M1-01-radio-edit-prefilled-PASS.png | M1-01 radio 選項預填 |
| FAN-95-M1-02-checkbox-edit-prefilled-PASS.png | M1-02 checkbox 選項預填 |
| FAN-95-M1-03-select-edit-prefilled-PASS.png | M1-03 select 選項預填 |
| FAN-99-M3-01-type-dropdown-zh.png | M3-01 類型下拉顯示中文 |
| FAN-99-M3-04-frontend-options-zh-PASS.png | M3-04 前台選項文字中文 |
| FAN-95-M5-04-frontend-submit-PASS.png | M5-04 前台送出成功 |
| FAN-95-M5-04-backend-submission-PASS.png | M5-04 後台記錄驗證 |
| M5-01-all-7-types-PASS.png | M5-01 全 7 種類型新增 |
| M1-04-M4-02-M4-03-frontend-PASS.png | M1-04 + M4-02/03 前台確認 |

---

## 結論

**FAN-95、FAN-99、FAN-129 三張工單的修復已全部驗證通過。**

- 22/23 案例 SUCCESS，通過率 95.7%
- 唯一 BLOCKED（M5-05）因 FAN-137（既存未測 Bug）阻擋，與本次修復無關
- FAN-137 已開立於 FAN-67 底下，待 RD 修復後重測 M5-05
