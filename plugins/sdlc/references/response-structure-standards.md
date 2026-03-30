# Response 結構化欄位規範

> 適用於撰寫規格書、驗證規格書

## 原則

所有 API Response 中的參照欄位（廠商、幣別、工別、模具、零件等），**必須**使用巢狀 Light Response 結構，不可回傳 flat 字串。

撰寫 BFS 前，**必須**先搜尋訂單模組（SalesOrder/Quotation/Shipping）的 Response 結構作為參考基準。

## 巢狀結構對照表

| 參照類型 | DB 欄位 | Request 屬性 | Response 巢狀結構 | Light Response 類別 |
|---------|---------|-------------|-------------------|-------------------|
| 廠商 | CUST_NO | VendorNo (string) | `vendor: { vendorId, vendorName }` | `GetLightVendorResponse` |
| 幣別 | BIL_NO | CurrencyCode (string) | `currency: { currencyId, currencyName, exchangeRate }` | `GetLightCurrencyResponse` |
| 工別 | MD_NO | ProcessTypeCode (string) | `processType: { id, processTypeCode, processTypeName }` | `GetLightProcessTypeResponse` |
| 模具 | DIE_NO | MoldNo (string) | `mold: { moldId, moldNo, moldName }` | `GetLightMoldResponse` |
| 零件 | SUB_NO | PartNo (string) | `part: { id, partNo, partName }` | `GetLightPartResponse` |
| 訂單 | ORDER_NO | SalesOrderNo (string) | `salesOrder: { salesOrderId, salesOrderNo }` | `GetLightSalesOrderResponse` |

## 撰寫規格書時

1. 識別所有參照欄位（FK 或 code 欄位）
2. 搜尋現有 Light Response 是否已存在：`grep -r "GetLight{Type}Response" {application_layer}/ --include="*.cs"`
3. Response 包含完整的參照物件資訊（不只返回 ID），便於前端顯示；Request 模型使用 flat code/id
4. JSON 範例中呈現巢狀物件

## 驗證規格書時

增加檢查項（CRITICAL）：

| 檢查項目 | 等級 |
|---------|------|
| Response 中參照欄位使用巢狀 Light Response（非 flat 字串） | CRITICAL |
| 巢狀結構與訂單模組一致 | CRITICAL |
| JSON 範例正確呈現巢狀物件 | WARNING |
