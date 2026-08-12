# ハーネス issue 第10波 マージゲート + クローズ (2026-08-12 マージ完了 / 同日 15:30 に事後記録)

- 対象: dendedev/claude-harness 第10波 **5 レーン**。マージ 5 本 = #392 (`72021e5`) → #389 (`51c9f64`) → #390 (`43a6c0c`) → #391 (`169ea09`) → #393 (`8c3b76f`)。**issue 23 本 close**、マージ後の open PR = 0 / open issue = 19
- 由来: 第9波の繰越 + 第9波が生んだ follow-up (#383 #385 #386) + 積み残しの essence-gate 系
- 新規起票 2 本: **#394** (テストが live の hook-fire ledger を汚す) / **#395** (essence-gate の (f)/(g) が「テストの在処」で発火し「検査対象」で発火しない)
- ゲートのラウンド数: **2** (第9波は 3)

## ⚠ 本ログの位置づけ — 第10波はゲート結果を GitHub へ転記していない

第9波までは 1 段目/2 段目の判定を PR コメントへ転記し、`issuecomment-*` の表をログに残していた。**第10波はそれをやっていない。** 本ログ執筆時の実測:

| 経路 | 実測 |
|---|---|
| PR #389〜#393 の review | **0 件** (5 本すべて) |
| PR #389〜#393 の comment | **1 件のみ** (#390 の `issuecomment-5260602139`。しかもゲート発ではなく**レーン発の返信**) |
| issue #394 / #395 の comment | **0 件 / 0 件** |
| scratchpad 成果物 (`VERDICT.md` / `SENDBACK-all-lanes.md` / `pr-{389..393}.md` / `verify-*.md`) | **全滅**。`~/.claude` / `~/.worktrees` / `~/.claude-worktrees` / cc-learn ツリー / `/tmp` / `/var/folders` を走査してヒット 0 |

つまり第10波のゲート判定の一次記録は **`.claude/handoff-state.md` だけ** で、それは cc-learn で **gitignored**。本ログが**唯一の Git 追跡された痕跡**になる。

**そのため本ログは出所を 2 系統に分けて書く**:

- **(実測)** = 本ログ執筆時 (2026-08-12 15:30 前後) に GitHub / git で測り直した値
- **(handoff 由来)** = 一次記録が消えており**再測不能**。`.claude/handoff-state.md` の記述をそのまま引く

## マージ 5 本 (実測)

| PR | merge commit | マージ時刻 (UTC) | branch |
|---|---|---|---|
| #392 | `72021e5` | 2026-08-12T00:43:34Z | `issue-364-280-362` |
| #389 | `51c9f64` | 2026-08-12T00:43:37Z | `issue-322-324-339-388-312` |
| #390 | `43a6c0c` | 2026-08-12T00:43:41Z | `issue-320-323-326-275-321` |
| #391 | `169ea09` | 2026-08-12T06:08:03Z | `issue-383-386-387-297-314` |
| #393 | `8c3b76f` | 2026-08-12T06:08:06Z | `issue-276-277-278-279-313` |

第 1R で 3 本 (00:43)、第 2R で 2 本 (06:08)。

## handoff の記述と実測が食い違った点 3 件

**この 3 件は「一次記録が消えると、確定形で書いた数値が誰にも検算されないまま残る」実例。** 本ログを書くために測り直してはじめて出た。

### (1) 「close した issue 23 本 (全件 COMPLETED)」→ 実測は **22 COMPLETED + 1 NOT_PLANNED**

`#280` だけ `NOT_PLANNED` で、close 日も他の 22 本 (08-12) と違い **08-11**。中身は「essence-gate 1053 行を `hooks/lib/gate_<stage>.sh` へ分割する」で、レーン① (#392) が「**分割せず条件付き免責**」を選んだ結果の意図的な not-planned。

皮肉なことに handoff 自身の「測定器の失敗」表の 2 行目が **「#280 の close 状態を測らずに『未実施』と確定形で書いた (実際は NOT_PLANNED)」** と自己申告している。**同じファイルの中で、上の実績欄は『全件 COMPLETED』のまま直っていなかった。**

### (2) 「測定器の失敗 8 件、うち 4 件がメイン」→ 同じ表を数えると **メイン 5 件 / 2 段目 3 件**

handoff の表は行 1・2・3・4・8 の 5 行がメイン、5・6・7 の 3 行が 2 段目。**見出しの「4 件」が自分の表と合っていない。**

### (3) 「構造的に書込不能」クラスの narrow 値 (16 行 / 8 ファイル) が**どのツリーでも再現しない**

handoff は次のように記録している:

```
skills/ narrow (逐語「構造的に書込不能」) = 16 行 /  8 ファイル
skills/ broad  (構造的にも不可・構造保証・structural guarantee 等) = 24 行 / 12 ファイル
agents/ broad                             = 20 行 / 16 ファイル
```

本ログ執筆時に **逐語** `構造的に書込不能` を第10波の commit **23 点**で測った結果 (走査集合 = `git log origin/main -30` の後方 20 点 + base `29f1ac2` + head `origin/main` + マージ commit `169ea09`。この 23 点で**マージ commit 5 本すべてを含む**):

| ツリー | skills/ 逐語 |
|---|---|
| base `29f1ac2` および第10波の大半 | **24 行 / 12 ファイル** |
| head `origin/main` (`8c3b76f`) ほか | **25 行 / 13 ファイル** |
| 16 行 / 8 ファイル | **一度も出ない** |

しかも 24/12 の内訳は **全件が fork skill の `SKILL.md`** (`llm-debate-*` 5 / `*-essentials-reviewer-fork` 3 / `gep-*-fork` 2 / `framing-advocate-*-fork` 2)。つまり handoff が **broad に割り当てた 24/12 が、逐語 (narrow) の実測値と一致する**。

原因は特定できない (scratchpad 消失)。**確実に言えるのは、broad の語彙集合が `等` で閉じておらず再現不能だったこと。** handoff 自身の検証の型に「**確定形の数値には測定条件 (何を含め何を除外したか) を添える**」が入っているのに、その条項を書いた当の数値が再現できない。

なお **live の反例**は今も物理的に実在する (実測):

```
agents/code-reviewer.md:36
  修正は実行しない (提案のみ)。tools から Edit/Write/Skill を除外済みで構造的にも不可
```

この定義で動いた第10波の 2 段目レビュアーが Bash でファイルを新規作成した (handoff 由来)。

## ゲートの構成と結果 (handoff 由来)

| 段 | 構成 | 結果 |
|---|---|---|
| 機械層 | メインが実施 | merge-base 5 本とも一致 / merge-tree conflict 0 / **ファイル交差 0** (76 ファイル) |
| 1 段目 | PR ごとの fresh `code-reviewer` を名前なし background で 5 体並列 | Critical 0 / **High 9** / Medium 27 / Low 24 |
| 2 段目 | 別の fresh agent 2 体が**指摘の反証**を試行 | **CONFIRMED 9 / REFUTED 0 / OVERSTATED 0 / UNVERIFIABLE 0** + 新規 High 3 |
| 1R 判定 | | 3 本マージ可 / 2 本 NO-GO |
| 2R | 各レーンが確定 High 12 件 (9 + 3) を処置 | 全緑 → 残り 2 本もマージ |

**第9波との最大の違い: 2 段目が 1 段目を一件も覆さなかった** (第9波は事実誤り 6 件を撤回・格下げ)。反証を試みて CONFIRMED 9/9。

## 本波の重い発見

### 1. 統合してはじめて出る「レーン間の設計矛盾」が 2 種類あった (handoff 由来)

**(a) レーン③ × レーン①**: レーン③ の assert が「tracked かつ 500 行超の全ファイルが BASELINE」を要求。一方レーン① の record は「行が増えて REGRESSION になるのは**仕様**」。両立しない。レーン③ が「repo の在庫を見ない前後比較」へ設計ごと差し替えて解消。

**(b) レーン⑤ × レーン②**: レーン⑤ が書いた実測主張をレーン② が同じ波で反証。しかもレーン⑤ の段落がレーン② の書き換えた 3 ファイルを「正本」と名指ししていた。**変更ファイル集合が素なので `merge-tree` は conflict ゼロで通り、1 段目の 2 人はどちらも構造的に気づけない。** レーン⑤ が撤回して決着。

### 2. ⚠ ゲート自身の盲点 — run-all の射程が repo の全テストを覆っていない (実測で独立確認)

本ログ執筆時に `origin/main` (`8c3b76f`) の木で測り直した:

```
repo 内の test-*.sh 総数            : 75
うち hooks/test/cases/test-*.sh     : 69  ← run-all の既定射程 (run-all.sh:49 `CASES="$DIR/cases"` / :216 `for t in "$CASES"/test-*.sh`)
hooks/ 配下で cases 以外の test-*.sh: 0
→ 射程外                            : 6 (全件 skills/ 配下)
```

射程外の 6 本 (パス逐語一致を確認):

```
skills/essence-reviewing-orchestrator/scripts/test-orchestrator-scripts.sh
skills/essence-reviewing-orchestrator/scripts/test-fork-scan-contract.sh
skills/accumulating-reviewer-feedback/scripts/test-accumulate-scripts.sh
skills/establishing-knowledge-persistence/scripts/tests/test-l2-gates.sh
skills/enforcing-strict-tdd-cycle/scripts/tests/test-runner-detect.test.sh
skills/enforcing-strict-tdd-cycle/scripts/lib/test-runner-detect.sh
```

**メインが第10波を通じて出してきた「N files passed」は、この 6 本を一度も走らせていなかった** (handoff 由来)。最終判定では 6 本すべて個別実走して全緑を確認した (handoff 由来。**本ログ執筆時にテストは再実行していない** — 確認したのは射程の構造だけ)。

**issue #395 が名指ししたのは 2 本だけで、残り 4 本はメインが repo 全数走査で見つけた。**

第11波で「run-all の射程を repo 全体へ広げるか、射程外を明示的に別経路で回すか」を決めること。

### 3. 測定器の失敗が 8 件 (メイン 5 / 2 段目 3 — 上記 (2) の訂正後) (handoff 由来)

| # | 誰 | 内容 | 検知 |
|---|---|---|---|
| 1 | **メイン** | branch 衝突検査を `refs/heads/` だけで撃ち remote-only を見落とす形。派生で「在庫 175 / うち 39」も分母と分子の走査範囲が食い違い | probe-before-persist |
| 2 | **メイン** | #280 の close 状態を測らず「未実施」と確定形で書いた (実際は `NOT_PLANNED`) | probe-before-persist |
| 3 | **メイン** | `grep -m1 "^tools:"` で agent の tools を測ろうとした。agents 32 本中 **26 本が LIST 形**ゆえ **Bash 持ち 21 本を丸ごと no-bash と誤判定**する形 | 自己検知 → 2 段目が原因特定 |
| 4 | **メイン** | 1 つ前のツリー (r2b) の結果を現 head の結果として報告しかけた | 自己検知 |
| 5 | 2 段目 (#390) | `CM="commit""er"` が `commiter` になっており、その「ALLOW」5 件は trigger 未発火の空振り | 自己検知・破棄・再測 |
| 6 | 2 段目 (#390) | `timeout` 不在で run-all 即死。「ledger 増分 0 行」は測定器が動いていない結果 | 自己検知・破棄 |
| 7 | 2 段目 (#391/#393) | 未回収 provision を 53 件と算出しかけた (`reclaimed` 226 行中 48 行が `path` を持たず `hash` のみ)。`hash` 基準で 5 件 | 自己検知 |
| 8 | **メイン** | `timeout 300 bash` で exit=127 (macOS に `timeout` 不在)。テスト失敗と読みかけた | 自己検知・撃ち直し |

**メインの推測違反 3 件も Stop hook が捕捉**した (「かもしれない」「可能性がある」「だと思う」)。うち 1 件は測り直して**推測が外れていた**ことが確定 (#391 と merged main の食い違いは実測ゼロ)。

## 環境の後始末 (実測 — 本ログ執筆時)

- **第10波の worktree 0 本** (`git -C ~/.claude worktree list` に第10波分なし)
- 残っているのは 2026-07-24 由来の essence 系 2 本 (`essence/harness-gap-compaction-boundary` / `essence/skill-gap-evaluation-grounded-authoring`、いずれも `1bcb0ca`)。第10波と無関係で触らない判断済み
- **Keychain 孤児 0 / 帰属不能 0** (handoff 由来)
- `~/.claude-worktrees/` に **probe の残骸 3 ファイル**が残置 (`probe_glob_outside.txt` / `probe_home_control.txt` / `probe_readctl.txt`、いずれも 08-12 02:00〜02:19)。dir ではないので殻検知に掛からない。**ゲートのレビュアーが残した副作用**

## リポジトリ状態 (実測 — 本ログ執筆時)

| repo | HEAD | vs origin | 備考 |
|---|---|---|---|
| cc-learn | `f3ce517` | **0 ahead / 0 behind** | 第9波ゲートログは push 済み (handoff の「ahead 1 未 push」は解消) |
| claude-harness | local `29f1ac2` / origin `8c3b76f` | **0 ahead / 40 behind** | ff-only merge が止まる |

**40 behind の原因は 1 ファイル**: `CLAUDE.md` が origin 側で `f77e29d` (Stack 排除宣言 #324 #339) により変更され、ローカルでも未 commit の変更 (ペルソナ差し替え) がある。残る未追跡 20 本 (`.docs/logs/local/` のレーン記録) は **origin 側で同パスの変更が 0 件**ゆえ merge の障害ではない。

**この状態で `git clone ~/.claude` すると clone の base も `29f1ac2` になる** (第10波で実測済み)。統合ツリーは GitHub から直接 clone すること。

## 第11波の在庫 (open 19 本 — 実測)

- **essence-gate 系 7 本**: #304 #306 #316 #319 #336 #337 #385 — レーン① (#392) が「分割せず条件付き免責」を選んだので**同一ファイルに集中したまま**。**依然として同一レーンへ束ねる必要がある**
- **本波の新規 2 本**: #394 (ledger 汚染) / #395 (essence-gate の (f)/(g) 発火条件)
- **繰越 6 本**: #273 #305 #325 #340 #342 #345
- **Apple レーン 4 本**: #218 #219 #224 #226

## 次波へ持っていく設計

1. **ゲート結果を GitHub へ転記する** — 第10波はこれを飛ばし、判定の一次記録が消えた。第9波の `issuecomment-*` 表の形へ戻す
2. **run-all の射程問題を決着させる** — 射程 69 / repo 全数 75。広げるか別経路で回すかを第11波の頭で決める
3. **2 段目は省かない** — 第10波は覆しゼロだったが、それは「反証を経て残った指摘は強い」の実証であって、省く理由にはならない
4. **統合ツリーは毎ラウンド作り直し、レーンの head が動いたら測り直す** (第10波で 1 つ前のツリーの結果を報告しかけた)
5. **確定形の数値には語彙集合と走査範囲を閉じて添える** — 「〜等」で開いたまま書くと本波の (3) のように再現不能になる
6. **指示書は各 worktree の `prompt/<name>.md` へ置き、貼るプロンプトは「それを読め」の 3 行にする** (第11波からの新方式。`~/.claude` の `.gitignore:2` が whitelist 方式ゆえ `prompt/` は自動で追跡外)

## 実測で確定している注意点 (第10波で足されたもの)

- **`~/.claude` を clone すると clone の base は local main になる** — local が behind なら統合ツリーの base が古くなる。GitHub から直接 clone するのが確実
- **`gtr rm` が殻 dir を残すかは非決定的** — 第10波は残らなかった。毎回測ってから判断する
- **worktree を掴むのは claude だけとは限らない** — 撤収条件は「worktree 内に居座る **claude** が無いこと」。zsh は残っていてよい
- **Keychain 回収コマンドは逐語で 1 件ずつ** (`for` ループにすると承認画面に「何を消すか」が映らない)
- **macOS に `timeout` コマンドは無い** — 使うと exit=127 になりテスト失敗と誤読しうる
- **gh の issue/PR コメント本文は必ず `--body-file`** / **commit message に保護パス token を含む時は `git commit -F <msgfile>`**

## 出典

- 第7波ゲートログ: `.docs/logs/shared/2026-08-09_harness-issue-wave7-merge-gate-and-close.md`
- 第8波ゲートログ: `.docs/logs/shared/2026-08-10_harness-issue-wave8-merge-gate-and-close.md`
- 第9波ゲートログ: `.docs/logs/shared/2026-08-12_harness-issue-wave9-merge-gate-and-close.md`
- セッション運用ログ (第7・8波): `.docs/logs/shared/2026-08-10_wave7-wave8-merge-gate-session.md`
- 第10波の一次記録 (gitignored・本ログの (handoff 由来) の出所): `.claude/handoff-state.md` / 指示書は `.claude/wave10-lane-payloads.md`
- worktree 隔離・撤収・Keychain 回収の手順正本: `~/.claude/.docs/progressive-disclosure/harness-worktree-isolation.md`
- 本波の issue / PR: `gh issue view <N> -R dendedev/claude-harness` / `gh pr view <N> -R dendedev/claude-harness --comments`
