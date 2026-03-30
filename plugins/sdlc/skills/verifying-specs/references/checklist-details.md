# 驗證檢查清單詳細說明

## 功能需求文件（FRD）詳細檢查

### 第 1 章：需求概述

**1.1 業務背景**
- [ ] 描述至少 2-3 段業務背景
- [ ] 說明目前痛點
- [ ] 說明不做的影響

**1.2 功能目標**
- [ ] 至少 3 個具體目標
- [ ] 每個目標有可衡量的成功指標
- [ ] 已排除的目標明確標記（~~刪除線~~）

**1.3 功能範圍**
- [ ] In Scope 列出所有要做的功能
- [ ] Out of Scope 列出明確排除的項目
- [ ] 排除項目有說明原因

**1.4 利害關係人**
- [ ] 列出所有相關角色
- [ ] 每個角色有關注重點和參與程度

### 第 2 章：使用者故事

**2.1 故事清單**
- [ ] 每個故事遵循 "身為...我想要...以便..." 格式
- [ ] 每個故事有優先級（Must/Should/Could/Won't）
- [ ] 已排除的故事明確標記

**2.2 故事詳述**
- [ ] 每個 Must 故事有詳細的 Given-When-Then
- [ ] 每個 Must 故事有例外情境處理
- [ ] Given 描述具體的前置條件
- [ ] When 描述明確的使用者操作
- [ ] Then 描述可驗證的預期結果

### 第 3 章：業務流程

- [ ] 至少一個主流程的 Mermaid 流程圖
- [ ] 流程圖包含決策點（菱形節點）
- [ ] 流程圖區分使用者操作和系統處理
- [ ] 若有狀態變化，有狀態圖
- [ ] 若有多系統互動，有時序圖

### 第 4 章：畫面設計

**4.1 畫面總覽**
- [ ] 列出所有畫面及進入方式
- [ ] 每個畫面有用途說明

**4.2 畫面詳細規格**
每個畫面必須包含：
- [ ] ASCII 示意圖或 UI 設計連結
- [ ] 查詢條件欄位表（欄位名稱、元件類型、必填、預設值、說明）
- [ ] 清單欄位表（欄位名稱、寬度、對齊、格式、說明）
- [ ] 操作按鈕表（按鈕、觸發條件、執行動作、備註）

### 第 5 章：欄位規格與驗證規則

**5.1 資料欄位定義**
- [ ] 每個欄位有編號（F-xx）
- [ ] 每個欄位有資料型態和長度
- [ ] 每個欄位標記是否必填
- [ ] 有預設值的欄位有標記

**5.2 輸入驗證規則**
- [ ] 每個必填欄位有對應 VR 規則
- [ ] 每個有長度限制的欄位有 VR 規則
- [ ] 每個有格式要求的欄位有 VR 規則
- [ ] 每條規則有明確的繁體中文錯誤訊息

**5.3 業務規則**
- [ ] 每條規則有編號（BR-xx）
- [ ] 每條規則有觸發時機
- [ ] 每條規則有處理方式（阻擋/警示）
- [ ] 複雜規則有詳細判斷邏輯說明

### 第 8 章：驗收標準

- [ ] 每個 Must 的 US 至少有一個測試案例
- [ ] 每個測試案例有前置條件
- [ ] 每個測試案例有明確的測試步驟
- [ ] 每個測試案例有可驗證的預期結果
- [ ] 有例外處理的測試案例

---

## 後端功能規格書（BFS）詳細檢查

### 第 2 章：業務邏輯

**2.1 核心業務規則**
- [ ] 每條規則有唯一編號（BL-xxx）
- [ ] 每條規則有明確的觸發時機
- [ ] 每條規則有優先級

**2.2 業務規則詳細說明**
- [ ] 高優先級規則有詳細說明表格
- [ ] 說明包含：規則目的、判斷條件、執行動作、例外處理、錯誤訊息
- [ ] 複雜規則有虛擬碼

**2.3 計算邏輯**
- [ ] 每個計算項目有明確公式
- [ ] 公式中的欄位名稱與資料模型一致
- [ ] 有加總/匯總的計算有 WHERE 條件

### 第 5 章：資料模型

**5.1 ER Diagram**
- [ ] 包含所有主要資料表
- [ ] 標示主鍵（PK）和外鍵（FK）
- [ ] 標示資料關聯類型（1:1, 1:N, N:1）
- [ ] 每個欄位有資料型態和長度

**5.4 欄位對應表**
- [ ] DB 欄位欄：每個 ER Diagram 中的欄位都出現
- [ ] Entity 屬性欄：使用語義化 PascalCase
- [ ] Request 屬性欄：標記自動產生/自動計算的欄位
- [ ] Response 屬性欄：關聯實體使用巢狀物件（如 vendor.vendorId）
- [ ] varchar 中文欄位有位元組長度備註

### 第 6 章：API 介面規格

**6.1 端點總覽**
- [ ] 列出所有端點（HTTP 方法 + 路由）
- [ ] 每個端點有權限說明

**每個端點（6.2+）必須包含：**
- [ ] HTTP 方法和完整路由
- [ ] Request 模型定義表格（屬性名稱 PascalCase、型態、必填、長度限制、說明、驗證規則）
- [ ] Request JSON 範例（camelCase，擬真資料）
- [ ] Response 模型定義表格
- [ ] Response JSON 範例（camelCase，擬真資料）
- [ ] 錯誤回應說明

**命名一致性：**
- [ ] 模型表格使用 PascalCase
- [ ] JSON 範例使用 camelCase
- [ ] 巢狀物件遵循 Light Response 命名慣例

### 第 7 章：驗證規則

**7.1 輸入驗證**
- [ ] 每個 Request 中 `必填=O` 的欄位有 VR 規則
- [ ] 每個有長度限制的欄位有 VR 規則
- [ ] 中文 varchar 欄位使用 MaximumByteLength（非 MaximumLength）
- [ ] 每條 VR 有繁體中文錯誤訊息

**7.2 業務驗證**
- [ ] 每條 BL 規則有對應 BR 驗證
- [ ] 唯一性檢查有明確的查詢條件
- [ ] 關聯檢查指明關聯表和欄位
- [ ] 刪除前置檢查涵蓋所有子表

**7.4 FoxPro 來源驗證（若適用）**
- [ ] 每條 FP 規則有來源 APP 檔名
- [ ] 每條規則有驗證狀態（✅/⚠️/❌）
- [ ] 未找到的規則有說明

### 第 10 章：測試案例

- [ ] 每個 Command Handler 有正常案例
- [ ] 每個 Command Handler 有異常案例（驗證失敗、資料不存在、業務規則違反）
- [ ] 每個 Query Handler 有至少一個案例
- [ ] 命名遵循 Method_Scenario_ExpectedResult 格式

---

## FoxPro 交叉比對詳細檢查

### 畫面元素比對

FoxPro 畫面可能包含的元素類型：
1. **TextBox** — 對應 Request/Response 欄位
2. **ComboBox/ListBox** — 對應下拉選單（通常需要 Light API）
3. **CheckBox** — 對應布林欄位
4. **Grid** — 對應明細（表頭-明細關係）
5. **CommandButton** — 對應 API 端點或前端操作
6. **Label** — 顯示用途，通常不需 API
7. **EditBox** — 對應多行文字欄位

### 事件類型對應

| FoxPro 事件 | 典型用途 | 新系統對應 |
|------------|----------|-----------|
| `form.Init` | 載入初始資料 | GET API（查詢） |
| `form.Load` | 設定預設值 | 前端初始化 |
| `form.Destroy` | 清理資源 | 前端 cleanup |
| `cmdAdd.Click` | 新增記錄 | POST API |
| `cmdEdit.Click` | 進入編輯模式 | PUT API |
| `cmdSave.Click` | 儲存資料 | POST/PUT API |
| `cmdDelete.Click` | 刪除記錄 | DELETE API |
| `cmdPrint.Click` | 列印 | 列印 API（若有） |
| `cmdFind.Click` | 查詢 | GET Paged API |
| `cmdExit.Click` | 關閉表單 | 前端路由 |
| `txtXxx.Valid` | 欄位離開驗證 | FluentValidation |
| `txtXxx.InteractiveChange` | 即時輸入反應 | 前端邏輯 |
| `cboXxx.InteractiveChange` | 選項變更觸發 | 前端聯動或 API 查詢 |
| `grdXxx.AfterRowColChange` | Grid 行列切換 | 前端事件 |
| `Timer.Timer` | 定時事件 | 排程/WebSocket |

### 驗證模式對應

| FoxPro 驗證模式 | 程式碼特徵 | 新系統對應 |
|----------------|-----------|-----------|
| 必填檢查 | `IF EMPTY(field)` | FluentValidation `.NotEmpty()` + VR 規則 |
| 唯一性檢查 | `SEEK value IN table` | Handler BR 驗證 + Repository.IsUniqueAsync() |
| 存在性檢查 | `SQLEXEC(conn, "SELECT...")` | Handler BR 驗證 + Repository.ExistsAsync() |
| 範圍檢查 | `IF value < min OR value > max` | FluentValidation `.InclusiveBetween()` + VR 規則 |
| 格式檢查 | `IF !LIKE(pattern, value)` | FluentValidation `.Matches()` + VR 規則 |
| 長度檢查 | `IF LEN(TRIM(value)) > max` | FluentValidation `.MaximumLength()` 或 `.MaximumByteLength()` |
| 關聯檢查 | `SEEK fk IN master` | Handler BR 驗證 |
| 計算驗證 | `field = expr1 * expr2` | Handler 計算邏輯 + BL 規則 |
| 條件必填 | `IF condition AND EMPTY(field)` | FluentValidation `.When()` + VR 規則 |
| 提示訊息 | `MESSAGEBOX("text", icon)` | VR/BR 錯誤訊息（繁體中文） |
