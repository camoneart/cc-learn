---
type: work
date: 2026-08-14
scope: dendedev/claude-harness 第14波 (issue 5 本 / 4 レーン) のマージゲート
base: 9089726
head: b8dfadf
---

# 第14波マージゲートのログ — 4 レーン 5 件完走 (マージ 4 本・issue 4 本 close)

## 結果 (確定値。すべて 2026-08-14 に実測)

| 項目 | 値 |
|---|---|
| マージ | **4 本** — `5ccad3c` (#427) / `4224704` (#433) / `e505218` (#434) / `b8dfadf` (#436) |
| base → 現在 | `9089726` → **`b8dfadf`** |
| close した issue | **5 本** (#424 #419 #421 #425 + **#416**。全て `state=CLOSED` `reason=COMPLETED` を実測) |
| 新規起票 | **6 本** (#428 #429 #430 #431 #432 #435。全て OPEN を実測) |
| open issue | 10 → **11** |
| open PR | **0** |
| レビュー判定 | 4 本とも **🟢 GO** (1 段のみ。2 段目なし) |
| post-merge 全量 | **78 file(s) passed / 0 failed** (elapsed 656s、live で実測) |
| `run-all --list` | 77 → **78** |
| ゲート運用者 (Lead) の誤り | **7 件、すべて自己検知** |

### レーン配分と diff 規模

| レーン | issue | PR | diff | 縄張り |
|---|---|---|---|---|
| **A** | #419 | #433 | +333/-6 (3 files) | `hooks/hook_pre_commit_gate.sh` + その test |
| **B** | #425 | #436 | +509/-18 (3 files) | `hooks/hook_pre_commit_grounds_gate.sh` + その test |
| **C** | #421 + #416 D/E/F | #434 | +558/-42 (9 files) | `skills/essence-reviewing-orchestrator/**` + skill fork の抽出器 |
| **D** | #424 | #427 | +564/-0 (5 files) | `skills/orchestrating-team-development/**` |

**縄張り交差は 0 ファイル**、branch 総当たり 6 組 + `main × 各 branch` 4 本の merge も**全て衝突なし**。

## 本波の主題: 「測定器が対象を読めていない」class が装置側と運用者側の両方に在庫していた

第12波「装置が守ると宣言した条件で発火しない」の**一段下**が本波の通奏低音になった。
**装置が測っているつもりで何も測っていない**という形が、レーンの成果物・新規起票・そしてゲート運用者自身の作業に同時に現れた。

### 装置側 (レーンが検出・起票したもの)

| issue | 形 |
|---|---|
| **#430** | fork の機械シグナルが `head -N` で無音に切り詰められ、`SCAN-END` は完全走査を宣言する |
| **#432** | 色 emoji のラベル併記チェック (層D) が orchestrator の references 2 本を走査せず、`✅❌⚠️` も射程外 |
| **#429** | 走査契約を 4 トークンへ広げたのに、それを最初に読む 3 fork の SKILL.md が 2/3 トークンのまま |
| **#431** | `essence_resolve_skill_files` が非 SKILL.md を SKILL.md として通し、存在しない違反候補を製造する |

### 運用者側 (Lead が本セッションで踏んだ 7 件。すべて自己検知)

| # | 形 | 症状 | 検知経路 |
|---|---|---|---|
| 1 | `set -- $pair` (zsh は unquoted 展開を単語分割しない) | `git -C` が rc=128 → 「ファイル不在」と読みかけた | 健全性チェック |
| 2 | `for L in $LANES` (同上) | **stall 検知が丸ごと死んだまま「静か」に見えた** | armed 行の per-lane 表示 |
| 3 | `PATH=/usr/bin:/bin` で jq を外したつもり | `jq` は `/usr/bin/jq` にあり外れていなかった | 出力が対照と同一 |
| 4 | grounds gate の fixture にツリー判定マーカーを置かなかった | ゲートが素通りし改名前後とも ALLOW → **「新実装は正しい」と誤報告しかけた** | **陽性対照** (改名 + 新規行) |
| 5 | `merge-tree <base> <A> <B>` を `grep -c '^<<<<<<<'` | **必ず衝突するペアでも 0** → 「衝突 0」を handoff へ書いた | **陽性対照** (書いた後) |
| 6 | `$REF:skills/...` (zsh の `:s` 履歴修飾と衝突) | `git show` が別パスを引いて exit 1 | 大きく落ちたので即判明 |
| 7 | 「ブランチが動いたか」の基準 hash を**事後**に取った | **PR #427 の 5 つ目のファイルをレビューせず 🟢 GO を出した** | マージ後に新規ファイルを発見 |

**7 件のうち 2 件 (#4 / #5) は陽性対照が無ければ誤った確定形が成果物へ残った。#5 は実際に handoff へ一度着地し、後から是正した。**

### 装置がこの class を構造で塞いでいた例 — run-all の自己申告

post-merge の全量 run の出力そのものが、本波の主題の実装例だった (逐語):

```
run-all: 78 file(s) passed, 0 file(s) failed
run-all: 測定条件 ts=2026-08-14T15:44:37+0900 elapsed=656s cwd=/Users/camone/.claude
         (inside-HOME) tree=main-worktree toplevel=/Users/camone/.claude config-dir=unset worktrees=11
run-all: 台帳隔離 dir=/var/folders/.../hook-fire-ledger-test.KBmo3K owner=run-all
         captured=4 行 (この run で隔離先へ落ちた = live へ行かずに済んだ行数)
run-all: [注] worktrees は存在数であって同時実行数ではない。並走レーンの実行有無・
         既知の環境結合テストの状態は本行では観測していない (件数を転記する時はこの限界ごと添えること)
```

- **測定条件を自分で刻む** (`elapsed` / `cwd` / `tree` / `toplevel`)
- **「汚さなかった」を数で示す** (`captured=4 行`)
- **転記する人へ向けて自分の観測の限界を先に開示する** (4 行目)

「78 passed」だけを転記すると測定条件が消える。**装置が測定条件を出力に埋め込むことで、転記者の規律に依存しなくなっている。**

## ゲートの型に足した点

### 1. 指示書の既定処方が誤りで、レーンが実測で棄却した (第14波の最重要事象)

指示書 (`.claude/wave14-lane-payloads.md`) は #419 の既定処方を「**処方 2 = 「処理内訳」節で範囲を限定**」と定めた。
**レーン A はこれを 2 点の実測で棄却し、Lead が独立に追試して両方とも確認した**:

| 棄却の根拠 | Lead の独立検証 |
|---|---|
| ① **節限定でも閉じない** — 在庫 `2026-08-13_120415_*` は `_CLOSED=3` に対し**節内 14 行** | ✅ `high_detected=3` / `critical_count=0` / `high_count=0` → `_CLOSED=3`、節内 `是正\|降格` = **14 行** (High 明記は 3 行、残り 11 行は Medium/Low) |
| ② **正当な record を落とす** — 内訳を別見出しへ書いた record が 7 件 | ✅ **8 件**を検出 (母数の定義差。Lead 側は `_CLOSED=0` も含む superset) |

見出しの表記ゆれも指示書の想定より広かった (実測): `##` / `###` / `####` の 3 レベル × 前置語つき 6 変種が実在。

```
9  ### 処理内訳          1  #### 検出 3 件 (High) の処理内訳
2  ## 処理内訳           1  ### High 3 件の処理内訳
                        1  ### 検出 → 残存 の処理内訳
                        1  ## 検出 High 1 件の処理内訳
```

**指示書どおりに実装していたら、穴を塞げないうえに正当な record を 7 件 block するものが出荷されていた。**

共通規律 8「ゲートが出す修正案は『案』— 採否はレーンが自分の木で測って決める。**ゲートの主張も検証対象**」が、初めて**指示書の既定を覆す形**で機能した。

### 2. レビューは「主張の再現」で行う — base と new を両方叩く

4 本すべてで、PR の主張表を Lead が自分で再現した。

| PR | 再現した内容 |
|---|---|
| **#433** | base=ALLOW → new=DENY(a2) (穴が閉じた) / 正当な C/H 内訳は両方 ALLOW (締めすぎていない) / severity 無表記は両方 ALLOW (**残穴**) |
| **#434** | base は folded scalar の本文を 1 バイトも出さないのに `skipped=0`、new は継続行 2 本を出す / `SCAN-EMPTY` が空・継続行なしの 2 形で発火し plain scalar では出ない |
| **#436** | 改名のみ=ALLOW / 改名+新規行=DENY(新規 1 行のみ) / **層またぎ改名=DENY** |
| **#427** | 自作 fixture (壊れ JSON / 空ファイル / 不在) で **4/4 発火・無音ゼロ** / 変異1 で**きっかり 5 assert が赤** |

**PR の数値をそのまま転記せず、同じ測定を自分で撃つ。** #436 では 1 度目の再現が測定失敗し、**陽性対照が救った**。

### 3. 「納品して静か」と「死んで静か」を分ける — 監視装置の設計

`Monitor` ツールで PR 出現とレーン stall を監視した。初版は transcript の mtime だけを見ていたため、**納品を終えて待っているレーンを stall と誤報**した。

分類を 4 つに割り、stall 検知時に納品状態 (`commit=N 未commit=M PR=#N/OPEN`) を取りに行く形へ直した:

| 記号 | 条件 |
|---|---|
| 🟢 | 新規 PR |
| ⚪ | 無音 + **未 commit 0 + PR あり** = 納品済み待機 (stall ではない) |
| 🟡 | 無音 + **未納品** = 本物の疑い |
| 🔴 | **transcript が 1 本も読めない** = 監視自体が効いていない |

🔴 を別立てにしたのが肝。**「stall が 0 件」と「測れていない」を同じ沈黙にしない。**
これは #424 (レーン D の担当 issue) が塞ごうとしている穴そのもので、**監視装置を作る側が同じ穴を 2 回踏んだ** (上表の #2 と誤報)。

### 4. 起票前番号の裏取りを機械で行う

レーン C の commit message が `#429-#431` へ降格と書いていた。`grounds-not-approval.md` が名指しする「**採番前の issue 番号を書く**」Gotcha の踏みやすい箇所ゆえ、**3 本すべての実在を `gh issue view` で確認**し、**健全性対照として存在しない `#99999` がエラーになることも確認**した。

## レーンの成果 (要点のみ。詳細は各 PR body)

### レーン A / #419 — commit gate (a2) の員数合わせ fail-open

分母 (`_CLOSED`) は Critical/High 限定なのに分子 (`_DISPO_N`) が record 全文の `- 是正:` / `- 降格:` を無差別に数えていた。**Critical 2 件を「是正した」と申告しながら内訳を 1 行も書かない record が allow された。**

採った形は **negative filter** (Medium/Low と明示された行を員数から外す。severity 無表記の行は残す)。
- severity 語彙を 3 系統受ける (語形 / `[M-1]` ブラケット略記 / `[MEDIUM]` 大文字ブラケット)
- **裸の大文字 `LOW` は受けない** — `ALLOW` に部分一致して誤除外する。gate の record は allow/deny を語るので実際に踏むクラス
- 在庫 172 件を base 版 gate と本実装の両方へ食わせて全数突合、**判定変化 0 件** (陽性対照つき)

**副産物**: 独立レビュー 2 体が別の欠陥を検出して是正 — 段の見出しが `# --- (` 形式でない段があり、**(a) が (h) へ / (b) が (a2) へ合算されて「どの段も正しく測れていなかった」**。#414 の規模ラチェットが判定入力にしていた値そのもの。

**残穴を自分で起票** (#428): severity 無表記の内訳行で員数は今も埋まる。**Lead が独立に確認した残穴と同一。**

### レーン B / #425 — grounds gate の rename 検出

`added_lines()` が per-file pathspec で `git diff` を撃つため rename 検出が成立せず、**改名しただけのファイルの全行が「追加行」として再浮上**していた。

**実害は live 台帳に残っていた** (Lead が独立確認):

| 時刻 (UTC) | rule | 検出行 |
|---|---|---|
| 2026-08-14T00:25:00Z | `fabricated-grounds` block | `# 裁定: 2026-07-25 PR #238 マージゲートレビュー / 〈人名〉` |
| 2026-08-14T00:50:02Z | 同上 | 同上 |
| 2026-08-14T00:50:43Z | 同上 | 同上 |

健全性対照として台帳全体の `fabricated-grounds` は **51 件** (非空)。
**第13波レーン A は、改名と無関係な在庫 2 行の書き換えを強いられていた。**

処方は経路ごとに pathspec 抜きで 1 回 diff を撃つ形。untracked 経路では **一時 index (`GIT_INDEX_FILE`) で committer が載せる予定の index を先取り**する (本物の index は 1 バイトも触らない)。

**独立レビューが挙げた High 2 件を実測で処置分けした**:
- **層をまたぐ rename で検出が消える** → `.docs/logs/a.md` → `rules/a.md` で 旧=DENY / 新=ALLOW = **本変更が作った新規の穴** → **同 PR で修正** (Lead が DENY へ反転していることを独立確認)
- **部分 stage + `git commit -a`** → 旧=ALLOW / 新=ALLOW = **旧実装から在る既存の穴** → **#435 へ降格** (`AskUserQuestion` 実結果「開示 + 別 issue」)

### レーン C / #421 + #416 D/E/F — orchestrator の契約層

**#421**: 走査契約の 3 状態 (`SCAN-SKIPPED` / `SCAN-OK (0 件)` / 生出力) に**第4状態が無く**、folded scalar の description が「走査成立・結果あり」に見えていた。

処方は 2 段 (片方だけでは閉じない):
1. **抽出器側** — frontmatter の状態機械へ書き換え、folded / literal scalar の継続行を本体として拾う
2. **契約側** — `essence_empty()` → `SCAN-EMPTY: <指標> (対象は読めたが当該属性を抽出できなかった = 未検証。「0 件」と読まない)`

**`SCAN-EMPTY` を `SCAN-SKIPPED` へ畳まなかった理由**: 判定は同じでも**処方の向きが逆**。`SCAN-SKIPPED` = パス・配備を直す / `SCAN-EMPTY` = **抽出器を直す**。畳むと根本原因が永久に「配備の問題」として誤診される。

**後方互換**: `essence_empty` は `skipped=` を加算し `SCAN-END` の書式を 1 バイトも変えないため、4 トークン目を知らない消費者も fail-close 側へ倒れる。**実在する消費者を名指しした** (`skills/coordination-harness-integrity-fork/SKILL.md:195`)。

> self-eval の harness fork が「そのような消費者は在庫に無い」と主張したのを、**レーンが在庫を実測して反証**した (fork は 3 essence fork しか探していなかった)。**fork の主張も検証対象**という規律の実例。

**#416 D**: `references/*.md` に行数ラチェットを新設 (既存ラチェットは fork の SKILL.md 用で references を 1 本も見ていなかった)。**全数性つき** — 宣言に無い doc が 1 本でも在れば落ちる。pin は本 PR の改修後の値で、同 PR 内で 2 回上げた事実と理由を開示している。

### レーン D / #424 — Agent Teams の stall 検知

**issue のスケッチ案を実測で棄却した** (Lead が独立確認):

| 測ったもの | 値 |
|---|---|
| 全 `timeline.jsonl` の `"state":"failed"` | **0 行** (健全性対照 `"state":"working"` = 461 行で非空) |
| `962ac51d` の `state.json` vs `timeline.jsonl` 最終行 | `failed` vs **`working`** |
| `13a8fc90` でのヒット率 | **160 / 180 行 (89%)** = 通知過多で Monitor が自動停止 |
| `bce7fcde` の `timeline.jsonl` | **存在しない** → `tail -f` は即終了 = 無音 |

**捕まえたい事故を 1 行も拾えない**ため信号源を `state.json` のポーリングへ変えた。判定の主軸は **mtime (`STALE`)**。

**設計の要を Lead が実測**: `state.json` の mtime は稼働中 job で **1 秒前**、死んだ job で 846,252〜1,991,364 秒前。主軸の前提は成立。

**#427 の Medium 1 (テストが run-all 未配線) を同 PR 内で自力解決**した — `hooks/test/cases/test-orchestrating-team-jobs-scan.sh` を新設し `# run-all-trigger:` で守る対象 3 本を引き金にした。**重量帯に入らないこと (6s < 閾値 30s) を実測してから配線**し、固定 sleep 方式だった時の 13s から待ち合わせ方式へ直した経緯も書いてある。

## ⚠ Lead のレビュー漏れ 1 件 (自己検知・開示)

**PR #427 の 5 つ目のファイルをレビューせずに 🟢 GO を出した。**

| 測ったもの | 値 |
|---|---|
| Lead がファイル一覧を取った時点 | `changedFiles=**4**` |
| 現在 | `changedFiles=**5**` |
| commit | `f99001b` (本体) + **`be12b44`** (wrapper 新設) |

「ブランチが動いていないか」を hash で確認したが、**基準値として取った hash が既に push 後の `be12b44`** だったため差分を検知できなかった。

**マージ後に当該ファイルをレビューした結果は良好**だったが、**「レビュー済み」の射程を誤って報告した事実は残る**。

**教訓**: 「動いたか」を測るなら **レビュー開始時点の hash を先に記録**する。事後に取った hash 同士の比較は何も言っていない。

**当該ファイルへの指摘 [Low]**: 層2 のフラグ検査が `if grep -q "$flag" SKILL.md` 条件付きゆえ、SKILL.md からフラグ記述が消えると assert が fail でなく**不在**になる (vacuous pass 経路)。

## 未処理 (第15波以降へ)

### 1. L1 計上 (`rules/grounds-not-approval.md` の再発マーク)

対象は 3 波ぶん: **第12波 5 件 + 第13波 9 件 + 第14波 7 件 + レーン B の自己申告 1 件**。
**2026-08-14 の `AskUserQuestion` 実結果で「既存 bullet (2 つ目 = 測定器がその対象を実際に読めたか) の再発として計上」を選択済み。**
メインセッションからは `Edit(~/.claude/rules/**)` が deny ゆえ **worktree セッションが要る**。

### 2. worktree の撤収 (第13波 4 本 + 第14波 4 本)

第14波ぶん = `issue-419` / `issue-425` / `issue-421-416` / `issue-424`。Keychain 払い出し記録: `1772d6d2` / `e0a73017` / `4c2d48f7` / `3356abfb`。
第13波ぶん = `issue-406` / `issue-408-415` / `issue-414` / `issue-416-409`。記録: `d1a22425` / `ddd604d7` / `805272f3` / `322c96c5`。
**セッションが cwd を掴んでいる間は `gtr rm` しない。** Keychain 回収 → `gtr rm` の順 (手順正本 §4-1)。

### 3. 新規 6 本の配分

#428 (#419 の残穴) / #429 #430 #431 (#421 の消費者側・残クラス) / #432 (層D の走査漏れ) / #435 (部分 stage の既存穴)。
**#429 #430 #431 #432 は「測定器が対象を読めていない」class で、本波の主題の直系。**

## 追記: #416 の close (ログ commit 後に実施)

**ゲートが 1 件取りこぼしていた。** PR #434 は `Closes #416` でなく `Refs #416` を書いていた (着手時点で C の扱いが未確定だったため) ので自動 close が発火せず、#416 が open のまま残っていた。

main (`b8dfadf`) で 6 項目すべてに処置が付いていることを実測して close した:

| 項目 | 状態 | 実測した根拠 |
|---|---|---|
| A / B | landed | PR #422 (`9d97758`) |
| C | 見送り (記録済み) | 1R の `AskUserQuestion` 実結果「1 ファイル維持 + 免責記録」 |
| **D** | **landed** | `scripts/test-orchestrator-scripts.sh:1892` にラチェット実在 |
| **E** | **landed** | `references/step-6.md` が **189 → 186 行**へ縮小 |
| **F** | **landed** | `references/output-format.md:114` に「既定は 1 つ (issue #416 F で一本化)」 |

痕跡: `https://github.com/dendedev/claude-harness/issues/416#issuecomment-5290460248` / `closedAt=2026-08-14T06:58:32Z`。

**確定値の訂正**: 上の結果表の「close した issue」は **4 → 5 本**、「open issue」は **12 → 11** へ是正済み。

**教訓**: **`Refs #N` の issue は自動 close が発火しないので、マージ後に残スコープを測って手で閉じるかを判断する。** 本波はこれを 1 度取りこぼした。

## 出典

- 第14波レーン指示書: `.claude/wave14-lane-payloads.md` (gitignored、497 行)
- 第13波ゲートログ: `.docs/logs/shared/2026-08-14_harness-issue-wave13-merge-gate-and-close.md`
- PR / issue 本文: `gh pr view <N> -R dendedev/claude-harness` / `gh issue view <N> -R dendedev/claude-harness`
