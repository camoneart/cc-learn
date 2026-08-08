---
date: 2026-08-08 15:00:16
type: study
topic: final-design-checklist-closure
session: 巻末「設計チェックリスト」の軽量版締め (取り入れフェーズ第20弾・最終回 — フル監査でなく照合 + スポット掃討。縮小設計は Dende の指摘による)
related_article: ".docs/references/260405_【「なぜか微妙」は偶然ではない】Claude Codeで学ぶ 7つの構造的制約とハーネス設計、AIエージェントの本質論/text.md (設計チェックリスト = checkbox 実範囲 2319〜2381行、35 問 = 7 グループ×5。関連: おわりに 2385〜 / 最初の一歩 2 アクション 2395〜2396)"
related_skill: [harness-adoption-audit, explain-in-html, logging]
related_log_ids: [2026-05-30_note-harness-gap-analysis, 2026-08-04_e-1-and-e-1-1-constraint-cascade-periodic-audit-deepdive, 2026-08-08_e-2-reason-based-generalization-deepdive, 2026-08-08_e-3-nondeterminism-last-deepdive]
related_log: [.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md, .docs/logs/shared/2026-08-08_e-3-nondeterminism-last-deepdive.md]
---

# 巻末「設計チェックリスト」締め — 判定: 照合完了・取り入れフェーズ完走 (35 問全問に測定痕跡・証拠強度は不均一と開示) · 道具側の差分 = **rubric 完全欠落 1 問 (CTX-5) + 部分 14 問** (欠落・部分の代表 3 問は全部ハーネスに実装が在る —「実装済みだが診断網に無い」型) + **agent-essence は原則全カバーだが質問形 (checkbox) ゼロ** · 初版の数値 4 つを独立検証が反証し是正した第 2 版

> 核心の構造事実: チェックリスト 35 問 (7 グループ×5、checkbox 実範囲 = text.md 2319-2381。`- [ ]` を awk で数えて 35 を確認) の**フル監査は行わない** — 各問は既に第 1〜19 弾で測定済みの原則の質問形であり、再照合は 19 弾の再検証になるだけ (Dende の指摘による縮小設計)。実施したのは ①**道具側カバー照合** (35 問 × rubric 25 指標 × essence 正本 4 本 — scan 委譲・主要値はメイン再測) ②**未深掘りだった T 系 5 問のスポット掃討** ③総括。**発見 3 つ (数値は独立検証の反証を再測して確定した第 2 版)**: (a) rubric に診断指標が**完全に無い**問いは **CTX-5 (順序・フレーミング検証) の 1 問のみ** — 字句 (フレーミング/順序/誘導/複数) と概念の両レベルで陰性 (独立検証が概念語彙でも走査)。初版が NONE とした BIAS-5 は **D4 が校正盲 (C-7) を原則として持ち「確信度に比例した検証強化」の機構だけが無い** = 部分、KNW-5 は **D3 が K-1.2「記憶は保存前に監査・修復する」を逐語で持ち probe/反例の具体性だけが無い** = 部分へ各格下げ (rubric L258-259 をメイン再測で確認)。集計 = **NONE 1 / 部分 14 / full 20**。**3 問ともハーネスに実装は在る** (detecting-framing-bias skill / harness-essentials:149 の質問形 / probe-before-persist rule+hook) — 欠けは診断側。(b) **agent-essence.md は 35 問全問を原則として持つが (ID 体系が記事と一致)、checkbox 形式の行は全 185 行中 0 件** (再測一致)。harness/skill-essentials 側の質問形カバーは**初版の「12 問」を撤回** — 独立検証が 104 checkbox (harness 59 + skill 45) を全読して突合し、**TRS だけで質問形 4** (harness:200-203 の決定論ゲート・非信頼入力データ扱い等、メイン再測で確認) を検出。保守的レンジ = **16〜20 問**。初版の「補充候補」数も 35−12 の算術ゆえ**同時に撤回** — 正確な全数突合は #345 レーンの第一作業に指定する (誤った作業リストを流すと、存在する質問形を再作成させる)。(c) rubric の原則カバレッジマップと本文の字句に乖離あり (宣言上カバーする指標の本文に該当語が無い)。

## 概要

取り入れフェーズ第20弾 = 最終回。当初 #256 は独立のフル監査として構想されたが、**Dende の指摘「吸収 (#344/#345) で実質終わりでは」を受けて軽量版へ縮小** — チェックリストは既監査原則の要約ゆえ、フル照合は冗長 (「賢ければ自発的にやることを書くな」の監査版)。残る実作業 = 道具側の照合表づくり (#344/#345 の前段として必須) + どの弾でも測っていない項目のスポット掃討のみ。

照合は read-only `Explore` 1 体 (`scan-256-coverage`、本文報告・回収と同時に停止) へ委譲し、集計値・行スポット・陰性は全てメインで撃ち直して一致を確認した (agent-essence checkbox 0/185・rubric 陰性 3 語 + 対照・harness:105/skill:33/agent:163,166 の逐語)。

## 内容

### 照合表の要約 (全 35 行の詳細は #344/#345 へのコメントに供給)

| グループ | rubric カバー (第 2 版で確定) | 測定元の証拠強度 (独立検証の指摘で開示) |
|---|---|---|
| コンテキスト管理 (CTX 1-5) | 4/5 (**CTX-5 = NONE** — 唯一の完全欠落) | 6 月の C 章監査群への**ポインタ止まり** (問いとの 1:1 対応は未整備) |
| 認知バイアス (BIAS 1-5) | 5/5 (**BIAS-5 = 部分** — D4 に原則あり・機構なし) | 6 月監査 + **本セッション運用 (循環成分あり — 開示)** |
| 知識管理 (KNW 1-5) | 5/5 (**KNW-5 = 部分** — D3 に K-1.2 逐語) | auto-memory 系への**ポインタ止まり** |
| タスク構造 (TSK 1-5) | 5/5 (部分 4) | 本弾スポット — TSK-1/2/3 は**機械アンカーで強**、TSK-4 後半・TSK-5 は**親バッチの宣言止まり + 循環成分** |
| 検証 (VER 1-5) | 5/5 | V 章 7 本の deepdive が 1:1 — **強** |
| 信頼と権限 (TRS 1-5) | 5/5 (部分 4) | S-1 系 5 本 — **強** |
| 環境設計 (ENV 1-5) | 5/5 (部分 1) | E 章 3 本 (第 17-19 弾) — **強** |

- **rubric NONE = 1 問 (CTX-5) / 部分 = 14 問 / full = 20 問** (初版の「NONE 3」は独立検証が 2 問を反証 — 各根拠は下記 (a))
- **essence NONE = 0 問** — 原則 ID 体系の一致により全問に対応記述。**agent-essence の checkbox = 0/185** (再測一致)。質問形の総数は初版計数を撤回しレンジ 16〜20 (上記)
- 35 行の生照合表 (scan 産・部分判定の per-row 根拠つき) は #344 へのコメントで供給

### 発見 (a) — rubric の穴 3 問は「実装済みだが診断網に無い」型

| 問い | ハーネスの実装 (実在) | rubric の状態 (第 2 版) |
|---|---|---|
| CTX-5 順序・フレーミングの複数検証 | `detecting-framing-bias` skill (2 フレーム並列の Devil's Advocate 装置) | **NONE** — 字句・概念の両レベルで陰性 |
| BIAS-5 高確信度ほど検証を強化 | `harness-essentials:149` に質問形で existent | **部分** — D4 が校正盲 (C-7) を原則として持つ。無いのは「確信度に比例した強化」の機構 |
| KNW-5 保存前の反例テスト | `probe-before-persist` rule + PreToolUse hook (本セッションでも毎回発火) | **部分** — D3 が K-1.2 を逐語で持つ (L258-259)。無いのは probe/反例の具体性 |

E 章監査で続いた「奥の原則は強く、運用の縁が弱い」と同じ形 — **作る側は記事を取り込んだが、診る側 (rubric) の解像度が追いついていない**。#344 の処方 = CTX-5 は新指標、BIAS-5/KNW-5 は既存指標 (D4/D3) の判定基準補強。

### スポット掃討 — T 系 5 問 (唯一の未深掘りグループ)

親バッチの 1 行判定 (L39「T タスク構造 全✅」) しか無かった T 系を、本セッションで実在確認済みの機械アンカーで裏取りした:

- **TSK-1 (関心事の同居禁止)**: `three-elements-harness` の hook 2 本が **settings.json に配線済み** (`block-team-in-macro.sh` = PreToolUse Task / `restrict-macro-writes.sh` = PreToolUse Write 系。E-1 監査の settings 全数調査で配線を実測) — Macro/Micro の分離を機械が強制
- **TSK-2 (計画と実行の分離)**: Plan Mode + `plan-workflow` rule (paths 条件注入) + plan 系 hook 4 本 (pre_plan_output_convention / stop_plan_archive / stop_plan_externalization_check / stop_plan_promote_reminder — E-1 監査で配線実測)
- **TSK-3 (計画の自己完結性)**: 「良い計画の 5 要素」の照合ログが既在 (`2026-06-30_good-plan-5elements-and-adr-container.md` + output に mapping HTML 2 本)
- **TSK-4 (ゲート検証と再注入)**: `orchestrating-team-development/references/phase-gate-protocol.md` 実在 (GO/NO-GO 語彙 18 ファイルの 1 つとして E-3 監査で列挙済み)。spec 再注入は親バッチが「T-2.2 明記」と記録
- **TSK-5 (弱→強の目標ドリフト)**: 親バッチが「統合非委任 (T-2.3 明記)」を記録。加えて本セッションの運用実践 = scan/gate の報告は必ずメインが再測してから採用 (弱い出力の無検証継承をしない)

→ **5 問とも実体アンカーあり**。T 系に改修 issue は立てない (深掘りの必要が出たら将来の弾で)。

### 発見 (b)(c) と巻末「最初の一歩」の決着

- **(b) agent-essence の質問形ゼロ**: 原則は全カバー・checkbox 0/185。質問形の補充は #345 (proposing-essence-updates 経由) の職分 — 補充候補の**確定リストは供給しない** (初版の「23 問」は撤回済み)。#345 の第一作業 = 104 checkbox × 35 問の全数突合で確定させる
- **(c) rubric のカバレッジマップと本文の乖離**: マップは C-2→D1,E1 等と宣言するが、当該指標の本文に「フレーミング」等の字句が無い (scan 実測・grep)。#344 のレーンで判定基準の補強と同時に突合するべき点として記録
- **最初の一歩 2 アクション** (2395-2396): ①CLAUDE.md 60 行以下 = **達成済み** (59 行、E-2 監査実測) ②deny 追加 = **半分採用済み・半分意図的不採用** (初版の「test/lint 不採用」は転記の精度落ちで独立検証が訂正): **lint 設定の deny は採用済み** (`Edit(**/.eslintrc*)` 等の実在を deny 179 件から再測 — 記事名指しの `.eslintrc` そのもの)。**意図的不採用はテストファイル本体 (`**/*.test.ts` / `**/*.spec.ts`) のみ** (親バッチ L46-48 の原文どおり — TDD ハーネスとの構造衝突を理由に却下、テスト改竄が実測されたら条件付き hook で、と記録済み) — どちらも決着済みで新規作業なし

### 処分

| 出力 | 行き先 |
|---|---|
| 照合表全文 (35 行の raw 表 + 部分 14 問の per-row 根拠) | **#344 へコメント供給** (CTX-5 新指標 + D4/D3 補強の弾薬。初版集計の撤回注記つき) |
| agent-essence 質問形ゼロの実測 + 突合の方法 (104 checkbox × 35 問) + 初版計数の撤回注記 | **#345 へコメント供給** |
| note セクション issue #253/#254/#255/#256 | **全て OPEN のまま残っていたことを本弾で発見** — 各監査ログ (push 済み) を根拠に completion コメント + close する (実行は step8 の HITL 後) |

判定: **巻末「設計チェックリスト」= 照合完了。取り入れフェーズ (全 20 弾) 完走** — 35 問はハーネス実体の側で全問に測定痕跡がある。**ただし証拠強度は不均一と開示する** (V/S/E/K-2 = deepdive 1:1 で強 / CTX/KNW = 6 月ログ群へのポインタ / BIAS の一部と TSK-5 = 本セッション運用を根拠に含む循環成分)。道具側の差分は #344 (NONE 1 + 部分 14 の補強) / #345 (質問形の全数突合から) が引き継ぐ。

## step6 独立検証の記録

fresh な read-only reviewer (`code-reviewer` 型・ファイル報告) 1 体。**verdict: CONDITIONAL** — CONFIRMED 多数 (35 問・387 行・0/185・逐語群・#253-256 OPEN・TSK 機械アンカー) / OVERCLAIM 5 / UNTRACEABLE 3。是正 5 点は全て検証者の実測をメインで再測してから反映 (二重測定):

| # | 指摘 | 初版 | 是正 |
|---|---|---|---|
| OC-1 | 質問形 12 問・補充候補 23 問 | scan の集計を未検証で採用 | **両方撤回**。検証者が 104 checkbox 全読で TRS 単独の質問形 4 を実測 (メイン再測一致) — レンジ 16〜20 とし、全数突合を #345 の第一作業に指定 |
| OC-2 | KNW-5 = rubric NONE | 字句陰性のみで判定 | **部分へ格下げ** — D3 が K-1.2 を逐語保持 (L258-259 再測) |
| OC-3 | 陰性の対照が「読めた」止まり | known-positive 1 件 | BIAS-5 も**部分へ格下げ** (D4 の校正盲)。grounds Gotcha「1 件の対照は陰性の検証にならない」の再演と認める — 概念語彙の走査を対照に追加 |
| OC-4 | 「test/lint deny = 意図的不採用」 | 親バッチからの転記で精度落ち | **lint は採用済み・test 本体のみ不採用へ訂正** (deny 実測: eslintrc True / test.ts False)。「転記は実測の代わりにならない」(#312 の規律) の再演 |
| OC-5 / UT-3 | 「全問測定済み」の均し・弾数の自己矛盾 | 一様な強さで記述 | **証拠強度を 3 段で開示** (1:1 / ポインタ / 循環成分) + 弾数の数え方を統一 |
| UT-1 | 行範囲 3 箇所ズレ + 範囲と数の不整合 | 2313-2377 に 35 問 (実際は 31) | **2319-2381 へ統一** (awk で 35 を確認)。おわりに 2385 / 最初の一歩 2395-2396 |
| UT-2 | 要約表の内部算術 (質問形 11≠12・VER 4+2=6) | scan 集計の転記 + 自前集計のミス | 表を再構築 — 誤った質問形列を撤去し、rubric 列を raw 表から再集計 (NONE 1 / 部分 14 / full 20) |

**このラウンドの学び**: 初版は「scan の集計値」と「親バッチの記述」を、個別行は検証したのに**集計と転記の層で未検証のまま採用**した — E-3 の「10 件目」問題の集計版。撃ち直しの射程は「個別の行」だけでなく「**集計・転記・算術**」まで含む。再ゲートは行わない (1 ラウンド固定。是正は二重測定値の採用、残余は step8 の Dende レビュー)。

## 関連ファイル

- `~/.claude/skills/review-harness/diagnosis-rubric.md` — 25 指標・カバレッジマップ・陰性 3 語の実測対象
- `~/.claude/.docs/essence/essence-docs/` 4 本 — checkbox 分布 (agent 0 / harness・skill に 12 問分)
- `~/.claude/settings.json` + `skills/three-elements-harness/scripts/hooks/` — TSK-1 の機械配線
- `~/.claude/skills/orchestrating-team-development/references/phase-gate-protocol.md` — TSK-4
- `~/.claude/skills/detecting-framing-bias/SKILL.md` / `~/.claude/rules/probe-before-persist.md` — rubric 穴 3 問の実装側

## 出典

- 記事本文: `.docs/references/260405_*/text.md` (チェックリスト 2311-2377 / おわりに 2379-2397)。索引 `~/.claude/.docs/references/BIBLIOGRAPHY.md` (番号 260405)
- 親バッチ: `.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md` (L39 = T 章 1 行判定)
- 測定ログ群: 弾番号つき 19 本 (7/19 以降の adoption-check / deepdive — K-2.x 3 / S 系 6 / V 系 7 / E 系 3。本弾が第 20 弾) + **弾番号の外の 6 月監査群** (C 系 c2-c3/c4・auto-memory 系 — 実在は ls で照合済みだが 35 問との 1:1 対応は未整備、上表の強度開示どおり)
- scan 報告: `scan-256-coverage` の本文報告 (35 行照合表。集計・スポットはメイン再測で一致)
