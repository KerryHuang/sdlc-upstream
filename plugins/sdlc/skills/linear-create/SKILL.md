---
name: linear-create
description: >-
  透過自然語言描述在 Linear 建立完整票層級結構（Feature: PM→QA→Dev / Bug: QA→Dev）。
  適用於需要開票、建立 Linear ticket、或建立功能/Bug 票層級結構。
  Use when user says "開票", "建立 Linear ticket", "create ticket", "我要開一張票", "幫我建 Linear issue", "建立工作票".
argument-hint: "{自然語言描述}"
---

# Linear 開票

透過自然語言描述自動判斷票類型，互動式引導收集資訊，建立完整票層級結構。

- **票預設值與描述範本**：見 `${CLAUDE_SKILL_DIR}/../../references/linear-ticket-defaults.md`
- **工作流程規範**：見 `${CLAUDE_SKILL_DIR}/../../references/linear-workflow.md`

## 輸入參數

```
$ARGUMENTS → {自然語言描述}
```

## 工作流程

### 1. 驗證輸入

- 若 `$ARGUMENTS` 為空，提示使用者輸入描述並終止
- 偵測 Linear MCP 是否可用，若找不到則提示設定 `.mcp.json` 並終止

### 2. AI 判斷類型 + 確認

分析 `$ARGUMENTS` 判斷 Feature 或 Bug，使用 `AskUserQuestion` 確認類型。

### 3. 詢問優先級

> Urgent（1）/ High（2）/ **Normal（3）預設** / Low（4）

### 4. 詢問 Labels

透過 Linear MCP `list_issue_labels` 取得列表，讓使用者選擇（可多選，可跳過）。

### 5. 詢問開發範圍

> 僅後端 / 僅前端 / 前端 + 後端

### 6. 生成各票描述草稿

#### 內容來源

1. **搜尋已完成的文件** — 在專案中搜尋對應的需求文件與規格書：
   - 需求文件（FRD / CR-FRD）
   - 系統分析文件（SAD）
   - 後端規格書（BFS / CR-BFS）
   - 前端規格書（FFS / CR-FFS）
2. 若找到文件，在對應票描述中**附上文件路徑**
3. 若無已完成文件，按 `${CLAUDE_SKILL_DIR}/../../references/linear-ticket-defaults.md` 範本從自然語言生成

#### 各票描述

| 票種 | 描述來源 | 範本參考 | 須附文件路徑 |
|------|----------|----------|:---:|
| PM 票（Feature） | FRD 需求文件 | §PM 票描述範本 | FRD |
| QA 票 | 表格式 CheckList（□ 通過標示） | §QA 票描述範本 | FRD |
| Dev 後端票 | BFS 後端規格書 | §Dev 後端票描述範本 | BFS、SAD |
| Dev 前端票 | **完整 SPEC**（UI/API/驗證/流程/驗收） | §Dev 前端票描述範本 | FFS、BFS |

**文件路徑格式**（附在票描述最上方）：

```markdown
## 參考文件

| 文件 | 路徑 |
|------|------|
| 需求文件 | `{FRD 路徑}` |
| 後端規格書 | `{BFS 路徑}` |
```

使用 `AskUserQuestion` 呈現草稿摘要，確認或修改後繼續。

### 7. 建票

依 `${CLAUDE_SKILL_DIR}/../../references/linear-ticket-defaults.md` 的 Assignee 與狀態預設值，依類型順序建立每張票，帶入 description + assignee + state + links。

> **重要**：使用狀態 ID 指定狀態，避免名稱匹配歧義。詳見 `references/linear-ticket-defaults.md`。

#### 資源連結（links）

若步驟 6 搜尋到文件，且 `{docs_repo_url}` 已設定（非佔位符），則在 `save_issue` 時帶入 `links` 參數，將對應文件掛為 URL 資源連結。

**連結 URL 組合方式**：`{docs_repo_url}/{URL encoded 文件相對路徑}`

> 文件相對路徑 = docs 目錄下的路徑（如 `生產模組/設變處理看板/設變處理看板_需求文件.md`），需 URL encode。

**各票掛連結規則**：

| 票種 | 掛的文件連結 |
|------|------------|
| PM 票（Feature） | FRD |
| QA 票 | FRD |
| Dev 後端票 | BFS |
| Dev 前端票 | FFS |

**範例**（`save_issue` 的 `links` 參數）：

```json
[{"url": "{docs_repo_url}/{encoded_path}", "title": "FRD 需求文件"}]
```

若 `{docs_repo_url}` 未設定，跳過 `links`，僅在票描述中附文件路徑（現有行為）。

**Feature 流程（PM → QA → Dev）：**

| 順序 | 標題格式 | 說明 |
|------|---------|------|
| 1 | `{標題}` | PM 票 |
| 2 | `【測試】{標題}` | QA 票 |
| 3 | `【後端】{標題}` | Dev 後端票 |
| 4* | `【前端】{標題}` | Dev 前端票（若需前端，blockedBy = 後端票） |

**Bug 流程（QA → Dev）：**

| 順序 | 標題格式 | 說明 |
|------|---------|------|
| 1 | `{標題}` | QA 票 |
| 2 | `【後端】{標題}` | Dev 後端票 |
| 3* | `【前端】{標題}` | Dev 前端票（若有） |

**父子關係**：Feature: PM → QA → Dev / Bug: QA → Dev

### 8. 輸出結果

```markdown
## 開票完成

### 票層級結構
PM-123 {標題}
  └── QA-456 【測試】{標題}
        ├── MP-789 【後端】{標題}
        └── MP-790 【前端】{標題} ← Blocked by MP-789

### 票資訊
| 票 | 團隊 | Assignee | 狀態 | 優先級 | Labels |
|----|------|----------|------|--------|--------|
| PM-123 | PM | {開票人} | Backlog | Normal | feature |
| QA-456 | QA | {assignee} | Revisable | Normal | feature |
| MP-789 | Dev | {assignee} | Revisable | Normal | feature |
| MP-790 | Dev | {assignee} | Revisable | Normal | feature |

### 下一步
- 使用對應的開發啟動工作流開始後端開發
```

## 錯誤處理

| 錯誤情境 | 處理方式 |
|---------|---------|
| 無 `$ARGUMENTS` | 提示輸入描述，終止 |
| Linear MCP 未設定 | 提示設定 `.mcp.json`，終止 |
| 建票失敗 | 顯示錯誤，列出已成功票 ID |
| Labels 取得失敗 | 跳過 Labels 步驟 |

## 使用範例

```bash
# 新功能
linear-create 新增模具報價功能，需要前後端支援

# Bug
linear-create 訂單查詢頁面篩選條件無效

# 功能增強
linear-create 客戶主檔新增匯出 Excel 功能
```

## 完整開發流程

詳見 `${CLAUDE_SKILL_DIR}/../../references/upstream-workflow.md`。
