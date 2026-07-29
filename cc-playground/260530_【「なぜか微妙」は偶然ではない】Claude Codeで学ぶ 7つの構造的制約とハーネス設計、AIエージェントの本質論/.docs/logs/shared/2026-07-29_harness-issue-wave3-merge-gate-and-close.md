---
date: 2026-07-29 23:13:10
type: work
topic: harness-issue-wave3-merge-gate-and-close
session: cc-learn 260530 メインセッション (第3波マージゲート)
related_skill: [handoff, pickup, commit, logging]
related_log_ids: [2026-07-26_harness-issue-wave1-close-and-wave2-review, 2026-07-27_harness-improvements-locale-failclosed-oracle-hardening]
related_log: [.docs/logs/shared/2026-07-26_harness-issue-wave1-close-and-wave2-review.md, .docs/logs/shared/2026-07-27_harness-improvements-locale-failclosed-oracle-hardening.md]
---

# ハーネス issue 第3波 (捏造ガード #257 + #220/#244/#245/#251) マージゲートレビューと全着地

> 5 レーン同時起動 → PR 5 本の独立レビュー (fresh code-reviewer 5 並列) → 是正 → 再ゲート (spend limit によりメインのインライン検証) → **5/5 マージ完了**。人間承認捏造ガード L1+L3 が live 稼働開始。

## 概要

第3波 = 5 レーン同時起動 (#257 捏造ガード / #251 / #245 / #244 / #220)。当初は「ガード先行→5レーン」のデンデ裁定だったが、本人が AskUserQuestion で「5 レーン同時 (マージは #267 最優先)」へ変更 (2026-07-28、本セッションの AskUserQuestion 履歴が痕跡)。PR 5 本 (#258/#259/#265/#266/#267) をレビュー → 条件付き判定 → レーン是正 → 再ゲート → 全マージ (2026-07-29)。

**着地の要**: 人間承認捏造ガード (L1 `rules/grounds-not-approval.md` + L3 `hook_pre_commit_grounds_gate.sh`) の live 稼働開始。第1波 12 + 第2波 6 箇所の捏造への恒久対処。

## 内容

### マージ結果 (時系列・全て実測確認済み)

| PR | issue | 初回判定 | 再ゲート | merge commit | 備考 |
|---|---|---|---|---|---|
| #267 | #257 | CONDITIONAL GO (H4/M6/L5) | GO | `1597e46` (03:25Z) | ガード。マージ直後に live pull |
| #258 | #245 | CONDITIONAL GO (H4/M2/L2) | GO | `b1928e9` | ui fork 単語境界 |
| #259 | #220 | CONDITIONAL GO (H4/M3/L5) | GO | `e3c406d` | agent tools 絞り |
| #266 | #251 | **NO-GO** (C1/H5/M5/L5) | GO | `89f4004` | !構文 CLAUDE_CONFIG_DIR 解決形 |
| #265 | #244 | CONDITIONAL GO (H1/M3/L7) | GO (rebase 後) | `a4da904` (13:57Z) | records dir 述語正本化 |

マージ順の裁定 (Lead 判断): **#267 → #258/#259 → #266 → #265**。#266→#265 の順は、SKILL.md 衝突解消で現実に落ちるリスク (= 他 PR の変更) を #266 の回帰テストが機械で捕まえられる側に倒すため。#265 レビュアーは逆順 (#265→#266) を推奨したが、「解消者が自分の変更を落とす」想定で論理が逆さまだった (解消者が落とすのは他人の変更)。レビュアー間の推奨が割れた時に Lead が理由付きで裁定した実例。

### 初回レビューの主要所見 (fresh code-reviewer 5 並列、各 PR 1 体、隔離 export + 実走)

**是正条件の逐語正本は各 PR のマージゲートコメント** (5110354978 / 5110355131 / 5110355280 / 5110355439 / 5110355695)。以下は要旨。

#### #267 (ガード) — 検出器自身の穴
- [H] 未 push 4 commits で PR head ≠ ローカル (レーン自身が規約違反と判定して外したログが着地する状態)
- [H] committer 引数解決が 9 形で silent fail-open (subdir cwd / 絶対パス / ./ / dir 引数 / 単引用 / bare / 複数呼出 / `git commit -a` / `--all` — 全形 probe 実走で立証)
- [H] commit message を 1 バイトも検査せず、L1 の宣言射程 > L3 実装で非開示
- [H] 「判断」の検出除外理由が論理誤り (`Lead` は人名網に無く正面衝突しない)。「デンデ判断」が素通し
- [M] **「確認していない値を確定形で書くな」を昇格させた当の PR に未再現数値 2 つ** (在庫 37 行 / 再現率 5/5)

#### #266 — 出荷証跡の層で NO-GO
- [C] record の High→Medium 降格理由に「Dende が AskUserQuestion で選択」— **機械痕跡なしの人間判定引用**が verdict 数値 (high_count: 0) へ連鎖
- [H] 必須バリデータ 2 本 (Step 6-3/6-3b) 未達のまま gate 通過 / orchestrator だけ新形 Gotcha 不在 / `set -u` で prelude 全滅 (実測) / permission 整合未確定 / #265 と衝突確定
- 実装自体は mutation 5 種全捕捉の良品 — 「ブロックは実装でなく証跡」という判定

#### #259 — 根拠の壊れ
- [H] 「決め手」の引用先 (`assets/config.json`) に該当文言が実在しない (実在は `references/review-workflow.md:28`、しかも射程違い)。**結論は正しいが根拠が壊れている**クラス
- [H] 新テストが `tools: null` / `tools: # コメント` で全緑 (mutation 実測) / security agent 本文が実装命令のまま Bash 迂回が未開示 / 未 push ログ二重化

#### #258 — 対象ドメイン外で測った偽陰性
- [H] `_` 語構成文字化で `Roboto_Mono` 等が 0 件 = **SCAN-OK (0 件) という清潔シグナルに洗浄** / design 語彙 (modernist 等) の取りこぼし / 自己参照が可視枠 10 行中 5 行 / 拡張禁止 assert がエスケープ段数事故で `\b` を検出不能 + 「23 FAIL」未再現
- 捏造スキャンで唯一の人間裁定引用を**レビュアーが transcript から独立に TRUE 立証** (locator 付き) — 後続の discharge 手法の原型

#### #265 — 検証の空洞化を自 PR で再生産
- [H] 出荷形 (`2>&1` 併合) が stdout 1 行契約を壊しうるのに、同 PR の assert は `2>/dev/null` 形しか測らない
- [M] log にしか無い申し送り 4 件 (→ issue 化を条件に) / 衝突予告の不足

### 捏造まわりの今波の学び

- #266 の Critical は「捏造」ではなく「**実在した対話を機械解決可能な形で書かなかった**」— メインが両レーン (issue-251 / issue-257) の worktree transcript (JSONL) から AskUserQuestion の実在を確認 (tool_use id + timestamp + 選択文言)。discharge = **逐語 + locator を record へ** (worktree 撤収で jsonl は消えるため逐語が恒久形)
- この形式は L1 ルール (grounds-not-approval) の要求そのもので、是正後の record は準拠形になった
- #267 レーンは是正時に「実測したと書いた値 2 件の誤り」を自己申告し、「本文の数値でなく再現コマンドを正とする」形へ置換。L0→L1 昇格 (「確認していない値を確定形で書かない」Gotcha) もレーン内 AskUserQuestion で決定・追加

### 再ゲート (条件 1:1 照合) — 方法の開示

**サブエージェント 5 体は月次 spend limit で全滅** (mid-flight 死)。第1波の前例 (spend limit 時はメインのインライン検証 + 開示) に従い、**メインがインラインで全条件を照合**した。独立性はサブエージェントに劣るが、初回の敵対的レビューは独立済みで、再ゲートは機械的照合が主。実施内容:

- #267: committer 9 形 probe 再実走 → **全 BLOCK 化**。`git commit -am` → BLOCK (走査源切替)。message 捏造 → WARN。「かいじゅう判断/デンデ判断/デンデ+選択」→ BLOCK、「Lead 判断」→ PASS (規約の正形を殺さない)。guide:50 悪例の 〈人名〉 化。単体テスト 53/0、run-all 48/1 (落ちる 1 件は main の clean export でも同一 = 既存環境依存、退行ゼロ)
  - 途中でメインの probe ハーネス不備 (marker 無しで gate 不活性) により偽の素通し観測 → 原因特定して marker 付きで再測。「検証装置自身を疑う」の実例として記録
- #266: record の逐語 + locator を transcript 実物と照合一致。`validate-verdict-consistency.sh` exit 0 実走。orchestrator 再走 + `validate-all-steps` COMPLETE (progress JSON は #263 未修正ゆえ本体へ落ちた — record が実測として開示、その JSON は 86d0df7 で追跡へ)。Gotcha must + TARGETS 4 skill + `set +u` prelude 22 箇所。bang テスト 30/0
- #265: `head -1` + 絶対パス case + 失敗時のみ stderr 再送を実装確認。stderr 吐き述語 stub で出荷形 1 行を実走確認、複数行述語 → fail-closed + junk 無し。出荷形 assert (stderr-noisy stub) の実在。96/0。issue 化 4 件 (#268-#271) の実在確認
- #258: fixture 実走 — `Roboto_Mono` 検出 / modernist・Beautifully・Elegance 検出 + cleanup 非検出 / 自己 dir を limit 前除外 (開示行付き)。拡張禁止 assert は `grep -cF` + 空回り防止の正対照付きへ書換え済み。契約テスト 321/0。「23 FAIL」の誤り訂正を record で確認
- #259: 引用訂正 4 箇所 (誤帰属の根因まで開示) / `tools: null` mutation → 10 passed 2 failed (検出力実証) / run-all 常時併走 2 行配線。フォロー issue だけ「起票します」宣言のまま未実施 → **メインが #272 を代行起票**
- #265 の rebase 後最終確認: 結合形 3 要素 (prelude / `2>&1 || echo` / 行49 見出し) の共存を grep で確認、両テスト 30/0・96/0 隔離実走、diff 8 ファイルでログ混入なし。レーンは rebase 中に「main の commit 済みログを旧 commit が削除しかける」のを modify/delete 衝突で検知し skip (事故の自力回避)

### メイン代行・後始末 (実施済み)

- **#272 起票** (security-devsecops-expert の本文/description と read-only tools の職務不整合 — #259 条件3 の未履行を代行)
- レーンログ 5 本を本体へ commit (`07862ff`) — #267 条件6 は worktree guard により「レーンから構造的に実行不能」が正当と確認しメイン代行
- 進捗 JSON 3 件 commit (`86d0df7`、前例 820b19e 準拠)。いずれもデンデが push 済み (origin/main = 86d0df7 → その後 a4da904)

### 定着した設計・運用 (今波の収穫)

1. **是正条件を PR コメントに機械可読で置く** — レーンが `gh` で読める・後続レビューが条件正本を参照できる。散文の申し送りより強い
2. **人間判定の引用は「逐語 + 機械痕跡 (locator)」が最低形** — transcript は揮発するため逐語が恒久形。L1 ルールと運用が一致した
3. **マージ順はレビュアー間で割れうる — 「解消で落ちるのは他人の変更」を機械 backstop が捕まえる側に倒す**
4. **spend limit 時のフォールバック** — メインのインライン再ゲート + 方法の開示 (第1波から 2 回目、運用として確立)
5. レーンの成熟: 事故の自力回避 (modify/delete)、実測誤りの自己申告、実行不能条件の構造的理由付き差し戻し

### 未解決 / 申し送り

- **CI ゼロ継続** — テスト緑は隔離実走が唯一の根拠。run-all の 1 件 (credstore テスト) は clean export で常に落ちる環境依存 — hermetic 化は未 issue (次波 #260-262 の隣接)
- フォロー issue 残: #260-#264 / #268-#272 (+既存 #221/#222/#223/#243/#246 と Apple 系 #218/#219/#224/#226)
- ガードの KNOWN GAP (レーンが開示済み): 人名を伏せた言い換え 4 形 / 2 行分割 / 「指示」語彙 / PR body・issue コメント (hook から不可視) は検出外 — L1 規律が塞ぐ領域
- essence 提案 2 本 (worktree essence-* 駐車中) のレビューは未実施

## 関連ファイル

- kaijutale/claude-harness PR #258/#259/#265/#266/#267 — 是正条件とレーン報告の逐語正本 (`gh pr view <N> -R kaijutale/claude-harness --comments`)
- `~/.claude/rules/grounds-not-approval.md` / `~/.claude/hooks/hook_pre_commit_grounds_gate.sh` — 今波の主着地物 (L1+L3)
- `~/.claude/.docs/logs/local/2026-07-29_issue-{220,244,245,251,257}-*.md` — レーン側セッションログ (commit 07862ff)
- `.docs/logs/shared/2026-07-26_harness-issue-wave1-close-and-wave2-review.md` — 前波ログ (捏造 12 箇所の検出と L1+L3 昇格の記録)
- note 260405 (本 PJ の参照記事) — glob `~/.claude/.docs/references/note-articles/260405_*/`。索引 `~/.claude/.docs/references/BIBLIOGRAPHY.md`
