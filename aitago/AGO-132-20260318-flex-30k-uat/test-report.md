# AGO-132 測試報告 — [UAT] Flex Message 超過30K錯誤訊息提示

| 項目 | 內容 |
|------|------|
| 工單 | [AGO-132](https://ai360c.atlassian.net/browse/AGO-132) |
| 測試人員 | williamsliu |
| 測試日期 | 2026-03-18 |
| 測試環境 | UAT — aitago-uat (https://uat.aitago.tw/) / stg-fanpokka (https://stg-aitago.fanpokka.ai/) / devfm (https://dev.familymart.aitago.tw/) |
| MeterSphere 計畫 | [[AGO-132][UAT] Flex Message 超過30K錯誤訊息提示](http://10.9.0.11:8081/test-plan/12696232564629504) |

---

## 測試環境

| 環境名稱 | URL | 測試結果 |
|----------|-----|----------|
| aitago-uat | https://uat.aitago.tw/ | ✅ 全部通過 |
| stg-fanpokka | https://stg-aitago.fanpokka.ai/ | ✅ 全部通過 |
| devfm | https://dev.familymart.aitago.tw/ | ✅ 全部通過 |

---

## 測試結果

| # | 測試案例 | 結果 |
|---|----------|------|
| 1 | 卡片訊息 — 發送前預覽 — 超過30K — 阻止發送 | ✅ Pass |
| 2 | 圖文結構 — 超過30K — 顯示錯誤訊息 | ✅ Pass |
| 3 | 圖文結構 — 內容接近30K — 正常儲存 | ✅ Pass |
| 4 | 卡片訊息 — 超過30K後修改回30K以內 — 可正常儲存 | ✅ Pass |
| 5 | 卡片訊息 — 錯誤訊息 — 內容清楚描述問題 | ✅ Pass |
| 6 | 卡片訊息 — 多區塊組合 — 超過30K — 顯示錯誤訊息 | ✅ Pass |
| 7 | 卡片訊息 — 單一文字區塊 — 超過30K — 顯示錯誤訊息 | ✅ Pass |
| 8 | 卡片訊息 — 單一文字區塊 — 正好30K邊界 — 正常儲存 | ✅ Pass |
| 9 | 卡片訊息 — 單一文字區塊 — 30K以內 — 正常儲存 | ✅ Pass |

---

## 總結

| 統計項目 | 數量 |
|----------|------|
| 總測試案例 | 9 |
| ✅ 通過 | 9 |
| ❌ 失敗 | 0 |

> 三個 UAT 環境（aitago-uat、stg-fanpokka、devfm）皆測試通過。Flex Message 超過30K時系統正確顯示錯誤訊息並阻止發送，邊界值行為符合預期。
