---
date: 2026-07-24 20:57:20
type: validation
topic: pr-225-227-merge-gate-review
session: PR #225 (issue-216 TDD チェーン multi-stack 化) + PR #227 (issue-217 検証 scripts Swift 対応) のマージ可否レビュー (かいじゅう割込指示。第16弾 S-1.4+S-1.5 深掘りを中断して実施)
related_pr: [kaijutale/claude-harness#225, kaijutale/claude-harness#227]
related_skill: [logging, handoff]
related_agent: [code-reviewer (reviewer-pr225 / reviewer-pr227 の 2 並列・フレッシュコンテキスト)]
related_log_ids: []
---

# PR #225/#227 マージゲートレビュー — 判定: 両 PR とも CONDITIONAL GO (このままのマージ不可 / #225=High4 · #227=High3 是正 + 配線宙吊りの追跡が条件)

> 核心: **PR 本文の「検証済み」主張と独立レビューの乖離が本レビューの主成果**。#227 の claim「102 assertion 全 PASS + shellcheck clean」は実走で完全一致 (誠実) だったが、**その 102 PASS は High 3 件 (systemic fail-open / coverage 水増し / TAMPERED 誤爆) をどれも検出していない** — テスト全緑は出荷判断の根拠にならない、を地で行く実例。#225 は是正 claim 6 件中 3 件が部分実在 (「残存 0 件」が偽、L0 テンプレ未配線、正本との逐語矛盾)。クロス PR では**両 PR が互いに残作業を相手 issue の責務と明記して押し付け合い、両方 close されると配線 4 点が宙に浮く**構図を実測確定。メイン (team-lead) の中間見立て「false green は出ない設計」は #227-H1 の実測 (main rc=2 → head rc=0、消費者は exit code しか読まない) で**反証され撤回**した。

## レビュー体制

- fresh `code-reviewer` × 2 並列 (reviewer-pr225 / reviewer-pr227)。PR 本文は未検証 claim として扱わせ、diff・実体・実走で独立裏取り。severity は judging-review-severity 準拠
- 対象 head: #225 = e9b272b / #227 = 2e8b1c5、base main = 1bcb0ca (ローカル ref と GitHub headRefOid の一致を rev-parse で確認済)
- #227 は `git archive` で `$CLAUDE_JOB_DIR/tmp/pr227-tree` へ隔離展開しテストスイート実走 (LIVE ハーネス・PR worktree 非接触)
- メイン (team-lead) はクロス PR 整合 (ファイル重複 comm / 配線 4 点の grep / open issue 全件照合) を担当

## 統合判定

| PR | VERDICT | マージ条件 |
|---|---|---|
| #225 | CONDITIONAL GO | H-1 (L0 テンプレに AMBIGUOUS/Toolchain 欄追加) / H-2 (入れ子 SwiftPM の package root 概念 or 縮退宣言) / H-3 (`STACK == "web"` 二分岐の Python/Go 回帰を反転) / H-4 (質問軸とトークン軸の 1:1 写像定義)。M-1 (正本との逐語矛盾 1 行) 同時投入推奨 |
| #227 | CONDITIONAL GO | H-1 (cycles 消費者側の `skipped:` 区別 — #225 側での是正+同時マージでも可) / H-2 (`-ignore-filename-regex` でテストターゲット除外) / H-3 (PRUNE に Pods/Carthage/vendor 追加) |
| クロス | — | #227 README「呼び出し側の未対応点」4 点が #225 head に 4/4 不在 + 追跡 open issue 不在 → #225 へ取り込むか追跡 issue 新設をマージ条件に加える |

推奨マージ順: **#225 (是正込) → #227 (是正込)** — fail-open の窓 (cycles rc2→rc0 が消費者未対応のまま先行) を作らないため。ファイル重複は 0 (comm 実測) ゆえ conflict は無い。

別 issue 推奨 (本 PR 外・main 既存): (a) 3 scripts のオプション値欠落で無限ループ (#227-H4、実質 Critical 級) (b) 非 Swift 経路が runner exit code を捨てる (M-4) (c) `_is_number` が `.` を通しゲート無効化 (M-5)。

## クロス PR 実測 (メイン担当分・probe 済)

- ファイル重複 0: `comm -12` で実測 (11 files vs 17 files)
- 配線 4 点の #225 head 不在を全件 grep/git show で確認: (1) `coder.md:254` `--branch-min 75` 固定 (2) `orchestrating SKILL.md:233`「exit 0=循環なし」のまま `no cycles:`/`skipped:` 言及なし (3) enforcing SKILL.md → scripts/README 参照 0 件 (4) thresholds.md は変更対象外
- open issue #218〜#226 に呼出側配線の追跡なし (gh issue list 全件照合)

## 副次観測 (ハーネス学び — issue 候補はかいじゅう判断)

1. **teammate 型で spawn した `code-reviewer` は SendMessage 非所持 (tools: Read/Grep/Glob/Bash) で親へ返信不能**。idle 通知だけが届き、レポートは transcript (`~/.claude/projects/<PJ>/<uuid>.jsonl`) から jq 抽出で回収した。さらに **shutdown handshake も不成立** — `shutdown_request` への応答 (`shutdown_response`) 自体が SendMessage を要するため、要求を送っても起きて何もできず idle へ戻るだけ (21:08 実測)。用済み後も表示に残り続ける。名前付き並列 spawn + 返信要求の運用をするなら (a) レビュア系にも通信手段を渡すか (b) 最初から同期 Agent 呼びにするかの設計判断が要る
2. 両レビュアとも「SendMessage が無い」を自己申告し最終テキストにレポートを置いた (fail-close 側の正しい縮退)
3. PR 本文の検証表が精緻でも独立レビューは省略不可 — #212 (autoMemoryEnabled 捏造) と同じ教訓の再確認。今回は捏造でなく「真の claim が High を覆い隠す」変種

## reviewer-pr225 レポート原文 (verbatim・transcript 回収)

# PR #225 独立レビュー最終レポート (head e9b272b / base 1bcb0ca, 11 files +754/-100)

## (1) findings — severity 順

### HIGH

**H-1 `agents/team-tester.md:71` — L0 の出力契約に `AMBIGUOUS` / `Toolchain:` が無く、L2 の新 halt ゲートが空撃ちしうる**
実測: `[Test Derivation Complete]` テンプレは `Stack: <web | apple-swiftpm | apple-xcodebuild>` のみで `Toolchain:` 行が存在しない (`git grep -n "Toolchain:" issue-216 -- agents/team-tester.md` のヒットは散文の 121 行のみ、テンプレ内 0)。一方 team-tester.md:121 は「`Toolchain: UNAVAILABLE` を報告して親へ差し戻す」と literal を要求し、`agents/coder.md:191` の `if "AMBIGUOUS" in red or "UNAVAILABLE" in red` がその literal を拾う設計。RED 報告を実際に書くのは L0 team-tester (自分の agent テンプレ) で、L1 `red-test-fork/SKILL.md:118-119` のテンプレとは別物。テンプレが割れている以上、C-2 Critical (RED 不成立で implement が走る) の是正は L0 まで配線されていない。
改善案 (`agents/team-tester.md:71` を置換):
```
Stack: <web | apple-swiftpm | apple-xcodebuild | AMBIGUOUS (<検出したシグナル一覧>)>
Toolchain: <OK | UNAVAILABLE (CommandLineTools only) | N/A (Web)>
```
同じ欠落が `skills/deriving-test-from-spec/SKILL.md:161-163` のテンプレにもある。

**H-2 `skills/enforcing-strict-tdd-cycle/references/stack-detection.md:99,114` — 入れ子 SwiftPM は「検出できるが実行できない」**
実測 (fixture: `ios/Package.swift` + `ios/App.xcodeproj`): `find . -maxdepth 2 -name "Package.swift"` → `./ios/Package.swift` を検出 (M-1 是正は機能)。しかし同 fixture で `find Sources -maxdepth 8` / `find Tests -maxdepth 8` は**空**、配置規約は root 相対 `Tests/<Target>Tests/`、runner は裸の `swift test`。`git grep -n "package-path" issue-216 -- agents skills` → **0 件**。結果、`apple-swiftpm` と判定後にテストをリポジトリ root の `Tests/` へ書き、`swift test` は `could not find Package.swift` で落ちる。M-1 の修正が「取りこぼし」を「誤配置 + 実行不能」に置き換えている。
改善案 (§6 / §7 SwiftPM 節 + 3 fork の runner 行):
```
- package root = 検出した Package.swift の親ディレクトリ (root 固定ではない)
  テスト = <package root>/Tests/<Target>Tests/… / 実装 = <package root>/Sources/<Target>/…
- runner は `swift test --package-path <package root>` (cd に依存しない)
- !構文の `find Sources/Tests` は root 直下のみ。非 root package では Glob tool `**/*.swift` で列挙
```

**H-3 `agents/coder.md:199-201, 240, 253` — `if STACK == "web"` の二分岐が Python/Go レーンを回帰させ、虚偽ラベルを出す**
実測: base (`git show main:agents/coder.md:161,193,199`) は `assert-tests-unchanged.sh` / `assert-coverage.sh` を**無条件**実行。head は `STACK == "web"` の時のみ実行し、else で `coverage_status = "SKIPPED (Swift 未対応 — issue #217)"`。`scripts/lib/test-runner-detect.sh:48-51` は `pyproject.toml` / `pytest.ini` / `setup.cfg` から `python3 -m pytest` を返し、`assert-coverage.sh:113` に `*pytest*) RUNNER="pytest"` が実在 = **Python の coverage 検証は動いていた**。head ではそれが実行されず、Python プロジェクトの完了レポートに「Swift 未対応」と書かれる。H-3 (他言語レーンを止めない) を L1/L0 で直しながら L2 で同型の回帰を新設している。
改善案:
```python
APPLE = STACK in ("apple-swiftpm", "apple-xcodebuild")
if not APPLE:
    cov = Bash(f'bash "{COVERAGE_ORACLE}" --line-min 80 --branch-min 75 --root "{WORK_DIR}"')
    coverage_status = {0:"PASS",1:"BELOW_THRESHOLD",2:"SKIPPED(provider無)"}[cov.exit_code]
else:
    coverage_status = "SKIPPED (Swift 未対応 — issue #217)"
```
integrity 側も `not APPLE` に統一し、else のラベルは stack 名を埋める (`N/A ({STACK} 未対応 — issue #217)`)。

**H-4 `skills/orchestrating-team-development/SKILL.md:59,65` — 質問の軸 (プラットフォーム) と記録トークンの軸 (ビルドシステム) が不一致。最優先入力が未定義写像で作られる**
実測: 選択肢は `Web (TS/Next.js) / iOS / macOS / その他`、書き出す token は `web / apple-swiftpm / apple-xcodebuild / auto`。「iOS」→ どちらの token かの写像は 11 ファイルのどこにも無い。token は優先 1 = **ファイル検出を上書き**するため、誤写像は SwiftPM を xcodebuild レーンへ確定的に流す。さらに未知トークン (L3 が素直に `ios` と書く等) は全層の表で未定義 —「3 値のいずれか → その値」にも「`auto`・空 → 優先 2 以下」にも該当しない。トークン集合を検証する hook / validator は存在しない。
改善案 (Step 1-0 の 2・3):
```
2. 選択肢を token と 1:1 に:
   Web (package.json) → web / Apple・SwiftPM (Package.swift) → apple-swiftpm /
   Apple・Xcode project (*.xcodeproj|*.xcworkspace) → apple-xcodebuild / その他 → auto
   ※ iOS か macOS かは destination の決定要素であって stack token ではない (§7-2 で決める)
3. 4 値以外を書かない。読み手は 4 値以外を検出したら auto と同義に倒し、その旨を報告する
```

### MEDIUM

**M-1 `skills/orchestrating-team-development/SKILL.md:52` — 正本と逐語矛盾 (claim (c) の「残存 0 件」が偽)**
head 当該行 = `| 1 | .docs/specs/CURRENT/target-stack.txt（既存の記録） | 記載値 | 記載に従う |`。正本 `stack-detection.md:23` は「**優先 1 の runner 列は「記載に従う」ではない**」と明示的に禁じる。`auto`・空の扱いもこの表に無い。8 ファイルは統一されたが、**stack を書く当事者である L3 の表だけ旧形式**。
改善案: 他 8 ファイルと同一文言へ置換 (`が web / apple-swiftpm / apple-xcodebuild → その値`、`auto`・空 = 指定なし)。

**M-2 `agents/coder.md:191` — 自由文 substring ゲートが誤 halt を作る**
`if "AMBIGUOUS" in red or "UNAVAILABLE" in red` は孫の報告テキスト全体への部分一致。ところが `red-test-fork/SKILL.md:118-119` のテンプレ自身が両 literal を enum 凡例として含む。孫が凡例ごと転記すると健全な Web サイクルが即 halt。同 PR の verify 側 (`coder.md:229`) は構造化フィールド `result.final_result == "UNDETERMINED"` を使っており実装が非一貫。
改善案: `parse_field(red,"Stack").startswith("AMBIGUOUS") or parse_field(red,"Toolchain").startswith("UNAVAILABLE")`。

**M-3 `stack-detection.md:53-56` vs `orchestrating-team-development/SKILL.md:63-67` — stale 値の掃除責務が所有者層に inline されていない**
ライフサイクル (feature 単位・毎回上書き・完了時削除) は正本 §3 のみ。実際に書く L3 Step 1-0 手順 3 には「1 トークンのみ書き出す」しかなく上書き/掃除の指示が無い (read-miss 原則の自己違反)。加えて `coder` は L3 を経由せず直接起動可能で、その経路では更新主体が居ない。stale な `apple-swiftpm` は優先 1 でファイル検出に勝ち、Web 案件が黙って Apple レーンへ倒れる。機械ガード (hook / validator / TTL) は無し。
改善案: 手順 3 に「必ず上書き」「feature 完了時に削除」「L2/L1 は spec.md より mtime が古い target-stack.txt を無視」を逐語追加。

**M-4 `skills/orchestrating-team-development/SKILL.md:42-45` — Step 1-0 が Mode 選択 (1a) より前で無条件必須**
Web 専用リポジトリでも調査/ドキュメント Mode でも毎回 target stack を質問する。かつ正本 `stack-detection.md:38-41` は L3 の質問を**曖昧検出時**の動作と定義しており、L3 の「無条件必須」と層間で矛盾 (受入基準「全層で矛盾しない」に抵触)。PR の「Web レーンは改修前と実質同一」は !構文展開の話で、L3 の対話フローは同一でない。
改善案: シグナルが 1 系統のみなら質問を省略、Web+Apple 同居 / 全部不在の時だけ AskUserQuestion に条件化。

**M-5 出力契約の `Stack:` enum に他言語レーンの値が無い** — `agents/team-tester.md:71-72` は `Stack:` を 3 値に限定しつつ直下の `Framework:` で `pytest|go-test|cargo-test|rspec|mix-test` を許す。H-3 で守った Python レーンが走ると孫は契約外の値を書くか `web` と偽るしかない。後者だと `coder.md:200` の `STACK == "web"` が真になり Web 用 oracle が Python に走る。改善案: 4 テンプレの enum を `<… | other (<runner>) | AMBIGUOUS (…)>` に拡張。

**M-6 Apple レーンは現ハーネスで実行不能 (実測)** — `settings.json` の allow(41)/ask(30)/deny(177) いずれにも `swift`/`xcodebuild`/`xcrun`/`swiftlint` は無く、subagent 内 permission prompt 依存。#226 により RED→GREEN 実走行は 0 件。新レーンは「一度も動かしていない仕様書」。PR 本文で開示済みだがマージ判断材料として明記。

### LOW

**L-1 `agents/coder.md:199` — `detect_stack()` が未定義。** 他の外部呼出はすべて `Bash(f'…')` の具体形。agent には !構文が無いため実際には coder 自身が Bash を叩く必要があり、C-4 gate を回すか否かの分岐がモデル判断に落ちている。改善案: `STACK = Bash('cat .docs/specs/CURRENT/target-stack.txt 2>/dev/null').stdout.strip() or parse_field(red,"Stack")`。

**L-2 `agents/coder.md:245` — `integrity_status` が dead 変数。** どこからも参照されず、計算されうる `SKIPPED (baseline 無し)` (exit 2) が完了レポート `coder.md:423` の enum に存在しない。改善案: テンプレを 3 値化し `{integrity_status}` を埋める。

**L-3 `stack-detection.md:80` — 正本自身が enum 外の値 `Stack: apple` を書いている。** 「食い違ったら正本が勝つ」と宣言している以上、ここは `apple-swiftpm` / `apple-xcodebuild` か `<確定した stack>`。

**L-4 検出 !構文の重複** — root 直下 `Package.swift` では `cat Package.swift` と `find . -maxdepth 2 -name "Package.swift"` が二重ヒットし「判定根拠」欄がぶれる。

**L-5 PR 受入基準表「cat×4 / find×5 / ls×1」は一意行数。** 実際の追加 `!` 行は 25 行 (+既存 3 行の `2>/dev/null` 付与)。単純形のみ・複合形ゼロという結論は独立に再現 (30 行すべて cat/find/ls)。

### 検証して問題無しと確認した項目 (誤検出の排除)

- `Bash(cat:*)` / `Bash(find:*)` / `Bash(ls:*)` はすべて `permissions.allow` に実在 → 追加 !構文は permission 面で安全。#224 が要るのは runner 実行 (`swift`/`xcodebuild`/`xcrun`/`swiftlint`) のみで、追加 !構文行に影響を受けるものはゼロ。
- Web fixture (package.json + tsconfig + src/ + node_modules) で追加 7 行を実行 → **全行 0 バイト出力**。`ls` 行は base/head とも `package.json` `tsconfig.json` のみ。後方互換の主張は再現した。
- 不在時のエラー汚染なし: 追加行はすべて `2>/dev/null` 付きで stderr がプロンプトに混ざらない (`cat` は rc=1 だが出力は空)。
- `record-loop-iteration.sh:60-61` は `GREEN|RED` 以外を exit 1 で拒否 → `coder.md:225-227` のコメントは正確。
- `agents/coder.md` は 502 行だが `hook_post_file_line_limit.sh:34` の対象拡張子は code のみで `.md` は除外 → 500 行ルール違反ではない。
- 参照整合: #216/#217/#218/#224/#226 はすべて OPEN で実在。`references/stack-detection.md` の相対リンク、`harness-modification-policy.md` (`.docs/progressive-disclosure/` に実在) ともリンク切れ無し。

## (2) claim 照合表

| # | claim | 判定 | 根拠 |
|---|---|---|---|
| (a) | Apple では oracle 非実行 → `Test integrity: N/A (Swift 未対応 — #217)` (UNCHANGED を出さない) | **実在** (副作用あり) | `coder.md:210` (Rules) / `240-249` (擬似コード else 分岐) / `423` (テンプレ) / `verify-test-fork:114,124` / 正本 §7.5。副作用 = 分岐が `web` vs それ以外の二分で Python も「Swift 未対応」になる (H-3) |
| (b) | coder Step 1 に halt 分岐 + Step 5 red-test-fork 直後の即エスカレーション | **部分実在** | `coder.md:114-115` に halt 分岐、`191-192` に `if "AMBIGUOUS" in red or "UNAVAILABLE" in red: return stack_escalation_report(...)` は実在。ただしトークンを出す L0 テンプレに欄が無く (H-1)、判定が自由文 substring (M-2) → 配線は未完 |
| (c) | `auto` の扱いを 8 ファイルで統一 / 「非空 → 記載値」残存 0 件 | **部分実在** | 8 ファイル (coder / team-tester / deriving / enforcing / stack-detection / implement / red / verify) の統一を grep で確認。**`orchestrating-team-development/SKILL.md:52` が「記載値 / 記載に従う」のまま**で正本 `stack-detection.md:23` と逐語矛盾 (M-1)。「`が非空` 残存 0」は文字列としては真、意味としては 1 件残存 |
| (d) | 「全部不在 → 差し戻し」を「Web/Apple 不在 かつ runner も特定不能」に緩和 / Python・Go 回帰なし | **実在** | 「全部不在」記述を 9 ファイルで確認、うち 7 ファイルに「runner を特定できない時だけ」。ただし L2 coder に別種の Python 回帰が新規混入 (H-3) |
| (e) | 出力契約テンプレに `Stack: AMBIGUOUS` / `Toolchain:` を正式追加 | **部分実在** | 3 fork (`red:118-119` / `implement:120-121` / `verify:126-127`) は追加済み。**L0 `agents/team-tester.md:71` と `deriving-test-from-spec:161` は未追加** (H-1) |
| (f) | `find . -maxdepth 2 -name "Package.swift"` で検出深度を対称化 | **実在** | 4 ファイルに存在。入れ子 fixture (`ios/Package.swift` + `ios/App.xcodeproj`) で `./ios/Package.swift` 検出を独立再現。ただし対称化は検出のみで、配置・runner は root 固定のまま (H-2) |

## (3) VERDICT

**CONDITIONAL GO — このまま main へマージするのは不可 (High 4 件がマージブロッカー)。**

マージ条件 (本 PR 内で是正、または該当箇所へ逐語で開示 + issue 化):

1. **H-1**: `agents/team-tester.md:71` (と `deriving-test-from-spec` テンプレ) に `AMBIGUOUS` enum と `Toolchain:` 行を追加。無いと C-2 Critical の是正が L0 に届かない。
2. **H-2**: 入れ子 SwiftPM の package root 概念を導入 (`swift test --package-path` / 配置を package root 相対 / Glob 列挙)。または「root 直下の Package.swift のみ対応」と縮退させ、root 外検出時は `AMBIGUOUS` で差し戻す。
3. **H-3**: `coder.md` の `STACK == "web"` を `not APPLE` に反転し、Python/Go の coverage oracle 実行を復元。else ラベルから固定文字列「Swift」を外す。
4. **H-4**: Step 1-0 の AskUserQuestion 選択肢を 4 トークンと 1:1 にし、未知トークン時の挙動 (`auto` 扱い + 報告) を優先 1 行に明記。

M-1 (正本との逐語矛盾) は 1 行修正ゆえ同時投入を推奨。M-2〜M-6 と Low 群は別 PR / issue で可。

補足: 本 PR は markdown のみ (`settings.json` 変更なし)、Web レーンの !構文展開が改修前と同一であることは独立に再現した。**ブロック理由は既存 Web レーンの退行ではなく、新設 Apple レーンと巻き添えになった他言語レーンの正しさ**にある。

## reviewer-pr227 レポート原文 (verbatim・transcript 回収。冒頭の送信不能連絡と persona 枠のみ省略)

**VERDICT: CONDITIONAL GO** (このままの main マージは不可。条件 3 件)

### (1) findings — severity 順

#### [HIGH-1] `skills/enforcing-strict-tdd-cycle/scripts/assert-no-cycles.sh:101` — 「未検査」が呼出側では「循環なし」になる (本 PR が持ち込む系レベル fail-open)

実測: 同一 SwiftPM root (`Package.swift` + `Sources/App/App.swift`) に対し **main = `rc=2` / head = `rc=0`**。唯一の消費者 `skills/orchestrating-team-development/SKILL.md:200` は「exit 0=循環なし」と明記し exit code しか読まない (同ファイルは本 PR で未変更)。stdout の `no cycles:` / `skipped:` 分離は正しいが、それを読む消費者が存在しない。PR 前は人が気づく exit 2、PR 後は「検査済み・循環なし」として通過。

```markdown
<!-- 最小修正: SKILL.md:200 -->
exit 0 かつ stdout が `no cycles:` で始まる時のみ「検査して循環なし」。
stdout が `skipped:` の exit 0 は未検査であり合格として扱わない
```

#### [HIGH-2] `lib/test-runner-detect.sh:148` — Swift の行 % にテストターゲット自身が混入し閾値が緩む

`xcrun llvm-cov report "${binary}" -instr-profile=...` に `-ignore-filename-regex` 無し。`<Pkg>PackageTests.xctest` にはテストターゲットのソースが静的リンクされ、TOTAL に合算される。実測 (本機 Swift 6.3.2、等価実験):

```
Lib.swift       5 2  60.00% ...  5 2  60.00%
LibTests.swift  1 0 100.00% ...  4 0 100.00%
TOTAL           7 2  71.43% ... 10 2  80.00%   ← production が読む $10 = 80.00% (合格)
-ignore-filename-regex='.*Tests.swift' 適用時 = 66.67% (不合格)
```

同一コードで合否が割れる。vitest/jest は既定でテストを除外するため Web と Swift で「80」の意味が違う。修正: `-ignore-filename-regex='(^|/)(Tests|\.build)/'`。

#### [HIGH-3] `assert-tests-unchanged.sh:77,117-119` — vendored 依存が Swift baseline に混入し正当な依存更新が TAMPERED

`PRUNE` は 6 個のみ・`.gitignore` 不参照で root 全体を走査。実測:

```
recorded 4 test file(s)
  Carthage/Checkouts/Dep/Tests/DepTests.swift
  Pods/SomeLib/Tests/PodTests.swift
  Tests/AppTests/AppTests.swift
  vendor/x/FooTests/V.swift
→ Pods/ を pod install 相当で書換 → Test integrity: TAMPERED / rc=1
```

P0 は coder.md のブロッキング gate。既存テスト (シナリオ11) は `${s7}/Tests/.build/...` という実在しない配置しか検証していない。修正: `PRUNE` に `Pods`/`Carthage`/`vendor` 追加、または `git check-ignore --stdin` 連携。

#### [HIGH-4] `assert-coverage.sh:57-66` / `assert-no-cycles.sh:41-47` / `assert-tests-unchanged.sh:39-46` — オプション値欠落で無限ループ (**pre-existing・diff 外**)

`shift 2` は `$# < 2` で失敗し何もシフトしない → `while [[ $# -gt 0 ]]` が回り続ける。実測 (3s 後も生存 → kill):

```
head assert-coverage.sh --line-min : HANG / --branch-min : HANG
head assert-no-cycles.sh --root    : HANG
head assert-tests-unchanged.sh --root : HANG
main assert-coverage.sh --line-min : HANG   ← main 既存 = 本 PR の退行ではない
```

rubric 上は Critical 例だが diff 外のため本 PR の条件からは外す (別 issue 化)。

#### [MEDIUM-1] `lib/test-runner-detect.sh:84-100` — テストバイナリ 1 本のみ採用

最初にマッチした 1 本だけを `llvm-cov` に渡す。複数 `*.xctest` / `Contents/MacOS/*` 複数時に測定対象が欠け、**警告なしに数値が変わる** (fail-closed にならない)。修正: 全件を `-object` 配列で渡す。

#### [MEDIUM-2] `lib/test-runner-detect.sh:111-153` — 新規 50 行に自動テストゼロ

`grep -rn "swift_llvm_cov_report\|_swift_test_binary" tests/` → **0 件**。102 assertion のどれも触れていない。Xcode 不要で書ける分岐 (root 不在 / profdata 不在 / xctest 不在 / llvm-cov 失敗) すら未テスト。record の残存リスク 2 は「実行経路未検証」までしか開示していない。

#### [MEDIUM-3] `assert-no-cycles.sh:77-83` — `.js` 1 個で誤 exit 2

実測: `docs/assets/site.js` を置いた純 Swift package → `rc=2`「検査対象を特定できない」。`dist/`・`.eslintrc.js`・`Scripts/*.js` 等どれでも発火。方向は安全側だが構造 gate が理由不明にブロック。

#### [MEDIUM-4] `assert-coverage.sh:147` — 非 Swift 経路が runner の exit code を捨てる (**pre-existing**、本 PR で非対称化)

テストが落ちてもカバレッジ表さえ出れば `Coverage OK` → exit 0。新設 Swift 経路 (lib:129) は失敗を fail-closed にしたため、同一 script 内で Web/Python だけが緩い。

#### [MEDIUM-5] `assert-coverage.sh:69-74` — `_is_number` が `"."` を通す (**pre-existing**)

実測: `--line-min .` (実 42.86%) → `Coverage OK: lines 42.86% >= .%` **rc=0**。awk が `.` を 0 に変換しゲート無効化。修正: `[[ "$1" =~ ^[0-9]+(\.[0-9]+)?$ ]]`。

#### [LOW-1] `scripts/README.md`「走査範囲の線引き」/ `assert-tests-unchanged.sh:108-109` / commit message — 「既存 baseline 集合を変えない」は事実と異なる

実測: `src/app/a.test.ts` + `src/packages/x/node_modules/dep/b.test.js` → main `recorded 2` / head `recorded 1`。変更方向は改善だが記述と PR 本文の「後方互換不変」が誤り。

#### [LOW-2] `lib/test-runner-detect.sh:135` — `--show-codecov-path` を 1 行に正規化していない

本機 (6.3.2) は 1 行と実測済みだが、ビルドログ混在版では `dirname` が壊れ「profdata が無い」という誤った理由で exit 2。`| tail -1` で解消。

#### [LOW-3] `tests/run-all.sh:22-23` — コメントが実装と不一致 (PIPESTATUS もパイプも未使用)

#### [LOW-4] `tests/run-all.sh:24` — 出力バッファリングで進捗が消える (Xcode 有り環境の `[B]` 段・数十秒が無音)

#### [LOW-5] `tests/swift-coverage-live.test.sh:121-125` — 実機検証が production と違う matcher (`/^TOTAL/`) を使用。出荷される `$1=="TOTAL"` が実機出力で正値を返すことは未検証

#### [LOW-6] `tests/fixtures/assert-coverage/swift-llvm-cov-*.txt` — Filename 行が `Sources/` 1 行のみ。出荷経路が生成する `Tests/**` 行入りの形状を parse テストが一度も見ていない (HIGH-2 と同根)

#### [LOW-7] `assert-no-cycles.sh:82` — `find` 失敗を握り潰し (`2>/dev/null`・pipefail 無し) → 「TS/JS 無し」と読まれ skip=exit 0 に倒れる

### (2) 実走結果

**run-all.sh** (展開 tree `/Users/camone/.claude/jobs/962ac51d/tmp/pr227-tree` 内で実行、所要 13.7s):

```
rc=0
PASS 行 = 102 / FAIL 行 = 0 / SKIP 行 = 1 / test ファイル = 7
内訳: 24 + 9 + 18 + 4 + 17 + (7 passed,1 skipped) + 23 = 102
最終行: ORACLE TESTS PASSED WITH SKIPS (7 file(s), 1 stage(s) skipped — 未実行の検証あり)
skip 内容: [B] swift test 段 - Xcode 不在のため SwiftPM の実 e2e は未検証
           reason 2 行 (XCTest は Xcode 同梱 / judged by: xcrun --show-sdk-platform-path) を表示
```

**claim「102 assertion 全 PASS (1 段 skip、suite 最終行に明示)」との差分 = ゼロ。** skip 有り時の `ALL ORACLE TESTS PASSED` 抑止も実挙動で確認。

**shellcheck** (0.11.0):

```
README 記載コマンド (shellcheck -x --source-path=tests --source-path=lib *.sh lib/*.sh tests/*.sh) → rc=0 指摘ゼロ
--source-path 省略時のみ SC1091 (info) ×7 — README の Gotcha 記載どおりの既知挙動
```

**claim「shellcheck clean」= 一致。**

**その他の実測**: `--branch-min 75` は 0.1s で `REASON=branch-metric-unsupported` 付き exit 2 (ビルド前中断を確認) / `--branch-min 0` の実行経路は 30.4s で `no such module 'XCTest'` を転記し fail-closed exit 2 / 0 件 baseline の exit code は main・head とも record=0・baseline=0 (互換 claim 一致) / awk の locale 依存 (`fr_FR`, `de_DE`) は無し / 全域 walk の実測コストは 4238 ファイルで 0.25s (性能問題なし)。

**未検証の開示**: `swift test` 成功経路 (= `swift_llvm_cov_report()` の happy path) は本機に Xcode が無く実行不能。HIGH-2 は構造的推論 + 等価実験による裏取り。

### (3) 最終 VERDICT

**CONDITIONAL GO** — マージ前必須条件 3 件:

1. **HIGH-1**: `orchestrating-team-development/SKILL.md:200` を `skipped:` 区別付きに直す (1 行) か、#216 と同時マージ。放置すると「未検査 → 循環なし」の silent pass を系に導入する。
2. **HIGH-3**: `PRUNE` に `Pods`/`Carthage`/`vendor` 追加 (または `git check-ignore` 連携)。Xcode 対応を謳う PR が Xcode の主要依存配置で誤 TAMPERED を出す状態では出荷不可。
3. **HIGH-2**: `-ignore-filename-regex` 追加。即時修正が重いなら「Swift の行 % はテストターゲットを含む」を README と残存リスクに明示開示して #216 へ引き継ぐ (現状は caller が `--branch-min 75` を渡すため経路が死んでおり、実害は #216 着地時に顕在化)。

**条件外・別 issue 推奨**: HIGH-4 (無限ループ)、MEDIUM-4 (非 Swift 経路の rc 無視)、MEDIUM-5 (`_is_number` の `.`) — いずれも main 既存・diff 外。

### まとめ (reviewer-pr227)

3 oracle の fail-closed 設計・skip の可視化・`$1=="TOTAL"` 完全一致化は狙い通り動く。だが README 自身が書いた「script 単体の fail-closed ≠ 系としての gate」が、cycles の exit 2→0 でそのまま現実化している。**「102 PASS / shellcheck clean」は本レビューの High 3 件をどれも検出していない** — 出荷判断の根拠にはならない。

## 追記 (2026-07-25): 是正の再ゲートレビュー → GO → マージ完了 → 申し送り再突合クローズ

- **是正 commit**: #225 = 38543a9〜c008c10 の 5 本 (head c008c10) / #227 = a0faf84〜3a5c553 の 3 本 (head 3a5c553)。worktree = remote 一致を実測してから条件 1:1 で実体照合。
- **#225 = 条件 6/6 クリア + 任意分**: H-1 (L0 テンプレ 2 箇所に AMBIGUOUS/Toolchain 追加、`other (<runner 名>)` で M-5 も解消) / H-2 (package root 定義 + `--package-path` 全層統一 + root 直下後方互換、入れ子 fixture 実測が PR 本文に) / H-3 (`if not APPLE` 反転、ラベルは stack 名埋込、verify-test-fork の同型も統一、Python fixture 実測) / H-4 (選択肢と token の 1:1 表 + iOS/macOS は destination 注記 + 未知トークン=auto) / M-1 (正本文言へ統一) / 配線 (cycles 消費者精密化・`--branch-min` は **issue #228 新設**・README 参照追加・thresholds.md 注記 +9 行)。任意推奨の M-2 (substring→構造化 line_value 読み) も是正済み。
- **#227 = 条件 2/2 クリア + 受入実走**: H-2 (`-ignore-filename-regex='(^|/)(Tests|\.build)/'` + Tests/ 行入り fixture 2 本とテスト固定) / H-3 (PRUNE へ Pods/Carthage/vendor + シナリオ 12 = 3 配置の再発防止テスト)。隔離コピーで **tests/run-all.sh = 121 PASS / 0 FAIL** (前回 102 → 是正テスト +19)・shellcheck rc=0。任意の LOW-2 (`tail -1`) と Swift 異常系 stub テストも追加されていた。
- **是正側の逆検出 (レビュアの指示も検証対象、の実例)**: 当レビューの条件「`skipped:` の exit 0 を不合格に」は、pre-#227 の現行 script が出さない phantom トークンだった — 実際の素通り穴は `no cycles: 0 module(s)` exit 0。#225 側の独立 essence レビューがこれを実測検出し、「0 件走査 = 未検査として不合格」+「`skipped:` は #227 マージ後の契約」の二段で現物準拠に是正した。
- **開示**: 本再レビューは月次 spend limit で独立 agent が起動不能だったため、メイン自身のインライン検証 (全条件の実体照合 + テスト実走)。是正側の独立 essence レビュー record が両 PR に同梱されており、独立性はそちらの層と組で担保。
- **マージ**: 推奨順どおり #225 (2026-07-24T21:36:25Z) → #227 (21:36:45Z) 完了、issue #216/#217 close、remote branch 自動削除。local main へ ff-only pull 済み (9583db3)。
- **申し送り再突合 (マージ後 live 3 ケース)**: 実検査 = `no cycles: 2 module(s) checked` exit 0 のみ合格 / SwiftPM = `skipped:` exit 0 は不合格 / 0 件走査 = `no cycles: 0 module(s)` exit 0 も不合格 — 消費者 (orchestrating SKILL.md:244) の契約と完全一致。**追加修正なしでクローズ**。
- **残フォロー**: #228 (branch-min の stack 依存化) OPEN。初回レビューで「別 issue 推奨」とした main 既存 3 件 (オプション値欠落の無限ループ / 非 Swift 経路の rc 無視 / `_is_number` の `.`) は、2026-07-25 にマージ後 main (9583db3) で再検証 — HANG は perl alarm 3s で exit 142 (live 再現)、`.` 通過は判定関数単体で live 再現、rc 揉み消しは是正 commit 未変更の静的確認一致 — の上で **issue #231 に 1 本化** (かいじゅう判断・受入基準案つき)。これで本レビュー起点の未追跡はゼロ。
