# 分析標準規範

> 適用於舊系統分析、系統分析、規格書驗證等分析類技能
> 本規範用於**規格產出階段**的資料驗證。

## MES 領域分析框架

### 模具生命週期階段

| 階段 | 典型功能 | 關鍵資料表 |
|------|---------|-----------|
| 設計 | 設計案管理、BOM 建立 | DES010, DES020 |
| 開模 | 工單開立、派工排程 | PCM020, PCM020E |
| 試模 | 試模報告、修模記錄 | 試模相關表 |
| 量產 | 報工、進度追蹤 | WIP020, PCM020E |
| 維修 | 維修工單、零件更換 | 維修相關表 |
| 報廢 | 報廢申請、資產沖銷 | 報廢相關表 |

分析任何功能時，**必須**標明其在生命週期中的位置與上下游關係。

### 工單與工序分析項目

| 分析項目 | 說明 | 嚴重等級 |
|---------|------|---------|
| 工單類型 | 內製/外包/混合 | CRITICAL |
| 工序順序 | 前後工序依賴關係 | CRITICAL |
| 工別代碼 | 車、銑、磨、放電、線割等 | CRITICAL |
| 標準工時 | 工時設定與實際比對 | WARNING |
| 報工方式 | 即時/批次/掃碼 | WARNING |
| 進度計算 | 完成率公式 | CRITICAL |

### 成本與計價分析項目

| 分析項目 | 說明 | 嚴重等級 |
|---------|------|---------|
| 成本類型 | 材料/人工/外包/間接 | CRITICAL |
| 計價方式 | 依工時/件數/重量/固定 | CRITICAL |
| 幣別處理 | 多幣別、匯率轉換 | WARNING |
| 稅務處理 | 含稅/未稅、稅率 | WARNING |

---

## 實際資料比對標準

### 必須查詢的資料項目

對每個涉及的資料表，使用 MCP 工具執行以下查詢：

| 查詢項目 | SQL 模板 | 嚴重等級 |
|---------|---------|---------|
| 列舉值 | `SELECT DISTINCT {col}, COUNT(*) FROM {table} GROUP BY {col}` | CRITICAL |
| 編號格式 | `SELECT TOP 10 {id_col} FROM {table} ORDER BY {id_col} DESC` | CRITICAL |
| 日期範圍 | `SELECT MIN({date}), MAX({date}) FROM {table}` | WARNING |
| 數值範圍 | `SELECT MIN({amt}), MAX({amt}), AVG({amt}) FROM {table}` | WARNING |
| 資料筆數 | `SELECT COUNT(*) FROM {table}` | INFO |
| NULL 分布 | `SELECT COUNT(*) - COUNT({col}) as null_cnt FROM {table}` | WARNING |
| 外鍵完整性 | `SELECT a.{fk} FROM {child} a LEFT JOIN {parent} b ON a.{fk}=b.{pk} WHERE b.{pk} IS NULL` | CRITICAL |

### 比對結果判定

| 狀態 | 條件 | 處理方式 |
|------|------|---------|
| ✅ 一致 | 規格書/FoxPro 定義與 DB 資料吻合 | 無需處理 |
| ⚠️ DB 有額外值 | DB 存在規格書未列舉的值 | 需確認含義並補充規格 |
| ⚠️ 精度差異 | 小數位數或長度不一致 | 以 DB 為準修正 |
| ❌ 不一致 | 型態/格式根本不同 | CRITICAL 缺失 |

---

## 使用者操作流程記錄標準

### CRUD 操作流程必記項目

每個 CRUD 操作必須記錄：

| 記錄項目 | 說明 |
|---------|------|
| 觸發方式 | 按鈕/快捷鍵/選單 |
| 前置條件 | 必須先選取記錄？必須特定狀態？ |
| 系統行為 | 表單狀態變化、欄位鎖定/解鎖 |
| 驗證時機 | 即時驗證（離開欄位）vs 提交驗證（點儲存） |
| 資料操作 | INSERT/UPDATE/DELETE 語句 |
| 交易管理 | BEGIN TRAN...COMMIT/ROLLBACK |
| 成功回應 | 訊息、畫面切換 |
| 失敗回應 | 錯誤訊息、保留狀態 |
| 連動操作 | 觸發其他表更新、通知、計算 |

### 事件類型對應

| FoxPro 事件 | 典型用途 | Web 版對應 |
|------------|----------|-----------|
| `form.Init` | 載入初始資料 | GET API（查詢） |
| `form.Load` | 設定預設值 | 前端初始化 |
| `cmdAdd.Click` | 新增記錄 | POST API |
| `cmdEdit.Click` | 進入編輯模式 | PUT API |
| `cmdSave.Click` | 儲存資料 | POST/PUT API |
| `cmdDelete.Click` | 刪除記錄 | DELETE API |
| `cmdPrint.Click` | 列印 | 列印 API |
| `cmdFind.Click` | 查詢 | GET Paged API |
| `txtXxx.Valid` | 欄位離開驗證 | FluentValidation |
| `txtXxx.InteractiveChange` | 即時輸入反應 | 前端邏輯 |
| `cboXxx.InteractiveChange` | 選項變更觸發 | 前端聯動或 API |
| `grdXxx.AfterRowColChange` | Grid 行列切換 | 前端事件 |

---

## 現有功能串接分析標準

### 串接類型

| 類型 | 說明 | 分析方式 |
|------|------|---------|
| API 重用 | 新功能可直接呼叫現有 API | 搜尋 Controller + Light API |
| Entity 共用 | 新功能使用已存在的 Entity | 搜尋 Domain/Entities/ |
| 資料依賴 | 新功能需讀取現有表資料 | 分析 FK 關聯 |
| 資料寫入 | 新功能會修改現有表資料 | CRITICAL - 影響分析 |
| 業務觸發 | 新功能操作觸發現有業務規則 | 搜尋相關 Handler |

### 影響評估矩陣

| 影響層級 | 條件 | 處理要求 |
|---------|------|---------|
| 無影響 | 完全獨立，無共用資料表 | 直接開發 |
| 低影響 | 僅讀取參照表 | 確認 Light API 存在 |
| 中影響 | 共用 Entity，僅新增欄位 | DB 加欄位 + 重新產生 Entity |
| 高影響 | 修改現有 Entity 欄位定義 | 跨模組影響分析，需逐一確認引用 |
| 極高影響 | 變更業務規則或刪除欄位 | 需所有相關模組同步調整 |

### 搜尋指令

> **注意**：以下路徑佔位符需依專案實際目錄結構調整。

```bash
# 現有 API 搜尋
grep -r "class {Entity}Controller" {presentation_layer}/ --include="*.cs"
grep -r "Get{Entity}Light" {application_layer}/ --include="*.cs"

# Entity 引用搜尋
grep -r "{Entity}" {application_layer}/ --include="*.cs" -l

# 外鍵依賴搜尋
grep -r "{EntityId}" {domain_entities}/ --include="*.cs" -l

# 同模組 Handler 搜尋
find {application_layer}/{Module}/ -name "*Handler.cs"
```

---

## 全新功能分析清單

全新功能（非 FoxPro 遷移）**必須**額外分析：

| # | 分析項目 | 說明 | 嚴重等級 |
|---|---------|------|---------|
| 1 | 是否與現有功能重疊 | 搜尋同模組現有 CRUD API | CRITICAL |
| 2 | 是否能重用現有 Entity | 搜尋 Entities/ 目錄 | CRITICAL |
| 3 | 是否需要新資料表 | 確認 DB 是否已有對應表 | CRITICAL |
| 4 | 參照表 Light API 是否已存在 | 搜尋現有 Light Response | WARNING |
| 5 | 是否影響現有資料 | 新增欄位/FK 對現有表的影響 | CRITICAL |
| 6 | 是否需要資料遷移 | 現有資料是否需要轉換/補值 | CRITICAL |
| 7 | 跨模組資料流 | 與其他模組的資料交換關係 | WARNING |
| 8 | MES 生命週期定位 | 在模具生命週期中的位置 | INFO |
| 9 | 工單/工序整合 | 是否與工單排程、報工相關 | WARNING |
| 10 | 成本影響 | 是否影響成本計算或計價 | WARNING |
