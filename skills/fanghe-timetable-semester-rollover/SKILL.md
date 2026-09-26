---
name: fanghe-timetable-semester-rollover
description: Reuse the existing Fanghe timetable GPT Project and GAS when a new semester's official timetable PDF arrives. Use for semester rollover such as 115-2: source lock, Canonical rebuild and validation, GPT Project refresh, meeting projection rebuild, GAS freshness-gate cutover, release validation, or deciding whether PDF/schema/policy drift requires a migration ticket instead of normal rollover.
---

# Fanghe Timetable Semester Rollover

以「新學期正式課表 PDF」作為每學期唯一的新來源輸入，重用既有 GPT Project、Canonical pipeline、`meeting_query.py`、meeting projection 與 GAS Web App。

本 skill 的 artifact 形態是 **SOP + Release Gate Checklists**：

- 完整工作路徑 → 讀 `references/ROLLOVER-SOP.md`
- 到高風險發布節點 → 讀 `references/RELEASE-GATES.md`
- 不把整份 SOP 改寫成大型 checkbox list

## 何時使用

當使用者要：

- 將 115-1、115-2 等新學期正式課表 PDF 換入既有系統；
- 更新既有 GPT Project 與 GAS，而不是重建新系統；
- 判斷新 PDF 是否可沿用既有 parser / schema / meeting policy；
- 驗證 Canonical、Projection、GPT、GAS 是否指向同一學期與同一 authority；
- 執行 GAS freshness 的 stale → fresh cutover；
- 完成學期 release / handoff / human acceptance。

## Authority chain

永遠維持：

```text
Official semester PDF
        ↓
Canonical JSON
        ├──→ GPT Project
        ↓
meeting_query.py
        ↓
meeting_projection.json
        ↓
GAS
```

### 不可違反

- **Canonical JSON 是唯一課表 authority。**
- `meeting_query.py` 是 deterministic meeting 判斷核心；正常 rollover 不重寫。
- `meeting_projection.json` 是 derived / disposable，不得人工成為第二權威。
- GAS 只消費 projection 並以 Canonical SHA freshness gate 決定是否放行。
- 不建立 DB、free-slots 表、第二份 current timetable 或第二套 meeting policy。
- `candidate_free` 只表示目前供應證據內無已知課務衝突，不代表教師已確認可出席。
- `unknown`、unmarked week、cross-table conflict 一律不得自動升格為 `candidate_free`。

## 先做正常 rollover / migration 分流

讀 `references/ROLLOVER-SOP.md` 的 Preflight。

如果只有新學期資料，且 PDF 結構、schema、教師 identity 規則、節次結構、週別語意、meeting policy 都相容：

> 走 **normal rollover**，重用既有系統。

如果其中任一項有結構性改變：

> **STOP normal rollover**，先開 migration / defect ticket；只修真正的變更點，完成 regression 後再回 rollover。

不要因為新學期就順手重構既有架構。

## Release gates

只有下列五個 pause points 使用 operational checklist：

1. Source Lock
2. Canonical Release
3. GPT Release
4. GAS Cutover
5. Semester Close

前四個以既有工作完成為前提，不用 checklist 教使用者整個程序。GAS Cutover 因為少見且順序敏感，使用 READ-DO；其他 Gate 使用 DO-CONFIRM。

## State semantics

狀態只是 workflow semantics，不等於 checklist item：

```text
source-locked
canonical-ready-for-human
canonical-accepted
gpt-accepted
gas-stale
gas-fresh
gas-accepted
semester-closed
```

不可把 created / uploaded / deployed 寫成 accepted / completed。

Human acceptance 沒有明確發生前，不得宣稱 `YH accepted`。

## 最小變更原則

依 Ponytail：

1. 先問新東西是否需要存在。
2. 優先重用既有 Project、GAS、schema、query engine、freshness gate。
3. 優先使用平台原生能力。
4. 不增加平行 authority 或同步層。
5. validation、error handling、security、可追溯性、fail-closed 不得為了精簡而刪除。

Drive operational file ID 若可安全維持，優先更新既有 current file；若 File ID 必須改變，只更新必要的 GAS 常數並重新跑 freshness / regression，不建立第二套 current 檔案。

## Field status

v0.1 是 provisional。115-2 應視為第一次正式 field test。

在 115-2 完成前，不宣稱本 skill 已穩定。以實際 failure / catch / rework evidence 決定後續刪除、移動、改寫或新增檢查。
