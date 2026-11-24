# 09 — Brainstorm 設計空間多 agent 擴散

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法（`flows/ultracode-brainstorm.md`、`workflows/brainstorm.js`、`prompts/brainstorm-explorer.md`、`brainstorm-judge.md`）

- Part 1：與使用者對齊需求，產出 `## Aligned requirement` 與 `## Context notes`。
- Part 2：N 個 explorer（預設 3 個角度：最小可行／風險最低、長期可維護／架構最乾淨、使用者與開發者體驗優先）各提一個設計 → judge（最強 alias）合成一個設計 → HARD-GATE A 使用者核准。
- 只傳需求區塊逐字全文，不傳對話紀錄；全程 read-only，死掉就重跑。

## 取捨

- 好處：避免單線對話太早收斂到第一個想到的設計。
- 代價：每個變更都多 3 explorer + 1 judge；需要 Workflow 工具（native 引擎沒有）；小變更明顯浪費。

## 當時建議

暫不建議預設開啟。若要做，做成選項（例如 `--explore` 或 brainstorm 時問一次），只在架構性變更使用。

## 討論時要決定的事

1. 做成選項還是不做？
2. native 引擎下要不要用多次 Agent dispatch 模擬？

## 補充分析（2026-11-24，對照 pstack）

pstack 處理同一個問題（避免太早收斂到第一個設計）的方式不是多 agent 擴散文字提案，而是 Prototype playbook 與 `/arena`：對能跑出結果的設計岔路做 2–3 個可丟棄的原型並排比較，由實際行為決定。她也提醒多數日常改動不需要這種重流程。來源：pstack `poteto-mode/playbooks/prototype.md`、原則 exhaust-the-design-space（cursor/plugins，2026-11-21 版）。

對這項的影響：若要做，比較貼近 super-spec 的形式是「brainstorm 時問一次是否要做原型比較」，只用在架構性變更，而不是預設擴散 explorer。這只是分析，決議仍待討論。

## 決議

（尚未討論）
