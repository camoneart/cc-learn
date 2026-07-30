# ハーネス issue 第4波 マージゲート + クローズ (2026-07-30 〜 07-31 未明)

- 対象: kaijutale/claude-harness 第4波 5 PR = #283 (issue-269-270) / #284 (issue-246-264) / #285 (issue-221) / #286 (issue-260-262) / #287 (issue-222-223)。マージで issue 10 本 close (#221/#222/#223/#246/#260/#261/#262/#264/#269/#270)
- 着地: 全 5 本マージ。merge commits = 62d9181 (#283) / 26a3335 (#285) / 9ca5426 (#286) / 2783bb3 (#284、かいじゅう直接マージ) / f79df5a (#287)。live pull 済み (HEAD = f79df5a)、最終 main の dispatch 検査 25/0

## 初回レビュー (fresh code-reviewer 5 並列)

- 1 PR 1 レビュアー独立。隔離実走 (archive → scratch export) + mutation probe + 捏造スキャン + 縄張り + VERDICT
- 結果: **5 本全部 CONDITIONAL** (Critical 0 / NO-GO 0)。条件数 = #285:6 / #284:6 / #283:8 / #286:7 / #287:6 (計 33)。High = #283:2 (テストの ambient env 依存回帰・書き手の沈黙 fail-open 再生産) / #286:3 (runner 不在の沈黙素通し・代償表が実測と矛盾 217s 接続・設計論理の自己反転) / #287:1 (deny リスト自身を守る entry が無 assert) / #284:0 / #285:0
- **捏造スキャン全 5 本ゼロ** — 人間判定引用は全部 transcript locator 付きで実在確認 (locator 欠落 1 件は Medium → 条件で是正)。第3波 #266 NO-GO の教訓 (逐語+locator) がレーン側に定着した初の波
- 条件は各 PR コメントへ正本化 (第3波定着運用)。運用発見: code-reviewer agent は SendMessage 非配線 → 報告はプレーンテキスト出力で消えるため、**transcript (projects/<slug>/<uuid>.jsonl) から jq で直接回収**する方式を確立

## 条件対応 → 再ゲート

- 全 5 レーンが実走証拠付き「条件対応完了」コメント。#284 は対応中に自分の locator 破損を自己検知して是正 (grounds Gotcha の実例)
- 再ゲート: 初回レビュアー 3 体を SendMessage 再開で再検証させようとして **spend limit で全滅** (第3波と同一の失敗形) → **メインのインライン検証へフォールバック** (mutation 撃ち直し・逆変異 (条件を戻すと赤くなるか)・起票実在の gh 確認・訂正の grep 突合)。全 33 条件通過 → 5 本 GO
- 検証の実効例: #285 は M8/M9 変異をメインが再現 (40/3・29/3 赤 → 復元で全緑)、#286 は 201s trigger を戻す逆変異で「重量テストの非接続」assert が赤くなることを実証、#287 は mut00/mut01 (deny entry 削除) の DETECTED をメインが再現

## 衝突とマージ順

- merge-tree 全 10 ペア実測: 唯一の衝突面 = `hooks/test/run-all.sh` の 3 者 (#284×#286×#287)。**初回測定は zsh の単語分割仕様でペアが潰れ全ペア偽 CONFLICT** → sanity check (各 branch×main = clean、gh の MERGEABLE と一致) で検出し測り直した
- 順序裁定: clean 組 (#283/#285) 先行 → **#286 を衝突クラスタ先頭へ** (宣言式 dispatch の登録漏れ検知 = 機械 backstop を先に main へ入れ、後続の解消ミスを機械が捕まえる側に回す) → #284 → #287
- #284 rebase: run-all.sh を #286 形 byte 一致へ、宣言 2 本追加。同乗した追加分 (#295 の 1 行是正 + essence gate に強制された High 4 件是正) はメインが diff 精査して承認
- #287 rebase: **メインの指示前提 2 点が誤り** (「force-attach ブロックへ 4 本目」→ #286 が機構ごと撤去済みで存在せず /「宣言不要」→ 宣言なしだと dispatch 層2 が赤)。レーンが盲従せず run-all.sh ヘッダと dispatch テストの機械基準に従って反証・是正 (宣言 + GATE_EXCLUDED_KNOWN 4 件目登録)。メインは誤指示を PR コメントで自己申告の上で承認

## インシデント / 学び

1. **並行払い出しの見積もり甘さ (かいじゅう指摘)**: 5 レーンは本体の仕事こそ互いに素だったが、「新テストの登録が中央ファイル run-all.sh に集まる」衝突点を払い出し前に開示しなかった (実測はゲート段階)。実害 = rebase 2 回・喪失ゼロ。**是正: 払い出し前チェックに「成果物が共有ファイル (登録簿・中央リスト) へ書き込むか」を追加** (第5波の選抜で実施済み)。なお #286 の宣言方式でこの衝突点自体が構造的に消滅
2. **レーン間統合欠け (#295)**: #283 の新 script と #286 の完全性検査が別々に正しくても合流した main は赤 — 並行マージの統合検証は「次のレーンの run-all 全緑要件」が拾った。1 行是正 + 起票で解消
3. **probe-before-persist の実効**: handoff 更新のたび hook が発火し、誤記 4 件を捕捉 (未来時刻 4 分 / 払い出し「昼」→ mtime 実測 21:45 / PR 数の鮮度切れ / worktree 状態の陳腐化)。「書く直前に測り直す」が機械強制で回った
4. **essence gate の追随強制**: #284 の rebase commit が gate に block され (record が最終状態を評価していない判定)、SKIP せず record 追随で解消 → その self-eval が High 5 を検出し 4 件是正 + 1 件 issue 化 (#297)。gate の意味論に従う側を選ぶ運用の成功例
5. **stop-words hook との摩擦**: 「かもしれ」(推測ルール)・「ついでに」(追加作業ルール) の字句検出でメイン発言が 2 回 block。どちらも実態は「直後に実測」「裁定済み手順の言い換え」で中身の違反なし — 字句層の精度限界として記録 (発火 ledger には残る)

## 新規起票 (第4波の出力)

- ゲート条件由来: #288 (essence-docs の Bash 素通し + sync gate 文言弱体化 = #222 の残り半分) / #289 (#264 class 残余 11 skill) / #290 (gate count 規約の穴) / #291 ((g) glob の外部 project blast radius) / #292 (assertion 床なし) / #293 (case 52 判別力ゼロ) / #294 (review-harness の述語直呼び)
- レーン作業中の起票: #276〜#282 (第4波レーン群) / #295 (main 赤の統合欠け) / #296 (fork 原則リストのドリフト) / #297 (trigger 宣言射程 > assertion 射程)
- 申し送り: #275 実装時に「201s テストの settings.json 宣言と HEAVY_TESTS 到達可能性定義を再訪」(#286 再ゲートコメント)

## 定着 (第4波で確立した運用)

- (a) **宣言方式** (# run-all-trigger) により「テスト登録 = 中央ファイル書込」が消え、複数レーン並行の衝突源が構造的に閉じた
- (b) **GATE_EXCLUDED_KNOWN** = gate 非到達宣言の機械台帳。「登録は可視化であって解消ではない」の区別を散文でなくリスト+失敗メッセージで運ぶ
- (c) 再ゲートのフォールバック標準 = **メインのインライン検証** (spend limit 下でサブエージェント再開は不可。mutation 撃ち直し・逆変異・起票実在・grep 突合の 4 点セット)
- (d) レビュアー報告の回収 = **transcript 直読み** (code-reviewer は SendMessage 非配線。idle 通知 → jq で最終 assistant text を抽出)

## 残タスク

- worktree 5 本 (issue-221/222-223/246-264/260-262/269-270) の撤収 + Keychain 回収 — かいじゅう指示待ち (multi-agent-safety: PR merge 済み・clean 確認後)
- 第5波 3 PR (#301 issue-268 / #302 issue-272 / #303 issue-274) 到着済み — マージゲート未着手
- ~/.claude の未追跡 progress JSON 2 件は #301 (ignore 化) が拾う見込み
