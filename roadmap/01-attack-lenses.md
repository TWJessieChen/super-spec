# 01 — Final-review 改成 core lens + 兩個攻擊型 lens

> ⚠ **狀態：未實作。動手前必須先與 repo 擁有者（JC）討論並取得同意。** 討論結果寫進最下方「決議」後才可以改。
> 來源：ultra-spec @ `701e685`（2026-08-22）。比對日期 2026-11-17。

## ultra-spec 的做法

- 把 super-spec 的 spec / lifecycle / coherence 三個 lens 合併成一個 **core lens**（`prompts/final-reviewer.md` 的 Part A/B/C，三部分都強制要做），只讀一次文件與 diff。
- 旁邊兩個 lens 都是攻擊型：
  - **adversarial**（`prompts/final-lens-adversarial.md`）：對每個新／改的入口點做惡意輸入、邊界值、依賴失敗注入、重複／亂序／中斷操作、可繞過的檢查；每個發現都要附**具體觸發條件**，否則 skeptic 會駁回。
  - **over-engineering**（`prompts/final-lens-overengineering.md`）：YAGNI，問「哪個 spec 情境或 design 決策要求這個抽象存在？」。三條硬規則：`is_design_flaw` 一律 false、嚴重度上限 `Important`、預設 `Minor`——避免它把變更推進 recover 這條最貴的路徑，也不占 skeptic 名額。
- Panel worst case 從多 lens 變成 3 lens → ≤ 4 skeptic（單票）→ judge。

## 移植到 super-spec 要動的檔

`prompts/final-lens-spec.md`、`final-lens-lifecycle.md`、`final-lens-coherence.md`（合併或刪除）、`prompts/final-reviewer.md`（Part A/B/C）、新增兩個 lens prompt、`workflows/final-review.js`（lens 清單、skeptic 數量與投票方式）、`flows/ultracode-review.md`（args 的 templates 清單）、`flows/native-verify.md`（是否同步）、`README.md` 模型表。

## 取捨

- 好處：super-spec 現在的 lens 都在「照計畫讀」，沒有一個在「攻擊」；簡潔性只是長清單裡的一條。
- 風險：core lens 一個 agent 做三件事可能截斷清單；ultra-spec 的緩解是「三部分都必做」條款，真的發生就把某部分拆回去。
- 注意 super-spec 有 `REVIEW.md` 的 diff 觸發檢查與 Android lifecycle 範例，lifecycle lens 對你的 Android/串流專案特別有價值——合併時不能讓它被稀釋。
- ultra-spec 把 skeptic 改成每個問題 1 票、最多 4 個問題；super-spec 現在是 2 票。驗證會變薄，要決定是否跟進。

## 當時建議

推薦優先討論。最小版本：**保留現有三 lens，只新增 adversarial lens**，over-engineering 先放進 `## Observe` 性質（只記 Minor）觀察幾次。

## 討論時要決定的事

1. 三 lens 合併成 core lens，還是保留現狀只加攻擊 lens？
2. over-engineering lens 要不要做？若做，照搬三條硬規則嗎？
3. skeptic 要維持 2 票，還是改 1 票＋上限 4？
4. native 引擎（無 Workflow）Deep 深度時要不要也跑 adversarial？

## 決議

（尚未討論）
