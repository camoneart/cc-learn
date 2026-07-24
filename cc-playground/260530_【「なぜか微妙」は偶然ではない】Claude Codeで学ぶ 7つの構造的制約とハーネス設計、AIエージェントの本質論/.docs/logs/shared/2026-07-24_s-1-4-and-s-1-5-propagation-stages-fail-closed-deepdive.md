---
date: 2026-07-24 21:20:00
type: study
topic: s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive
session: S-1.4「防御は伝播停止の段階で評価する」+ S-1.5「安全側をデフォルトにする (fail-closed)」合同深掘り (取り入れフェーズ第16弾 — 両者薄い (各11行) ため合同、handoff 指示どおり)
related_article: .docs/references/260405_【「なぜか微妙」は偶然ではない】Claude Codeで学ぶ 7つの構造的制約とハーネス設計、AIエージェントの本質論/text.md (S-1.4 = 2112〜2122行 / S-1.5 = 2124〜2130行、関連: S-1.3 Tier表 2100〜2110 / K-1.2 永続化 / S-1.1 中継 / E-1 開始 2138)
related_skill: [harness-adoption-audit, explain-in-html, logging]
related_log_ids: [2026-07-24_s-1-2-and-s-1-3-machine-safety-least-privilege-deepdive, 2026-07-24_s-1-1-memory-origin-tracking-deepdive, 2026-07-23_s-1-trust-boundary-three-layer-gate-deepdive, 2026-07-20_s-chapter-trust-boundary-permissions-adoption-check, 2026-07-24_pr-225-227-merge-gate-review]
related_log: [.docs/logs/shared/2026-07-24_s-1-2-and-s-1-3-machine-safety-least-privilege-deepdive.md, .docs/logs/shared/2026-07-20_s-chapter-trust-boundary-permissions-adoption-check.md, .docs/logs/shared/2026-07-24_pr-225-227-merge-gate-review.md]
---

# S-1.4「防御は伝播停止の段階で評価する」+ S-1.5「安全側をデフォルトにする (fail-closed)」合同深掘り — 判定: 両者とも取り入れ済み · 記事超え = 「配備≠実効」の live 実測文化 + fail-closed/open の部品役割規約 · 反例狩りの新規収穫 = block gate 5 本の jq 依存 fail-open (うち 1 本は偽の fail-closed 宣言つき、1 本は本体書込 guard の武装時・自己規約との乖離・未追跡) + 発火観測を歪める集計器の locale 融合バグ

> 核心の構造事実: **S-1.4** = 伝播 4 段 (曝露→永続化→中継→実行) の全段に機構実在。曝露 = 検疫タグ hook 2 本 (warn) + CLAUDE.md Prohibition inline / 永続化 = probe hook (warn) + commit gate 3 本 + 保護パス書込 deny (block) / 中継 = agent tools 絞り 28/32 + fork 分離 21 skill (構造) / 実行 = allow 41 · ask 30 · deny 177 + PreToolUse block hook 群。強制度が段で勾配する (曝露=warn は「曝露率ほぼ100%で止められない」の記事前提と整合、実行=block 最厚は「実行遮断が最重要」と一致)。**中継段だけ防御的機械層ゼロ** (SubagentStop に hook 4 本あるが全て装飾通知 = echo/効果音/osascript/say で検疫・中和なし、中継データを検査する rule 0 hit — 第14弾で記録済みの platform 依存既知)。**S-1.5** = hook-authoring-guide「根因2」が『防御 gate は fail-closed / sensor は fail-open』を規約化 + 設計外デフォルトは `defaultMode: "default"` (グレーゾーンは人間に聞く) が現行実装。「auto は fail-open」の自己分析は 2026-05-29 log に実在するが、同 log の決定は「auto のまま + surgical ask」= 切替を退けた記録で、実際の default 切替は別 repo (pixel-leap) の同日 log Finding E に「ask はモード依存で auto では空回り」という機構発見として残る (初稿の「移行記録」出典は step6 第 2 ラウンドが誤りと確定し差し替え)。**反例狩りの新規収穫 = block gate が jq 不在の異常系で検査を全スキップして approve/exit 0 に落ちる実装が 5 本** (pre_commands / stop_words / essence_gate / settings_churn / worktree_bash_write_guard 武装時) — read_secret は同型の穴を是正済 (L14-16 是正記録) なのに水平展開されておらず、自己規約 (根因2「入力異常時の exit 0 禁止、特に secret/権限系」) と乖離、全 97 issue (closed 含む) のどれにも未追跡 = **新規 Medium**。**とりわけ settings_churn は行50 コメントで「jq 不在→block (fail-closed)」と自称するが、行121 の `[ "$TOOL_NAME" != "Bash" ] && exit 0` が先に素通りするため行306 の deny normalizer-unavailable に構造的に到達しない = 偽の fail-closed 宣言 (dead code)**。4 本目 (偽宣言) は step6 第 1 ラウンド、5 本目 (武装 guard) は第 2 ラウンドが捕捉 (初稿は settings_churn を live で撃たず行50 コメントを額面通り転記し、「全 gate 走査」も 6/13 の over-claim だった = 本ログ自身が掲げた「配備≠実効/live で撃て」規律の違反を独立検証が 2 度是正した実例)。

## 概要

取り入れフェーズ第16弾。S-1.4 (11 行)・S-1.5 (7 行) はともに薄いため handoff 指示どおり合同 1 本。親バッチ (07-20) は S-1.4「✅ 多層で各段階に防護」/ S-1.5「✅ 記事超え (fail-closed/open の使い分け)」と判定済み。本深掘りの差分焦点は (a) 4 段のどこにも掛からず実行へ届く伝播経路の反例狩り、(b)「fail-closed 宣言」と「異常系の実装」の乖離の系統的実測 (= gate 自身が死んだ時に fail-open へ倒れないか。第15弾が捕捉した read_secret 旧実装 fail-open の前科が他 gate に残っていないか)。ハーネス実測 (step3) は read-only Explore (scan-s14-s15) に委譲、512 行の構造化 scan を回収。**引継ぎ規律 (S-1.1 由来) を適用**: gap 判定前に (a) essence 棄却候補表 (b) 既存 issue #220〜#226 (c) 意図的相違/開示済み制約を照合した。

## 内容

### note の定義

- **S-1.4** (2112〜2122): たとえ = 水漏れの階層。外部入力の曝露は避けられない (曝露率ほぼ100%) — 大事なのは入った後どの段階で伝播を止めるか。4 段 = 曝露 → 永続化 (→K-1.2) → 中継 (→S-1.1) → 実行 (→S-1.3)。**最終段階 (実行) の遮断が最も重要**。入口で弾くことだけに頼らず各段階に防護層。
- **S-1.5** (2124〜2130): たとえ = スマホ通知のデフォルトON。設定し忘れた時に許可される (fail-open) か拒否される (fail-closed) かで安全性が根本的に違う。S-1.3 の Tier 分類に応じてデフォルト切替 (Tier1 読取=allow / Tier2-3 ローカル変更=都度確認 / Tier4 不可逆=拒否)。**S-1.3 が「何を許可するか」を設計し、S-1.5 が「設計していないものをどう扱うか」を決める**。

### S-1.4 実測照合 — 4 段の防護マップ (scan A 全文より)

| 段 | 機構 (実体) | 強制度 |
|---|---|---|
| 曝露 | `hook_post_external_input_notify.sh` (WebFetch/WebSearch へ検疫タグ注入) / `hook_post_mcp_notify.sh` (mcp__* へ同上) / CLAUDE.md:43-44 Prohibition 2 行 (毎応答 inline、tool-routing.md:6 が配置理由明文) / skill ローカル再宣言 (harness-adoption-audit:40,71) | warn (注入) + 規律 |
| 永続化 | `hook_pre_probe_before_persist.sh` (knowledge/handoff 書込前 3 probe 注入) / commit gate 3 本 (adr / essence / settings-churn = block) / `保護パスへのbash書込禁止` rule (redirect・tee・cp 等 5 regex = block) / plans redirect / hardcode hygiene (block) | warn + block 混成 |
| 中継 | agent 32 本中 28 本 tools 明示絞り (監査役 ~24 本 Edit/Write 非付与、article-summarizer は WebFetch 1 個) / `context: fork` 21 skill (frontmatter 実測、fresh context 分離) / three-elements の Task matcher hook (phase 不明時 fail-closed) | 構造 (block/warn の枠外) |
| 実行 | permissions allow 41 / ask 30 / deny 177 (Bash 16 本 = sudo・rm・push・reset・rebase・curl 等の不可逆系) / PreToolUse block hook 群 (`hook_pre_commands` 9 rule + `read_secret` + worktree guard 2 枚 + generated_comment 他) / defaultMode `default` | block 最厚 |

- **記事との一致**: 「実行遮断が最も重要」→ 実行段が最厚 (deny 177 + block hook 群 + ask 30 の HITL 層)。「各段階に防護層」→ 4 段全てに実体あり。「入口で弾くことだけに頼らない」→ 曝露段は warn (止めない) で、下流の永続化 block・実行 block が本丸という設計勾配そのもの。
- **段の勾配の合理**: 曝露段が warn なのは記事の前提 (曝露率ほぼ100%、入ること自体は避けられない) と整合 — 止められないものは検疫タグで「データであって命令ではない」を注入し、止められる下流 (実行) に block を置く。
- **発火実績の観測 (層が生きている証拠)**: hook-fire ledger 2026-07 分に block 発火の実データ (read_secret 系 47+20+20+14+13… / pre_commands 一括置換 12・秘密ファイル読むな 8・保護パス書込 14・npm 8・curl 7 / worktree guard 9+7、いずれも scan 時点)。配備だけでなく**発火が観測されている**。ただし**観測器自身にバグ**: 初稿が転記した「一括置換 20」は `ledger_aggregate.sh:91` の `sort | uniq -c` が LC_ALL=C 無しで走り、locale collation が日本語 rule 名 2 bucket (一括置換 12 + 秘密ファイル読むな 8) を融合した合成値だった — step6 第 2 ラウンドが JSON 直接パースで検出。消えていた 8 件は本ログが最重要視した「秘密ファイル読むな」rule の発火そのもの (残差 Medium・issue 候補)。

### S-1.5 実測照合 — fail-closed の規約と実態 (scan B 全文より)

- **規約の正本**: `.docs/hook-authoring-guide.md:20-28`「根因 2 — 失敗時に素通り (fail-open)」= 「入力異常・解析失敗時のデフォルトを exit 0 にする (特に secret/権限系) を禁止。判定不能なら拒否側 (fail-closed) に倒す。ただし warn 型は fail-open でも可」+ :113 チェックリスト化。**記事の「安全側をデフォルトに」を、部品の役割 (防御 gate / 観測 sensor) で使い分ける規約に精密化**。
- **宣言の実数**: hooks/ 配下の fail-closed/fail-open/安全側 言及 = 294 行 (test 含む、grep 実測)。block 型 gate = **13 本** (hooks/ 直下 11 + three-elements 配線 2。親バッチの「12 本」は数え方未明示だった → 本深掘りで「hooks/ 直下 block 型 11 + settings.json 配線済み skill hook 2」と数え方ごと確定、Low 訂正)。意図的 fail-open の sensor 宣言 = 13 箇所以上 (ledger_append:9 / credstore_orphan_report:20 / probe:38-40 ほか)。**逆契約** (fail-open にしない sensor) も 1 本明文 (credstore_ledger_append:22「沈黙しない」)。
- **設計外デフォルト (記事 S-1.5 の核心「設計していないものをどう扱うか」)**: `defaultMode: "default"` = グレーゾーン (allow にも deny にも無い操作) は人間に聞く。分析と決定の記録は **2 つのログに割れて実在**: ① `~/.claude` の 2026-05-29 log:83 に「`defaultMode: auto` は fail-open — グレーゾーンを人間に聞かず自動承認する。3 層ゲートを組んだつもりでも実態は 2 層 + グレーゾーン自動承認」の自己分析。ただし同 log:70 の決定は「auto のまま + 危険操作だけ surgical に ask」= **切替を退けた記録**。② 実際の default 切替は別 repo (pixel-leap) の同日 live 検証 log Finding E — 「ask はモード依存。auto では確認窓が出ず ask が空回り、default でのみ発火 → 恒久変更」という**機構発見が動機**。git 履歴上 settings.json に auto は一度も存在しない (git log -S 実測、初 commit から default)。初稿は ① の分析だけ読んで「① が切替の記録」と誤引用していた (step6 第 2 ラウンドが反転)。
- **Tier 対応**: allow 41 (読取系) / ask 30 (eval・commit・keychain 等の可逆-要確認) / deny 177 (不可逆・秘密) — 第15弾 S-1.3 で照合済みゆえ再掲しない (関連ログ参照)。

### 反例狩り (差分の核) — 「自称 fail-closed の楽屋裏」systematic 実測

第15弾が捕捉した前科: `hook_pre_read_secret_check.sh:14-16`「旧実装は exit 0 で素通り (fail-open) しており、入力異常時に秘密ファイル防御が無効化していた」→ 是正済 (現行は入力異常で exit 2 = fail-closed)。**本深掘りで block gate の異常系を系統走査した** (初稿 scan B-2 は 13 本中 6 本のみで「全 gate 走査」は over-claim — step6 第 2 ラウンドが残り 5 本を追加走査して完了)。結果:

**同じ「jq 不在」異常系に対し、gate ごとに 3 通りの挙動が混在する:**

| gate | jq 不在時の挙動 | 宣言 |
|---|---|---|
| `hook_pre_read_secret_check` | exit 2 = **fail-closed** | あり (L14-16 是正記録) |
| `hook_pre_commit_adr_gate` | 素通し (gate 対象外のみ)、判定器欠落は block | **あり** (:58-59 宣言付き fail-open) |
| `hook_pre_commands` (9 rule、秘密 Bash 読取 deny 含む) | TOOL_NAME 空 → 検査全スキップ → approve = **fail-open** | **なし** (:17,20,107 実装からの読み取り) |
| `hook_pre_commit_essence_gate` | 行131 jq 抽出→空→行134 `!= "Bash"` で exit 0 = **fail-open** | **なし** (:131,134) |
| `hook_stop_words` (Stop gate、全131行) | transcript 不在扱い → 行130 approve/行131 exit 0 = **fail-open** | **なし** (:11-12,130-131) |
| `hook_pre_commit_settings_churn_normalize` | 行121 `!= "Bash"` で exit 0 = **fail-open** (行306 deny normalizer-unavailable に到達せず = dead code) | **偽宣言** (:50「jq 不在→block」と自称するが到達不能) |

**step6 第 2 ラウンドの追加走査 (残り 5 本、jq 排除 PATH での実行検証):**

| gate | jq 不在時の挙動 | 宣言 |
|---|---|---|
| `hook_pre_worktree_bash_write_guard` (武装時) | exit 0・無音 = **fail-open** (jq 有なら同 payload を deny) | **部分のみ** (:62 は jq 不在を名指ししない) |
| `hook_pre_worktree_write_guard` (武装時) | exit 2 = **fail-closed** | あり (:79-82) |
| `hook_pre_plans_redirect` | exit 0 = fail-open (jq 有: exit 2) | あり (:16、jq 名指し) |
| `hook_pre_generated_comment_ban` | exit 0 = fail-open | あり (:40、jq 名指し) |
| `hook_pre_hardcode_hygiene_check` | exit 0 = fail-open | あり (:22、jq 名指し) |

- `worktree_bash_write_guard` は `settings_churn:121` と完全同型の構造 (:68-69 `TOOL_NAME` 空 → 非 Bash 扱い exit 0) で、守る対象は本体 `~/.claude` への誤書込 (#97) = 権限系の中核。対の `worktree_write_guard` は同条件で exit 2 に倒れ、**guard 2 枚が非対称**。plans/generated/hardcode の 3 本は宣言つき fail-open (documented) ゆえ新規 Medium に含めない。

- **新規 Medium (未開示・未追跡)**: 実効 fail-open は計 5 本 (宣言なし 3 = pre_commands/stop_words/essence_gate、偽宣言 1 = settings_churn、部分宣言 1 = worktree_bash_write_guard 武装時)。特に `hook_pre_commands` は「秘密ファイル読むな」(Bash cat .env 等) を含む 9 rule 全てが jq 不在で消える = 根因 2 の「特に secret/権限系」に正面から該当。read_secret L16 の是正が**水平展開されていない**。**さらに悪質なのが settings_churn** — 行50 で「fail-closed で止めます (issue 195)」と明示自称しながら実測は fail-open (step6 独立検証が jq 排除環境で撃って exit 0 を確定。行121 の Bash 判定早期 exit が行306 の jq-不在 deny を無効化する dead code 構造)。「配備≠実効」を地で行く偽宣言 gate で、初稿はここを live で撃たず行50 コメントを額面通り「fail-closed」に誤分類していた (step6 が是正)。rules JSON 不在時の素通し (pre_commands 宣言なし / essence_gate はコメント明示) も同族。発生確率は低い (jq はハーネス運用の常在依存) が、規約と実装の乖離 + 宣言の不在 (settings_churn は逆に偽の fail-closed 宣言) が問題の本体。→ **切り分け結果: (a) 棄却候補表に fail-closed/伝播段階を名指しする棄却項目なし (scan C-1 全文走査) = 棄却済み候補の再導入ではない。(b) #220〜#226 のいずれにも該当なし、さらに全 97 issue (closed 含む) を jq/fail-open 語で串刺し検索しても該当なし (step6 第 2 ラウンド) = 未追跡。(c) 開示なし (ヘッダ・guide とも)・settings_churn は逆に偽の fail-closed 開示 = 意図的相違でも開示済み制約でもない。→ 新規 issue 候補 (issue 化はかいじゅう判断)**。
- **Low (宣言の非対称)**: hook timeout 到達時に Claude Code が hook を kill して tool を通す (= fail-open) 事実の明文が `adr_gate:60-62` の 1 箇所のみ。同じ構造穴は全 PreToolUse block gate に共通 (platform 挙動ゆえ塞げない — 塞げないことと宣言しないことは別)。hook crash 時の tool 帰趨を汎用に説明する doc も不在 (scan D、hook-authoring-guide に 1 節で閉じる)。
- **既知・開示済み (新規 issue 不要と判定)**:
  - 中継段の機械層ゼロ (subagent 境界 hook 0 本 / teammate・SendMessage を扱う rule 0 hit) — **第14弾で「subagent 中和は ~/.claude に決定論機構なし・platform 依存」と記録済みの既知**。S-1.4 レンズでの再収束 (構造は tools 絞りが担い、中継データの検疫は platform の system-reminder に依存)。
  - sandbox.denyRead 137 本 = dead config (`sandbox.enabled` 不在で 1 本も効かない、2026-07-17 live 実測 #186) — **開示 7 箇所以上** (hook コメント :371-374,:385 / 正本ログ / local ログ / テスト 2 ファイル / essence-review-records 3 本 — 初稿の「4 箇所」は undercount、step6 第 2 ラウンド訂正)。`enabled:true` は E2BIG で足せない (ARG_MAX 1MB 超過) = 設定だけでは直せないことまで実測済み。
  - Bash 経路の秘密読取に専用層なし (hook は matcher=Read で射程外、sandbox dead) — hook コメント :371-396 で開示済み + 「実際に守っているのは hook §3 絶対パス判定」+ pre_commands「秘密ファイル読むな」rule が部分カバー。「未測定は恒久の言い訳にしない」宣言つき。
  - worktree guard の非武装時 allow (#97 契約) / essence gate の corner 2 件 (:113-117, :88-90) / churn の awk 不在不活性 (:148) — いずれもコメントで開示済みの documented fail-open。
  - #220 (tools 無指定 4 本 = 中継段の入口全開) / #222 (essence-docs deny 非保護) / #223 (hooks/lib 界面) — 実行段の既追跡残差。#223 は「gate が fail-closed 設計ゆえ雑な lib 編集は gate を止める方向に倒れる」と severity 下方の根拠自体が fail-closed 設計の効用になっている。

### 記事超え

1. **「配備≠実効」の live 実測文化 (S-1.4)**: 記事は「各段階に防護層を設ける」まで。ハーネスは**設けた層が本当に効いているかを live で撃って確認する**段階まで進む — #186 の rig (probe-deny-glob.sh) が sandbox.denyRead 202 本 (当時) を「静的検査もレビューも PR も通った上で 1 本も効いていない」と暴いた。「live で撃つまで、配備は仮説にすぎない」(正本ログの教訓節)。さらに hook-fire ledger が各層の発火実績を月次観測 = 「層が生きている」ことの常時計測。
2. **fail-closed/open の部品役割規約 + 射程の非一枚岩 (S-1.5)**: 記事の「安全側をデフォルトに」を「防御 gate = fail-closed / 観測 sensor = fail-open (センサーの失敗で作業を止めない) / 逆契約 sensor = 沈黙しない」の 3 分化まで精密化 (hook-authoring-guide 根因 2 + checklist)。加えて `hook_stop_handoff_check:60-61` は「fail-open は一枚岩ではない — 壊れ方によって倒れる方向が反転する」を実測で記録 (substring 非一致は通知側 / substring を含む壊れ JSON は沈黙側)。
3. **設計外デフォルトの移行記録 (S-1.5)**: `auto`→`default` 切替を「auto は fail-open」の自己分析ごと文書化。記事の S-1.5 定義 (「設計していないものをどう扱うか」) への回答が、結論だけでなく**意思決定の過程ごと**残っている。

### 同日の傍証 (PR #227 レビューとの共鳴)

同日実施の PR #225/#227 マージゲートレビュー (related_log 参照) で、#227 の scripts が「script 単体は fail-closed でも系としては gate 不在」(assert-no-cycles の exit 2→0 変更を、exit code しか読まない消費者が「循環なし」と解釈する fail-open) を独立レビューが捕捉 → CONDITIONAL GO。**S-1.5 の「fail-closed は部品でなく系で評価する」がマージ前ゲートで実働した実例** (本深掘りの判定材料ではなく傍証)。

## 残差サマリ (severity 付き)

| severity | 内容 | 切り分け |
|---|---|---|
| Medium | block gate 5 本 (pre_commands / stop_words / essence_gate / settings_churn / worktree_bash_write_guard 武装時) の jq 依存 fail-open — 宣言なし 3・偽宣言 1 (settings_churn 行 50 vs 行 121 早期 exit = 行 306 deny が dead code)・部分宣言 1 (worktree_bash_write_guard、settings_churn 同型で guard 2 枚非対称)。根因 2 規約と乖離・read_secret L16 是正の水平未展開。rules JSON 不在時の素通しも同族 | 新規 (棄却表×/全 97 issue×/開示×)。issue 化はかいじゅう判断 |
| Medium | `ledger_aggregate.sh:91` の LC_ALL=C 欠落 — locale collation で日本語 rule 名の bucket が融合し、発火実績の集計が合成値になる (一括置換 12 + 秘密ファイル読むな 8 → 「23」等)。S-1.4 の「層が生きている証拠」を観測器自身が歪める | 新規 (ハーネス実バグ、step6 第 2 ラウンド発見)。issue 化はかいじゅう判断 |
| Low | timeout fail-open (hook kill → tool 通過) の明文が adr_gate 1 箇所のみ — 全 block gate 共通の構造穴なのに宣言が非対称。hook crash 時挙動の汎用 doc も不在 | 新規 (doc 追記で閉じる) |
| Low | 親バッチ「防御 gate 12 本」→ 実測 13 本 (hooks/ 直下 block 型 11 + three-elements 配線 2)。数え方の明示ごと確定 | 計数訂正 |
| Low | 初稿の計数・表記の訂正 (step6 第 2 ラウンド): 「auto→default 移行記録」の出典差し替え (High-2) / S-1.5 行範囲 2124〜2136→2124〜2130 (7 行) / sandbox 開示 4→7 箇所以上 / 行番号ずれ 3 件 (:372-374→:371-374 ほか) / scan 515→512 行 | ログ側訂正 (全て本文反映済み) |
| — (既知) | 中継段の機械層ゼロ (第14弾記録済・platform 依存) / sandbox dead config (開示4箇所・実測済) / Bash 秘密読取層 (開示済) / #220・#222・#223 | 新規 issue 不要 |

## step6 独立検証の記録

2 ラウンド + 収束確認で実施 (いずれも fresh code-reviewer・read-only・アンカリング防止のためログ本文と ~/.claude 読取権限のみ渡し):

- **第 1 ラウンド (07-24 21時台)**: settings_churn の偽 fail-closed 宣言 (行 50 自称 vs 行 121 早期 exit = 行 306 deny 到達不能) を jq 排除環境の実行で捕捉し、fail-open 3 本 → 4 本へ是正。**レポート原文はセッション断で回収不能となった** (サブエージェント出力はセッション内限り) ため、第 2 ラウンドで全主張をゼロから再検証した。
- **第 2 ラウンド (07-24 22時台, verifier-round16)**: 全数トレース再実施。**核心は全て live 再現で確定** — gate × jq 不在の 6 行表 (全行を jq 排除 PATH farm で実行・exit code 突合)、settings_churn の dead code (制御フローと実行の両方)、block 型 13 本 (11+2)、allow 41/ask 30/deny 177、fork 21 skill、agent 28/32 tools 明示 (Edit/Write 非付与 24 本)、SubagentStop 4 本全装飾、根因 2 と逆契約と一枚岩でない fail-open の逐語引用、#186 正本ログ、棄却候補表と全 97 issue の切り分け、記事原文 (2112〜2130) 逐語一致。**要修正 7 件 (High 3 / Medium 1 / Low 3) を検出、全件本文へ反映済み**:
  - High-1: 「一括置換 20」は `ledger_aggregate.sh:91` の LC_ALL 欠落による 2 rule 融合の合成値 (実数 12+8)。→ 発火実績を実数へ訂正 + 観測器バグを残差 Medium 化
  - High-2: 「auto→default 移行記録 (05-29 log)」は出典誤り — 当該ログは「auto のまま + surgical ask」という逆の決定の記録。実切替は別 repo (pixel-leap) の Finding E (動機 = ask がモード依存で空回り)。→ 核心・S-1.5 節・記事超え 3 を差し替え
  - High-3: 「全 block gate 系統走査」は 6/13 の over-claim。残り 5 本の追加走査で worktree_bash_write_guard 武装時 fail-open (settings_churn:121 同型・guard 2 枚非対称) を捕捉 → Medium 5 本目 + 追加走査表を本文へ
  - Medium-4: 本記録節が placeholder のまま本文が step6 を既成事実化 → 本節で解消
  - Low-5〜7: S-1.5 行範囲 (2124〜2130 = 7 行)・開示 4→7 箇所以上・行番号ずれ 3 件・scan 512 行 → 各所訂正
- **検証者の副作用開示 (第 2 ラウンド)**: live 撃ちで hook-fire ledger 2026-07 に 7 行 append (全て 07-24T13:15Z 以降 = scan 時点の計数に影響なし)。
- **総評 (第 2 ラウンド原文より)**: 落ちた 3 High はいずれも「自分の外にある道具の出力 (集計器・過去ログ・scan) を独立に確かめず転記した」箇所 — 本ログが記事超えに掲げた「live で撃つまで配備は仮説」が、初稿の検証手続き自身に 3 回適用されていなかった。
- **収束確認 (第 3 ラウンド, 07-25)**: fresh 検証者 (verifier-round16c) へ再投入したが**月次 spend limit で失敗し、独立エージェントは起動不能に** (transcript 末尾は上限エラーのみ — 部分成果の転記はしない)。skill の縮退規定「reviewer 系 agent が取れなければ step 8 の人間が独立検証者を務める (HITL backstop = fail-close 側)。検証ゲート自体は飛ばさない」を適用:
  - (a) **変更主張の機械確認 9 点はメイン自身が Bash/Read で再実測し全一致**: ledger scan 時点実数 (一括置換 12・秘密読むな 8、JSON 直接パース) / aggregate :88-93 の `sort | uniq -c` に LC_ALL 無し / 05-29 log:70 (auto のまま + surgical ask) と :83 (fail-open 分析) / pixel-leap Finding E 逐語 / `git log -S '"defaultMode": "auto"'` 0 件 / worktree_bash_write_guard :62 宣言 (jq 非名指し)・:68-69 settings_churn 同型 + worktree_write_guard :79-82 fail-closed + plans :16・generated :40・hardcode :22 の jq 名指し宣言 / read_secret :371 / essence_gate :113-117 / text.md 範囲 (2112=S-1.4 見出し・2122=本文末・2124=S-1.5 見出し・2130=本文末・2131〜=E 章) / 内部整合 (旧値 515・2112〜2123・2124〜2136・開示 4 箇所の残存なし — 訂正行での意図的言及のみ)。
  - (b) **独立性の担保は HITL へ委譲**: 自己検証のみで commit しない — かいじゅうが本ログを読んで承認してから commit する。第 2 ラウンド (独立・完了済) が事実の土台を検証済みで、本ラウンドは転記の正しさの機械確認である点を開示する。

## 出典

- note 記事: `.docs/references/260405_【「なぜか微妙」は偶然ではない】Claude Codeで学ぶ 7つの構造的制約とハーネス設計、AIエージェントの本質論/text.md` 2112〜2130 行 (S-1.4=2112〜2122 / S-1.5=2124〜2130)。索引 `~/.claude/.docs/references/BIBLIOGRAPHY.md` (番号 260405)
- ハーネス実測: read-only Explore (scan-s14-s15) の 512 行 scan (2026-07-24。settings.json 22318 bytes 2026-07-24 20:23 時点) + step6 第 2 ラウンド (verifier-round16) の live 実行検証。主要一次ファイル: `~/.claude/hooks/hook_pre_commands.sh` / `hook_stop_words.sh` / `hook_pre_commit_essence_gate.sh` / `hook_pre_commit_adr_gate.sh` / `hook_pre_read_secret_check.sh` / `.docs/hook-authoring-guide.md` / `.docs/logs/shared/permission-deny-glob/2026-07-17-live-effectiveness.md`
- 既存トレース: issue #220/#222/#223 (`gh issue view <N> -R kaijutale/claude-harness`) / `~/.claude/.docs/essence/essence-sources/harness-sources.md` 棄却候補表
