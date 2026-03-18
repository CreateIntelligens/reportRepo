# AGO-135 測試報告 — [UAT] 訊息發送預設顯示文字修正

| 項目 | 內容 |
|------|------|
| 工單 | [AGO-135](https://ai360c.atlassian.net/browse/AGO-135) |
| 子工單（Feature 測試） | [AGO-142](https://ai360c.atlassian.net/browse/AGO-142) |
| 測試人員 | williamsliu |
| 測試日期 | 2026-03-18 |
| 測試環境 | UAT — aitago-uat (https://uat.aitago.tw/) / stg-fanpokka (https://stg-aitago.fanpokka.ai/) / devfm (https://dev.familymart.aitago.tw/) |
| MeterSphere 計畫 | [[AGO-135][UAT] 訊息發送預設顯示文字修正](http://10.9.0.11:8081/test-plan/12715886334976000) |

---

## 測試環境

| 環境名稱 | URL | 測試結果 |
|----------|-----|----------|
| aitago-uat | https://uat.aitago.tw/ | ✅ 全部通過 |
| stg-fanpokka | https://stg-aitago.fanpokka.ai/ | ✅ 全部通過 |
| devfm | https://dev.familymart.aitago.tw/ | ✅ 全部通過 |

---

## 測試結果

### 訊息發送 — 單素材

| 素材類型 | 測試項目 | 結果 |
|----------|----------|------|
| 圖文訊息 | 必填測試 | ✅ Pass |
| 圖文訊息 | 好友列表訊息預覽 | ✅ Pass |
| 多頁訊息 | 必填測試 | ✅ Pass |
| 多頁訊息 | 好友列表訊息預覽 | ✅ Pass |
| 優惠券 | 必填測試 | ✅ Pass |
| 優惠券 | 好友列表訊息預覽 | ✅ Pass |
| 版型訊息 | 必填測試 | ✅ Pass |
| 版型訊息 | 好友列表訊息預覽 | ✅ Pass |
| 圖片 | 不會顯示預設文字欄位 | ✅ Pass |
| 圖片 | 預設文字顯示 LINE 官方預設文字「xxx 傳送了一張圖片」 | ✅ Pass |
| 影片 | 不會顯示預設文字欄位 | ✅ Pass |
| 影片 | 預設文字顯示 LINE 官方預設文字「xxx 傳送了一則影片」 | ✅ Pass |
| 文字 | 不會顯示預設文字欄位 | ✅ Pass |
| 文字 | 預設文字直接顯示該訊息文字內容 | ✅ Pass |
| 超連結圖片 | 不會顯示預設文字欄位 | ✅ Pass |
| 超連結圖片 | 預設文字顯示該超連結圖片的「素材名稱」 | ✅ Pass |

---

### 訊息發送 — 雙素材（正確顯示最後素材對應樣式）

| 素材類型 | 結果 |
|----------|------|
| 圖文訊息 | ✅ Pass |
| 多頁訊息 | ✅ Pass |
| 優惠券 | ✅ Pass |
| 版型訊息 | ✅ Pass |
| 圖片 | ✅ Pass |
| 影片 | ✅ Pass |
| 文字 | ✅ Pass |
| 超連結圖片 | ✅ Pass |

---

### 訊息發送 — 三素材（正確顯示最後素材對應樣式）

| 素材類型 | 結果 |
|----------|------|
| 圖文訊息 | ✅ Pass |
| 多頁訊息 | ✅ Pass |
| 優惠券 | ✅ Pass |
| 版型訊息 | ✅ Pass |
| 圖片 | ✅ Pass |
| 影片 | ✅ Pass |
| 文字 | ✅ Pass |
| 超連結圖片 | ✅ Pass |

---

## 備註

> Feature 環境測試（AGO-142）已確認**圖片**與**影片**素材在手機版與電腦版顯示文字略有不同，確認為 LINE 平台行為，非系統缺陷。UAT 三環境結果一致，行為符合預期。

---

## 總結

| 統計項目 | 數量 |
|----------|------|
| 總測試案例 | 32 |
| ✅ 通過 | 32 |
| ❌ 失敗 | 0 |

> 三個 UAT 環境（aitago-uat、stg-fanpokka、devfm）皆測試通過。UAT 測試沿用 Feature 測試（AGO-142）之 32 個案例，所有素材類型的預設顯示文字行為均符合規格。
