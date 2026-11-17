# 10 — 只保留 ultracode 引擎、移除 native

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法

移除 HARD-GATE B2 的引擎選擇；沒有 Workflow 工具時 pre-flight 直接停止，沒有 native fallback。所有 phase 都走 Workflow 腳本，prompt 以絕對路徑傳給 subagent 自行讀取（不讓 prompt 內容進主 context）。

## 取捨

- 好處：只維護一條路徑；結構化輸出取代字串協定。
- 代價：沒有 Workflow 工具的環境（部分 Claude Code 版本、雲端或其他 surface）完全不能用；super-spec 的 `flows/native-verify.md`、Phase 4 native 迴圈全部失去意義。

## 當時建議

不建議。super-spec 保留 native 是刻意的退路。可單獨考慮的子項目：「prompt 只傳路徑、由 subagent 自己讀」以節省主 context——這可以在不移除 native 的前提下做。

## 討論時要決定的事

1. 只做「prompt 傳路徑」這個子項目嗎？

## 決議

（尚未討論）
