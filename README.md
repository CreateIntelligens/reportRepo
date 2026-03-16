# reportRepo — QA 測試報告倉庫

> CreateIntelligens QA Team | 所有專案的測試計畫與測試結果報告集中存放處

---

## 專案列表

| 專案 | 目錄 | 任務來源 | 狀態 |
|------|------|---------|------|
| Fanpokka | [fanpokka/](./fanpokka/) | Jira (FAN) | 進行中 |
| Aitago | [aitago/](./aitago/) | Jira (AGO) | 進行中 |
| Aicreate360 | [aicreate360/](./aicreate360/) | Jira (WEB) | 進行中 |
| Open333 CRM | [open333crm/](./open333crm/) | GitHub Issues | 進行中 |
| OpenVman | [openVman/](./openVman/) | GitHub Issues | 進行中 |

---

## 目錄結構規範

```
reportRepo/
└── {專案名}/
    └── {工單號}-{YYYYMMDD}-{標題簡述}/
        ├── test-plan.md      ← 測試計畫（開始測試前產出）
        └── test-report.md    ← 測試結果報告（測試完成後產出）
```

### 資料夾命名規則
- Jira 專案：`{JIRA-KEY}-{YYYYMMDD}-{title-slug}`
  - 範例：`FAN-93-20260316-login-flow`
- GitHub Issues 專案：`issue-{N}-{YYYYMMDD}-{title-slug}`
  - 範例：`issue-37-20260316-user-auth`

---

## 報告格式

各專案報告產生方式可不同，最終輸出為 `.md` 格式即可。

| 檔案 | 用途 | 產出時機 |
|------|------|---------|
| `test-plan.md` | 測試範圍、測試案例清單、預期結果 | 開始測試前 |
| `test-report.md` | 執行結果、Pass/Fail 統計、Bug 列表、結論 | 測試完成後 |

---

## 新增專案

新開專案時，在此 repo 根目錄新增對應專案資料夾：
```bash
mkdir -p reportRepo/{新專案名}
# 並建立 {新專案名}/README.md
```

> 此步驟已整合至 `init_project.py` 自動執行。
