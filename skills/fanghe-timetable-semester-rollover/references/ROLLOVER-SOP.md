# Rollover SOP

## 1. Purpose

把每學期新的正式課表 PDF 安全換入既有「芳和全校課表」系統，同時重用既有 GPT Project 與 GAS，並確保兩個 consumer 最終只服務同一份已驗收的 Canonical。

## 2. Scope

正常 rollover 的新輸入：

- 新學期正式課表 PDF。

正常 rollover 原則上重用：

- GPT Project；
- GAS Web App；
- Canonical schema / validation；
- 教師與班級 identity 規則；
- `meeting_query.py`；
- projection builder；
- Freshness Gate；
- regression / acceptance / handoff pattern。

每學期重新產生或更新：

- source manifest / source SHA evidence；
- Canonical JSON 內容；
- dataset version；
- Canonical SHA-256；
- issues / evidence；
- `meeting_projection.json`；
- semester regression evidence；
- release / handoff snapshot。

## 3. Preflight：normal rollover 還是 migration？

在任何轉錄前比較新 PDF 與現行 pipeline。

### Normal rollover

只有資料內容改變，且下列結構仍相容：

- PDF 版面 / 表格定位；
- schema；
- 教師 identity 規則；
- 班級 identity 規則；
- 星期 / 節次結構；
- odd / even / all / unmarked 語意；
- meeting candidate policy。

→ 繼續正常 rollover。

### STOP / migration

若發生下列任一情形，不得硬套舊 pipeline：

- PDF 表格結構改到既有 parser 無法可靠定位；
- schema 必須新增 / 改義；
- identity 規則變更；
- 星期或節次結構變更；
- 週別標記語意變更；
- meeting policy 變更；
- 已證實既有程式存在 defect。

處理方式：

1. 暫停 normal rollover。
2. 開 migration / defect ticket。
3. 只修改真正受影響的 authority / code seam。
4. 跑既有 regression，避免破壞舊行為。
5. 回到本 SOP。

## 4. Source intake

保存新學期官方 PDF 原始檔，不先轉成另一份 authority。

記錄：

- semester；
- original filename；
- source date / acquired date（可取得時）；
- file size / page count（可取得時）；
- source SHA-256；
- source manifest；
- 版本衝突、缺頁、重複來源或尚未確認狀態。

若同時存在多個「正式版」候選，先解決 source authority，不得靠 checklist 假裝已決定。

完成 source intake 後執行 **PP1 Source Lock**。

## 5. Canonical build

以既有 pipeline 重建 / 更新 `school_timetable.pilot.json`。

至少保留：

- 新 dataset version；
- source traceability；
- teacher / class identity；
- 星期與節次；
- week semantics；
- issues / conflicts / unknown evidence；
- Canonical SHA-256。

### Validation

依現行專案 acceptance 重跑：

- JSON parse / schema；
- 預期教師與班級集合；
- slot completeness；
- odd / even / all / unmarked；
- cross-table checks；
- known issue mapping；
- sampled source-to-canonical verification；
- 既有 protected regression。

安全邊界不變：

```text
unmarked ≠ all
unknown ≠ candidate_free
cross-table conflict → unknown / review
```

Validation 完成後進入 **PP2 Canonical Release**。

Canonical 未 human accepted 前：

- 不更新 GPT active knowledge；
- 不把 GAS 恢復到 fresh；
- 不宣稱 semester data 已發布完成。

## 6. GPT Project rollover

重用既有 GPT Project。

更新 active semester knowledge，使 current authority 指向新學期 accepted Canonical 與相對應 current state / handoff / source manifest。

舊學期資料可以封存在 release pack，但不得與新學期資料同時形成第二份「current Canonical」。

### GPT regression

依新資料挑選實際案例，測試類型至少包含：

- 單一教師課表；
- 兩位教師共同候選；
- busy；
- odd / even；
- unknown；
- cross-table issue（若本學期存在）。

驗證：

- dataset version 是新學期；
- 查詢使用新 Canonical；
- 不混入上一學期 current data；
- unknown 不被推測成 free；
- candidate_free 語意不變。

完成後執行 **PP3 GPT Release**。

## 7. GAS rollover

重用既有 GAS Web App，不重建第二個 Web App。

### 7.1 更新 current Canonical

優先在現行 operational location 更新 accepted Canonical。

若 Drive operational file 可以安全保留同一 File ID，優先保留；若 File ID 改變，只更新必要的 `CANONICAL_TIMETABLE_FILE_ID`，不得建立平行 current authority。

Canonical 更新後：

```text
Canonical SHA = new
Projection canonical_sha256 = old
→ Freshness mismatch
→ GAS must become STALE / fail closed
```

此 stale 是預期 cutover 狀態。

### 7.2 重建 Projection

使用既有 `meeting_query.query_meeting()` / projection builder，以 accepted Canonical 產生新的 `meeting_projection.json`。

驗證：

- dataset version 對應新 Canonical；
- projection.canonical_sha256 = 新 Canonical SHA；
- teacher set / slots 正確；
- odd/even deterministic states 正確；
- unknown 不被放行；
- protected meeting regression PASS。

不要人工編輯 projection 來「修正」結果。

### 7.3 更新 current Projection

更新 GAS 所讀的 current projection。

若可安全保留同一 File ID，優先保留；若 File ID 改變，只更新必要的 `MEETING_PROJECTION_FILE_ID`。

更新後：

```text
Canonical SHA = Projection canonical_sha256
→ Freshness = FRESH
```

### 7.4 Web App regression

確認：

- freshness = enabled + SHA match；
- 新學期教師清單；
- shared meeting candidate；
- unknown summary / detail；
- known issue detail；
- desktop；
- mobile。

整個 7.1–7.4 依 **PP4 GAS Cutover** 的 READ-DO checklist 執行。

## 8. Closeout

只有 GPT 與 GAS 都完成 human acceptance 後才能進入 closeout。

封存：

- source manifest / source hash；
- accepted Canonical；
- Canonical SHA；
- projection；
- acceptance / regression evidence；
- current state；
- handoff snapshot；
- final checksums / release pack。

current operational filenames可保持穩定；歷史版本放進 semester release / archive，不用在 current layer 製造 `final_final_v3` 類平行檔名。

最後執行 **PP5 Semester Close**。

## 9. Failure handling

### Source ambiguity
停在 source intake，先確認正式來源。

### Format drift
停 normal rollover，開 migration / defect。

### Canonical validation failure
不得發布任何 consumer。

### GPT regression failure
GPT 不得標示 accepted；不影響 Canonical authority 本身。

### GAS stale
保持 fail closed；先重建 / 更新 projection，不關閉 gate。

### Freshness read/hash error
視同 stale / unavailable，不用 modified time 或人工直覺放行。

### Human acceptance missing
保持 ready-for-human / pending；不宣稱 closed。
