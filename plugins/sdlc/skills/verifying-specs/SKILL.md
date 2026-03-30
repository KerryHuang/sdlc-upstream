---
name: verifying-specs
description: >-
  以資深開發工程師/系統架構師視角驗證規格書，確認架構可行性、資訊流/資料流正確性、與現有系統一致性。
  使用資料庫 MCP 工具驗證 DB 結構與實際資料真實性。
  支援單文件驗證與跨文件一致性驗證（追溯矩陣）。
  適用於使用者說「驗證規格」、「檢查規格」、「validate spec」、「analyze」。
  適用於規格書已撰寫完畢，需要在實作前進行品質關卡審查。
  適用於檢查 FRD、SAD、BFS、計劃與任務文件之間的一致性。
argument-hint: "[document-path] [--cross]"
---

# Verifying Specs — 架構師級規格書驗證

核心問題：**「這份規格書能不能直接寫 code？寫出來能不能在現有架構和真實資料上跑？」**

> **規則依賴**：`${CLAUDE_SKILL_DIR}/../../references/analysis-standards.md`（MES 領域、資料比對、串接分析標準）

## 模式偵測

| 模式 | 觸發方式 | 說明 |
|------|---------|------|
| **單文件驗證** | `{path}` | 驗證單一 FRD/SAD/BFS 文件品質 |
| **跨文件一致性** | `--cross {功能目錄}` | 驗證 FRD↔SAD↔BFS↔計劃↔任務之間的對齊 |

解析 `$ARGUMENTS`：
- 包含 `--cross` → **跨文件模式**，執行 Task 1 + Task 8（跨文件一致性）
- 不包含 `--cross` → **單文件模式**，執行 Task 1~7（原有流程）

## Task Initialization (MANDATORY)

若 TodoWrite 可用，建立以下任務清單：

**單文件模式：**
1. 識別文件類型與架構定位
2. **資料流驗證** — DB 結構 × 實際資料 × 規格書三方比對
3. **資訊流驗證** — API + 讀寫邏輯 + 驗證規則 + 現有系統整合
4. **架構可行性** — 架構規範合規 + DB 變更影響 + 向後相容
5. 舊系統交叉比對（若適用）
6. MES 領域一致性
7. 生成架構師驗證報告

**跨文件模式：**
1. 識別文件類型與架構定位（定位所有相關文件）
8. **跨文件一致性驗證** — 追溯矩陣 + 文件間對齊檢查
7. 生成架構師驗證報告

發現問題 → 記錄到缺失清單（CRITICAL/WARNING/INFO），繼續下一項。

---

## Task 1: 識別文件類型與架構定位

讀取 `$ARGUMENTS` 指定的文件。**必須讀取對應範本逐章比對，不可憑記憶。**

| 文件特徵 | 類型 | 範本路徑 |
|----------|------|---------|
| `FRD-*` | 功能需求文件 | `${CLAUDE_SKILL_DIR}/../../templates/frd.md` |
| `BFS-*` | 後端功能規格書 | `${CLAUDE_SKILL_DIR}/../../templates/bfs.md` |
| `CR-*` | 功能調整文件 | 對應 CR 範本；**必須**執行 [CR 交叉驗證](references/cr-validation.md) |

修訂歷史提及 "FoxPro" → **必須**執行 Task 5。

---

## Task 2: 資料流驗證（核心）

**DB 是 Single Source of Truth。**

### 2.1 DB 結構逐欄比對

用 MCP 工具查詢每個涉及的資料表（欄位、關聯、索引）：

| 檢查項目 | 等級 |
|---------|------|
| 欄位名稱/型態/長度與 DB 一致 | CRITICAL |
| NOT NULL 與必填標記一致 | CRITICAL |
| decimal 精度一致 | CRITICAL |
| 規格書未遺漏 DB 欄位 | CRITICAL |
| varchar 長度與驗證規則一致 | CRITICAL |
| 外鍵在 ER Diagram 有對應 | CRITICAL |
| 預設值一致 | WARNING |

### 2.2 實際資料真實性驗證

對已存在且有資料的表，**必須**用資料庫 MCP 工具查詢：

| 查詢項目 | 等級 |
|---------|------|
| 列舉欄位 DISTINCT 值涵蓋規格書定義 | CRITICAL |
| 編號格式與規格書一致 | CRITICAL |
| 計算公式抽樣比對 | CRITICAL |
| 外鍵無孤兒記錄 | CRITICAL |
| NULL 分布與必填標記一致 | WARNING |

詳細 SQL 模板 → [references/db-validation-queries.md](references/db-validation-queries.md)

### 2.3 資料流向追蹤

追蹤每個 CRUD 操作：API 入口 → Request→Entity 映射 → DB 操作 → 連動影響 → Response。

### 2.4 表頭-明細一致性（若有主從表）

明細增刪後表頭重算(C)、表頭刪除時明細處理(C)、明細排序邏輯(W)。

---

## Task 3: 資訊流驗證

### 3.1 API 端點架構

| 檢查項目 | 等級 |
|---------|------|
| 路由符合專案 API 路由規範 | CRITICAL |
| 每個端點有 Request/Response 表格 + JSON 範例 | CRITICAL |
| 模型 PascalCase、JSON camelCase | CRITICAL |
| Light API Response 足夠 | WARNING |

### 3.2 讀寫邏輯設計可行性

| 檢查項目 | 等級 |
|---------|------|
| 查詢邏輯與修改邏輯明確分離 | CRITICAL |
| 複雜查詢的過濾/排序/分頁條件已定義 | CRITICAL |
| 操作後續動作（副作用）的觸發時機已說明 | WARNING |
| 複雜查詢是否需特殊優化已評估 | WARNING |

### 3.3 驗證規則完整性

| 檢查項目 | 等級 |
|---------|------|
| 每個必填欄位有驗證規則 | CRITICAL |
| 字串欄位有長度/字節驗證 | CRITICAL |
| 每條業務邏輯有對應業務規則 | CRITICAL |
| 唯一性驗證排除已刪除資料 | CRITICAL |
| 刪除前置檢查涵蓋所有 FK 子表 | CRITICAL |
| 驗證規則有語意明確的錯誤訊息 | WARNING |

### 3.4 現有系統整合

用 Grep/Glob 搜尋，逐表盤點：

| 檢查項目 | 等級 |
|---------|------|
| 同模組已有相似 API（避免重複） | CRITICAL |
| Entity 已存在（需直接使用） | CRITICAL |
| Light API 已存在（規格書引用的參照表） | CRITICAL |
| 路由與現有 Controller 衝突 | CRITICAL |
| 修改的 Entity 被其他模組引用 | CRITICAL |

詳細搜尋指令 → [references/existing-resources-check.md](references/existing-resources-check.md)

---

## Task 4: 架構可行性

| 檢查項目 | 等級 |
|---------|------|
| 遵循專案分層架構（如 Clean Architecture） | CRITICAL |
| 資料存取透過 Repository 介面 | CRITICAL |
| 新增 NOT NULL 欄位有預設值/遷移策略 | CRITICAL |
| 不移除現有 Response 欄位（向後相容） | CRITICAL |
| 不變更現有欄位型別 | CRITICAL |
| 新增欄位為 nullable | CRITICAL |
| 是否需要資料庫 Migration | WARNING |
| 索引規劃合理 | WARNING |

---

## Task 5: 舊系統交叉比對（條件式）

**觸發條件：** 舊系統遷移時**必須**執行。搜尋同目錄或同模組的分析報告。

逐一比對：
- **畫面欄位** → Request/Response 欄位追溯
- **驗證規則** → VR/BR 追溯
- **CRUD 事件** → API 端點追溯
- **實際資料** vs 舊系統 vs 規格書三方比對（用 SQL 查詢）

詳細事件/驗證對應 → [references/checklist-details.md](references/checklist-details.md)

---

## Task 6: MES 領域一致性

**依據 `${CLAUDE_SKILL_DIR}/../../references/analysis-standards.md` MES 領域分析框架。**

- 生命週期定位（設計→開模→試模→量產→維修→報廢）
- 領域特有驗證（工單狀態流轉、工序依賴、工別代碼、成本歸集、外包計價、BOM 結構）
- 跨模組資料流（用 SQL 確認參照表有資料、FK 關聯成立）

---

## Task 8: 跨文件一致性驗證（--cross 模式）

**觸發條件：** `$ARGUMENTS` 包含 `--cross` 時執行。

### 8.1 定位所有相關文件

探索專案文件目錄，搜尋所有相關文件：

| 文件 | 類型 | 狀態 |
|------|------|------|
| 需求文件 (FRD) | [path] | ✅/❌ |
| 系統分析文件 (SAD) | [path] | ✅/❌/N/A |
| 規格書 (BFS) | [path] | ✅/❌ |
| 實作計劃 | [path] | ✅/❌/N/A |
| 任務列表 | [path] | ✅/❌/N/A |

### 8.2 FRD ↔ SAD 一致性

| 檢查項目 | 等級 |
|---------|------|
| FRD 每個使用者故事在 SAD 工作流中有對應 | CRITICAL |
| FRD 欄位規格在 SAD 資料流中有對應 | CRITICAL |
| FRD 業務規則在 SAD 資訊流處理邏輯中有對應 | CRITICAL |
| SAD 標明的 MES 生命週期位置合理 | WARNING |

### 8.3 SAD ↔ BFS 一致性

| 檢查項目 | 等級 |
|---------|------|
| SAD 涉及的資料表在 BFS 中都有定義 | CRITICAL |
| SAD 可重用 API 在 BFS 中有引用或說明 | WARNING |
| SAD 實現策略建議在 BFS 中有體現 | WARNING |
| SAD 風險項目在 BFS 中有因應方案 | WARNING |

### 8.4 BFS ↔ 計劃/任務 一致性

| 檢查項目 | 等級 |
|---------|------|
| BFS 每個 API 端點在計劃/任務中有對應 | CRITICAL |
| 計劃中的每個元件都有對應任務 | CRITICAL |
| 任務相依性符合架構分層（如 Clean Architecture） | CRITICAL |
| 無孤立任務（每個任務可追溯到 BFS） | WARNING |
| 包含測試任務 | WARNING |

### 8.5 追溯矩陣（--cross --full 時產出）

| 需求 ID | FRD 章節 | SAD 分析 | BFS 元件 | 任務 ID | 狀態 |
|--------|---------|---------|---------|--------|------|

---

## Task 7: 生成架構師驗證報告

報告格式 → [references/report-template.md](references/report-template.md)

**必須包含**：架構師評估摘要（一段話結論）、驗證結果表、缺失清單、DB 實際資料驗證明細、現有資源盤點。

| 結論 | 條件 |
|------|------|
| ✅ **可直接開發** | 0 CRITICAL, 0 WARNING |
| ⚠️ **需修正後開發** | 0 CRITICAL, >0 WARNING |
| ❌ **不可開發，需重大修訂** | >0 CRITICAL |

---

## Red Flags

**禁止捷徑**：標題在 ≠ 內容完整、必須用 MCP 查 DB 比對、必須查資料值、必須 Grep 搜尋現有 API/Entity、驗證規則逐一比對、必須追蹤完整資料流路徑。

## References

- [checklist-details.md](references/checklist-details.md) — 結構檢查與舊系統事件對應
- [db-validation-queries.md](references/db-validation-queries.md) — DB 驗證 SQL 模板
- [existing-resources-check.md](references/existing-resources-check.md) — 現有資源盤點指令
- [cr-validation.md](references/cr-validation.md) — CR 交叉驗證規則
- [report-template.md](references/report-template.md) — 報告格式範本
- [pipeline.md](${CLAUDE_SKILL_DIR}/../../references/pipeline.md) — 完整開發流程

---

## 下一步（Pipeline 銜接）

**驗證通過後**，執行：

```
superpowers:writing-plans
```

輸入文件：FRD + SAD + BFS（全部傳入作為實作計畫依據）。

**驗證未通過**：修正規格書後重新執行 `sdlc:verifying-specs`。

完整 pipeline 見 `${CLAUDE_SKILL_DIR}/../../references/pipeline.md`。
- [analysis-standards.md](${CLAUDE_SKILL_DIR}/../../references/analysis-standards.md) — MES 領域分析標準
