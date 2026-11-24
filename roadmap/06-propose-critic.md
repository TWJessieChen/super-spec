# 06 — Propose 階段 critic 一次審四份文件

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法（`flows/ultracode-propose.md`、`workflows/propose.js`、`prompts/propose-critic.md`）

- orchestrator 單獨起草 proposal／design／spec deltas，`writing-plans` 產出 `tasks.md` 後，**一個 critic** 一次壓測四份文件（含 `tasks.md` 的拆分）。
- critic 只讀磁碟與 git；gating 問題由 orchestrator 折回文件，再做 planning commit。
- 啟動前把 critic 的 model 給使用者確認。

## 理由

super-spec 在 Phase 4 開始前，沒有任何自動審查看過 `tasks.md` 的拆分，而它是最貴的階段要執行的計畫。

## 移植要動的檔

`phases/02-propose.md`／`phases/03-plan.md`（在 planning commit 前插入 critic）、新增 `prompts/propose-critic.md`；ultracode 引擎可用 Workflow，native 引擎用單一 Agent dispatch 即可。

## 取捨

每個變更多一次 dispatch（建議 sonnet/opus 一次）；換來早期抓到拆分錯誤，避免 Phase 4 多輪 FAIL。

## 當時建議

可考慮，且可以兩種引擎都支援（不需要 Workflow）。

## 討論時要決定的事

1. 每個變更都跑，還是只在 Task 數 ≥ N 或深度 Deep 時跑？
2. critic 用哪個 model？

## 補充分析（2026-11-24，對照 pstack）

Lauren Tan（pstack 作者）的實務做法和這項相反：她不對「還沒有實物的計畫」做對抗審查，理由是審查用的 agent 讀不到能驗證的東西，容易憑空列出不會發生的風險，反而增加修改成本。她的替代做法是對能觀察的岔路直接做一個可丟棄的原型，讓結果決定設計，或在跨模組時先寫出型別與介面。來源：pstack `/interrogate` 說明、使用指南第 4 章（cursor/plugins，2026-11-21 版）。

對這項的影響：如果要做，critic 應該只審「能對照現有程式碼查證」的部分（例如 tasks.md 引用的檔案與 API 是否存在、拆分是否和 `## File Structure` 一致），不審抽象的風險推測；或改成在 Phase 2 設計有疑義時做一個原型。這只是分析，決議仍待討論。

## 決議

（尚未討論）
