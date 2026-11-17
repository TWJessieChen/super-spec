# 07 — Plan-review gate + waves 平行 apply

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法（`flows/plan-review-gate.md`、`workflows/apply.js`、`scripts/lint-execution-plan.py`）

- Phase 2 結束、apply 之前的 HARD-GATE B：orchestrator 用 Grep 只讀每個 Task 的 `**Interfaces**`（Consumes）與 `**Files:**`，建 DAG：依賴邊＋檔案重疊邊；「如盤點出缺口才 Modify」這種執行期才知道的目標降為單獨序列波。
- 推導 waves、每個 Task 的 implementer／reviewer model 與 TDD／Simple mode，給使用者確認後寫成 `execution-plan.md`（之後 resume 直接解析，不靠記憶；缺檔則全序列＋預設）。
- `apply.js` 依 waves 用 git worktree 平行跑互不相交的 Task，再 reconcile；revision Task 不在 waves 裡，一律一 Task 一波。

## 取捨

- 好處：大變更的 Phase 4 牆鐘時間大幅縮短；Task 拆分在執行前多一次人工檢視。
- 代價：worktree 與合併的失敗模式（建置快取、Gradle／AOSP 大型 build 在多個 worktree 會很重）；`apply.js` 複雜度大增（ultra-spec 447 行 vs super-spec 296 行）；native 引擎無法支援。
- 對 Android／AOSP 專案：平行 build 很可能吃滿資源，收益存疑。

## 當時建議

暫不建議整套搬。若想要「apply 前人工審 Task 拆分」，可只搬 gate 的檢視部分，不做平行。

## 討論時要決定的事

1. 你的專案實際上有多少 Task 是可平行、且 build 不互相拖累？
2. 只要檢視閘、不要平行，可以嗎？

## 決議

（尚未討論）
