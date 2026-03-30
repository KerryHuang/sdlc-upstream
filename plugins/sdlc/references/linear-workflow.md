# Linear 工作流程

## 團隊與識別碼

| 團隊 | 識別碼前綴 | 角色 |
|------|-----------|------|
| `{PM_team}` | PM- | 產品管理、需求規劃 |
| `{QA_team}` | QA- | 測試驗證、Bug 管理 |
| `{Dev_team}` | MP- | 後端/前端開發 |

## 工作流程類型

### 類型一：Feature / 外部 Issue / 外部 Bug

**票層級結構**：
```
PM-XXX (需求文件)
  └── QA-XXX (測試文件)
        ├── MP-XXX (後端規格書/開發)
        └── MP-YYY (前端規格書/開發) ← Blocked by 後端
```

**流程**：
1. **PM** 建立需求 → 產生需求文件 → 建立 QA 子票
2. **QA** 建立測試 → 產生測試文件 → 建立 Dev 子票（後端、前端）
3. **Dev** 後端先行，前端被後端 Blocked

### 類型二：內部 Bug

**票層級結構**：
```
QA-XXX (問題描述/測試)
  ├── MP-XXX (後端問題描述/解決方案)
  └── MP-YYY (前端問題描述/解決方案) ← Blocked by 後端（若有）
```

**流程**：
1. **QA** 建立問題描述 → 建立 Dev 子票
2. **Dev** 後端先行（若有），前端被後端 Blocked

## 狀態流轉

### 規劃階段

所有相關票 → **Backlog**

### 實作階段（Feature / 外部 Issue / 外部 Bug）

| 階段 | 執行者 | 狀態變化 | 交接動作 |
|------|--------|----------|----------|
| 工作開始 | - | Backlog → **Todo** | 所有相關票同步 |
| 後端開發 | Dev | Todo → **In Progress** → **Done** | 前端開始 |
| 前端開發 | Dev | Todo → **In Progress** → **Done** | QA 狀態改 **Testing** |
| 測試驗證 | QA | Testing → **Done** | PM 狀態改 **Testing** |
| PM 確認 | PM | Testing → **Done** | 完成 |

### 實作階段（內部 Bug）

| 階段 | 執行者 | 狀態變化 | 交接動作 |
|------|--------|----------|----------|
| 後端開發 | Dev | Todo → **In Progress** → **Done** | 前端開始（若有） |
| 前端開發 | Dev | Todo → **In Progress** → **Done** | QA 狀態改 **Testing** |
| 測試驗證 | QA | Testing → **Done** | 完成 |

## 狀態對照表

| 狀態 | 類型 | 說明 |
|------|------|------|
| Backlog | backlog | 規劃中 |
| Todo | unstarted | 待開始 |
| In Progress | started | 開發中 |
| Testing | started | 測試中 |
| In Review | started | 審查中 |
| Done | completed | 已完成 |
| Canceled | canceled | 已取消 |

## 交接操作

### 後端完成 → 交接前端

```
# 更新後端票為 Done
Linear MCP update_issue(id: "MP-XXX", state: "Done")

# 留言說明完成內容
Linear MCP create_comment(issueId: "MP-XXX", body: "完成報告...")
```

### 前端完成 → 交接 QA

```
# 更新前端票為 Done
Linear MCP update_issue(id: "MP-YYY", state: "Done")

# 更新 QA 票為 Testing
Linear MCP update_issue(id: "QA-XXX", state: "Testing")
```

### QA 驗證完成 → 交接 PM（Feature / 外部流程）

```
# 更新 QA 票為 Done
Linear MCP update_issue(id: "QA-XXX", state: "Done")

# 更新 PM 票為 Testing
Linear MCP update_issue(id: "PM-XXX", state: "Testing")
```

## Blocked 關係

前端票建立時需設定 Blocked by 後端票：

```
Linear MCP create_issue(
  title: "【前端】...",
  team: "{Dev_team}",
  blockedBy: ["MP-XXX"]  # 後端票 ID
)
```

## Git 分支命名

- Feature: `feature/<ticket-id>` (例: `feature/mp-693`)
- Bug: `bugfix/<ticket-id>`
- Hotfix: `hotfix/<ticket-id>`

## 完成報告範本

```markdown
## 實作完成報告

### Commit
- **Hash**: `xxxxxxxx`
- **訊息**: `feat(scope): 描述`

### 變更檔案
| 檔案 | 說明 |
|------|------|
| file1.cs | 新增 |
| file2.cs | 修改 |

### 修復問題（如有）
- 問題描述
- 解決方案
```

## 注意事項

- 所有留言使用**繁體中文**
- 後端完成前，前端不可開始（Blocked 狀態）
- 交接時主動更新下游票的狀態
- 完成報告需包含可追溯的 commit hash
