# Release Gate Checklists

本檔只保護高風險 transition。完整工作方法在 `ROLLOVER-SOP.md`。

## Reliability target

**Purpose:** 在新學期 rollover 中，避免錯來源、錯 Canonical、舊學期資料混用、stale projection 被放行，以及未完成驗收就宣稱結案。

**Avoidable failures:**

1. 拿錯 PDF / 舊版本開始轉錄。
2. PDF 已 format drift，舊 pipeline 仍照跑。
3. Canonical 未驗收就發布，或 unknown / unmarked / conflict 被錯誤放行。
4. GPT 同時存在兩份 current semester authority。
5. Canonical 已換但 GAS 繼續使用舊 Projection。
6. GPT / GAS 未全部驗收就宣稱 semester closed。

**Target user/team:** 芳和課表行政維護者（YH）與協作 agent。

---

# PP1｜Source Lock

**Pause point / trigger:** 收到新學期 PDF，準備開始 Canonical 轉錄前。  
**Mode:** `DO-CONFIRM`  
**Owner/facilitator:** YH / 當次 rollover 執行者。

### Task checks

- [ ] 學期、來源名稱與指定的正式完整 PDF 相符。
- [ ] 原始 PDF 的 SHA-256 / source manifest 已可追溯。
- [ ] Format drift 已明確判定為「相容」；若不相容，已 STOP normal rollover。

### Communication check

只有當存在多個相互衝突的正式版候選時：在繼續前，由來源 owner / YH 明確指定 current source。

### Proceed condition

唯一來源已鎖定，且沒有未處理的 format drift。

---

# PP2｜Canonical Release

**Pause point / trigger:** Canonical build + machine validation 完成，任何 consumer 更新之前。  
**Mode:** `DO-CONFIRM`  
**Owner/facilitator:** YH。

### Task checks

- [ ] dataset version 已切到新學期，Canonical SHA-256 已鎖定。
- [ ] schema / identity / slot / protected regression 全部 PASS。
- [ ] `unmarked`、`unknown`、cross-table conflict 沒有被錯誤放行。
- [ ] Source-to-canonical 抽核與必要 issue evidence 已完成。
- [ ] YH 已明確接受這份 Canonical。

### Proceed condition

Canonical = accepted。只有此狀態可以發布至 GPT 或用來恢復 GAS fresh。

---

# PP3｜GPT Release

**Pause point / trigger:** GPT Project 已換入新學期資料，準備宣告 active / accepted 前。  
**Mode:** `DO-CONFIRM`  
**Owner/facilitator:** YH。

### Task checks

- [ ] Active current Canonical / dataset version 是新學期。
- [ ] 舊學期資料沒有形成第二份 current authority。
- [ ] 單一教師、共同候選、week、unknown / issue regression 已以新學期案例 PASS。
- [ ] `candidate_free` 的證據邊界仍清楚，沒有被改寫成「教師已確認可出席」。
- [ ] YH 已完成 GPT Project human acceptance。

### Proceed condition

GPT = accepted。

---

# PP4｜GAS Cutover

**Pause point / trigger:** 已有 accepted Canonical，準備把 GAS 從舊學期切換到新學期。  
**Mode:** `READ-DO`  
**Owner/facilitator:** rollover 執行者；必要 human gate 由 YH 驗收。

這是一個低頻、順序敏感的 established sequence，依序執行：

1. [ ] 更新 GAS 所指向的 accepted Canonical。
2. [ ] 確認 freshness 進入 **STALE / blocked**，舊候選不可使用。
3. [ ] 以 accepted Canonical 和既有 deterministic engine 重建 projection。
4. [ ] 更新 GAS 所指向的 projection。
5. [ ] 確認 freshness 恢復 **FRESH / SHA match**。
6. [ ] 新學期 desktop + mobile regression PASS；unknown / issue detail 仍 fail closed。

### Proceed condition

GAS = FRESH，且 regression / human acceptance 完成。

若第 2 步沒有進入 STALE，或第 5 步無法恢復 FRESH，立即停止，不用人工跳過 freshness gate。

---

# PP5｜Semester Close

**Pause point / trigger:** 準備把整個 semester rollover 標示 CLOSED 前。  
**Mode:** `DO-CONFIRM`  
**Owner/facilitator:** YH。

### Task checks

- [ ] GPT Project = YH accepted。
- [ ] GAS = FRESH + YH accepted。
- [ ] Release evidence、SHA / checksums、handoff / current-state 已鎖定。

### Proceed condition

只有以上全部成立，才可標示：

```text
Semester = CLOSED / YH accepted
```

---

# Professional judgment / intentional omissions

以下不放進 checklist，因為它們屬於 SOP、專業判斷或只有在特定情況才需要處理：

- PDF 逐頁如何轉錄；
- parser / schema 的完整實作細節；
- 每一個教師與班級的普通 production step；
- migration 的設計方案；
- 每個可能的課表例外；
- 一般 Drive / Apps Script 教學；
- release pack 的每個檔名細節。

Format drift、source authority、是否需要 migration 仍需要根據當期證據判斷；checklist 不取代這個判斷。

---

# Design rationale

- **PP1** 在任何資料衍生前阻止錯來源與 format drift。
- **PP2** 是最高槓桿 release gate，防止錯 Canonical 擴散到兩個 consumer。
- **PP3** 防止 GPT current knowledge 混入上一學期或改寫證據邊界。
- **PP4** 使用 READ-DO，因為 GAS cutover 低頻且順序錯誤會造成舊 projection 暴露；freshness gate 本身就是安全鎖。
- **PP5** 防止 uploaded / deployed 被誤寫成 accepted / closed。

每個 gate 只保留具有明確 consequence、可現實漏掉、且能在該 pause point 觀察確認的 killer checks。

---

# Field-test plan｜v0.1

**First test context:** 115-2 真實換版。  

**Primary learning question:**  
這五個 pause points 是否足以阻止錯學期、錯來源、stale projection 與 false completion，同時沒有把正常 rollover 變成繁瑣 checkbox 工作？

**Observe:**

- 哪個 Gate 需要提醒才會被執行；
- 是否有 item 被略過 / shortcut；
- 是否有 wording 被不同理解；
- 是否有 checklist item 其實只是 SOP leakage；
- GAS 在 Canonical 更新後是否確實 catch 到 STALE；
- 是否發生因 gate 過晚而產生的 rework；
- 每個 Gate 的實際完成時間。

**Revision criterion:**

優先順序：

1. 刪除；
2. 移到更好的 pause point；
3. 改寫得更精確；
4. 只有 genuinely different pause point 才拆分；
5. 只有 field evidence 支持時才新增 item。

在 115-2 field evidence 完成前，本 checklist 保持 provisional。

---

# Outcome measures

優先追蹤：

- 錯學期 / 錯來源資料被發布：目標 0；
- Canonical 更新後舊 Projection 仍可查：目標 0；
- unknown / unmarked / conflict 被升格為 candidate_free：目標 0；
- normal rollover 為了換資料而修改核心 meeting policy：目標 0；
- GPT / GAS 未完成 acceptance 卻宣稱 semester closed：目標 0；
- 因 release gate 過晚造成的重做 / rollback 次數。

Checklist completion rate 可以記錄，但不能單獨證明可靠性。
