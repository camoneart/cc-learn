---
date: 2026-08-08 09:29:21
type: study
topic: e-3-nondeterminism-last-deepdive
session: E-3「非決定論性は最後に活用する」単独深掘り (取り入れフェーズ第19弾 — E-3 の個別判定としては初回)
related_article: .docs/references/260405_【「なぜか微妙」は偶然ではない】Claude Codeで学ぶ 7つの構造的制約とハーネス設計、AIエージェントの本質論/text.md (E-3 = 2271〜2309行、関連: E-2 2201〜2269 / 巻末設計チェックリスト開始 2311 / Steinberger 可視化引用 2295)
related_skill: [harness-adoption-audit, explain-in-html, logging]
related_log_ids: [2026-05-30_note-harness-gap-analysis, 2026-08-08_e-2-reason-based-generalization-deepdive]
related_log: [.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md, .docs/logs/shared/2026-08-08_e-2-reason-based-generalization-deepdive.md]
---

# E-3「非決定論性は最後に活用する」単独深掘り — 判定: 取り入れ済み (原則が essence 正本に逐語存在・決定論ファースト検査 5 項目中 4 項目に機構実在・fan-out は明示呼出 + HITL マージで抑制) · 残差 = **並列 agent の稼働可視化 [Medium]** (roster は在るが停止時刻・露出・subagent 掲載が欠ける — 実害を本セッションで観測: 6h45m 放置 + 報告不達 3 体) + 回収・停止規律の未文書化 [Low] · 特記 = **親バッチに E-3 の個別判定が存在せず、本監査が初回判定**

> 核心の構造事実: **E-3 の原則はハーネスの essence 正本に逐語で存在する** — `agent-essence.md` L165-167「同一仕様で並列実行して最良を選ぶ戦略は有効だが、**まずハーネスで失敗率を下げること。並列実行は品質管理の代替ではない。**」(末文は記事 2309 行と同一文)。記事の「決定論ファーストの実践的チェックリスト」5 項目は **4 項目に実体がある**: ①テストスイート + 数値測定 = `hooks/test/` 60 cases + `run-all.sh` の pass/fail 数値出力 ②lint 自動実行 = PostToolUse lint hook 7 本 ④隔離 = worktree 規律一式 (rule 節 + 41KB の隔離手順正本 + gtr skill 3 本 + write guard hook 2 本) ⑤客観的選択基準 = GO/NO-GO/CONDITIONAL 判定語を持つファイル 18 本。**欠けているのは③「単一実行を 10 回試し 8 回以上」型の成功率サンプリング計測のみ** (陰性・対照実験つき) — ただしハーネスの並列は「同一仕様 N 候補から最良を選ぶ」型でなく**役割分離・視点分散型** (6-role debate / 5-lens / 2-frame) が主で、best-of-N 選抜の前提自体が稀ゆえ [Low]。fan-out の 4 条件は、客観基準・隔離・**自動マージしない** (HITL 明文) が機構であり、「単一成功率が十分高いか」のゲートは機械でなく**高コスト fork の明示呼出専用** (人間が起動を裁定) が代替する。**最大の残差は Steinberger の指摘そのもの** — 「並列の難しさは実行でなく管理。可視化がなければカオス」に対し、ハーネスの並列 agent 稼働観測は実運用上 idle 通知 (「情報量ゼロ」と自己文書化済み) + session 終了時 UI の目視に依存する。**永続 roster (`teams/*/config.json`) は実在するが、停止時刻・露出導線・Task 型 subagent の掲載が欠ける** (独立検証が初版の「台帳なし」を反証し、残差はこの 3 点へ再構成)。**実害は本セッションで観測済み**: scan agent 2 体の 6h45m 放置 (Dende のスクショ指摘で発覚) + 報告不達 3 体 (transcript 直読みで回収)。永続痕跡 = 本 PJ handoff frontmatter の next_phase 記載 (2 箇所、grep 実測)。

## 概要

取り入れフェーズ第19弾。**親バッチ (`2026-05-30_note-harness-gap-analysis.md`) に E-3 の個別判定が存在しない** — `E-3|並列|fan-out` の grep が 0 件 (対照: 同ファイルで「制約カスケード」1 件ヒット = 検索器は健全)。E 章「大半✅」の根拠列挙は全て E-1/E-2 のもので、**E-3 は個別根拠なく章まとめに包含されていた**。よって本監査は差分深掘りでなく初回判定。

実測 (step3) は read-only `Explore` 1 体 (`scan-e3-determinism`) へ委譲。**運用の学び**: Explore 型は Write 非搭載でファイル報告が構造的に不可 (agent 自身が制約を自己申告し本文報告に切替) — 以後、scan (Explore) は本文報告・gate (code-reviewer) はファイル報告と様式を分ける。判定に使う数値は全てメインで撃ち直し、**scan の誤帰属 1 件を検出** (下記)。

## 内容

### note 側の定義 (E-3 = 2271〜2309 行)

一行サマリー (2273 行): 並列実行で「当たり」を引こうとする前に、まず単発の成功率を上げる。設計パターン「決定論ファースト + 並列探索」: **Phase 1** = ハーネスで矯正 (V-1)・制約カスケード (E-1)・理由ベース指示 (E-2) で 10 回実行して 8-9 回は許容可能な状態にする (2289 行) / **Phase 2** = 安定したら Fan-out。並列の 4 条件 (2291 行): (1) 客観的な選択基準 (2) 単一成功率が十分に高い (3) 各インスタンスが隔離されている (4) 結果は「候補」であり自動マージしない。Steinberger の引用 (2295 行): **並列実行の難しさは「実行」ではなく「管理」にある。可視化の仕組みがなければカオスになる**。決定論ファーストの実践的チェックリスト 5 項目 (2297〜2307 行)。締め (2309 行、逐語): **「並列実行は品質管理の代替ではない。品質管理の上に成り立つ、最後の最適化手段だ。」**

### ハーネス実体の対応表

| 記事の要素 | ハーネスの実体 (実測) | 状態 |
|---|---|---|
| E-3 原則そのもの | `agent-essence.md` **L165-167 に原則 ID「E-3」として逐語存在**。末文「並列実行は品質管理の代替ではない」は記事 2309 と同一文。順序規律「まずハーネスで失敗率を下げること」も同節に含まれる | **取り入れ済み** |
| チェックリスト① テスト + 数値測定 | `hooks/test/cases/` = **60 本** + `run-all.sh` が pass/fail をファイル単位の数値で出力 (L196-212 実読)。なお記事①の「成功率」は確率計測を含意しうるが、ハーネスの実体は**決定論テストの binary pass 率** — この読み替えは③の陰性と対で開示する | **実在** |
| チェックリスト② lint/型の自動実行 | PostToolUse (`Edit\|Write\|MultiEdit`) の lint 系 hook **7 本** (prettier+eslint / emoji / fork矛盾 / 500行 / bang自己参照 / frontmatter / locale照合) | **実在** |
| チェックリスト③ 単発成功率の計測 (10 回試行 8 回) | **現役の rules/ + skills/*/SKILL.md に無い** (同スコープのヒット **13 件**は全て別文脈。`agents/` を足すと 14 件目 = coder.md:31 でこれも別文脈。対照: `essence-for-implementer:43`「成功率は積で劣化」がヒット = 検索器は健全)。**最も近い実装は `skills-disabled/empirical-prompt-tuning`** (L29-47 に成功/失敗 2 値 + 精度% + 再試行回数の測定ループ) だが**無効化済み** | **陰性 [Low]** (下記) |
| チェックリスト④ 隔離 (worktree) | `multi-agent-safety.md` の worktree 節 (削除条件・回収・起動形) + `harness-worktree-isolation.md` (41KB の手順正本) + gtr 系 skill 3 本 + worktree write guard hook 2 本 | **実在 (最厚)** |
| チェックリスト⑤ 客観的な比較・選択基準 | GO/NO-GO/CONDITIONAL の判定語を持つファイル **18 本** (`grep -rlE "GO/NO-GO\|NO-GO" skills/ agents/ --include=*.md`)。verdict は数値 (critical/high count) 付きの機械可読形式 (essence-review-records) | **実在** |
| fan-out 4 条件の (4) 自動マージしない | `proposing-essence-updates/SKILL.md:7`「④レビュー・⑤マージは HITL (skill 外、**自動マージしない**)」+ `essence-reviewing-orchestrator/references/essence-summary.md:18`「判断基準の正しさは現時点で人間にしか保証不能、**AI 自動マージ禁止**」(検証者が発見したより的確な明文)。なお「自動マージで共存できる」という記述も別所に在るが、それは git のファイルマージ (非重複キー領域) の話で、agent 出力の採用可否とは別対象 | **明文** |
| fan-out 4 条件の (2) 単一成功率ゲート | 機械化されていない。代替 = **高コスト fork の明示呼出専用** — `llm-debate`「明示呼出専用…自動誘発なし」/ `detecting-framing-bias`「明示呼出専用 (高コスト fork 起動)・自動誘発なし」/ `essence-reviewing-orchestrator`「高コスト fork×3 のため…明示呼出を強く推奨」(各 frontmatter 逐語)。**並列を起動するか自体を人間が裁定する**形 | **代替機構** |
| Steinberger: 並列の管理・可視化 | `teammateMode: "tmux"` (settings.json)。teammate 管理の実測知見は **`debating-roles/SKILL.md` L342/L354/L364** に文書化 —「**idle_notification は情報量ゼロ**」「shutdown_response 未返却時は shutdown 未達のまま idle 生存」「実データ受領は SendMessage 経由のみで判定」。永続 roster = `teams/*/config.json` (13 team dir・joinedAt/isActive) が実在。**欠け = 停止時刻・集約露出・Task 型 subagent の掲載** (ledger 名検索では 2 種のみ = 名前ベース測定の限界を独立検証が指摘) | **部分 [Medium]** (下記) |

### 個別照合 — 並列様式の差分 (gap ではなく設計差)

記事の Fan-out は「**同一仕様を複数インスタンスに投げ、客観基準で最良を選ぶ**」best-of-N 型。ハーネス側は「並列」の語を含む skill が 18 本 (`grep -l` 実測)、**うち並列を実際に編成するオーケストレータは 10 本前後** (残りは trigger 語・言及・部品側 — 検証者の目視分類)。その実装は **役割分離・視点分散型が主**: debating-roles (6 role) / llm-debate (5 lens) / detecting-framing-bias (2 frame) / essence-reviewing-orchestrator (3 領域) / マージゲートの fresh reviewer 並列。同一仕様 best-of-N の実装は見当たらない。

これは「できていない」ではなく「**使っていない**」— E-3 の主旨は「並列は最後の最適化」であり、並列の用途を検証・視点分散 (= 決定論的な選抜基準が最初から在る領域) に限定している現状は、むしろ順序規律の遵守側。best-of-N が必要になった時に 4 条件 (基準・成功率・隔離・候補扱い) を検査する運用が入口 (明示呼出) に既に在る。

### 残差 / 改善候補

- **[Medium] 並列 agent の稼働可視化 — 台帳は在るが「停止時刻・露出・subagent」が欠ける**: 独立検証が初版の「永続台帳が無い」を反証した — `teams/<team>/config.json` が **teammate 単位の永続 roster** (13 team dir・`joinedAt`/`isActive` 保持・セッションを越えて残存) として実在する。**それでも残差が生き残る欠け 3 点**: (a) 停止時刻 (`stoppedAt` 相当のフィールドが member スキーマ和集合に無い) (b) 集約・SessionStart への露出導線が無い (c) **Task 型 subagent は roster に載らない** — 本セッションで放置された scan 2 体も members に不在 (検証者実測)。実運用の観測は idle 通知 (「**情報量ゼロ**」と `debating-roles:342` が自己文書化) + session 終了時 UI の目視。**実害は本セッションで観測済み**: ①scan agent 2 体を報告回収後に停止し忘れ **6h45m 放置** (Dende のスクショ指摘で発覚。片方の 6h44m はスクショのライブ観測のみで永続痕跡なし) ②SendMessage 非搭載 agent の報告不達 3 体。永続痕跡 = 本 PJ handoff の next_phase (2 箇所)。**処方 (検証者の指摘で差し替え)**: 新規 jsonl の増設は車輪の再発明 — **既存の `teams/*/config.json` + `jobs/*/timeline.jsonl` (セッション粒度の state 記録、検証者発見) を読む集約器**を SessionStart レポートの隣に置く方が筋。
- **[Low] 報告回収・停止の運用規律が未文書化**: `TaskStop` の語はハーネスの rules/skills/progressive-disclosure に **0 件** (grep 実測)。本セッションで確立した運用 (報告はファイル書き出し or 本文・回収と同時に停止・様式は agent 型で分岐) は **handoff にのみ存在 = セッション揮発**。`failure-promotion-trigger` の L0→L1 昇格候補 (自己検知の反復: 放置 2 体 + 不達 3 体)。昇格先の候補 = `multi-agent-safety.md` (subagent 運用の既存の家) or `debating-roles` の知見節への追記。
- **[Low] チェックリスト③ (単発成功率のサンプリング計測) の不在**: 10 回試行 8 回型の計測機構は現役ハーネスに無い (最近縁 = `skills-disabled/empirical-prompt-tuning`、無効化済み)。**Low とする理由**: ハーネスの品質保証は決定論テスト (60 cases の binary pass/fail) + merge gate の実証 (mutation) で行われており、確率サンプリングの前提となる best-of-N 並列自体を使っていない。best-of-N を導入する時に前提条件として要実装、の位置づけ。

判定: **E-3「非決定論性は最後に活用する」= 取り入れ済み** (原則の逐語存在 + 検査 5 項目中 4 項目の機構 + fan-out の抑制ゲート)。残差は「並列を使う時の可視化・管理」の側に集中しており (Medium 1 + Low 2)、これは記事が Steinberger 引用で予告した弱点の実証でもある。

## step6 独立検証の記録

fresh な read-only reviewer (`code-reviewer` 型) 1 体へ判定ログ本文と読み取り権限のみを渡した (報告はファイル書き出し = 163 行、grep 3 エンジン交差 + corpus の HEAD 固定性確認つき)。**verdict: CONDITIONAL** — UNTRACEABLE 1 / OVERCLAIM 4。**判定本体「取り入れ済み」は支持** (「結論を覆す証拠は出なかった」と明記)。

| # | 指摘 | 初版 | 是正 |
|---|---|---|---|
| A | UNTRACEABLE | 実害の出典を「handoff の E-1 反省節」と名指し | **相互に誤りがあった** — 検証者の「6h4[0-9] = 0 件」はメイン再測で反証 (`6h45m` と報告不達は handoff next_phase に 2 箇所実在、grep -c 実測)。だが検証者の核 =「E-1 反省節という名前の節は無い」は正しく、locator を「handoff frontmatter の next_phase」へ是正 + **6h44m は永続痕跡なしと開示** |
| B | OVERCLAIM | [Info] 残差「E5 定義行が無い」 | **偽 — L343 に実在** (メイン再測で確認)。残差ごと削除。**scan の主張を 1 件だけ未検証のまま昇格させていた** — 9 件撃ち直しても 10 件目の未検証が漏れる実例 |
| C | OVERCLAIM | 「永続の稼働台帳が無い」 | **反証 — `teams/*/config.json` が永続 roster** (joinedAt/isActive、13 team dir)。残差を「停止時刻・露出・subagent 掲載の 3 点が欠ける」へ再構成。処方も「新規 jsonl」(車輪の再発明) から**既存 roster + `jobs/*/timeline.jsonl` を読む集約器**へ差し替え |
| D | OVERCLAIM | ③陰性「14 件・無い」 | 掲示スコープでは **13 件** (agents/ 込みで 14)。`skills-disabled/empirical-prompt-tuning` に最近縁の実装が実在 (無効化済み) — 射程を明示 |
| E | OVERCLAIM | 「並列実装 18 skill」 | 18 は**語のヒット数**。実装オーケストレータは 10 本前後 — ラベルを分割 |

軽微 2 件 (例外開示の対象ずれ → `essence-summary.md:18`「AI 自動マージ禁止」へ差し替え / ①の binary pass 率の 1 語) も反映。再ゲートは行わない (1 ラウンド固定・是正は二重測定値の採用。残余は step8 の Dende レビュー)。

**このラウンドの学び**: ①検証者も誤る — A の「0 件」はメインの再測が反証した。**gate の指摘も鵜呑みにせず撃ち直す**という E-2 からの規律が、今回は逆方向 (検証者の誤りの検出) に効いた ②B は「scan の 10 件目」問題 — 9 件撃ち直しても、撃ち直さなかった 1 件が漏れる。E-2 の学び「対照・補助の数値も撃ち直す」の適用射程は「判定に載る主張すべて」まで広げる必要がある。

## 自己是正の記録

- **scan の誤帰属 1 件をメインが検出**: scan は teammate 管理知見の在処を「`rules/multi-agent-safety.md` L340-369」と報告したが、同ファイルは **12 行** (本セッションで 4 回実測) で L340 は存在し得ない。実体は **`debating-roles/SKILL.md` L342/L354/L364** (grep で確定)。scan 自身が「(debating-roles/SKILL.md と一部重複)」と注記しており内容は正しかったが、主帰属が誤り。**「行番号つきの引用は、ファイルの行数と突き合わせる」**が今回の教訓 — L340 を 12 行のファイルに帰属させる主張は、行数を知っていれば一目で弾けた。
- **運用の学び (様式)**: Explore 型 agent は Write 非搭載でファイル報告が構造的に不可 (今回、agent 自身が制約を自己申告して本文報告に切替 = 正しい挙動)。E-2 まで「報告はファイル書き出し標準」としていたが、**scan (Explore) = 本文報告 / gate (code-reviewer) = ファイル報告**と agent 型で様式を分けるのが正。

## 関連ファイル

- `~/.claude/.docs/essence/essence-docs/agent-essence.md` — L165-167 (原則 E-3 の逐語)
- `~/.claude/hooks/test/` — cases 60 本 + `run-all.sh` の数値出力 (L196-212)
- `~/.claude/settings.json` — PostToolUse lint 7 本 / `teammateMode: "tmux"`
- `~/.claude/rules/multi-agent-safety.md` + `~/.claude/.docs/progressive-disclosure/harness-worktree-isolation.md` — 隔離の規律と手順正本
- `~/.claude/skills/debating-roles/SKILL.md` — L342/L354/L364 (teammate 管理の実測知見。idle 通知の限界)
- `~/.claude/skills/{llm-debate,detecting-framing-bias,essence-reviewing-orchestrator}/SKILL.md` — 高コスト fork の明示呼出ゲート (frontmatter)
- `~/.claude/skills/proposing-essence-updates/SKILL.md` — L7 (自動マージしない HITL)
- `~/.claude/skills/review-harness/diagnosis-rubric.md` — L377 (E-3 行の stale 参照疑い)
- `~/.claude/.docs/essence/essence-sources/_wip-note-distillation/260405.md` — L187 (順序規律の蒸留)

## 出典

- 記事本文: `.docs/references/260405_*/text.md` (E-3 = 2271〜2309 行)。索引 `~/.claude/.docs/references/BIBLIOGRAPHY.md` (番号 260405)
- 親バッチ: `.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md` (E-3 の個別判定なし — grep 0 件・対照つきで確認)
- 直前の深掘り: `.docs/logs/shared/2026-08-08_e-2-reason-based-generalization-deepdive.md` (第18弾)
- 実害の観測記録: 本 PJ `.claude/handoff-state.md` の E-1 反省節 (6h45m 放置・報告不達 3 体)
