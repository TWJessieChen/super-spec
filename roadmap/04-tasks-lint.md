# 04 — `tasks.md` lint 腳本

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法（`scripts/`）

- `lint-tasks.py`（670 行）＋ `test_lint_tasks.py`（733 行）：規則 `impl-body`（Task body 不得貼完整實作函式，只允許 file:line 錨點、意圖、不變量、陷阱、逐字簽章）、`illustrative-unmarked`、`code-ratio`、`missing-section` 等。code fence 用**黑名單**判斷：沒標語言或語言不認得的一律當實作候選。exit code 0／1（error）／2（只有 warning）。
- `lint-execution-plan.py`（411 行）＋測試：檢查 waves／per-task model 檔，依賴 [07](07-waves-parallel-apply.md)。
- 由 Phase 2 在 planning commit 前呼叫。

## 理由

Task body 貼完整實作 → implementer 照抄、reviewer 無從判斷，且每次 dispatch 冷啟動成本上升。

## 移植要動的檔

新增 `scripts/lint-tasks.py`（＋共用 lib＋測試）；`phases/03-plan.md` 寫完 `tasks.md` 後執行；`flows/pre-flight.md` 檢查 `python3`；`README.md` 依賴表。

## 取捨

- 多一個 Python 3 依賴（你的 Windows／macOS 環境都要有）。
- 規則門檻（多少行算 impl-body）要依你的專案校準，Kotlin/C++ 的簽章可能較長。

## 當時建議

可考慮，只搬 `lint-tasks.py`，不搬 execution-plan 那支。

## 討論時要決定的事

1. 接受 Python 依賴嗎？
2. lint 失敗是硬擋還是只提醒？

## 決議

（尚未討論）
