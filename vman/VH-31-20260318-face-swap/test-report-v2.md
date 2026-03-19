# VH-31 換臉生成時間縮短測試 — 詳細測試報告

> **環境**：STG（https://line.uat.tatung2025.aitago.tw）  
> **日期**：2026-03-18 | **分支**：tatung（f8adb53f）  
> **MeterSphere**：http://10.9.0.11:8081/test-plan/12964994438144000

---

## 摘要卡片

| 總案例 | 自動通過 | FINDING | 手動待執行 | 效能資料點 |
|--------|---------|---------|-----------|-----------|
| 21 | 8 ✅ | 2 ⚠️ | 11 🙋 | 28 📊 |

> **2026-03-19 補充**：追加 API 重測 14 筆 + 前端 E2E 計時 3 筆；MeterSphere A1/A3/A5/A6/A7/C1 已標記通過。

---

## 效能測試完整資料

### 原始資料（全 14 筆，按耗時排序）

| # | Phase | task_id | user | submit(s) | total(s) | status |
|---|-------|---------|------|-----------|----------|--------|
| 1 | P3-3con | 1890 | wt26031814 | ~0.43 | 31.8 | completed |
| 2 | P3-3con | 1891 | wt26031815 | ~0.43 | 31.8 | completed |
| 3 | P3-3con | 1892 | wt26031816 | ~0.43 | 31.8 | completed |
| 4 | P2-r2 | 1887 | wt26031811 | ~0.42 | 32.7 | completed |
| 5 | P2-r1 | 1884 | wt26031808 | ~0.43 | 36.4 | completed |
| 6 | P2-r1 | 1885 | wt26031809 | ~0.43 | 36.4 | completed |
| 7 | P2-r3 | 1888 | wt26031812 | ~0.43 | 36.1 | completed |
| 8 | P2-r3 | 1889 | wt26031813 | ~0.43 | 36.1 | completed |
| 9 | P1-single | 1882 | wt26031806 | 0.43 | 37.3 | completed |
| 10 | P1-single | 1883 | wt26031807 | 0.43 | 37.9 | completed |
| 11 | P1-single | 1881 | wt26031805 | 0.41 | 40.5 | completed |
| 12 | P1-single | 1880 | wt26031804 | 0.47 | 40.8 | completed |
| 13 | P2-r2 | 1886 | wt26031810 | ~0.42 | 41.4 | completed |
| 14 | P1-single | 1879 | wt26031803 | 0.39 | 41.9 | completed |

### 分層統計

| Phase | n | min | max | avg | P95 |
|-------|---|-----|-----|-----|-----|
| P1 單次 | 5 | 37.3s | 41.9s | 39.7s | 41.9s |
| P2 2並發 | 6 | 32.7s | 41.4s | 36.5s | 41.4s |
| P3 3並發 | 3 | 31.8s | 31.8s | 31.8s | 31.8s |
| **全域** | **14** | **31.8s** | **41.9s** | **36.6s** | **41.9s** |

---

## 功能測試詳細記錄

<details>
<summary>✅ A1 — GET /templates 端點可達性（PASS）</summary>

**執行指令**：
```bash
GET https://line.uat.tatung2025.aitago.tw/api/face-swap/templates
```
**結果**：HTTP 200  
**回應**：17 個模板，id=4（棒球大同寶寶）確認存在  
**耗時**：< 1s

</details>

<details>
<summary>✅ A3 — 回應 Schema 驗證（PASS）</summary>

**執行指令**：
```bash
POST /api/face-swap
body: userId=dev_user_williamsliu_test, template_id=4, file=Ma_Ying-jeou_2015.jpg
```
**結果**：HTTP 200  
**回應結構確認**：
```json
{
  "status": "success",
  "result": { "task_id": 1875 }
}
```

</details>

<details>
<summary>✅ A5 — 空 body → 400（PASS）</summary>

**執行指令**：`POST /api/face-swap`（空 body）  
**結果**：HTTP 400  
**message**：`使用者ID不能為空`

</details>

<details>
<summary>✅ A6 — 缺 template_id → 400（PASS）</summary>

**執行指令**：`POST /api/face-swap`（有 userId + file，無 template_id）  
**結果**：HTTP 400  
**message**：`模板ID不能為空`

</details>

<details>
<summary>✅ A7 — 上傳 .txt → 400（PASS）</summary>

**執行指令**：`POST /api/face-swap`（file 為 .txt 文字檔）  
**結果**：HTTP 400  
**message**：`請上傳有效的圖片文件`

</details>

<details>
<summary>✅ C1 — 3s Timeout 模擬（PASS / INFO）</summary>

**執行指令**：`curl --max-time 3 POST /api/face-swap`  
**結果**：HTTP 200，回應時間 **0.39s**（Submit API 極快，3s timeout 不觸發）  
**結論**：Submit 本身無 timeout 風險，生成時間由 AI Server 非同步處理

</details>

---

## 缺陷詳情

<details>
<summary>🔴 BUG-1（P1）— /status 二次查詢回傳 500</summary>

**受影響端點**：`GET /api/face-swap/status/{taskId}`  
**問題說明**：任務完成後再次查詢（如使用者刷新頁面）回傳 HTTP 500

**重現步驟**：
```bash
# Step 1: 送出換臉
POST /api/face-swap → 取得 task_id

# Step 2: 輪詢直到 completed（第一次會成功）
GET /api/face-swap/status/{task_id}  → status: completed ✅

# Step 3: 再次查詢（第二次 → 500 ❌）
GET /api/face-swap/status/{task_id}  → HTTP 500
```

**實際回應**：
```json
{
  "status": "error",
  "statusCode": 500,
  "message": "檢查任務狀態失敗"
}
```

**根因分析（FaceSwapController.php）**：
```php
$avatar = Cache::lock('face_swap:' . $taskId, 30)->block(15, function () use ($taskId) {
    $avatar = $this->avatarService->findByTaskId($taskId);
    if (!$avatar->isCompleted())
        return $this->avatarService->checkTaskStatus($avatar);
    // ← avatar 已 completed 時，callback 沒有 return
    // → 外層 $avatar = null → $avatar->id 拋出 NullPointerException
});
```

**建議修復**：
```php
// 在 if block 後加入：
return $avatar;
```

**影響評估**：  
- 使用者在結果頁重新整理 → 看到 500 錯誤  
- KIOSK 若有輪詢重試機制 → 每次重試都失敗

</details>

<details>
<summary>🟡 BUG-2（P2）— /status 回應結構不一致</summary>

**受影響端點**：`GET /api/face-swap/status/{taskId}`

**問題說明**：`/status` 的回應被 middleware 包裝，`images` 在 `result` 內層；但 submit 端點的結構不同，可能導致前端取值邏輯不一致

**Submit 端點回應**：
```json
{
  "status": "success",
  "result": { "task_id": 1879 }
}
```

**Status 端點回應**：
```json
{
  "status": "completed",
  "result": {
    "id": 582,
    "status": "completed",
    "images": ["https://storage.googleapis.com/..."],
    "template_id": "4",
    "coupon_code": "XI1632544996942"
  }
}
```

**影響**：前端若用 `response.images` 取圖 URL → undefined；需改用 `response.result.images`

**建議**：統一 API 回應結構文件，或在 Controller 層確保結構一致

</details>

---

## 2026-03-19 補充測試資料

### API 重測（n=14）

| # | Phase | task_id | submit(s) | total(s) | status |
|---|-------|---------|-----------|----------|--------|
| 1 | P1-1 | 1902 | 0.44 | 44.23 | completed |
| 2 | P1-2 | 1903 | 0.88 | 43.27 | completed |
| 3 | P1-3 | 1904 | 0.90 | 43.44 | completed |
| 4 | P1-4 | 1905 | 0.40 | 39.20 | completed |
| 5 | P1-5 | 1906 | 0.40 | 45.26 | completed |
| 6 | P2-r1-A | 1907 | 0.38 | 40.24 | completed |
| 7 | P2-r1-B | 1908 | 0.43 | 39.18 | completed |
| 8 | P2-r2-A | 1909 | 1.02 | 40.76 | completed |
| 9 | P2-r2-B | 1910 | 1.03 | 42.09 | completed |
| 10 | P2-r3-A | 1911 | 0.37 | **49.44** ⚠️ | completed |
| 11 | P2-r3-B | 1912 | 0.40 | 40.48 | completed |
| 12 | P3-1 | 1913 | 0.40 | 41.88 | completed |
| 13 | P3-2 | 1914 | 0.40 | 42.79 | completed |
| 14 | P3-3 | 1915 | 0.40 | 41.17 | completed |

| Phase | n | min | max | avg |
|-------|---|-----|-----|-----|
| P1 單次 | 5 | 39.2s | 45.3s | **43.1s** |
| P2 2並發 | 6 | 39.2s | 49.4s | **42.0s** |
| P3 3並發 | 3 | 41.2s | 42.8s | **41.9s** |

> ⚠️ task 1911 有 49.44s 離群值；P3 並發優勢較 03-18 消失，推測 AI Server 當日負載較高。

---

### 前端 E2E 計時（Playwright，n=3）

> 計時範圍：點擊「開始生成」→ 結果頁「重新生成」按鈕出現  
> 工具：Playwright headless=false，viewport 390×844（行動裝置）

| # | total(s) | api_completed(s) | click→POST(ms) |
|---|----------|-----------------|----------------|
| E2E-1 | 41.44 | 41.38 | 66 |
| E2E-2 | 38.90 | 38.78 | 57 |
| E2E-3 | 42.43 | 41.94 | 57 |
| **avg** | **40.9s** | **40.7s** | **60ms** |

> **與 API 直測差異**：前端瀏覽器輪詢間隔更短（150-400ms），比 Python 腳本（500ms）更快抓到 completed，實測快約 2-3s。  
> 前端 E2E avg **40.9s** 為最接近真實使用者感知的基準數據。

---

## 測試環境資訊

| 項目 | 說明 |
|------|------|
| STG API | https://line.uat.tatung2025.aitago.tw |
| AI Backend | http://60.248.142.147:7588 |
| 模板 | id=4 棒球大同寶寶（正式模板） |
| 圖片壓縮 | 上傳前 max 2048px / JPEG 85%（Service 層） |
| Retry 設定 | HTTP timeout=30s, retry=2, interval=1s |
| 生成次數限制 | 每 userId 最多 4 次（dev_user_* 前綴例外） |

---

## 關鍵結論

1. **Submit 極快（~0.4s）**：API 接收無瓶頸，生成時間完全由 AI Server 決定
2. **生成時間（2026-03-18）**：單次 avg **39.7s**，全域 avg **36.6s**
3. **生成時間（2026-03-19 重測）**：單次 avg **43.1s**，2並發 avg **42.0s**，3並發 avg **41.9s**（整體偏慢，P3 並發優勢消失）
4. **前端 E2E 計時（2026-03-19）**：avg **40.9s**（min 38.9s / max 42.4s）
5. **計時範圍說明**：API 計時 = POST submit 送出 → /status 回傳 completed；E2E 計時 = 點擊「開始生成」→ 結果頁「重新生成」按鈕出現（含最後一次 /status 6s 回應 + avatars 0.2s，完整使用者感知時間）
6. **BUG-1 需優先修復**：二次查詢 500 會直接影響使用者體驗
7. **缺乏優化前基準**：本次測試為 tatung 分支現況，無舊版數據可比對「縮短幅度」
