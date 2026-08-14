# ハーネス issue 第13波 マージゲート + クローズ (2026-08-14 起動 〜 同日 マージ完了)

対象リポジトリ: `dendedev/claude-harness`。base `b1b62bc` → **`9089726`** (マージ **5 本**)。

## 結果

| 項目 | 値 |
|---|---|
| レーン | **4 本** (A=#414 / B=#406 / C=#416 #409 / D=#408 #415) |
| PR | **5 本すべてマージ** — `89176da` (#418) / `9d97758` (#422) / `8c9ee7a` (#420) / `96d0df1` (#423) / **`9089726` (#426 = ゲート自身の後始末)** |
| close した issue | **5 本** (#406 #408 #409 #414 #415) |
| 新規起票 | **4 本** (#419 #421 #424 #425) |
| open issue | 11 → **10** (検算: 11 − 5 + 4) |
| レビュー判定 | **4 本とも 🟢 GO** (1 段のみ。2 段目は無し — 下記「縮退」) |
| post-merge full run | **77 file(s) passed / 0 failed** (live `9089726` で実測、assert 2858) |
| ゲート運用者の誤り | **9 件、すべて自己検知** (外部指摘 0)。内訳はセッション運用ログ |

## 本波の特徴 — 「装置の保証」を撃つ波だった

第12波の High の過半が「装置が守ると宣言した条件で発火しない」class だったのを受け、本波は 4 レーンのうち **2 本 (C / D) がその class の機構化**に充てられた。

| レーン | issue | やったこと |
|---|---|---|
| A | #414 | 規模ラチェット条件 A の成立 → **改名を実行** (`hook_pre_commit_essence_gate.sh` → `hook_pre_commit_gate.sh`)。25 files / +547 -142 |
| B | #406 | review-harness の Phase 6 テンプレを detected 2 フィールドへ追随 + 書出直後の決定論検証 (6-3) を新設 |
| C | #416 A/B + #409 | 結論行 verdict の**併記契約を機械強制** + 任意 step の実施を痕跡化。15 files / +884 -77 |
| D | #408 + #415 | integrity fork の**検出層を機械実装** (0 本 → 325 行) + 誤発火計測 rig の **fail-close**。7 files / +1438 -114 |

## レビューで検出したもの

### 🟡 Medium 2 件 (どちらも「その PR の欠陥ではない」)

**① #419 — 処理内訳の員数合わせで gate (a2) が fail-open** (PR #418 のレビューで検出)

- 正本 `output-format.md:119` が **Medium/Low の検出数を処理内訳の行数で持つこと**を許す
- gate `:1822` の `_DISPO_N` は record **全文**の `- 是正:` / `- 降格:` 行を**無差別に**数える
- gate `:1819` の `_CLOSED` は **Critical/High だけ**の差分

→ **Medium/Low の記録行が Critical の消化行として数えられる。**

**実 gate + 陽性/陰性対照で実測** (再実装でなく本物の検出器):

| ケース | 実 gate |
|---|---|
| 陽性対照: 内訳 0 行 / `_CLOSED=2` | **DENY** (測定器の生存証明) |
| 陰性対照: 正当な内訳 2 行 / `_CLOSED=2` | ALLOW |
| **Critical の内訳 0 行 + 無関係な Medium/Low の是正 2 行** | **ALLOW** |

縄張りが gate (A) と正本テンプレ (C) に跨るため 1 レーンで閉じない → 起票。

**② 4 本マージ後に残る旧名参照 3 件** (PR #423 のレビューで検出)

改名の掃きは #423 の base に対しては完全だったが、**他レーンの branch と縄張りには届かない**:

| ファイル | 縄張り | 理由 |
|---|---|---|
| `skills/review-harness/SKILL.md` | B | **B が新規追加した参照**。#423 の base に無かった |
| `references/step-4-5.md` | C | C が触ったが旧名を残した |
| `scripts/check-essence-sync.sh` | C | 縄張り外 |

→ **ゲート自身が PR #426 で掃いた** (下記)。

### 🔵 Low

- **#418**: 処理内訳が正本の 2 規則に跨る / PR 本文のファイル数誤り → **レーン B が検算して 2 件とも採用・修正** (`f2373ae`)
- **#422**: 検証表の `test-skill_args_contract.sh` 行が「HEAD 33/1 fail」だが実測は **4 箇所すべてで 34 passed / 0 failed**。開発中の一時状態を HEAD 列に置いた誤ラベル
- **#423**: 「要 issue 化」と書いた `grounds_gate` の rename 検出問題が未起票 → **#425 で起票**
- **#420**: **未開示の欠陥は検出できなかった**

## レビューの検証方法 (本波で確立した型)

**「装置を作った」PR には変異テストを当てる。** 主張を読むのでなく、壊して赤くなるかを測る。

| PR | 変異テスト | 結果 |
|---|---|---|
| #420 | `viol()` を 1 トリガーずつ黙らせる (A-1 / A-2 / B-2 / B-3 / C-1 / C-2 / C-3a) | **7 件すべて赤へ反転**、復元で 34/0。空振り assert ゼロ |
| #420 | rig を base 版 (177 行) へ差し戻す | **8 passed / 26 failed** → 改修版 34/0 |
| #422 | `validate-verdict-consistency.sh` を HEAD へ戻す (M1) | **18 assert 落下** (PR の主張と完全一致)、往復 346→328→346 |
| #422 | #409 の 4 ケースを自作 fixture で実挙動確認 | (a) emoji 単独 base rc=0→PR rc=1 / **(b) ラベル単独+不整合 base rc=0 skip→PR rc=1** / (c) 整合は rc=0 (狭めすぎ無し) / (d) 語順の罠で理由が偽→正確 |
| #423 | base と PR を**同一条件の temp 展開**で gate テスト比較 | base **231/5** → PR **250/5**、**落ちる 5 件は完全同一** = 新規の赤ゼロ |

**レーンの木は 1 バイトも触っていない** — `git archive` で PR の commit を temp へ展開し、そこで変異させた。

## ゲート自身の後始末 PR (#426)

**旧名参照 3 件を掃いた。名前だけでなく行番号も同じ commit で是正** — gate は `+232/-94` で伸びており引用行番号が全部動いていた (`:1717`→`:1855` / `:1884`→`:2022`)。名前だけ直すと別クラスの stale を作る。

### ⚠ この commit は essence gate を通っていない

**hook は Bash 実行の**前**に発火し `git rev-parse --show-toplevel` を**その時点の cwd** から撃つ**。マージゲートのセッションは cwd が別プロジェクトに固定されており、コマンド内に `cd <worktree>` と書いても hook からは見えない。実際に 2 回 fail-closed で止められた:

1. `$R` (未展開のシェル変数) を引数に → `committer 引数を repo 相対へ解決できず`
2. `cd <別ツリー> && committer` → `essence-sync 検証を完了できませんでした (exit 2)`

**`SKIP_ESSENCE_GATE=1` は使っていない。** 代わりに gate の 6 段を手で回した:

| 段 | 手動代替の結果 |
|---|---|
| (a)(a2)(a2-2) | `validate-verdict-consistency.sh` **rc=0** / 残存 0-0 / detected 0-0 / 内訳行 0 (要求 0) |
| (c) | full run **77 file(s) passed / 0 failed** (assert 2859) |
| (d) | `check-essence-sync.sh` **rc=0** |
| (f) | `test-fork-scan-contract.sh` **361 / 0** |
| (g) | `test-orchestrator-scripts.sh` **346 / 0** |
| (h) | `launch_form_sync.sh` **正本と全複製 4 箇所が byte 一致** |

**限界も record と PR 本文へ書いた**: 「gate が通った」ではなく「各段を手で回した」。**手動経路は回し忘れを機械が検知しない。**

## 本波で判明した機構上の穴・盲点

1. **`Closes #N` をバッククォートで囲むと GitHub の自動 close が発火しない** — PR #422 が `` `Closes #416` `` `` `Closes #409` `` と書いており、**2 本とも close されなかった**。#409 は手動 close、#416 は**部分完了ゆえ open 維持**が正しかった (規律「部分完了なら `Refs`」に照らすと `Closes` 自体が不適切だった)
2. **交差 0 でも横断矛盾が出るのは 3 波連続** — ファイル交差は全 6 ペアで 0、`merge-tree` も conflict 0 だったが、**参照の意味は跨った** (旧名参照 3 件)
3. **gate テストの結果は場所依存** — 素の temp 展開では **231/5**、実ハーネスツリーでは **236/0** 前後。レーン C 報告 (232/4) とゲート実測 (231/5) が assert 2 件ぶんずれた。**「N 件」と確定形で渡さず測定条件を添える**必要がある
4. **worktree の置き場は `gtr new` を打つ端末の cwd で決まるが、Bash ツールの非対話シェルでは `cd ~` が `chpwd` を再計算しない** — ゲートの sweep worktree が**系統 3** (`~/dev/claude-code/.worktrees/claude-code-learn/`) へ落ちた。手順正本 §1 が警告する形を実地で踏んだ

## 縮退 (無音で品質を食わせないため明記)

**本波のレビューは 1 段のみ。2 段目 (別の fresh reviewer による反証) を回していない。** 本セッションは agent 起動を禁じられていたため。

PR 側の独立性も揃っていない:

| PR | essence レビューの独立性 |
|---|---|
| #418 | `independent_review: none` (Lead 単独) |
| #420 | `fork_invoked: 0` / `independent_review: false` |
| **#422** | **3 領域 fork を実走** (self-eval v15。High 4 検出 → 3 件是正 / 1 件を #421 へ降格) |
| #423 | **harness / skill の 2 fork が独立に同一 High 2 件を検出**し是正 |
| #426 | `fork_invoked: 0` (ゲート自身) |

= **#418 / #420 / #426 は PR 側もレビュー側も自己申告の域を出ていない。**

## 出典

- 第13波レーン指示書: `.claude/wave13-lane-payloads.md` (gitignored、349 行)
- 第12波ゲートログ: `.docs/logs/shared/2026-08-14_harness-issue-wave12-merge-gate-and-close.md`
- レビューコメント: `issuecomment-5287148767` (#418 初回) / `5288069881` (#418 追認) / `5288010330` (#420) / `5288205762` (#422) / `5288433633` (#423)
- issue 本文: `gh issue view <N> -R dendedev/claude-harness`
