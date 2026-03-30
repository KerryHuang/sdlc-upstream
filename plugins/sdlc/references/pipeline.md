# Development Pipeline

完整軟體開發生命週期流程，整合 `sdlc` plugin（文件產出）與 `superpowers` plugin（流程驅動）。

## 標準流程（新功能）

```
┌─────────────────────────────────────────────────────────┐
│                   UPSTREAM PHASE (sdlc)                 │
│                                                         │
│  [ superpowers:brainstorming ]                          │
│       需求探索、設計確認                                  │
│       輸出：docs/specs/*-design.md                      │
│              ↓                                          │
│  [ sdlc:requirement ]                                   │
│       PM 角色，撰寫功能需求文件                           │
│       輸出：*_需求文件.md (FRD)                          │
│              ↓                                          │
│  [ sdlc:system-analysis ]                               │
│       SA 角色，分析工作流/資訊流/資料流                   │
│       輸出：*_系統分析文件.md (SAD)                      │
│              ↓                                          │
│  [ sdlc:specify ]                                       │
│       建立後端 API 規格書                                │
│       輸出：*後端功能規格書.md (BFS)                     │
│              ↓                                          │
│  [ sdlc:verifying-specs ]                               │
│       架構師視角驗證規格書                               │
│       輸出：驗證報告                                     │
│              ↓                                          │
│  [ sdlc:linear-create ]                                 │
│       建立 Linear ticket 層級結構                        │
│       輸出：Linear tickets                              │
└─────────────────────────────────────────────────────────┘
               ↓
┌─────────────────────────────────────────────────────────┐
│              IMPLEMENTATION PHASE (superpowers)         │
│                                                         │
│  [ superpowers:writing-plans ]                          │
│       輸入：FRD + SAD + BFS                             │
│       輸出：實作計畫文件                                 │
│              ↓                                          │
│  [ superpowers:subagent-driven-development ]            │
│       平行實作，每個 task 獨立 subagent                  │
│              ↓                                          │
│  [ superpowers:verification-before-completion ]         │
│       實作完成前驗證                                     │
│              ↓                                          │
│  [ superpowers:requesting-code-review ]                 │
│       代碼審查                                          │
│              ↓                                          │
│  [ superpowers:finishing-a-development-branch ]         │
│       收尾，merge/PR                                    │
└─────────────────────────────────────────────────────────┘
```

## CR 流程（既有功能調整）

> CR 流程需求已明確，跳過 brainstorming，直接從 requirement 開始。

```
[ sdlc:requirement --change ]     輸出：CR-FRD
         ↓
[ sdlc:specify --change ]         輸出：CR-BFS + CR-FFS
         ↓
[ sdlc:verifying-specs ]          輸出：驗證報告
         ↓
[ sdlc:linear-create ]            輸出：Linear tickets
         ↓
（同標準流程 → superpowers:writing-plans 之後）
```

## 各 Skill 銜接點

| Skill | 前置 | 後續 |
|-------|------|------|
| `sdlc:requirement` | `superpowers:brainstorming`（推薦） | `sdlc:system-analysis` |
| `sdlc:system-analysis` | FRD 已完成 | `sdlc:specify` |
| `sdlc:specify` | SAD 已完成 | `sdlc:verifying-specs` |
| `sdlc:verifying-specs` | BFS 已完成 | `superpowers:writing-plans` |
| `sdlc:linear-create` | 驗證通過 | `superpowers:writing-plans` |

## Plugin 分工

| Plugin | 職責 | Skills |
|--------|------|--------|
| **sdlc** | 文件產出（upstream phase） | requirement, system-analysis, specify, verifying-specs, linear-create, foxpro-analyzer |
| **superpowers** | 流程驅動（process & implementation） | brainstorming, writing-plans, subagent-driven-development, verification-before-completion, requesting-code-review, finishing-a-development-branch |
