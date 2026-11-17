# 08 — Cost-fit 模型分配 + 每次啟動前確認

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法（`flows/model-orchestration.md`，59 行，執行期規則檔）

- 不用固定模型表；依「該角色的風險門檻」選剛好夠的 alias：生成與機械角色用便宜模型，驗證角色用強模型；implementer 依每個 Task 難度個別挑（禁止不評估就套預設，但評估後相同是允許的）。
- 最強 alias 只給兩個高槓桿點：Phase 1 synthesis judge（設計錯了全盤皆錯）與 Phase 4 skeptic。judge 只負責組文字（verdict 由腳本計算），用最便宜的。
- 每次啟動 Workflow 前都把模型配置以一行表給使用者確認。

## super-spec 現況

README「Subagent model assignment」固定表；ultracode 在 HARD-GATE B2 自動推導並寫進 `proposal.md` 的 `## Ultracode Agent Models`。

## 取捨

- 好處：成本更貼合難度；設計理由集中在一個規則檔。
- 代價：每次啟動多一次確認互動；模型選擇變得不固定，不同次執行難比較；review stats 的可比性下降。

## 當時建議

暫不建議。可以只借用「implementer 依 Task 難度選 sonnet/opus」這一點（super-spec 已有 TDD (Sonnet)／TDD (Opus) mode，接近了）。

## 討論時要決定的事

1. 你在意的是成本還是可預測性？
2. 每次啟動前確認，會不會太煩？

## 決議

（尚未討論）
