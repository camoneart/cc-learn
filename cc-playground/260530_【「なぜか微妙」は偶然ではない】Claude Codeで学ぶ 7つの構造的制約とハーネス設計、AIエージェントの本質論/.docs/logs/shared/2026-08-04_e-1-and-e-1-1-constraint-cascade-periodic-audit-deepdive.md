---
date: 2026-08-04 20:05:54
type: study
topic: e-1-and-e-1-1-constraint-cascade-periodic-audit-deepdive
session: E-1「制約が品質を生む」+ E-1.1「ドリフトを前提に定期的に掃除する」合同深掘り (取り入れフェーズ第17弾 — E-1 本体 35行 / E-1.1 26行、連続する 1 論点ゆえ合同)
related_article: .docs/references/260405_【「なぜか微妙」は偶然ではない】Claude Codeで学ぶ 7つの構造的制約とハーネス設計、AIエージェントの本質論/text.md (E-1 = 2138〜2172行 / E-1.1 = 2174〜2199行、関連: 環境設計の導入 2134〜2136 / E-2 開始 2201 / S-1.5 fail-closed 2124〜2130)
related_skill: [harness-adoption-audit, explain-in-html, logging]
related_log_ids: [2026-05-30_note-harness-gap-analysis, 2026-07-24_s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive]
related_log: [.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md, .docs/logs/shared/2026-07-24_s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive.md]
---

# E-1「制約が品質を生む」+ E-1.1「ドリフトを前提に定期的に掃除する」合同深掘り — 判定: E-1 = 取り入れ済み (4 層すべてに実体が実在・機械強制まで到達は Layer2 と Layer3 の一部) / E-1.1 = 部分 gap (基準 2 本が hook へ渡らず固着・除草の実行は手動・時計駆動 0 件) · 反例狩りの最大収穫 = **無効化 skill への参照追従が網羅的でない** (無効化 28 本中 9 本が live 15 ファイルから参照。**ただし issue #84 が 2026-07-09 に検出済み・🟢 Low として意識的に受容**していた — 本ログの「新発見」フレーミングは独立検証により撤回) · 独立検証 2 ラウンドで自分の主張 28 件が倒れた (第1版 16 / 第2版 12)

> 核心の構造事実: **E-1 制約カスケード 4 層は全層に実体が実在する** (ただし機械強制まで到達しているのは Layer2 と Layer3 の一部で、Layer1 は規律のみ — 本文 L55 の表と整合させた表現)。Layer1 命名規則 = skills 71/71・agents 32/32・rules 13/13 が kebab-case で逸脱ゼロ (**この遵守率の計測に使った判定器** `^[a-z0-9]+(-[a-z0-9]+)*$` は逆変異 6 種で全件 reject を確認済み。**ハーネス同梱の agent 判定器は別物で 6 種中 2 種を素通りさせる** — 残差⑤) **だが `hooks/*.sh` には規則外 2 件が現存** / Layer2 ディレクトリ = `permissions.deny` のハーネス自己保護 13 entry で `Edit` 経路を封鎖 / Layer3 メタデータ = 全 71 skill・全 32 agent が frontmatter 保持 / Layer4 参照実装 = **32 skill** が `references/` を持つ。**反例狩りの核心収穫は 2 つ**。①**Layer3 の (1)→(2) 断絶**: 基準を定義した validator が 2 本あるのに、どちらも実在庫を通さず、かつ hook へ配線されていない — `quick_validate.py` は 71 skill 中 **27 FAIL**、`validate-agent-definition.sh` は 32 agent 中 **21 FAIL**、配線は対照実験つきで **0 件**を確認。記事 E-1.1 の **(1) 基準を定義する は済んでいるが (2) Hooks で自動検知する へ渡せていない**。②**無効化 skill への参照追従が網羅的でない**: 無効化 28 本のうち **9 本が live の 15 ファイルから参照**されている (`skills/README.md` を除く実測)。うち `skills/authoring-skills/references/skill-type-taxonomy.md:32` は `nano-banana` を「**参照実装**」として名指ししており、記事 Layer4 の定義そのものに触れる。**ただしこれは新発見ではない** — `essence-review-records/2026-07-09_134619_issue-84-skills-inventory-budget.md` L64-69 が同じ現象を検出し、実害寄りの 2 本を「(現在無効化)」注記で修正した上で、**残りを 🟢 Low の既知課題として明示的に受容**していた (独立検証の指摘により、初稿の「経路が無い」「独立検証が発見した最大の収穫」を撤回)。正確に言えるのは「**機械強制が無く、受容から約 1 ヶ月を経ても未解消**」まで。**E-1.1 の判定は 2 ヶ月前の親バッチから差分あり**: 時計駆動 (cron / launchd / cloud routine) は**今も 0 件**だが、当時の「daemon は workers 空 = 未着手」から **「意図的な SessionStart 代替 + 限界の自己開示」へ設計判断が昇格**しており、実発火も 2 週連続で確認できた (7/24 金 → 7/31 金。**ただし 7/24 の脚は他文書からの転記で独立検証不能**)。**本ログは独立検証ゲートを 2 ラウンド通し、通算 28 件の主張を是正した第 3 版**である (経緯は `## step6 独立検証の記録`)。

## 概要

取り入れフェーズ第17弾。E 章 (環境設計 — コードベースの構造で品質を底上げする、導入 2134-2136) の先頭 2 節を合同で扱う。E-1 (制約カスケード) と E-1.1 (定期監査) は「制約を敷く」→「敷いた制約が腐るのを防ぐ」の連続する 1 論点ゆえ分割しない。

親バッチ (`2026-05-30_note-harness-gap-analysis.md` L43/L68) の判定は **「E 環境設計 = 大半✅ / △1」「△ = E-1.1 定期監査: 検知 hook (hardcode_hygiene / validate_claudemd) はあるが、定期実行スケジュールなし (daemon は存在するが workers 空、review-harness は手動起動)」**。**2 ヶ月前の判定**であり、その間にハーネスは第1〜6波の改修を受けている (**PR 本数は母集団の定義が記録に無く再構成できなかったため数値を書かない**。独立検証が `git log --merges` で 141 件を数え、波記録の合算は約 29 で、初稿の「24」はどの数え方でも再現できなかった)。本深掘りの差分焦点は次の 3 点に置いた:

- (a) **Layer3 メタデータの「基準はあるが効いていない」経路の反例狩り** — 定義された基準が実在庫に対して本当に通るのか、全件実走で確かめる (宣言の読み取りで済ませない)
- (b) **E-1.1 の △ が 2 ヶ月でどう動いたか** — 「未着手」のままか、「意図的な代替」へ昇格したかを、機構の実在と発火痕跡の両方で測る
- (c) **記事が名指しする具体例 (PreToolUse でファイル名 kebab-case 違反を検出する hook) の有無** — 記事の処方箋そのものを持っているか

ハーネス実測 (step3) は read-only の `Explore` 2 体へ委譲した (`scan-e1-cascade` = カスケード 4 層 / `scan-e11-drift` = 定期監査 3 ステップ)。両者に「陰性 (0 件) を報告する時は使った検索コマンドと対照実験を必ず併記せよ」を明示指示している (`grounds-not-approval.md` の陰性検証条項)。

**scan の報告をそのまま判定へ載せない方針を取ったが、それでも不十分だった** — step6 の独立検証を **2 ラウンド**回し、**通算 28 件**の主張が倒れた (第1ラウンド 16 = 数値と事実が中心 / 第2ラウンド 12 = 機序・フレーミング・一般化が中心)。倒れた主張の内訳は `## step6 独立検証の記録` に全件記録する。

## 内容

### note 側の定義 (E-1 = 2138〜2172行 / E-1.1 = 2174〜2199行)

E-1 の主張 (2148 行、逐語): **「出力品質を上げる最も確実な方法は、良いプロンプトを書くことではなく、解決空間を狭めることだ。」**

設計パターン「制約カスケード」(2158〜2166 行) — 粒度の異なる 4 層を積み重ねて段階的に解決空間を狭める:

| Layer | 制約の種類 | 記事の例 | 記事が挙げる効果 |
|---|---|---|---|
| 1 | 命名規則 | kebab-case, PascalCase, `*.test.ts` | ファイル名の揺れが消える |
| 2 | ディレクトリ構造 | `src/features/auth/`, `src/shared/` | 配置先を構造が決める |
| 3 | メタデータ | YAML フロントマター (`status: stable` 等) | 既存パターンを模倣する |
| 4 | 参照実装 | 「`create-invoice.ts` を踏襲せよ」 | **最も強力**。全スタイルが暗黙に制約される |

カスケードの核心 (2166 行、逐語): **「カスケードの核心は「書かなくていい指示」を増やすこと。」**

E-1.1 の主張 (2180 行、逐語): **「良い癖も悪い癖も複利で効く。」** 設計パターン「定期監査パターン」(2184〜2188 行) は 3 ステップ — **(1) 基準を定義する** (リンターのルール、テンプレート、規約) / **(2) Hooks で自動検知する** (ファイル名・配置・パターンの違反を即座に検出。エージェント自身が修正する) / **(3) 既存コードの逸脱を定期的に除去する** (週次レビュー、テンプレート修正、参照実装の更新)。

記事が挙げる具体例 (2190〜2197 行) は **PreToolUse Hook で `Write|Edit` を捕まえ、ファイル名が kebab-case でなければ block し、エージェント自身に直させる**もの。要点 (2199 行、逐語): **「完璧な規律を期待するのではなく、逸脱の発生速度より修正速度を上回らせること」**。

### ハーネス実体の対応表 (E-1 制約カスケード 4 層)

| Layer | ハーネスの実体 | 強制の形 | 本セッションの実測 |
|---|---|---|---|
| 1 命名規則 | skills / agents / rules の kebab-case、hooks の `hook_<timing>_<name>.sh` | **機械強制なし (規律のみ)** | skills 71/71・agents 32/32・rules 13/13 が適合 (逸脱 0)。**ただし `hooks/*.sh` 34 本中 2 件が規則外** (`hook_validate_claudemd.sh` = timing プレフィックス無し / `statusline.sh` = `hook_` 接頭辞無し) |
| 2 ディレクトリ構造 | `~/.claude` 直下 30 dir、`.docs/README.md` L221-238 の inventory 表 | `permissions.deny` 179 entry のうち**ハーネス自己保護は 13 entry** | `harness-policy.md` L30 は「6 系統」ラベルで 7 行の表を持つが、実 deny には `Edit(**/CLAUDE.local.md)` `Edit(~/.claude/.docs/hook-authoring-guide.md)` 等も在り 13 entry。同 doc L45 が「main では skills/** への **Bash 書込は ALLOWED**」と射程の限界を自己開示 |
| 3 メタデータ | SKILL.md / agent frontmatter、ADR frontmatter | `hook_post_skill_frontmatter_schema.sh` (PostToolUse・warn・`name:`/`description:` の有無のみ) / `verify-adr.sh` (`hook_pre_commit_adr_gate.sh` 経由・**block**) | frontmatter 保持率は 71/71・32/32。ただし**基準 validator 2 本が実在庫を落とす** (下記) |
| 4 参照実装 | **32 skill** が `references/`、`templates/`、`init_skill.py` の `SKILL_TEMPLATE`、`agent-definition-template.md` | 規律 (「参照実装」の名指し) | 「参照実装」として名指しされる skill は 4 件。**うち 1 件 (`nano-banana`) が `skills/` に不在 = リンク切れ 1/4** |

**記事の重み付けとの対比**: 記事が「最も強力」と呼ぶ Layer4 は、ハーネスでも量的には最も厚い (32/71 skill が `references/` を持ち、雛形生成器もある)。**ただし「厚い」ことと「健全である」ことは別**で、参照実装の名指し 4 件中 1 件がリンク切れという実測が出た (初稿はここを「リンク切れ 0」と誤って書き、Layer4 = 最厚を E-1 取り入れ済みの根拠にしていた。独立検証が反証)。逆に記事が最初に挙げる Layer1 は、ハーネスでは**最も薄い層** (機械強制ゼロ) になっている。

### 個別照合 (a) — Layer3 の反例狩り: 「基準はあるが効いていない」

記事 E-1.1 の (1)→(2) の受け渡しが成立しているかを、**validator を実在庫へ全件実走**して測った。

| validator | 実体パス | 対象 | 実走結果 (2026-08-04) | hook 配線 |
|---|---|---|---|---|
| `quick_validate.py` | `skills/authoring-skills/scripts/` | 71 skill | **PASS 44 / FAIL 27** | **無し** |
| `validate-agent-definition.sh` | `skills/authoring-agent-definitions/scripts/` | 32 agent | **PASS 11 / FAIL 21** | **無し** |
| `validate-knowledge.py` | `skills/establishing-knowledge-persistence/scripts/` | `.docs` 515 ファイル | **PASS 3 / FAIL 512** | **無し (理由開示済み)** |

**配線ゼロは対照実験つきで独立確認**: `grep -rln "quick_validate" hooks/ settings.json` → 0 件 / `grep -rln "validate-agent-definition" hooks/ settings.json` → 0 件。同じ検索器で `grep -rln "verify-adr" hooks/` は **3 ファイル** (`hook_pre_commit_adr_gate.sh` / `test-hook_pre_commit_adr_gate.sh` / `test-settings_harness_self_deny.sh`) を返す = **検索器の空振りではない**。なお `validate-knowledge` は `hook_pre_probe_before_persist.sh` に出現するが、**コメント内の言及であって呼出ではない** (実読で確認)。

**3 本は性質が違う。同列に扱わない**:

- `validate-knowledge.py` の 512 FAIL は **gap ではない**。`.docs/README.md` L27-51 が趣旨として「`.docs` は L2 知識永続化の 6 カテゴリの器ではない」「validator 未配線は怠慢ではなく、`.docs` が L2 の器でないことの必然的帰結」(L50 逐語) と理由を開示しており、**設計上の意図的な不適用**。さらに同 README L44-45 は「総数は `.docs/` にファイルが増えれば動く。**耐久する事実は「合格は 2 件のみ」の方**であって総数ではない」と先回りしている (本ログの自己是正②を参照)。
- 対して `quick_validate.py` / `validate-agent-definition.sh` の FAIL は **skill / agent という、その validator が本来の対象とする在庫**で起きている。ここは「器が違う」の弁明が立たない。

**FAIL の内訳 (`quick_validate.py`、27 件)**:

- 24 件 = `ALLOWED_PROPERTIES = {'name','description','license','allowed-tools','metadata','compatibility'}` の集合外キー。実在庫の frontmatter キー実測は `name:` 71 / `description:` 71 / `context:` 21 / `subagent:` 19 / `allowed-tools:` 6 / `model:` 5 / `argument-hint:` 2 / `agent:` 2 — **`context` `subagent` `model` `argument-hint` `agent` の 5 種が validator の許可集合に無い**。fork 系 skill は `context:` + `subagent:` を必ず使うため、fork 系は構造的にほぼ全滅する。
- YAML パース失敗 2 件 (`auditing-aio` / `authoring-claude-md`) — 原因は引用符なし description 中の `Trigger: ` (コロン+空白) で `yaml.safe_load` が ScannerError を出す。**ただしランタイムは読めている** (本セッションの利用可能 skill 一覧に両者とも description 付きで載っている) = **strict YAML パーサとランタイムパーサの寛容度の差**であって機能破損ではない。
- 山括弧禁止 1 件 (`capturing-ui-evidence`) — description が block scalar `>` で書かれ `<timestamp>` を含む。YAML としては正常 (独立パースで `parse OK`)。

**測定が言えること / 言えないこと (初稿の断定を後退させた点)**:

- **言える**: 基準と在庫は不一致である。不一致は 27/71 (skill) と 21/32 (agent)。
- **言えない**: 「基準側が悪い」か「在庫側が悪い」か。`context:` / `subagent:` はハーネス独自の運用キーだが、`ALLOWED_PROPERTIES` が上流 spec 由来である可能性を本監査は排除できていない。**どちらを正とするかは設計判断であって実測ではない** (初稿は「大半は基準側が追従していない」と確定形で書いた — 独立検証が OVERCLAIM と判定し、撤回)。
- **実害の形 (機序を訂正)**: `quick_validate.py` は `if len(sys.argv) != 2: sys.exit(1)` の**単一 skill 専用 CLI** であり、新規 skill に回しても既存 27 件の赤は視界に入らない (初稿の「既存 27 件が赤だから『赤が普通』になる」という機序は成立しない)。**実在する害は別の形** — 新規に fork 系 skill を作ると `context:` / `subagent:` を必ず使うため、**その skill は必ず赤になる**。どちらが偽か (基準か在庫か) は上記のとおり未決ゆえ「偽陽性」とは書かない (初稿の差し替え文が『正しく書いたのに偽陽性』と書いており、直前で撤回したはずの裁定を語彙を変えて復活させていた — 第2版の独立検証が検出)。言えるのは**基準を信頼できないまま放置されている**ところまで。

### 個別照合 (b) — E-1.1 ステップ3「定期的に除去する」の 2 ヶ月差分

**時計駆動の機構は 0 件**:

| 経路 | 実測コマンド | 結果 | 独立検証の可否 |
|---|---|---|---|
| system crontab | `crontab -l` | `crontab: no crontab for camone` (exit 1) | 可 (再現済み) |
| cloud routine | `CronList` ツール | `No scheduled jobs.` | **不可** (下記) |
| user LaunchAgents | `ls -la ~/Library/LaunchAgents/` | plist 5 個、**Claude 関連 0 件** (Adobe/Google のみ) | 可 (再現済み) |
| system LaunchAgents | `ls -la /Library/LaunchAgents/` | plist **9 個**、**Claude 関連 0 件** | 可 (再現済み) |

**`CronList` の限界を開示する**: 同ツールは step6 の独立検証者の環境では利用できず**再現できなかった**。`proposing-essence-updates` skill 同梱の `references/cron-setup.md` にも 2026-07-30 時点の「cloud routine の登録 = 無し (`CronList` = 0 件)」という記載があるが、**これはハーネス内部の自己申告であり、独立した第三者検証ではない**。「2 時点で一致」と言えるが、**2 時点とも同系統の自己申告**である点を明記する。crontab / LaunchAgents の 0 件は独立に再現されている。

**親バッチが指した `daemon` の正体 (「のみ」を撤回して内訳を提示)**: `~/.claude/daemon*` 一式は実在し空ではない (`daemon/` = control.key / dispatch/ / roster.json、`daemon.log` = 1100 行)。内訳を実測分類すると:

| 分類 | 行数 | 割合 |
|---|---|---|
| auth (トークン更新) | 650 | 59% |
| `[bg]` ワーカー管理 (`bg adopt` / `bg spare spawned` / `bg settled` 等) | 222 | 20% |
| version / upgrade / self-restart 系 | 143 | 13% |
| **essence / audit / gate / hook_pre_commit / proposing への言及** | **0** | **0%** |

対照実験: 既知陽性の `grep -cE 'auth' daemon.log` = 650 で検索器の稼働を確認した上での 0 件 (初稿は対照を `wc -l` = 非空で済ませており、`grounds-not-approval.md` が要求する「測定器がその対象を実際に読めたか」を満たしていなかった — 独立検証の指摘により撃ち直した)。

→ **daemon は現に worker プールを回している** (20% が `[bg]` 管理) ので、初稿の「中身は認証と自己更新**のみ**」は誤り。ただし**ハーネス監査 (essence / gate 等) への言及は 0 件**なので、**「ハーネス監査用のワーカーではない」という結論は維持**する。親バッチの「workers 空」は `roster.json` の観測時点スナップショットとしては正しく、log は稼働実績を示している — 両者は矛盾しない。

**差分 = 「未着手」から「意図的な代替 + 自己開示」への昇格**:

`hook_session_start_essence_proposal_reminder.sh` の冒頭コメント (逐語): **「cron の代替: 時計でなくセッション開始イベントで『金曜 かつ 前回通知から下限期間経過』を判定して通知する。SessionStart 駆動ゆえ金曜に起動した時のみ発火する (金曜に起動しない週はスキップ)。」**

つまり cron 不採用は**取りこぼしを承知の上での設計判断**として文書化されている。同型の代替がもう 1 本: `hook_session_start_hook_fire_report.sh` = hook-fire ledger の月次閾値レポート (issue #66 Sensor A)。

**代替は実際に発火している** (発火痕跡の実測):

- `.docs/essence-reminder-state` = `1785433940` → **2026-07-31 02:52 JST (金)**
- `references/cron-setup.md` の記載は「最終更新が 2026-07-24 (金)」 → **2 週連続の金曜発火**。**ただし 7/24 の脚は独立検証不能**: `.docs/essence-reminder-state` は `.gitignore` L24 で追跡外かつ値を 1 個しか保持しないため、7/24 の唯一の根拠は同じハーネス内部文書の観測記述 (2026-07-30 記) である。**7/31 のみが state 実値と mtime の 2 経路で再現可能**
- 本ログ作成時点 2026-08-04 (火) で、最終通知以降に到来した金曜は **0 日** (次の金曜は 8/7) = **取りこぼしの実測証拠は無い**。構造限界は宣言されているが実害はまだ観測されていない

**hook-fire ledger は生きている**: `2026-07.jsonl` = 1523 行 / 27 日分 / 17 hook 種別、`2026-08.jsonl` = 4 行 / 1 日分 (すべて 08-04)。**8 月が薄い機序 (2 度の是正を経た最終形)**: 行が出る主経路は hook の発火だが、初稿の「セッションが走っていない日は行が出ない」も、第2版の「**発火分岐でのみ**呼ばれる」も、**どちらも不正確だった** (後者は第2版の独立検証が反証)。7 月の sensor 分布を実測すると `recurrence 1330 / (フィールド無し) 118 / liveness 56 / judgment 19` で、**非発火由来が 75 行 (4.9%)** ある — `judgment` は `hooks/lib/line_limit_ratchet.sh` の `record` サブコマンド (人/Claude が手で打つ CLI。呼出元 hook が呼ぶのは `report` だけ) から、`liveness` は `hook_stop_plan_externalization_check.sh:222` が違反の有無に関わらず無条件に追記する分母から来る。正確には「**行 ≠ セッション**であり、大半 (95%) は hook 発火由来だが、手動 CLI と無条件 liveness も混じる」。7 月も 31 日中 27 日分しかない。

**除草の実行そのものは全て手動**: `essence-reviewing-orchestrator` = 「『3 領域 essence』…**でのみ発動**。高コスト fork×3 のため Skill ツール明示呼出を強く推奨」/ `accumulating-reviewer-feedback` = 「**明示呼出専用 (Skill ツール経由)・自動誘発なし**」。`review-harness` の description にはこの種の文言が無く、自然言語トリガー句のみ (= 明示呼出専用とも自動とも書いていない)。

→ **E-1.1 のステップ 3 は「促し (リマインダー) までは自動化され、除草の実行は人間の起動に依存する」構造**。記事の (3) が挙げる「週次レビュー、テンプレート修正、参照実装の更新」のうち、**週次レビューの起動トリガーだけが半自動化され、修正・更新は手動**。

### 個別照合 (c) — 記事が名指しする具体例 (kebab-case 検出 hook) の有無

**無い**。`grep -rn "kebab" hooks/` のヒット 4 件はすべてコメント / テストのラベル文字列で、**ファイル名の命名規則を検査するロジックではない** (対照: 同 grep は `hook_pre_read_secret_check.sh` 等を返しており検索器は機能している)。

kebab-case を検査するロジック自体は存在する — ただし **hook ではなく skill 同梱の手動スクリプト 2 本の内部** (`quick_validate.py` の `^[a-z0-9-]+$`、`validate-agent-definition.sh` の `^[a-z0-9][a-z0-9-]*[a-z0-9]?$`)。いずれも上記のとおり**自動発火しない**。

**親バッチとの未整合を明示する**: 親バッチ L43 は E 章「大半✅」の根拠に **「制約カスケード, kebab-case hook, 理由ベースrules」** を挙げている。だが本監査の実測では **kebab-case を検査する hook は存在しない**。2 ヶ月前の判定が誤っていたか、当時存在した機構が失われたかのいずれかで、**本監査は「現時点で存在しない」ことのみを確定**する (過去の存否は測っていない)。差分深掘りの成果として、親の根拠 1 件が現時点で成立しないことを記録する。

**反論の検討 (問題ゼロを疑う、の逆方向)**: 「遵守率 100% なら hook は不要では?」— 実測では skills/agents/rules の逸脱は 0 件。だが `hooks/*.sh` には **命名系統から外れる実物が 2 件現に在る**。ただし 2 件とも**理由の説明がつく**逸脱 (`hook_validate_claudemd.sh` は SessionStart `matcher='compact'` へ配線済み、`statusline.sh` は `statusLine` キー配下で hook ではない) であり、規約違反というより**規約の射程外**に近い。ゆえに severity は Medium 止まりとし、Critical/High には上げない。

### 個別照合 (d) — 無効化 skill への参照追従 (既知受容記録との突合)

> **フレーミングの訂正**: 初稿はこの節を「独立検証が発見した最大の収穫」と題し「経路が無い」と結論した。**どちらも撤回する** — 第2版の独立検証が、**同じ現象が 2026-07-09 に検出され、意識的に受容されていた記録**を提示したため。以下は既存記録との突合を含めた版。

**現象の広さ (本セッションの再計数)**: `skills-disabled/` の 28 本を live の `skills/**/*.md` へ全数照合すると (`skills/README.md` を除く)、**9 本が live の 15 ファイルから参照されている**: `article-explainer` (2) / `auditing-web-quality` (1) / `boris` (1) / `changelog` (3) / `directing-ai-development` (1) / `enumerating-verifiable-workflows` (1) / `linking-ticket-evidence` (2) / `nano-banana` (3) / `visualizing-article` (1)。

初稿はこれを「2 箇所」と数えていた。**母集団が狭かった** — 抽出を `grep -rn "参照実装"` に限ったため、`skills/authoring-skills/SKILL.md:110` の「**例: `nano-banana`**」のように別書式の名指しが構造的に見えなかった (第2版の独立検証が指摘)。

**参照実装として名指しされる skill は 4 件、うち 1 件がリンク切れ**:

| 名指しされた skill | `skills/` | `skills-disabled/` |
|---|---|---|
| `agent-teams-patterns` | 実在 | — |
| `commit` | 実在 | — |
| `designing-beautiful-frontends` | 実在 | — |
| **`nano-banana`** | **不在** | **実在** |

出所は `skills/authoring-skills/references/skill-type-taxonomy.md` L32 — **生きている skill 権威 doc が、無効化済み skill を「参照実装」として名指ししている**。

**`skills/README.md` にも同根の欠落**: README が記載する 19 skill のうち実在しない 9 件を `skills-disabled/` と突き合わせると、**8 件が現存**する:

| README 記載で `skills/` に不在 | `skills-disabled/` |
|---|---|
| `auditing-web-quality` / `boris` / `camone-ralph-loop` / `changelog` / `generating-gitignore` / `installing-project-mcps` / `setup-post-tool-use-hooks` / `translating-technical-articles` | **実在 (8 件)** |
| `skill-creator` | 不在 (真に消滅) |

→ 実態は「存在しない skill を案内している」ではなく **「無効化に参照側が追従していない」**。

**既存の受容記録 (これを開示せずに「発見」と書いたのが初稿の誤り)**: `.docs/essence-review-records/2026-07-09_134619_issue-84-skills-inventory-budget.md` は、未使用 20 本を `skills/ → skills-disabled/` へ純粋 rename で無効化した際の commit 前 self-eval であり、**独立 reviewer (`harness-essentials-reviewer` subagent) の CONDITIONAL GO を経て verdict 🟢 GO** で着地している。同記録 L64-69 が、本節の現象を**そのまま列挙している** (逐語):

> - `skills/establishing-knowledge-persistence/references/knowledge-categories.md:170,206` — `linking-ticket-evidence` (無効化) への言及 (参照 doc)。
> - `skills/visualizing-as-html/references/url-fetch-spec.md:41,136,138` — `/article-explainer` (無効化) 委譲の spec 記述 (実装 spec doc)。
> - doc/lineage の参照実装パス言及 (`authoring-skills` の nano-banana 参照実装例、`coordination-harness-integrity-fork` の lineage) — skill は skills-disabled/ に実在し**例示として無害**。
> - `skills/README.md` の一覧が無効化 20 本を含む (**再生成は別 chore**)。

つまり**検出も判断も済んでいた**。しかも実害寄りの 2 本 (`capturing-ui-evidence` / `visualizing-as-html`) は同じ commit で「**(現在無効化)**」注記へ修正されており、**追従の慣行そのものは実在する** (本セッションでも 3 箇所の注記を実測確認)。`managing-skills` の mv 手順に参照側更新が含まれていないのは事実だが、「経路が無い」は言い過ぎだった。

**それでも残差として立てる理由 (Lead 判断)**: ①受容の根拠は「**例示として無害**」だが、`skill-type-taxonomy.md:32` の書式は「例:」ではなく「**参照実装**: `nano-banana/SKILL.md` — CLI init + …完全構成」であり、**記事 Layer4 が「最も強力」と呼ぶ「これを踏襲せよ」の名指しそのもの**。読み手が辿れば不在に当たる。②受容から本監査まで **約 1 ヶ月** 経つが、`skills/README.md` の「再生成は別 chore」は未着手 (カバー率 71 分の 10 は本セッション実測)。③機械強制が無いため、**次に無効化した時も同じ状態が再生産される**。

**免責基準の非対称を是正する**: 本ログは `validate-knowledge.py` の 512 FAIL を「`.docs/README.md` に理由開示があるから gap ではない」と免責した。同じ基準を当てれば、無効化追従も「issue #84 に受容記録があるから gap ではない」となりうる。**両者を分けるのは「開示の有無」ではなく「受容時の前提が今も成り立つか」**とする — `.docs` が L2 の器でないことは今も真だが、「例示として無害」は `skill-type-taxonomy.md` の「参照実装」書式については今も真とは言い切れない。この線引き自体が Lead 判断であることを明記する。

### 記事超え点

- **限界の自己開示が制度化されている**: `.docs/README.md` は inventory 表の直後 (L240-241) に「この表は手書きの inventory であり、dir を追加しても機械検出されない (既知の限界)」と明記。`harness-policy.md` L45 は deny の射程が Edit ツール経路までで Bash 書込は通ることを明記。**記事は制約を敷く方法を説くが、敷いた制約の穴を宣言する規律までは説いていない**。
- **ドリフト計器の存在**: hook-fire ledger (17 hook 種別・1523 行/月) は、記事の 3 ステップには無い「**逸脱の発生を時系列で計量する**」層。記事の「増殖速度 vs 除草速度」を実際に数えられる形にしている。

> **初稿から撤回した記事超え点**: 「`hooks/rules/*.json` だけを deny から個別除外して粒度差をつけており、記事の 4 層モデルには無い『同一ディレクトリ内で書込可否を分ける』層を持つ」— **実体は逆だった**。`permissions` を機械分解すると `hooks/rules` に言及する entry は **allow 0 / ask 0 / deny 3** で、`Edit(~/.claude/hooks/**)` で既に deny 済みの配下を**さらに重ねて deny** しているだけ。carve-out (除外) は存在しない。独立検証の指摘により撤回。

### 残差 / 改善候補

- **[Medium] 基準 (validator) 2 本が実在庫を落としたまま hook 未配線で固着**: `quick_validate.py` = 71 skill 中 27 FAIL / `validate-agent-definition.sh` = 32 agent 中 21 FAIL、両者とも `hooks/` `settings.json` からの参照 0 件 (対照実験済み)。記事 E-1.1 の (1)→(2) の受け渡しが切れている箇所。**残差**: どちらを正とするか (基準を実態へ合わせるのか、在庫を基準へ合わせるのか) の設計判断が先に要る。判断を経ずに配線すると 48 件が赤で止まる。
- **[Medium — 新発見ではなく既知 Low の再評価提案] 無効化 skill への参照追従に機械強制が無い**: 無効化 28 本のうち **9 本が live の 15 ファイルから参照**されており、うち `skill-type-taxonomy.md:32` は `nano-banana` を「参照実装」として名指ししている。**issue #84 (2026-07-09) が同じ現象を検出し 🟢 Low として意識的に受容済み** — 本項はその受容の再評価提案であって新発見ではない。**Medium へ上げる根拠**: ①受容理由「例示として無害」が「参照実装」書式には当てはまりにくい (記事 Layer4 の核心に触れる) ②受容から約 1 ヶ月、`skills/README.md` の再生成 chore は未着手 (カバー率 10/71) ③機械強制が無いため次の無効化でも再生産される。**残差**: mv 時に参照側を検査する lint、あるいは README を機械生成へ寄せる。**トレードオフ**: 手作業の追従慣行は実在する (「(現在無効化)」注記 3 箇所を実測) ため、Critical/High には上げない。
- **[Medium] 記事が名指しする「ファイル名の命名規則を検査する hook」が存在しない**: kebab-case 検査ロジックは 2 本の手動スクリプト内にあるだけで、PreToolUse/PostToolUse のどこからも発火しない。現在の遵守率は skills/agents/rules で 100% だが、**強制しているのは規律であって機構ではない**。`hooks/*.sh` には規則外 2 件が現存する (ただし 2 件とも射程外に近い逸脱)。加えて**親バッチ L43 がこの hook の存在を E 章 ✅ の根拠に挙げていた**ため、判定の根拠 1 件が現時点で成立しない。
- **[Medium] E-1.1 ステップ3「定期的に除去する」は促しまでで、除草の実行が手動**: 時計駆動 0 件 (crontab / LaunchAgents ×2 は独立再現済み、`CronList` は独立検証不能で自己申告 2 時点のみ)。代替の SessionStart 駆動は実際に 2 週連続で発火しており機能しているが、「金曜に起動しない週はスキップ」は hook 自身が宣言する構造限界。監査 skill 3 本はいずれも人間の明示起動に依存する。**トレードオフの弁明**: cron を採らないのは「時計で起こしても人がいなければ結果を読む者がいない」という設計判断として文書化されており、単なる未着手ではない。ゆえに Critical/High には上げない。
- **[Medium — 第2版の独立検証が誘発した新規発見] agent 名の判定器に検出漏れがある**: `validate-agent-definition.sh:71` の実正規表現 `^name:[[:space:]]*[a-z0-9][a-z0-9-]*[a-z0-9]?[[:space:]]*$` へ逆変異 6 種を投入すると、**`trailing-` と `double--hyphen` の 2 種が素通りする** (REJECT されたのは `Camel-Case` / `-leading` / `with_underscore` / `UPPER` の 4 種)。本ログが「遵守率 100%」の計測に使った判定器 `^[a-z0-9]+(-[a-z0-9]+)*$` は 6/6 を落とすので、**両者は同じ規則を実装していない**。残差③ (kebab hook 不在) と合わせると、Layer1 は「**機械強制が無く、手動で回せる検査の側にも穴がある**」状態。**現在の在庫 32 agent はすべて適合しており実害は出ていない** (逸脱 0 を実測)。**残差**: `^[a-z0-9]+(-[a-z0-9]+)*$` 相当へ揃える。
- **[Low] `.docs/knowledge/` 不在の扱いは gap ではない**: 空ディレクトリのまま 2 ヶ月放置され issue #138 で `rmdir` 済み。`.docs/README.md` L53「先に器を作っても埋まらない (実証済)」+ L66「器は「使い始める時に生やす」 (adopt-on-use)」が設計判断を記録している。記事の Layer3 を満たさない状態に見えるが、**意図的な不採用**として記録済みゆえ残差に数えない (本項は「gap でない」ことの明示記録)。

判定: **E-1「制約が品質を生む」= 取り入れ済み** (4 層すべてに実体が実在する。ただし**機械強制まで到達しているのは Layer2 と Layer3 の一部**で、Layer1 は規律のみ・Layer4 は名指しの規律のみ)。**E-1.1「ドリフトを前提に定期的に掃除する」= 部分 gap** — (1) 基準の定義と (2) hook 検知は実在するが、**基準 2 本が (2) へ渡らないまま固着**し、**無効化 skill への参照追従は 9 本 15 ファイルで未解消 (既知・受容済み・機械強制なし)**、(3) 定期除去は促しのみ自動・実行は手動で、時計駆動の機構は 0 件。

## step6 独立検証の記録

**2 ラウンド回した。両方 CONDITIONAL。倒れた主張は通算 28 件** (第1ラウンド 16 / 第2ラウンド 12)。各ラウンドとも fresh な read-only reviewer (`code-reviewer` 型、`~/.claude` を full 読解・claim ごと突合・改修禁止) へ、**判定ログ本文と読み取り権限のみ**を渡した (著者の推論は渡さない = アンカリング防止)。第2ラウンドには**第1ラウンドの指摘内容も渡していない** — 第2版を初見として独立に測らせるため。

### 第1ラウンド (初稿に対して)

**verdict: CONDITIONAL** — CONFIRMED 20 クラスタ / **UNTRACEABLE 8** / **OVERCLAIM 8**、是正 7 条件。

**持ちこたえた側 (CONFIRMED の要点)**: 検証者は「特に疑え」と指定した 13 項目のうち、遵守率 100% 系と 0 件系を**自分の手で撃ち直して再現**した。kebab 判定器には**逆変異 6 種** (`Camel-Case` / `trailing-` / `double--hyphen` / `-leading` / `with_underscore` / `UPPER`) を投入して全件 reject を確認。validator 3 本の実走値・frontmatter キー出現数 8 種・ledger の日別分布・逐語引用 (記事 4 箇所 + `.docs/README.md` + hook コメント + 監査 skill 3 本) はいずれも一致した。自己是正①については、検証者が**偽測定を再現して「matched 32 / 0」を実際に出し**、指摘が正しいことを確認している。

**倒れた側と是正 (7 条件、全て適用済み)**:

| # | 指摘 | 初稿 | 是正後 (自分でも再測) |
|---|---|---|---|
| U-1 | 参照実装のリンク切れ | 「4 件すべてリンク切れ 0」 | **4 件中 1 件がリンク切れ** (`nano-banana` は `skills-disabled/` にのみ存在)。Layer4 の判定根拠を書き直し、**新規残差 [Medium] として独立項目化** |
| U-2 | `hooks/rules` の deny | 「deny から個別除外して粒度差」 | **実体は重複 deny** (allow 0 / ask 0 / deny 3)。記事超え点から**撤回**し、撤回の経緯を明記 |
| U-3 | `references/` を持つ skill 数 | 29 | **32** (`ls` 方式・`find` 方式の 2 経路で一致) |
| U-4 | system LaunchAgents の plist 数 | 8 | **9** |
| U-5 | `verify-adr` の grep ヒット数 | 2 (同一文書内で 3 とも書いていた) | **3** (ファイル名まで列挙) |
| U-6 | 自己是正②の出典行 | `.docs/README.md` L46-48 | **L44-45** (L46 は空行、L47-48 は別内容) |
| U-7 | `.docs` が L2 の器でない旨の引用 | 「」で括った第 1 文が逐語非存在 | 「**趣旨として**」へ改め、逐語一致する第 2 文 (L50) のみ引用符で括る |
| O-1 | validator の実害機序 | 「既存 27 件が赤だから赤が普通になる」 | `quick_validate.py` は**単一 skill 専用 CLI** ゆえ機序が成立しない。**実害は「新規 fork 系は必ず赤になる」**へ差し替え (第2ラウンドで「偽陽性」の語をさらに撤回 → O-B) |
| O-2 | 基準と在庫のどちらが正か | 「大半は基準側が追従していない」(確定形) | **測定は不一致を示すのみ**。どちらを正とするかは設計判断であって実測でないと明記し、残差の処方も「判断が先に要る」へ後退 |
| O-3 | `daemon.log` の内訳 | 「認証と自己更新のみ」 | **auth 650 / `[bg]` worker 管理 222 / version 系 143 / 監査系 0** の内訳を提示し「のみ」を撤回。結論 (監査 daemon ではない) は維持 |
| O-4 | 陰性の対照実験 | `wc -l` = 非空 | **既知陽性 (`grep -c auth` = 650) で撃ち直し**。`grounds-not-approval.md` が要求するのは「測定器が対象を読めたか」であってファイルの非空ではない |
| O-5 | ledger が薄い機序 | 「セッションが走っていない日は行が出ない」 | 「`ledger_append.sh` は**発火分岐でのみ**呼ばれる」へ差し替え → **この差し替え自体が第2ラウンドで反証された** (U-A)。最終形は「大半は発火由来だが手動 CLI と無条件 liveness で 75 行 4.9%」 |
| O-6 | `skills-disabled` 追従欠落の severity | Low (「存在しない skill を案内」) | **Medium へ昇格**。ただし第2ラウンドで「新発見」フレーミングと計数 (2 箇所) を撤回 → O-C / U-D |
| O-7 | 「本セッション中に 2 行 → 4 行へ増えた」 | 独立検証不能な観察 | 当該主張を**本文から削除** |
| U-8 | `CronList` の 0 件 | 「2 時点で一致」 | **独立検証不能**であること、2 時点とも**ハーネス内部の自己申告**であることを開示 |
| O-8 | 見出しの「逸脱ゼロ」 | skills/agents/rules のみ見て断定 | 見出し行に **`hooks/*.sh` の規則外 2 件を併記** |
| 付随 | 親バッチとの未整合 | 未言及 | 親バッチ L43 が「kebab-case hook」を ✅ の根拠に挙げているが**現時点で存在しない**ことを個別照合 (c) に明記 |
| 付随 | `permissions.deny` の系統数 | 「6 系統」+ 括弧内 8 項目 | 実 deny の**ハーネス自己保護 13 entry** を計測し、`harness-policy.md` のラベルとの差を注記 |

### 第2ラウンド (第2版に対して — 別の fresh reviewer、前ラウンドの指摘は非開示)

**verdict: CONDITIONAL** — **UNTRACEABLE 6 / OVERCLAIM 6**。「18 指定項目のうち計数・逐語系は全一致、捏造ゼロ」と評価された上で、**機序・フレーミング・一般化の側が倒れた**。

| # | 指摘 | 第2版 | 是正後 (自分でも再測) |
|---|---|---|---|
| U-A | ledger の機序 | 「発火分岐で**のみ**呼ばれる」 | **反証された**。sensor 分布 `recurrence 1330 / なし 118 / liveness 56 / judgment 19` = 非発火由来 75 行 (4.9%)。**O-5 で是正した機序が、さらに不正確だった** |
| U-B | `adopt-on-use` の引用行 | L66 | 「先に器を作っても埋まらない (実証済)」は **L53**、L66 は別見出し。**U-6 と同じクラスの誤りが別箇所に残っていた** |
| U-C | 参照実装の抽出母集団 | `grep "参照実装"` のみ | `authoring-skills/SKILL.md:110` の「**例: `nano-banana`**」を構造的に取りこぼし。母集団を全 `skills-disabled/` 照合へ拡大 |
| U-D | 現象の計数 | 「2 箇所」 | **無効化 28 本中 9 本が live 15 ファイルから参照** (本セッション再測) |
| U-E | 「第1〜6波で 24 PR」 | 確定形 | **どの数え方でも再現不能** (`git log --merges` = 141、波記録合算 ≈ 29) → **数値を書かない**形へ |
| U-F | step6 表の 8/8 | U-7 / O-7 が文書内に不在 | **本表に U-7 / O-7 を追加**して内訳を辿れるようにした |
| O-A | 「手順を書かなかった 0 件が死んだ」 | 運用規律へ一般化 | **n=1 の事後パターン。撤回**。検証者が反証データを提示 — 第2版にも手順なしの 0 件主張が 2 件あり (逸脱 0 / allow 0・ask 0)、**どちらも真だった** |
| O-B | 「正しく書いたのに偽陽性」 | O-1 の差し替え文 | **O-2 で撤回した裁定 (在庫側が正) を語彙を変えて密輸していた**。「fork 系は必ず赤になる」まで後退 |
| O-C | 無効化追従の「経路が無い」「最大の収穫」 | 新発見として提示 | **issue #84 (2026-07-09) が検出・🟢 Low で受容済み**。フレーミングを撤回し受容記録を開示。**免責基準の非対称** (`validate-knowledge.py` だけ既存開示を探した) も是正 |
| O-D | 「実発火 2 週連続を確認」 | 実測ラベル | **7/24 の脚は他文書からの転記**。state は追跡外かつ値 1 個保持ゆえ再現不能 — 開示を追加 |
| O-E | 「4 層すべてに機構実在」 | 見出し + サマリー | **同文書 L55 の表 (Layer1 = 機械強制なし) と矛盾**。「実体が実在・機械強制は Layer2 と Layer3 の一部」へ |
| O-F | 「判定器は逆変異 6 種で全件 reject」 | 器を明示せず | 計測に使った器と**ハーネス同梱の agent 判定器は別物**。後者は `trailing-` / `double--hyphen` を素通り → **残差⑤として新規起票** |

### 2 ラウンドから得たこと (一般化はしない)

第1ラウンド後に「手順を書かなかった 0 件が死んだ」を運用規律として立てようとしたが、**第2ラウンドが n=1 の事後パターンだと反証した** (第2版の手順なし 0 件 2 件はどちらも真)。実際に効いていたのは手順の有無ではなく「**測定器がその対象を読めたかを対照で確かめたか**」で、それは `grounds-not-approval.md` が既に持つ条項。**新しい規律を立てる根拠は無い** — この「規律を作りたくなる衝動」自体が、本ログが O-2 で戒めた「測定から言えないことへ踏み出す」の同型だった。

**2 ラウンド回して分かった構造**: 第1ラウンドで倒れたのは**数値と事実**が中心、第2ラウンドで倒れたのは**機序・フレーミング・一般化**が中心だった。数値の誤りは 1 回の独立検証で払底したが、**「どう解釈するか」の誤りは 2 回目でしか出てこなかった**。fail-close ゲートを 1 回で打ち切っていたら、O-C (既存受容記録の未開示) は永久に見つからなかった。

## 自己是正の記録 (判定確定までに捕まえた測定器の失敗 4 件)

本 skill の Gotcha「『実測』ラベルは自動で真にならない」と `grounds-not-approval.md` の陰性検証条項に照らし、**判定に載る前に潰した誤り**を開示する。うち 3 件は自己検知、1 件は独立検証が発見した。

1. **[自己検知] 測定器の空振りで数値が反転**: `validate-agent-definition.sh` を全 32 agent へ当てた一次測定で「**FAIL 0**」と出た。だが生出力を確認したところ `code-reviewer.md` は `=== FAIL: 3 件のエラー ===` を返していた。原因は合否判定の `grep -E "PASS|valid|✓|OK"` が、**推奨セクション警告の文面「⚠ ## Behavioral Traits 推奨 (任意、欠落しても PASS)」に含まれる "PASS" を掴んでいた**こと。exit code 基準へ切り替えて撃ち直し **FAIL 21 / PASS 11** と判明。step6 の検証者が偽測定を再現し「matched 32 / 0」を実際に出して裏付けた。
2. **[自己検知・撤回] 出典未確認の過大主張**: 中間報告で「`validate-knowledge.py` の失敗が 319 (7/12) → 512 (8/4) へ増えた = 除草速度より増殖速度が上回っている実測」と述べた。その後 `.docs/README.md` を直接読んだところ、**「総数は `.docs/` にファイルが増えれば動く。耐久する事実は『合格は 2 件のみ』の方であって総数ではない」と先回りで開示済み**だった。耐久値 (合格数) は 2 (7/12) → 3 (8/4) でほぼ不変であり、**総数の増加は非 L2 文書が増えただけ = ドリフトの証拠ではない**。当該主張は撤回した。**出典を当たる前に「実測」と称して因果を語ったのが誤り**。
3. **[自己検知] 抽出器の空振り**: `skills/README.md` の記載 skill 名を最初に抽出した際、正規表現が README の実フォーマット (`**ディレクトリ**: \`name/\``) と噛み合わず **19 件のところ 1 件しか拾えなかった**。この状態で「乖離 0 件」と結論しかけたが、抽出件数の異常に気づいて実フォーマットを確認し、抽出器を直して測り直した。
4. **[自己検知] `grep -E` に `\|` を渡してエスケープが literal 化**: `daemon.log` の内訳を測る際、`grep -cE 'version\|upgrade\|self-restart'` と書いたため **literal 文字列 `version|upgrade|self-restart` を探して 0 件**を返した。同じ書き方をしていた `'essence\|audit\|gate\|hook_pre_commit'` も 0 件を返しており、**判定の根拠になる「監査系への言及 0 件」が偽の 0 になりかけた**。ERE として正しく書き直したところ version 系は 143 件に変わり、監査系は**本物の 0 件**だった (対照 `grep -cE 'auth'` = 650)。**同一セッション内で 4 回目の測定器失敗**であり、`grounds-not-approval.md` の「測定器がその対象を実際に読めたか」条項が繰り返し効いた。

**さらに [独立検証が発見・第1ラウンド]**: 自己是正②の出典行が **L46-48 と誤記**されていた (正: L44-45)。**捏造を潰した記録それ自体の出典が未確認**という、本ログが戒めている型の再発。

**さらに [独立検証が発見・第2ラウンド]**: 上を是正した第2版にも、**同じクラスの出典行ずれが別箇所に残っていた** — 「先に器を作っても埋まらない (実証済)」の行番号を L66 と書いたが正は **L53** (L66 は別見出し)。**1 箇所を直しても同種の誤りは他所に残る**という、grep での横展開を怠った実例。

**同じく第2ラウンド**: `grep -E` の `\|` literal 化 (自己是正④) と**同型の「測定器を作り替えたら別の穴が開く」失敗**が機序の側でも起きた。初稿の ledger 機序を第2版で「発火分岐で**のみ**」へ差し替えたが、**その差し替え自体が不正確**だった (sensor 分布の実測で非発火由来 75 行を確認)。**是正が新しい誤りを持ち込む**ことがある以上、是正版こそ独立検証に掛け直す必要がある — step6 を 2 ラウンド回した価値はここに出た。

**再発計数の扱い**: 5 件とも `grounds-not-approval.md` Gotcha の同族 (確定形の値を、測定器の健全性や出典を確かめずに書く) だが、**いずれも永続成果物へ着地する前に潰した**ため、同 rule の「再発マーク」には計上しない (マークは「日付の裏が取れた再発ぶんだけ」= 着地した失敗が対象)。本節は L0 の揮発を防ぐ記録として残す。

## HITL 処分記録 (2026-08-06 追記)

残差 4 件 (Medium) の扱いを HITL で裁定した (AskUserQuestion 実結果 = 「選抜起票」。gh で独立確認済み):

| 残差 | 処分 | 痕跡 |
|---|---|---|
| ① validator 2 本が実在庫を落としたまま未配線 | **新規起票 #333** (正の決定 → 是正 → warn 配線の順序固定) | `dendedev/claude-harness#333` |
| ② 無効化 skill への参照追従 | **新規 issue とせず #84 へ再評価コメント** (既知・受容済みの再評価という性格に合わせる) | `#84` comment `issuecomment-5200317860` |
| ③ 記事名指しの kebab-case hook 不在 | ~~**見送り** (2026-08-06。理由: 遵守率 100% 実測 + 昇格ラダーに照らし根拠なし)~~ → **撤回・起票 #338** (2026-08-08、Dende 裁定)。撤回の根拠: (a) 昇格ラダーは事後反応型で、E-1.1 の前提「ドリフトは発生を前提」の予防文脈を覆わない (b) 現状は検出器ゼロ — 最初の 1 違反は在庫模倣で増幅されるまで不可視 (規律の 100% は「最初の違反が起きるまで」の 100%) | `dendedev/claude-harness#338` |
| ④ agent 判定器の正規表現の検出漏れ | **新規起票 #334** (正規表現修正 + 逆変異テスト) | `dendedev/claude-harness#334` |

なお E-1 本体 (カスケード 4 層) からの改修 issue は 0 件 — カスケードは「作る機能」でなく実物に埋め込む設計姿勢であり、実物は取り入れ済みのため。改修対象になったのは全て E-1.1 (維持機構) の側。

## 関連ファイル

- `~/.claude/skills/authoring-skills/scripts/quick_validate.py` — 許可キー集合の定義と全 71 skill への実走
- `~/.claude/skills/authoring-skills/references/skill-type-taxonomy.md` — 参照実装の名指し 4 件 (L14/23/32/41)、うち L32 が `nano-banana` (リンク切れ)
- `~/.claude/skills/authoring-agent-definitions/scripts/validate-agent-definition.sh` — 全 32 agent への実走 (exit code 基準)
- `~/.claude/skills/establishing-knowledge-persistence/scripts/validate-knowledge.py` — `.docs` 515 ファイルへの実走
- `~/.claude/.docs/README.md` — validator 未配線の理由開示 (L27-51、耐久値の先回り開示は L44-45)、inventory 表と既知の限界 (L221-241)、adopt-on-use (L66)
- `~/.claude/settings.json` — hooks 配線全件、`permissions.deny` 179 entry (ハーネス自己保護 13)
- `~/.claude/hooks/hook_session_start_essence_proposal_reminder.sh` — cron 代替の設計判断の逐語宣言
- `~/.claude/hooks/hook_session_start_hook_fire_report.sh` — ledger 月次集計 (issue #66 Sensor A)
- `~/.claude/hooks/lib/ledger_append.sh` — ledger 追記の発火条件 (実読)
- `~/.claude/hooks/hook_post_skill_frontmatter_schema.sh` — 配線済みの frontmatter 検査 (warn・キー有無のみ)
- `~/.claude/.docs/hook-fire-ledger/2026-07.jsonl` / `2026-08.jsonl` — 発火台帳の日別分布
- `~/.claude/.docs/essence-reminder-state` — 金曜リマインダーの発火痕跡
- `~/.claude/skills/proposing-essence-updates/references/cron-setup.md` — cron 未登録の自己実測 (2026-07-30)
- `~/.claude/skills/README.md` / `~/.claude/skills-disabled/` — 在庫ドリフトと無効化追従の実測対象 (28 本を live 全照合)
- `~/.claude/.docs/essence-review-records/2026-07-09_134619_issue-84-skills-inventory-budget.md` — **無効化追従の既知受容記録** (L64-69。独立 reviewer の CONDITIONAL GO を経て 🟢 GO)
- `~/.claude/skills/authoring-skills/SKILL.md` — L110 の「例: `nano-banana`」(本ログの初回抽出が取りこぼした別書式の名指し)
- `~/.claude/hooks/lib/line_limit_ratchet.sh` — `record` サブコマンド (ledger の非発火由来 `judgment` 行の出所)
- `~/.claude/hooks/hook_stop_plan_externalization_check.sh` — L222 の無条件 `liveness` 追記
- `~/.claude/daemon.log` — 内訳分類 (auth / `[bg]` / version / 監査系 0)
- `~/.claude/.docs/progressive-disclosure/harness-policy.md` — deny の射程と限界の自己開示 (L30 表 / L45)

## 出典

- 記事本文: `.docs/references/260405_*/text.md` (E 章導入 = 2134〜2136行 / E-1 = 2138〜2172行 / E-1.1 = 2174〜2199行)。索引 `~/.claude/.docs/references/BIBLIOGRAPHY.md` (番号 260405)
- 親バッチ判定: `.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md` (L43 = E 章の 1 行判定 / L68 = E-1.1 の △ 内訳)
- 直前の深掘り: `.docs/logs/shared/2026-07-24_s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive.md` (取り入れフェーズ第16弾)
- 陰性検証条項と確定形数値の規律: `~/.claude/rules/grounds-not-approval.md` の Gotcha 2 条
