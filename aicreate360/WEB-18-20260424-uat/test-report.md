# WEB-18 短影音頁面改點驗證 — UAT 測試報告

| 項目 | 內容 |
|------|------|
| 父工單 | [WEB-17 短影音替換影片和文字說明](https://ai360c.atlassian.net/browse/WEB-17) |
| QA 子單 | [WEB-18 UAT測試](https://ai360c.atlassian.net/browse/WEB-18) |
| 發現 Bug | [WEB-19 「社群影音經驗」文案缺句號（P3 Low）](https://ai360c.atlassian.net/browse/WEB-19) |
| 測試人員 | williamsliu |
| 測試日期 | 2026-04-23 ~ 2026-04-24 |
| 測試環境 | UAT — https://uat.aicreate360.com/shortVideo |
| MeterSphere | [WEB-18 UAT測試 — 短影音頁面改點驗證](http://10.9.0.11:8081/#/test-plan/testPlanIndex?id=42904850681372675&orgId=100001&pId=1103411458088960) |
| 測試工具 | chrome-devtools MCP（Chrome）+ 人工判讀 |
| 測試設計技術 | ISTQB 等價分割 / 邊界值 / 決策表 / 狀態轉換 |

---

## 執行摘要

| 項目 | 數值 |
|------|------|
| 總測試案例 | 35 |
| 自動化執行（chrome-devtools） | 34 |
| 人工執行（英文翻譯判讀） | 1 |
| **通過（SUCCESS）** | **34** |
| **失敗（ERROR）** | **1**（A-05 P2 標點，降級 P3 開 Bug 請 PM 確認） |
| **保留（PENDING）** | 0 |
| 發現 Bug | 1 張（WEB-19 P3，已交 PM 確認） |
| 通過率 | **97.1%**（34/35） |

### 需求改點（來自 WEB-17 父單）

| # | 改點 | UAT 現況 | 結果 |
|---|------|----------|------|
| C1 | Hero 版投影片替換為 YouTube Shorts `jaYPFkcD-0w` | iframe src 正確 | ✅ |
| C2 | 後期製作文案更新 | 文案完全吻合含句號 | ✅ |
| C3 | 「電視台製播經驗」→「社群影音經驗」標題 + 新文案 | 標題與文案皆更新，但文案結尾缺句號 | ⚠️ |
| C4 | 移除「電視台」相關字樣 | 全頁 0 次命中 | ✅ |

---

## 測試結果

### A — 改點驗證（5 案例）

| ID | 測試項目 | 結果 | 備註 |
|----|---------|------|------|
| A-01 | Hero 版投影片替換為 jaYPFkcD-0w | ✅ SUCCESS | iframe src 含 `youtube.com/embed/jaYPFkcD-0w` |
| A-02 | 後期製作文案符合改點 C2 | ✅ SUCCESS | 文案完整含結尾句號 |
| A-03 | 「社群影音經驗」標題 + 文案符合改點 C3 | ✅ SUCCESS | 標題與改點一致 |
| A-04 | 全頁不含「電視台」字樣（改點 C4） | ✅ SUCCESS | body.innerText 搜尋 0 筆 |
| A-05 | 改點文案標點符號完整性 | ❌ ERROR | C3 文案「更加豐富有趣」缺句號，開 [WEB-19](https://ai360c.atlassian.net/browse/WEB-19) 請 PM 確認 |

### B — 熱門範例影片（6 案例）

| ID | 測試項目 | 結果 | 備註 |
|----|---------|------|------|
| B-01 | 第一頁 4 支影片 ID 順序 | ✅ SUCCESS | EAifZAh3YUc → 5HFjTS_yZ6A → hE2rfyrO0tI → jaYPFkcD-0w |
| B-02 | 第二頁 4 支影片 ID 順序 | ✅ SUCCESS | m87chZNM1q8 → n2BTKt-FdmI → uo8hinMbbRw → SlvCiizJJiY |
| B-03 | 第三頁 4 支影片 ID 順序 | ✅ SUCCESS | qOgOH4QsyHQ → s6uhcioNadA → i7vkuuMsq_A → BQkYzTkq0Ug |
| B-04 | 12 支縮圖載入成功 | ✅ SUCCESS | 72 張 img（含重複）全 naturalWidth > 0 |
| B-05 | 點擊縮圖開新分頁到 YouTube | ✅ SUCCESS | target=_blank + href 正確 |
| B-06 | 分頁切換動畫流暢性 | ✅ SUCCESS | 切換正常、console 無 error |

### C — Hero 區互動與播放（5 案例）

| ID | 測試項目 | 結果 | 備註 |
|----|---------|------|------|
| C-01 | Hero 影片 autoplay=1 | ✅ SUCCESS | iframe src 參數正確 |
| C-02 | Hero 影片 mute=1 | ✅ SUCCESS | 符合瀏覽器自動播放政策 |
| C-03 | Hero 影片 loop=1 + playlist | ✅ SUCCESS | playlist=jaYPFkcD-0w |
| C-04 | Hero 影片 controls=0 | ✅ SUCCESS | 無播放控制 UI |
| C-05 | Reload ×3 仍正確播放 | ✅ SUCCESS | 3 次 F5 Hero iframe 皆正確掛載 |

### D — CTA 導向（3 案例）

| ID | 測試項目 | 結果 | 備註 |
|----|---------|------|------|
| D-01 | Hero「立即聯繫，由專人服務」 | ✅ SUCCESS | 點擊後導向 `/contact` |
| D-02 | 頁尾「立即聯繫」 | ✅ SUCCESS | 點擊後導向 `/contact` |
| D-03 | 「立即諮詢」 | ✅ SUCCESS | 點擊後導向 `/contact` |

### E — RWD 斷點（6 案例）

| ID | 寬度 | 結果 | scrollWidth | innerWidth | 溢出 | 截圖 |
|----|------|------|-------------|------------|------|------|
| E-01 | 320 | ✅ SUCCESS | 320 | 320 | 0 | [E-01-320.png](screenshots/E-01-320.png) |
| E-02 | 375 | ✅ SUCCESS | 375 | 375 | 0 | [E-02-375.png](screenshots/E-02-375.png) |
| E-03 | 768 | ✅ SUCCESS | 768 | 768 | 0 | [E-03-768.png](screenshots/E-03-768.png) |
| **E-04** | **1024**（WEB-14/15 回歸點） | **✅ SUCCESS** | 1024 | 1024 | **0** | [E-04-1024.png](screenshots/E-04-1024.png) |
| E-05 | 1440 | ✅ SUCCESS | 1440 | 1440 | 0 | [E-05-1440.png](screenshots/E-05-1440.png) |
| E-06 | 1920 | ✅ SUCCESS | 1920 | 1920 | 0 | [E-06-1920.png](screenshots/E-06-1920.png) |

> **回歸確認**：WEB-11 曾在 1024~1283px 出現 Header 溢出（Bug WEB-14、WEB-15 皆已修復）。本次 1024px 機器判定 `diff=0`，**無回歸**。

### F — 英文版（7 案例）

| ID | 測試項目 | 結果 | 備註 |
|----|---------|------|------|
| F-01 | 切換 English 後頁面可開啟 | ✅ SUCCESS | `/en-US/shortVideo` 載入成功 |
| F-02 | 英文版頁面結構與 zh 一致 | ✅ SUCCESS | 13 個 heading，四大區塊齊全 |
| F-03 | 英文版無中文殘留 | ✅ SUCCESS | main 內 0 個中文字符 |
| F-04 | 英文版 Hero 影片仍為 jaYPFkcD-0w | ✅ SUCCESS | iframe src 一致 |
| F-05 | 英文版無 TV station / broadcast 字眼 | ✅ SUCCESS | body.innerText 搜尋 0 筆 |
| F-06 | 英文版 CTA 導向正確 | ✅ SUCCESS | 「Contact Us」→ `/en-US/contact` |
| F-07 | 英文版翻譯合理性（人工判讀） | ✅ SUCCESS | 整體通順；2 處細節已在 Jira comment 提示（未開 Bug） |

**F-07 對照官方翻譯表驗證**（翻譯表：`1ZZ6ZoEYXrHtcmRusLnv5LXEBJFBy6U5avf6X4kItpdA` 短影音分頁，版位 1~5）：

| 版位 | 欄位 | 官方譯 = UAT 實況 | 結果 |
|---|---|---|---|
| 版位 1 | H1 + 副標 | `Short-form Video` + `Content / Video Production / Social Media Management / IP Development` | ✅ |
| 版位 1 | Hero 按鈕 | `Contact Us` | ✅ |
| 版位 4 | Social Video Production Experience 內文 | `A strong grasp of trending content rhythms to create scripts that resonate with social media and captivate audiences.` | ✅ |
| 版位 4 | Fast delivery within 7 days 內文 | `Scriptwriting takes 3 days and post-production takes 3 days. With a clear and efficient workflow, results can be delivered in the shortest possible time.` | ✅ |
| 版位 4 | Precise audience targeting 內文 | `With years of social media management experience and a strong understanding of platform algorithms, we ensure that your videos reach the right target audience.` | ✅ |
| 版位 5 | Our services 小標 | `We provide support at various stages to create high-quality video content.` | ✅ |
| 版位 5 | Creative short videos and scriptwriting 內文 | 完全符合 | ✅ |
| 版位 5 | Post-production 內文（改點 C2） | `Backed by years of social media expertise, our editing goes beyond technique—strategically crafted to capture attention and maximize video completion rates.` | ✅ |
| 版位 5 | Social media management 內文 | 完全符合 | ✅ |
| 版位 5 | IP expansion 內文 | 完全符合 | ✅ |
| 版位 9 | 尾標 + 按鈕 | `Short-form videos that make your creations more valuable.` + `Contact Us` | ✅ |

→ **WEB-18 改點範圍（版位 1~5）英文翻譯 100% 與官方翻譯表一致**，F-07 通過。

**另發現 2 處全站共用文案瑕疵**（非 WEB-18 範圍，不開 Bug，僅口頭告知 PM）：

1. 全站「聯絡我們」CTA 區塊「Arranging consult for free to turn your business difficulties into opportunities.」→ 建議 `Arrange a free consultation...`（動名詞起首不自然）
2. 全站頁尾「To truly prioritize the enterprise, utilize the least amount of budget to achieve optimal results.」→ 動詞 `utilize` 缺主詞

> 註：上述 2 處不在「官方短影音頁翻譯表」中，屬全站共用 CTA / 頁尾文案，不屬於 WEB-17 改點範圍，不影響 WEB-18 判定。

### G — 跨頁面導航（3 案例）

| ID | 測試項目 | 結果 | 備註 |
|----|---------|------|------|
| G-01 | 首頁 Header 選單「短影音」 | ✅ SUCCESS | 導向 `/shortVideo` |
| G-02 | 英文版選單「Short Video」 | ✅ SUCCESS | 導向 `/en-US/shortVideo` |
| G-03 | 短影音 → contact → 瀏覽器返回 | ✅ SUCCESS | history.back 正確回到 `/shortVideo` |

---

## 缺陷列表

### WEB-19（P3 Low — 文案標點）

| 項目 | 內容 |
|------|------|
| Jira | [WEB-19](https://ai360c.atlassian.net/browse/WEB-19) |
| 標題 | [UAT][短影音] 「社群影音經驗」文案結尾缺句號，請 PM 確認是否調整 |
| 嚴重度 | P3 (Low) |
| 指派 | emily320（WEB-17 父單負責人） |
| Sprint | 169 |
| 關聯 | Blocks WEB-17 / Relates WEB-18 |
| 環境 | UAT |
| 對照 | 改點原文：「⋯更加豐富有趣。」／ UAT 現況：「⋯更加豐富有趣」（缺句號） |
| 立場 | 請 PM 確認文案原文句號是否刻意保留或移除後，再決定是否調整 |

---

## 結論

1. **四項改點 C1~C4 全部正確反映於 UAT**（含中英文版本）。
2. **WEB-14 / WEB-15 修復未回歸**：1024px 斷點機器判定 0 溢出，Header 選單正常。
3. **中英文翻譯對照**：WEB-18 改點範圍（版位 1~5）英文翻譯與官方翻譯表 100% 一致，Post-production（C2）、Social Video Production Experience（C3）英譯專業自然；另於全站共用 CTA 與頁尾發現 2 處文法細節（非 WEB-18 範圍），不開 Bug 僅口頭告知 PM。
4. **通過率 97.1%（34/35）**，唯一 ERROR（A-05）為 P2 標點細節已降級並開 WEB-19 請 PM 確認。
5. **無 Blocker 級缺陷**，WEB-18 UAT 測試結案建議「接受」，待 PM 對 WEB-19 做出文案決定後即可收尾。

---

## 相關檔案

| 類型 | 路徑 |
|------|------|
| MS Plan | http://10.9.0.11:8081/#/test-plan/testPlanIndex?id=42904850681372675&orgId=100001&pId=1103411458088960 |
| 測試計畫文件 | `aicreate360/test-plans/WEB-18/`（requirements / strategy / cases / results） |
| RWD 截圖 | `screenshots/E-01-320.png` ~ `E-06-1920.png` |
| 英文版截圖 | `screenshots/F-English.png` |
