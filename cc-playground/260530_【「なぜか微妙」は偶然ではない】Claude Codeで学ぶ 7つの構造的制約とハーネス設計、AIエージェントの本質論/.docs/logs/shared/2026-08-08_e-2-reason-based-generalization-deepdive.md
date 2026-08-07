---
date: 2026-08-08 04:15:17
type: study
topic: e-2-reason-based-generalization-deepdive
session: E-2「ルールより理由で汎化する」単独深掘り (取り入れフェーズ第18弾)
related_article: .docs/references/260405_【「なぜか微妙」は偶然ではない】Claude Codeで学ぶ 7つの構造的制約とハーネス設計、AIエージェントの本質論/text.md (E-2 = 2201〜2269行、関連: E-1 2138〜2199 / E-3 開始 2271 / 判断基準 2265〜2267 / C-5 報酬ハッキング 569〜581 / Steinberger-Ronacher 引用 2269)
related_skill: [harness-adoption-audit, explain-in-html, logging]
related_log_ids: [2026-05-30_note-harness-gap-analysis, 2026-08-04_e-1-and-e-1-1-constraint-cascade-periodic-audit-deepdive]
related_log: [.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md, .docs/logs/shared/2026-08-04_e-1-and-e-1-1-constraint-cascade-periodic-audit-deepdive.md]
---

# E-2「ルールより理由で汎化する」単独深掘り — 判定: **分割** — essence 正本 + rules/ = 取り入れ済み / **CLAUDE.md 本体 = 未達 [Medium]** (bullet 33 本中 理由持ち 3 本・規則 4 セクションは理由ゼロ・記事 Before 型の日本語同型 L42 が現存) · 記事超えは狭い 1 点のみ (合理化文字列の rule 本文へのインライン列挙 — 「悪用面への防御が記事に無い」は誤りで C-5 が扱う) · 初版は独立検証で NO-GO (指摘 11 件) を受けた第 2 版

> 核心の構造事実: **記事の判断基準「十分に賢ければ Claude が自発的にやることか?」は、ハーネスの essence 正本に原則 ID「E-2」として英語逐語で存在する** (`agent-essence.md` L161-163「Would Claude do this anyway if it were smart enough?」が Yes なら書かない)。失敗から昇格した rule 3 本 (grounds-not-approval / failure-promotion-trigger / decisive-answers) は理由トークンが本文の骨格 (4+9 / 2+6 / 1+1) で、増殖防御 3 機構 (コスト自己開示 +41% / ADR 却下理由 REQUIRED / read-miss 対策) + GC issue #322 も実在する — **ここまでは取り入れ済み**。しかし**記事 E-2 が名指しで対象とする CLAUDE.md 本体は未達**: bullet 33 本中、理由トークンを持つのは **3 本のみで、その 3 本は全て Persona / 口調セクション**。Response / Prohibition / Interview / Stack の**規則 4 セクションに理由を持つ bullet はゼロ**。L42「絶対禁止: 敬語・忖度・過大評価・お世辞・罵倒・見下し・迎合・イエスマン」の 8 項目裸列挙は、**記事 Before 型 (2221-2226 の NEVER 6 行列挙) の日本語での完全な同型**。初版は「59 行 = 理想の内側」「ALWAYS/NEVER 字面 0 件」という**代理指標で CLAUDE.md 分を『取り入れ済み』に含めており、独立検証が直接測定で反証** (NO-GO・指摘 11 件)。本版はその全指摘を、検証者の実測値をメインで再測した上で反映した。**事前蒸留メモの推定も反証された** — `_wip-note-distillation/260405.md` L200 は「CLAUDE.mdの各ルールに『理由』コメントを付与する慣習 (確信度 中)」と推定していたが、直接測定は**その慣習が rules/ には在り CLAUDE.md には無い**ことを示した。

## 概要

取り入れフェーズ第18弾。親バッチ (`2026-05-30_note-harness-gap-analysis.md` L43) の判定は「理由ベース rules ✅」の 1 行のみ。本深掘りの差分焦点: (a) 理由の保持率の全数実測 (b) 判断基準の実在 (c) 増殖防御 (d) 反例狩り = Before 型の残存。

実測 (step3) は read-only `Explore` 1 体へ委譲 (ファイル報告・回収と同時に停止 — E-1 の運用反省を適用)。**step6 独立検証は 1 ラウンド固定の運用で、初版が NO-GO** (CONFIRMED 10 / UNTRACEABLE 2 / OVERCLAIM 3 / severity 過小 1 / 判定の支持不足 1)。是正は検証者の実測値を**メインで全件再測してから**反映した (禁止 6 行 8 出現 / 迂回 128・91 ファイル / CLAUDE.md L33-L42 逐語 / bullet 33 中 3 / `git log -S` の同一 commit — 全て一致)。再ゲートは行わない (Lead 判断。理由: 是正は二重測定済みの値の採用であり、意味論の残余リスクは step8 の Dende レビューが受け持つ — E-1 で確立した 1 ラウンド運用のとおり)。

## 内容

### note 側の定義 (E-2 = 2201〜2269 行)

一行サマリー (2203 行): **CLAUDE.md には**「何をすべきか」のルール列挙より「なぜそうすべきか」の理由を書く (対象は名指しで CLAUDE.md — 2220 のコード例も `# CLAUDE.md`)。Before 型の 3 つの問題 (2228〜2238 行): ①未知のケースに対応できない ②衝突時の優先順位がわからない ③ルールが増殖する。判断基準 (2265〜2267 行、逐語): **「十分に賢ければClaudeが自発的にやることか?」この問いが Yes なら、そのルールは書く必要がない。**同 2267 行は「列挙しても**コンテキストを消費するだけだ**」とコストにも言及する。関連: 記事は **C-5 報酬ハッキング (569〜581 行)** で「目標を達成するための合理的な (しかし意図されていない) 最適化だ。**賢いモデルほど、こうした抜け道を見つけるのがうまい**」と、モデルが規則を合理化で回避する面も正面から扱っている (E-2 節内 2261 行の「理由が書いてあれば…趣旨に沿った判断ができる」は無条件の性善説だが、記事全体はそうではない)。

### ハーネス実体の対応表

| 記事の要素 | ハーネスの実体 (実測) | 状態 |
|---|---|---|
| 判断基準「賢ければ自発的にやるか」 | `agent-essence.md` L161-163 に**原則 E-2 として英語逐語で存在**。`review-harness/diagnosis-rubric.md:53` も「賢いClaudeなら言われなくても分かること」として参照 | **取り入れ済み** |
| rules/ の理由ベース化 | 失敗昇格型 3 本の理由トークン = grounds-not-approval **4+9** / failure-promotion **2+6** / decisive-answers **1+1**。両トークン 0 の 6 本のうち 3 本は字面不一致なだけで目視理由あり、3 本 (`build-test-protocol` / `frontend-aesthetics` / `research-phase`) はポインタのみ (理由は指し先に住む設計 = ADR-0001) | **取り入れ済み** |
| 増殖への防御 | ①「このファイルのコスト」節 (L46-48、148→210 行 +41% の自己開示) ②ADR `_TEMPLATE.md` L24-26「検討した代替案と**却下理由**」REQUIRED ③ADR-0001 L49「読み損ねれば詳細ルールが効かない。**対策として**各ポインタに想起トリガーを明記し、Read 依存に耐えない核は inline 死守**する**」(逐語) ④常時注入層 GC = **#322 OPEN** (gh 実測) | **取り入れ済み** |
| **CLAUDE.md 本体への E-2 適用** | 59 行で行数理想 (60 行未満) の内側 — **ただし行数は K-2.1 の軸であって E-2 (ルール vs 理由) の軸ではない**。直接測定: **bullet 33 本中、理由トークン (`理由\|ため\|ゆえ\|ので\|から`) を持つのは 3 本のみ (L16/L17/L22、全て Persona / 口調)。Response / Prohibition / Interview / Stack の規則 4 セクションは理由ゼロ**。L42「絶対禁止: 敬語・忖度・過大評価・お世辞・罵倒・見下し・迎合・イエスマン」は記事 Before 型 (2221-2226) の日本語同型。ALWAYS/NEVER の**英字**は 0 件 (対照実験済み: 走査 334 行 / 14 ファイル、known-positive は skills/ でヒット) だが、この 0 件は「Before 型の排除」を意味しない | **未達 [Medium]** |

**親バッチからの差分**: CLAUDE.md は 146 行 (5/30 時点、同 log L42「146>60」) → **59 行**へ減量が着地。ただし減量 (K-2.1 の軸) と理由ベース化 (E-2 の軸) は別物で、後者は未達のまま — 「理由ベース rules ✅」という親バッチの 1 行は **rules/ に限れば正しく、CLAUDE.md には当てはまらない**ことが分布実測で判明した。

### 個別照合 — 反例狩り: 裸の命令行 (第 2 版で再計数)

理由が同一行にも直後行にも無い命令行 = **8 件** (初版の 5 件は undercount、独立検証が 4 件を追加検出しメインで再測。初版計上の `harness-modification-policy:12` は `→ doc` で終わるポインタ行のため carve-out 側へ再分類 — ポインタのみ rule 3 本と同じ基準を適用):

- `CLAUDE.md:33` (冒頭/末尾マーカー必須) / `:34` (三段構成固定) / `:39` (曖昧回答禁止 — json ポインタのみ) / `:42` (**絶対禁止 8 項目の裸列挙**) / `:45` (Claude Only・組込禁止) / `:49` (AskUserQuestion 必ず使用)
- `multi-agent-safety.md:7`「`git stash`操作→明示指示なしで不可（`--autostash`含む）」/ `:8`「`git worktree`操作→明示指示なしで不可」

`--autostash` の括弧列挙は記事の「増殖」が向かう先の**形**をしている — **ただし `git log -S` では規則本体と括弧が同一 commit `9d8bf2e` で同時投入されており、事後追記の痕跡は無い** (snapshot commit ゆえそれ以前の履歴は追えない = 「増殖の過程を経た」とは言えない。初版の「増殖パターンの初期形」という時系列断定は独立検証が反証し撤回)。

### 記事超え点 (第 2 版で 1 点に絞り込み)

- **合理化文字列の rule 本文へのインライン列挙**: `decisive-answers.md` L16 / `grounds-not-approval.md` L23 の「迂回パターン (LLM の合理化) — これらを禁止する」節 (bullet 3+4 = 7 件。見出し完全一致 grep でこの 2 ファイルのみ)。記事は C-5 で「賢いモデルほど抜け道を見つける」を扱う (569-581 行) が、**E-2 節では扱わず** (2261 は無条件の性善説)、かつ**実際に観測された合理化の文字列を、回避先の rule 自身へ列挙して塞ぐ**という実装形は記事のどこにも無い。超えているのはこの狭い一点であり、初版の「悪用面への防御は記事に無い層」は**過大で撤回** (C-5 が一般形を扱っている)。
- 却下理由の必須化 (ADR REQUIRED) は記事 After 型の一段先として維持。コストの「制度化」(issue 管理 #322 + 定量開示) も記事に無い形として維持 — ただし**記事 2267 も「コンテキストを消費するだけだ」とコスト自体には言及している** (初版の「記事はコストに触れない」は不正確で訂正)。
- 補記: 「迂回」の語は `~/.claude` 全体で **128 ファイル** (`*.md` 限定 91) にヒットするが、多くは hook バイパス env (`SKIP_*_GATE`) の別概念 (`.docs/hook-authoring-guide.md:37/38/111` 等)。**ただし全部ではない** — `CLAUDE.md:41`「合理化での迂回禁止: reference `rules/decisive-answers.md`」は同一概念のポインタ (初版の「残りは別概念」という全称は反例 1 件で崩れ訂正)。

### 残差 / 改善候補

- **[Medium] CLAUDE.md 本体の規則セクションが理由ゼロ + 裸列挙 8 件**: 記事 E-2 の名指し対象そのもので未達。severity を Medium とする理由: ①E-2 の中核対象での不適合 (Low にできない) ②ただし実害 (未知ケースでの誤判断・ルール衝突) は未観測で、59 行ゆえ「肥大による任意扱い」の条件も薄い (Critical/High にしない)。**処方**: 行単位で「理由を足す / 指し先の rule へ委譲する (L39・L41 型) / 削る」を判断する琢磨。**#322 (常時注入層 GC) への合流は不適** — GC は行を減らすレーンで、理由付与は行が増える施策と方向が逆 (初版の処方は独立検証の指摘で差し替え)。**HITL 処分 (2026-08-08、AskUserQuestion 実結果 = 「issue 起票」)**: `dendedev/claude-harness#339` として起票済み (OPEN を gh で独立確認)。処方の優先順 = 委譲・削除 > 追記 (60 行理想との張力ゆえ)。実作業は CLAUDE.md が Dende の私物ファイルであるため行ごとの HITL 前提。
- **[gap に数えない] ポインタ型の理由ゼロ**: rule 3 本 + `harness-modification-policy:12` + `CLAUDE.md:39/:41`。理由は指し先に住む設計 (ADR-0001)。read-miss リスクとのトレードオフとして記録のみ。

判定: **E-2「ルールより理由で汎化する」= 分割判定** — **essence 正本 (原則の逐語存在) + rules/ (失敗昇格型の理由骨格 + 増殖防御 3 機構) = 取り入れ済み** / **CLAUDE.md 本体 = 未達 [Medium]** (記事が名指しする対象で、規則 4 セクションの理由ゼロと Before 型日本語同型が実測された)。

## step6 独立検証の記録

fresh な read-only reviewer (`code-reviewer` 型) 1 体に判定ログ本文と読み取り権限のみを渡した (アンカリング防止・報告はファイル書き出し)。**verdict: NO-GO** — CONFIRMED 10 / UNTRACEABLE 2 / OVERCLAIM 3 / severity 過小 1 / 判定の支持不足 1 (指摘 11 件)。

**倒れた側 (全て本版へ反映済み。値はメインで再測して一致を確認)**:

| # | 指摘 | 初版 | 是正 |
|---|---|---|---|
| High-1 | 判定の支持不足 | CLAUDE.md 分を「59 行」「英字 0 件」の**代理指標**で取り入れ済みに算入 | **判定を分割** — 直接測定 (bullet 33 中 理由 3・規則 4 セクション 0) に基づき CLAUDE.md = 未達 [Medium] |
| High-2 | OVERCLAIM | 「悪用面への防御は記事に無い層」 | **C-5 (569-581) が一般形を扱う**と認め、超え点を「合理化文字列のインライン列挙」の 1 点に縮小 |
| Med-3 | severity 過小 | 裸命令 5 件・[Low] | **8 件** (CLAUDE.md 4 件追加・ポインタ 1 件を carve-out へ再分類)・**[Medium]** |
| Med-4 | OVERCLAIM | 「--autostash は増殖の初期形」(時系列断定) | `git log -S` = 同一 commit `9d8bf2e` 同時投入で**撤回**、形の類似のみ残す |
| Med-5 | UNTRACEABLE | 「迂回 97 ファイル」 | **128 (全) / 91 (*.md)** へ置換 + 走査範囲併記 |
| Med-6 | OVERCLAIM | 「残りは hook バイパス env の別概念」(全称) | `CLAUDE.md:41` の反例を明記し「多くは」へ |
| Low-7 | UNTRACEABLE | 「禁止 10 件」 | **6 行 / 8 出現**へ置換 |
| Low-8 | 分類の内部矛盾 | `harness-modification-policy:12` をポインタ carve-out と裸命令の両方に | carve-out 側へ統一 |
| Low-9 | 引用精度 | 「」内引用 2 箇所が語を脱落 | ADR-0001 L49 を真の逐語へ・hmp:12 は再分類で解消 |
| Low-10 | 比較の不誠実 | 「記事はコストに触れない」 | 記事 2267 の「コンテキストを消費するだけだ」を明記して訂正 |
| Info-11 | 記載漏れ | 両トークン 0 の 6 本目が「ほか」に埋没 | 本版は 3 本を名指しし分類を明示 |

**持ちこたえた側**: 行番号・逐語の精度は「1 件の誤りも無し」と評価 (L161-163 / 記事 7 範囲 / 分布値 4+9・2+6・1+1 / 59 行 / bullet 3+4)。0 件の陰性も対照込みで有効。

**学び (検証者の分析をそのまま採る)**: メインが「撃ち直した」と申告した 4 値は 4 件とも一致し、**外れた 2 値 (10 件・97 ファイル) はいずれも撃ち直しの対象外だった**。申告と精度が正確に対応している — **見出し級の数値だけでなく、対照実験・補助の数値も撃ち直し対象に含める**のが次回への処方。

**再ゲートを行わない判断 (Lead)**: E-1 で確立した「検証 1 ラウンド固定」の運用による。是正は検証者の実測値をメインが独立に再測した上での採用 (二重測定) であり、新しい未検証主張を持ち込んでいない。意味論の残余リスクは step8 の Dende による content review が受け持つ。

## 関連ファイル

- `~/.claude/CLAUDE.md` — 59 行・bullet 33 本・裸命令 6 件の実測対象
- `~/.claude/rules/*.md` (13 本) — 理由トークン分布・迂回パターン節
- `~/.claude/.docs/essence/essence-docs/agent-essence.md` — L161-163 (原則 E-2)
- `~/.claude/skills/review-harness/diagnosis-rubric.md` — L53
- `~/.claude/skills/authoring-claude-md/references/context-design-principles.md` — L46/L88/L158-159/L182-203
- `~/.claude/.docs/decisions/_TEMPLATE.md` — L24-26 / `0001-claude-md-detail-placement.md` — L49
- `~/.claude/rules/multi-agent-safety.md` — L7-8 (+ `git log -S` = `9d8bf2e`)
- `~/.claude/.docs/essence/essence-sources/_wip-note-distillation/260405.md` — L200 (推定「確信度 中」が反証された事前蒸留)

## 出典

- 記事本文: `.docs/references/260405_*/text.md` (E-2 = 2201〜2269 行 / C-5 = 569〜581 行)。索引 `~/.claude/.docs/references/BIBLIOGRAPHY.md` (番号 260405)
- 親バッチ判定: `.docs/logs/shared/2026-05-30_note-harness-gap-analysis.md` (L42-43)
- 直前の深掘り: `.docs/logs/shared/2026-08-04_e-1-and-e-1-1-constraint-cascade-periodic-audit-deepdive.md` (第17弾)
- scan / 検証の報告: セッション scratchpad `scan-e2-reasons.md` (251 行) / `gate-e2-v1-verdict.md` (345 行)
