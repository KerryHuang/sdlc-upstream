# FoxPro 資料表對照表

## 檔案位置

預設路徑：`C:\WDMIS\`

## 業務模組資料表

### Sales（銷售模組）

| FoxPro 表格 | 說明 | 主鍵 | 相關子表 |
|------------|------|------|---------|
| SAL010 | 客戶主檔 | CUST_NO | SAL010E, SAL013E |
| SAL010E | 客戶聯絡人 | CUST_NO + SEQ | - |
| SAL013E | 付款條件 | CUST_NO + ITEM_NO | - |
| SAL020 | 銷售訂單主檔 | SAL_NO | SAL020E |
| SAL020E | 銷售訂單明細 | SAL_NO + SEQ | - |
| SAL030 | 報價單主檔 | QUO_NO | SAL030E |
| SAL030E | 報價單明細 | QUO_NO + SEQ | - |

### Production（生產模組）

| FoxPro 表格 | 說明 | 主鍵 | 相關子表 |
|------------|------|------|---------|
| PCM010 | 模具主檔 | MOLD_NO | PCM010E |
| PCM010E | 模具零件 | MOLD_NO + PART_NO | - |
| PCM020 | 工單主檔 | WO_NO | PCM020E |
| PCM020E | 工單工序 | WO_NO + SEQ | - |
| WIP020 | 工時報工 | - | - |

### Purchasing（採購模組）

| FoxPro 表格 | 說明 | 主鍵 | 相關子表 |
|------------|------|------|---------|
| PUR010 | 供應商主檔 | VEND_NO | PUR010E |
| PUR010E | 供應商聯絡人 | VEND_NO + SEQ | - |
| PUR020 | 採購單主檔 | PO_NO | PUR020E |
| PUR020E | 採購單明細 | PO_NO + SEQ | - |

### Design（設計模組）

| FoxPro 表格 | 說明 | 主鍵 | 相關子表 |
|------------|------|------|---------|
| DES010 | 設計案主檔 | DESIGN_NO | - |
| DES020 | 設計變更 | ECN_NO | - |

### System（系統模組）

| FoxPro 表格 | 說明 | 主鍵 | 用途 |
|------------|------|------|------|
| SYS010 | 系統參數 | PARAM_NO | 系統設定 |
| SYS020 | 代碼主檔 | CODE_TYPE + CODE | 下拉選單 |
| PER010 | 員工主檔 | EMP_NO | 人員資料 |
| DEP010 | 部門主檔 | DEPT_NO | 組織架構 |

## 常見欄位對照

### 共用欄位

| FoxPro 欄位 | 說明 | 型態 | C# 屬性名 |
|------------|------|------|----------|
| DEL_MARK | 刪除標記 | char(1) | IsDeleted (bool) |
| UTIME | 更新時間 | datetime | UpdatedAt |
| UUSER | 更新人員 | char(10) | UpdatedBy |
| CTIME | 建立時間 | datetime | CreatedAt |
| CUSER | 建立人員 | char(10) | CreatedBy |

### 客戶相關

| FoxPro 欄位 | 說明 | 型態 | C# 屬性名 |
|------------|------|------|----------|
| CUST_NO | 客戶編號 | char(15) | CustomerId |
| CUST_NAME | 客戶名稱 | varchar(100) | CustomerName |
| SUBNAME | 簡稱 | char(12) | CustomerSubname |
| CONTACTER | 主聯絡人 | varchar(28) | PrimaryContact |
| TEL1 | 電話1 | varchar(30) | Phone1 |
| TEL2 | 電話2 | varchar(30) | Phone2 |
| FAX | 傳真 | varchar(30) | Fax |
| EMAIL | 電子郵件 | varchar(100) | Email |
| ADDRESS | 地址 | varchar(200) | Address |
| CURRENCY | 交易幣別 | char(3) | CurrencyId |
| TAX_RATE | 稅率 | decimal(5,2) | TaxRate |
| TRADE_TERM | 貿易條件 | char(10) | TradeTermId |
| REMARK | 備註 | text | Remark |

### 聯絡人相關

| FoxPro 欄位 | 說明 | 型態 | C# 屬性名 |
|------------|------|------|----------|
| SEQ | 序號 | char(3) | Sequence |
| CONTACT_NAME | 聯絡人姓名 | varchar(28) | ContactName |
| DEPT | 部門 | varchar(30) | Department |
| TITLE | 職稱 | varchar(30) | Title |
| MOBILE | 手機 | varchar(20) | Mobile |
| IS_MASTER | 是否主要 | char(1) | IsPrimary |
| BIRTHDAY | 生日 | date | Birthday |
| LEAVE_STATUS | 離職狀態 | char(1) | LeaveStatus |

### 付款條件相關

| FoxPro 欄位 | 說明 | 型態 | C# 屬性名 |
|------------|------|------|----------|
| ITEM_NO | 項目序號 | char(2) | ItemNo |
| ITEM_NAME | 項目名稱 | varchar(10) | ItemName |
| RATE | 比例 | decimal(5,2) | Rate |
| DAYS | 天數 | int | Days |

## 索引結構

### 主要索引類型

| 索引類型 | 說明 | 範例 |
|---------|------|------|
| Primary | 主鍵索引 | CUST_NO |
| Unique | 唯一索引 | CUST_NO + DEL_MARK |
| Regular | 一般索引 | SUBNAME |
| Candidate | 候選索引 | EMAIL |

### 常見複合索引

| 表格 | 索引欄位 | 用途 |
|------|---------|------|
| SAL010E | CUST_NO + SEQ | 客戶聯絡人查詢 |
| SAL013E | CUST_NO + ITEM_NO | 付款條件查詢 |
| SAL020E | SAL_NO + SEQ | 訂單明細查詢 |

## 關聯關係

```
SAL010 (客戶主檔)
├── SAL010E (客戶聯絡人) [1:N via CUST_NO]
├── SAL013E (付款條件) [1:N via CUST_NO]
├── SAL020 (銷售訂單) [1:N via CUST_NO] ← 刪除限制
└── SAL030 (報價單) [1:N via CUST_NO]

PCM010 (模具主檔)
├── PCM010E (模具零件) [1:N via MOLD_NO]
└── PCM020 (工單) [1:N via MOLD_NO]

PUR010 (供應商主檔)
├── PUR010E (供應商聯絡人) [1:N via VEND_NO]
└── PUR020 (採購單) [1:N via VEND_NO]
```
