# DB 結構交叉驗證查詢模板

## 概述

使用可用的資料庫 MCP 工具（如 `mcp__specurai`、`mcp__mssql` 或其他已設定的 DB 工具）查詢實際 DB 結構，與規格書逐項比對。具體工具依用戶實際環境而定。

**原則：規格書的每個欄位定義都必須與 DB 實際結構一致。不一致 = CRITICAL。**

---

## 1. 欄位結構比對

### 1.1 取得資料表欄位清單

使用資料庫 MCP 工具取得指定資料表的所有欄位，例如：

```
# 依實際可用的 MCP 工具選擇
mcp__specurai__get_columns(table: "{schema}.{TableName}")
# 或
mcp__mssql__execute-readonly-sql(query: "SELECT COLUMN_NAME, DATA_TYPE, ... FROM INFORMATION_SCHEMA.COLUMNS WHERE TABLE_NAME = '{TableName}'")
```

回傳資訊：欄位名稱、資料型態、長度、是否可 NULL、預設值。

### 1.2 比對項目

| 規格書欄位 | DB 實際欄位 | 比對方式 | 嚴重等級 |
|-----------|-----------|---------|---------|
| ER Diagram 欄位名稱 | COLUMN_NAME | 完全匹配 | CRITICAL |
| 資料型態（varchar/int/decimal...） | DATA_TYPE | 完全匹配 | CRITICAL |
| 長度（varchar(50)） | CHARACTER_MAXIMUM_LENGTH | 數值匹配 | CRITICAL |
| 精度（decimal(10,2)） | NUMERIC_PRECISION + NUMERIC_SCALE | 數值匹配 | CRITICAL |
| 必填標記（O/X） | IS_NULLABLE (YES/NO) | NOT NULL = 必填 | CRITICAL |
| 預設值 | COLUMN_DEFAULT | 值匹配 | WARNING |

### 1.3 輸出格式

```markdown
| # | DB 欄位 | DB 型態 | DB NULL | 規格書欄位 | 規格書型態 | 規格書必填 | 狀態 |
|---|--------|--------|---------|-----------|-----------|-----------|------|
| 1 | PUR_NO | varchar(12) | NO | PurchaseNo | varchar(12) | O | ✅ |
| 2 | AMOUNT | decimal(10,2) | YES | Amount | decimal(10,0) | X | ❌ 精度不符 |
| 3 | REMARK | varchar(100) | YES | -- | -- | -- | ❌ 規格書遺漏 |
```

---

## 2. 外鍵關聯驗證

### 2.1 取得資料表關聯

使用資料庫 MCP 工具取得外鍵關聯，例如：

```
mcp__specurai__get_relations(table: "{schema}.{TableName}")
```

### 2.2 比對項目

| 檢查項目 | 嚴重等級 |
|---------|---------|
| DB 外鍵都在 ER Diagram 有對應關聯線 | CRITICAL |
| ER Diagram 關聯的方向（1:N, N:1）與 DB 一致 | WARNING |
| 規格書 BR 的存在性檢查涵蓋所有外鍵 | CRITICAL |
| 刪除前置檢查涵蓋所有子表外鍵 | CRITICAL |

---

## 3. 索引驗證

### 3.1 取得索引資訊

```
mcp__specurai__get_indexes(table: "{schema}.{TableName}")
# 或使用其他可用的資料庫工具查詢索引
```

### 3.2 比對項目

| 檢查項目 | 嚴重等級 |
|---------|---------|
| 唯一性 BR 規則有對應 UNIQUE INDEX | WARNING |
| 規格書提到的查詢條件欄位有索引 | INFO |

---

## 4. Request/Response JSON 結構驗證

### 4.1 Request 驗證

從規格書提取每個 API 端點的 Request JSON 範例，逐欄位比對：

| 檢查項目 | 比對來源 | 嚴重等級 |
|---------|---------|---------|
| JSON 欄位名稱是 camelCase | 規格書命名規範 | WARNING |
| JSON 欄位對應的 DB 欄位存在 | DB 欄位清單 | CRITICAL |
| JSON 必填欄位對應 DB NOT NULL | DB NULL 設定 | CRITICAL |
| JSON 字串值長度不超過 DB varchar 長度 | DB 長度 | WARNING |
| JSON 數值精度符合 DB decimal 精度 | DB 精度 | CRITICAL |

### 4.2 Response 驗證

| 檢查項目 | 比對來源 | 嚴重等級 |
|---------|---------|---------|
| Response 涵蓋所有 DB 非系統欄位 | DB 欄位清單 | WARNING |
| 巢狀物件的外鍵欄位有對應 Light API | 端點總覽 | WARNING |
| Response JSON 欄位名稱是 camelCase | 規格書命名規範 | WARNING |

---

## 5. 驗證規則與 DB 約束一致性

### 5.1 varchar 長度比對

```
規格書 VR: MaximumByteLength(50) ←→ DB: varchar(50) ✅
規格書 VR: MaximumLength(50) ←→ DB: varchar(50) ❌ 中文欄位應用 MaximumByteLength
規格書 VR: MaximumByteLength(30) ←→ DB: varchar(50) ❌ 長度不一致
```

### 5.2 NOT NULL 比對

```
DB: NOT NULL + 無 DEFAULT → 規格書必須有 VR 必填驗證 (CRITICAL)
DB: NOT NULL + 有 DEFAULT → 規格書可選（系統自動填入）(INFO)
DB: NULLABLE → 規格書不應標記為必填 (WARNING)
```

### 5.3 decimal 精度比對

```
DB: decimal(10,2) → 規格書 JSON 範例應有 2 位小數 (WARNING)
DB: decimal(5,0) → 規格書 JSON 範例應為整數 (WARNING)
```

---

## 6. 資料值驗證（已存在資料表）

**觸發條件：** 規格書涉及的資料表在 DB 已存在且有資料。

**目的：** 確保規格書描述（列舉值、格式、計算公式、業務規則）與 DB 現有資料的實際狀況一致，避免上線後資料不相容。

### 6.1 列舉值/類別值驗證

規格書中出現的下拉選單、狀態碼、類別代碼，必須涵蓋 DB 所有實際值：

```sql
-- 取得欄位所有 DISTINCT 值與筆數
SELECT {col} AS [值], COUNT(*) AS [筆數]
FROM {schema}.{table}
GROUP BY {col}
ORDER BY {col}
```

**比對方式：**
- 規格書列舉 ⊇ DB DISTINCT 值 → ✅
- DB 有值但規格書未列舉 → ❌ CRITICAL（上線後該資料無法正確顯示/操作）
- 規格書有值但 DB 無資料 → ⚠️ WARNING（可能是新增選項，需確認）

### 6.2 編號/格式規則驗證

規格書描述的編號規則、日期格式等，必須與 DB 現有資料格式一致：

```sql
-- 抽樣取得欄位值，觀察格式
SELECT DISTINCT TOP 20 {col}
FROM {schema}.{table}
WHERE {col} IS NOT NULL
ORDER BY {col}
```

**比對方式：**
- 規格書格式能解析所有 DB 現有值 → ✅
- DB 有不符合規格書格式的資料 → ❌ CRITICAL（需在規格書中加入相容處理或資料遷移計畫）

### 6.3 計算公式驗證

規格書描述的計算邏輯（金額=單價×數量、稅額=金額×稅率 等），抽樣比對：

```sql
-- 抽樣驗證計算公式（範例：金額 = 單價 × 數量）
SELECT TOP 10
    {price_col} AS [單價],
    {qty_col} AS [數量],
    {amount_col} AS [金額（DB）],
    {price_col} * {qty_col} AS [金額（計算）],
    CASE WHEN {amount_col} = {price_col} * {qty_col} THEN '✅' ELSE '❌' END AS [狀態]
FROM {schema}.{table}
WHERE {amount_col} IS NOT NULL
```

**比對方式：**
- 抽樣全部符合 → ✅
- 有不符合的資料 → ❌ CRITICAL（公式可能遺漏條件，如折扣、進位規則）

### 6.4 外鍵參照資料驗證

規格書的外鍵參照表必須有對應資料，且 Light API 的顯示欄位能涵蓋實際使用場景：

```sql
-- 驗證參照表有資料
SELECT COUNT(*) AS [筆數] FROM {ref_schema}.{ref_table}

-- 取得參照表實際使用的值（了解顯示欄位）
SELECT TOP 20 * FROM {ref_schema}.{ref_table}

-- 驗證外鍵完整性（是否有孤兒記錄）
SELECT COUNT(*) AS [孤兒筆數]
FROM {schema}.{table} t
LEFT JOIN {ref_schema}.{ref_table} r ON t.{fk_col} = r.{pk_col}
WHERE r.{pk_col} IS NULL AND t.{fk_col} IS NOT NULL
```

**比對方式：**
- 參照表有資料 + 無孤兒記錄 → ✅
- 參照表無資料 → ⚠️ WARNING（可能是新表，需確認）
- 有孤兒記錄 → ❌ CRITICAL（規格書需定義孤兒資料處理策略）

### 6.5 業務規則與現有資料一致性

規格書的 BL/BR 規則，必須與現有資料的實際模式相容：

```sql
-- 範例：驗證「數量必須大於 0」規則是否與現有資料一致
SELECT COUNT(*) AS [違規筆數]
FROM {schema}.{table}
WHERE {qty_col} <= 0

-- 範例：驗證「狀態為已完成時必須有完成日期」
SELECT COUNT(*) AS [違規筆數]
FROM {schema}.{table}
WHERE {status_col} = 'Done' AND {complete_date_col} IS NULL
```

**比對方式：**
- 違規筆數 = 0 → ✅ 規則與現有資料相容
- 違規筆數 > 0 → ❌ CRITICAL（規格書規則與現有資料衝突，需加入資料遷移或例外處理）

### 6.6 輸出格式

```markdown
### 資料值驗證結果

| # | 驗證項目 | 規格書描述 | DB 實際值 | 狀態 |
|---|---------|-----------|----------|------|
| 1 | 訂單類別 (ORDER_TYPE) | A:模具, B:零件 | A, B, C (C=35筆) | ❌ 遺漏類別 C |
| 2 | 編號格式 (PUR_NO) | PUR-YYYYMMDD-NNN | 抽樣 20 筆皆符合 | ✅ |
| 3 | 金額計算 | 單價×數量 | 抽樣 10 筆皆符合 | ✅ |
| 4 | 供應商參照 (SUP_NO) | 參照 SUP_MASTER | 3 筆孤兒記錄 | ❌ 需處理孤兒資料 |
| 5 | 數量 > 0 規則 | BR-003: 數量必須大於 0 | 12 筆違規 | ❌ 需資料遷移計畫 |
```

---

## 7. 自動修復策略

> 包含結構不一致與資料值不一致的修復。

當 DB 驗證發現不一致時，以下項目可自動修復：

| 不一致類型 | 自動修復方式 |
|-----------|------------|
| 規格書欄位型態不符 DB | 以 DB 為準，更新規格書 ER Diagram 和欄位對應表 |
| 規格書遺漏 DB 欄位 | 從 DB 補充到欄位對應表，標記為 `[DB 補充]` |
| varchar 長度不一致 | 以 DB 長度為準，同步更新 VR 規則 |
| NOT NULL 與必填不一致 | 以 DB 為準，更新 Request 必填標記和 VR 規則 |
| decimal 精度不一致 | 以 DB 為準，更新 ER Diagram 和 JSON 範例 |
| 遺漏外鍵關聯 | 從 DB 關聯補充到 ER Diagram 和 BR 規則 |
| Request JSON 範例欄位缺失 | 從欄位對應表補充到 JSON 範例 |
| 規格書列舉值遺漏 DB 實際值 | 從 DB DISTINCT 值補充到規格書列舉清單 |
| 規格書格式規則與 DB 資料不符 | 以 DB 現有格式為準，更新規格書格式描述 |
| 業務規則與現有資料衝突 | 在規格書加入例外處理或資料遷移計畫 |
| 孤兒記錄存在 | 在規格書加入資料清理或容錯處理策略 |

**修復原則：DB 結構與資料是 Single Source of Truth。規格書必須與 DB 一致。**
