# AGO-213 / AGO-197 / AGO-198 測試報告 — 全家專案

> **測試日期**：2026-04-24 ~ 2026-04-27
> **執行者**：williams（QA）/ QA Agent（C17 API 負向測試）
> **環境**：devfm（https://dev.familymart.aitago.tw/ ／ API：https://api.dev.familymart.aitago.tw/）
> **版本**：1.12.4-fm
> **測試依據**：RD 驗收單 AGO-217（對應 AGO-197）／AGO-218（對應 AGO-198）／AGO-219（對應 AGO-213）
> **設計方法**：以 RD 驗收單為主、ISTQB EP/BVA/DT/STT 輔助、使用者視角

---

## 一、總體結果

| 工單 | RD 驗收單 | Case 數 | PASS | FAIL | BLOCKED | 通過率 |
|------|:--------:|:-----:|:----:|:----:|:-------:|:-----:|
| AGO-213 | AGO-219 | 5 | 5 | 0 | 0 | 100% |
| AGO-197 | AGO-217 | 6 | 6 | 0 | 0 | 100% |
| AGO-198 | AGO-218 | 7 | 7 | 0 | 0 | 100% |
| **合計** | — | **18** | **18** | **0** | **0** | **100%** |

**結論**：18 筆驗收項目全數通過，無新發現缺陷，三張單可進入結案流程。

---

## 二、測試帳號

| 角色 | LINE 顯示名稱 | uid | barcode |
|------|:------------:|:---:|:-------:|
| 帳號 A（已綁全家會員） | Daniel | `01kg4jk5c63bpjjxtp6005pd35` | `1340000350` |
| 帳號 B（未綁） | Williams | `01km03ddvy6zmbnk5f6nshnrtd` | `-`（綁定後變有值） |

**綁定限制**：帳號 B 僅能綁定一次（無 unbind endpoint）→ AGO-198-C07 與 AGO-213-C04 合併為一場「綁定大會戰」一次驗完。

---

## 三、AGO-213 時區 workaround 驗證（5 cases，對應 AGO-219）

| Case | 名稱 | Priority | 結果 | MS # |
|------|------|:------:|:----:|:----:|
| AGO-213-C01 | exec_type=1 立即發送 時間欄位儲存 | P0 | ✅ PASS | 100427 |
| AGO-213-C02 | exec_type=2 等待 N 小時 時間欄位儲存 | P0 | ✅ PASS | 100428 |
| AGO-213-C03 | exec_type=3 指定時間儲存與回顯 | P0 | ✅ PASS | 100429 |
| AGO-213-C04 | 排程時間正確性 — 綁定觸發（與 AGO-198-C07 合併） | P0 | ✅ PASS | 100430 |
| AGO-213-C05 | 重複編輯儲存 累積劣化 | P1 | ✅ PASS | 100431 |

**核心驗收（AGO-214 workaround）**：
- C03 指定時間欄位儲存／回顯：DevTools payload 為 `YYYY-MM-DDT15:30:00.000Z`（台北數字+Z），編輯頁仍顯示 15:30，未發生提早 8h 偏移 ✅
- C04 綁定觸發後三支腳本：立即／等待 1hr／指定時間 三種類型 LINE 訊息均按預期時間送達 ✅
- C05 重複進出編輯頁儲存 3 次：start_at 維持 10:00，未累積劣化 ✅

**MS 計畫**：http://10.9.0.11:8081/#/test-plan/testPlanIndexDetail?id=44098439272800261&orgId=100001&pId=1102518104891392

---

## 四、AGO-197 全家會員編號顯示驗收（6 cases，對應 AGO-217）

| Case | 名稱 | Priority | 結果 | MS # |
|------|------|:------:|:----:|:----:|
| AGO-197-C01 | 帳號 A 列表 TableView 顯示 barcode | P0 | ✅ PASS | 100432 |
| AGO-197-C02 | 帳號 B 列表顯示 `-` | P0 | ✅ PASS | 100433 |
| AGO-197-C03 | 帳號 A 詳細 EditView 顯示 barcode 唯讀 | P0 | ✅ PASS | 100434 |
| AGO-197-C04 | 帳號 B 詳細顯示 `-` 唯讀 | P0 | ✅ PASS | 100435 |
| AGO-197-C05 | 訊息中心右側 BasicInfoSection | P1 | ✅ PASS | 100436 |
| AGO-197-C06 | API include barcode 正確 | P1 | ✅ PASS | 100437 |

**核心驗收**：
- 三個顯示位置（會員列表 TableView / 會員詳細 EditView / 訊息中心右側 BasicInfoSection）對帳號 A 一致顯示 `1340000350`、對帳號 B 一致顯示 `-` ✅
- 欄位皆為唯讀 ✅
- API request URL 帶 `include=fm_mart_user`、response `result.fm_mart_user.barcode` 與 UI 顯示一致 ✅

**MS 計畫**：http://10.9.0.11:8081/#/test-plan/testPlanIndexDetail?id=44098473632538626&orgId=100001&pId=1102518104891392

---

## 五、AGO-198 綁定全家會員觸發條件驗收（7 cases，對應 AGO-218）

| Case | 名稱 | Priority | 結果 | MS # |
|------|------|:------:|:----:|:----:|
| AGO-198-C01 | UI 觸發條件 5 選項 | P0 | ✅ PASS | 100438 |
| AGO-198-C03 | 執行時機選項限制 | P0 | ✅ PASS | 100439 |
| AGO-198-C04 | 儲存 payload 驗證 | P0 | ✅ PASS | 100440 |
| AGO-198-C05 | 列表頁顯示此腳本 icon | P1 | ✅ PASS | 100441 |
| AGO-198-C06 | 編輯既有腳本條件正確 | P1 | ✅ PASS | 100442 |
| AGO-198-C07 | 真綁定觸發 + 多腳本共存 + AGO-213-C04 合併 | P0 | ✅ PASS | 100443 |
| AGO-198-C17 | exec_type=4 + fm_mart_user_join 擋下（API 負向） | P0 | ✅ PASS | 100444 |

**核心驗收**：
- C01 / C03：UI 觸發條件卡新增「綁定全家會員」選項、icon=card_membership、執行時機禁選週期性掃描 ✅
- C04：儲存 payload `script_items[0].interactive_item="fm_mart_user_join"`、operate=equal、condition="" ✅
- C07（綁定大會戰）：帳號 B 完成綁定後 LINE 立即收到「立即」訊息、會員列表 barcode 由 `-` 變 `1340000350`、等待腳本 +1hr ±5 分鐘送達、指定時間腳本按 UI 設定時間送達 ✅
- C17（API 層負向）：見下節單獨記錄

**MS 計畫**：http://10.9.0.11:8081/#/test-plan/testPlanIndexDetail?id=44098490812407821&orgId=100001&pId=1102518104891392

---

## 六、AGO-198-C17 API 負向測試詳細證據

### 測試動機

UI 已在 C03 驗證「綁定全家會員 → 週期性掃描選項不可選」，但這只是前端擋。C17 補後端防線：直接呼叫 API 送違規組合，驗後端是否亦擋下。

### 測試設備

- 工具：curl（macOS）
- 認證：使用 devfm 後台登入後從 DevTools Network 取得的 Bearer JWT
- 端點：`POST https://api.dev.familymart.aitago.tw/api/proxy/marketing_scripts`

### 第一次嘗試（v1）— 缺 exec_dt 觸發格式擋

**Payload**：
```json
{
  "name": "AGO-198-C17 negative test - DO NOT ACTIVATE",
  "exec_type": 4,
  "exec_dt": null,
  "scan_interval": 60,
  "is_active": false,
  "script_items": [
    { "interactive_item": "fm_mart_user_join", "operate": "equal", "condition": null }
  ]
}
```

**Response**：HTTP 422
```json
{
  "status": "validation error",
  "message": "執行類型為週期性掃描時，執行時間必須是 HH:MM 格式（代表每日發送時間）",
  "result": {
    "errors": {
      "exec_dt": ["執行類型為週期性掃描時，執行時間必須是 HH:MM 格式..."]
    }
  }
}
```

→ 此次擋下的是「格式錯誤」（缺 exec_dt），尚未觸發目標規則。補上 exec_dt 後重試。

### 第二次嘗試（v2）— 補 exec_dt="10:00" 觸發業務規則擋

**Payload**：相同 payload + `"exec_dt": "10:00"`

**Response**：HTTP 422
```json
{
  "status": "validation error",
  "message": "事件型條件（fm_mart_user_join）不能用於週期性掃描，請使用統計型條件",
  "result": {
    "errors": {
      "script_items.0.interactive_item": [
        "事件型條件（fm_mart_user_join）不能用於週期性掃描，請使用統計型條件"
      ]
    }
  }
}
```

→ **PASS**：後端業務規則擋下，錯誤訊息明確、欄位精準（指向 `script_items.0.interactive_item`）。

### 防禦深度結論

後端對此違規組合具備兩段防禦：
1. **格式擋**：缺 exec_dt → 422
2. **業務規則擋**：補上 exec_dt 後 → 422（事件型 vs 週期性互斥）

兩段擋下後均無腳本被建立（驗證後 GET marketing_scripts 0 筆殘留）。

---

## 七、合併執行設計：AGO-198-C07 + AGO-213-C04

### 為何合併

帳號 B 僅能綁定一次（無 unbind endpoint）。若分開測試只剩一次扣板機機會，分開執行會在第二次無法重綁，因此合併為單一場次：

- **AGO-198-C07** 觀察 LINE 訊息送達 + barcode 顯示變化（事件型觸發行為）
- **AGO-213-C04** 觀察三支腳本（立即／等待 1hr／指定時間）的執行時間是否符合預期（時區 workaround 行為）

### 執行順序

1. 前置：建 3 支觸發條件=綁定全家會員的腳本，分別設定立即／等待 1hr／指定時間 = 1 小時後整點
2. 帳號 B 從全家 dev rich menu 走「綁定手機」流程完成綁定
3. 觀察點：
   - 綁定後數秒內 LINE 收到「AGO-198-立即」訊息 ✅
   - 後台會員列表帳號 B「全家會員編號」由 `-` 變成 `1340000350` ✅
   - 綁定時間 +1 小時 ±5 分鐘內 LINE 收到「AGO-198-等待 1hr」 ✅
   - UI 設定的台北時間 ±5 分鐘內 LINE 收到「AGO-198-指定時間」（不提早 8h、不在綁定當下立即觸發）✅
   - 重開三支腳本編輯頁，原時間欄位回顯一致（不偏移）✅

→ AGO-214 workaround 在事件型觸發場景下**有效**。

---

## 八、相關文件

- 測試計畫：`aitago/docs/test-plan-AGO-197-198-213-familymart.md`
- 測試案例清單：`aitago/docs/test-cases-AGO-213-197-198.md`
- RD 驗收單：[AGO-217](https://ai360c.atlassian.net/browse/AGO-217) / [AGO-218](https://ai360c.atlassian.net/browse/AGO-218) / [AGO-219](https://ai360c.atlassian.net/browse/AGO-219)
- 工單：[AGO-213](https://ai360c.atlassian.net/browse/AGO-213) / [AGO-197](https://ai360c.atlassian.net/browse/AGO-197) / [AGO-198](https://ai360c.atlassian.net/browse/AGO-198)

---

## 九、結案建議

- 18/18 PASS、無 Bug、無 Blocked
- AGO-214 workaround 在前端時區處理 + 後端綁定觸發排程兩個面向皆驗收成功
- 後端對 fm_mart_user_join 與 exec_type=4 的互斥規則具備完整防禦
- 三張單可推進至結案流程
