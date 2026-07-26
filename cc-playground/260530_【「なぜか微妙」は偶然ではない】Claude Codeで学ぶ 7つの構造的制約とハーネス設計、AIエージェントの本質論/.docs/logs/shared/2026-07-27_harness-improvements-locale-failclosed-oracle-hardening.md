---
date: 2026-07-27 07:54:48
type: work
topic: harness-improvements-locale-failclosed-oracle-hardening
session: ハーネス issue 並行改修 (改善内容の記録)

related_skill: [enforcing-strict-tdd-cycle, essence-reviewing-orchestrator, committer]
related_log_ids:
  - 2026-07-26_harness-issue-wave1-close-and-wave2-review
  - 2026-07-24_s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive
related_log:
  - .docs/logs/shared/2026-07-26_harness-issue-wave1-close-and-wave2-review.md
  - .docs/logs/shared/2026-07-24_s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive.md
---

# ハーネス改善の実体 — locale 固定 / 防御 gate の fail-closed / oracle の異常系 hardening

> S 章の深掘りで検出した反例を issue 化し、並行 worktree で改修した内容の記録。第1波 4 件はマージ済み (live 稼働中)、第2波 5 件は PR 4 本が OPEN。**共通するのは「正常系は緑なのに異常系で静かに壊れる」クラスの欠陥**。

## 概要

取り入れフェーズ (note 記事とハーネスの照合) で「記事の原則は取り入れ済み」と判定する過程で、**反例狩り**が実際の欠陥を掘り当てた。その issue 化 → 並行改修の記録。

本ログは**改善の実体** (何が壊れていて、何が直り、どう検証されたか) を残す。セッションの経緯・レビューで検出した問題・メイン自身の手順違反は別ログ (`2026-07-26_harness-issue-wave1-close-and-wave2-review.md`) にある。

## 内容

### 第1波 — マージ済み (live 稼働中)

#### 1. locale 照合による bucket 融合 (#230 / PR #235)

**壊れていたもの**: `hooks/lib/ledger_aggregate.sh` (hook 発火実績の集計器) の `sort | uniq -c` が呼び出し側 locale を継承していた。`en_US.UTF-8` の照合規則では **CJK だけで構成された別の rule 名を同値と判定して bucket を融合**する。

**実害**: 集計器が「一括置換禁止 = 23」と出力していたが、権威計数 (JSON 直接パース) では 12。融合していた 8 件は監査が最重要視した「秘密ファイル読むな」rule の発火実績で、**月次報告から丸ごと消えていた**。取り入れフェーズ第16弾の判定ログ初稿がこの融合値を転記し、独立検証で捕捉・訂正された。

**直ったもの** (`ledger_aggregate.sh:57`):

```bash
export LC_ALL=C
```

出力契約行 (`:31`) にも `bucket 境界と行順は呼び出し側 locale に依存しない (Guaranteed by: export LC_ALL=C)` を明記。実 ledger で **102 bucket → 103 bucket** (差分 1) を確認。

**両刃の開示**: `LC_ALL` は `LC_COLLATE` と同時に `LC_CTYPE` も C にする。この script の `sed`/`awk` は行頭 ASCII しか触らないので無害だが、rule 名そのものへ文字クラス処理を足すと多バイト文字を破壊する — とコメントに残っている。

#### 2. 防御 gate の jq 依存 fail-open (#229 / PR #238)

**壊れていたもの**: block 型 gate 5 本が、jq 不在・入力が非 JSON の異常系で **検査を全スキップして approve / exit 0 に落ちていた**。`hook_pre_commands` は「秘密ファイルの Bash 読取 deny」を含む 9 rule 全てが消える。`hook_pre_commit_settings_churn_normalize` に至っては **ヘッダで「jq 不在 → block (fail-closed)」と宣言しながら、実装は構造的に到達不能な dead code** だった (偽宣言)。

**直ったもの**: 5 本すべてに入力解析可能性の probe を追加し、**部品の役割で落とし先を分けた**:

| hook | 役割 | jq 不在時 |
|---|---|---|
| `hook_pre_commands` | 防御 gate | exit 2 (fail-closed) |
| `hook_pre_commit_essence_gate` | 防御 gate | exit 2 |
| `hook_pre_commit_settings_churn_normalize` | 防御 gate | exit 2 (偽宣言を解消) |
| `hook_pre_worktree_bash_write_guard` | 防御 guard | 武装確定後は exit 2 (対の第1弾と対称化) |
| `hook_stop_words` | Stop gate | **exit 0 + 警告つき approve** (宣言つき fail-open) |

Stop gate だけ非対称なのは、fail-closed にすると jq が復旧するまでセッションを終われなくなるため (可用性コストが防御利得を上回る)。**「判定不能を握り潰す許可」ではなく「役割による意図的な非対称」**として明記されている。

**レビューで見つかった追加の穴**: 空入力 (jq は存在する状態) で `pre_commands`/`essence_gate` が exit 2 に倒れ、**迂回 env がないため復旧不能**になる経路が新設されていた。根因は `printf '' | jq -e .` が **rc=4** を返すこと。5 本を素通しで揃えて解消した。

#### 3. oracle 群の入力異常系 hardening (#231 / PR #236)

**壊れていたもの** 3 件:

| バグ | 実害 |
|---|---|
| `--line-min` などのオプション値が欠落すると **無限ループ** | `shift 2` は引数が 1 個しか残っていない時シフトせず失敗し、`while [[ $# -gt 0 ]]` が同じ引数で回り続ける。blocking gate が呼ぶ script が黙って固まる (実質 Critical 級) |
| `_is_number` が単独ドット `.` を通す | `--line-min .` が検証を通過し awk が `.` を 0 に変換 → 実測 42.86% でも `Coverage OK` rc=0 = **閾値ゲート無効化** |
| 非 Swift 経路が runner の exit code を捨てる | テストが失敗してもカバレッジ表さえ出れば exit 0 |

**直ったもの**:

```bash
# 値欠落ガード — 共通ヘルパーへ切り出し、3 オプションで共有
--line-min)   _need_value "$#" "$1"; LINE_MIN="$2"; shift 2 ;;
--branch-min) _need_value "$#" "$1"; BRANCH_MIN="$2"; shift 2 ;;
--root)       _need_value "$#" "$1"; ROOT_ARG="$2"; shift 2 ;;

# 数値検証 — 正規表現へ置換 (単独 . / 末尾 . / 1.2.3 を拒否)
_is_number() { [[ "$1" =~ ^[0-9]+(\.[0-9]+)?$ ]]; }
```

runner 失敗は `REASON=test-run-failed` を stderr に出して exit 2 (Swift 経路と対称化)。

**実測での修正確認** (メインが独立に叩いた):

- 3 scripts × 2 オプションの **6 ケース全部が exit 2 で即終了** (修正前は `perl alarm` で 5 秒生存 = exit 142)
- `.` / `1.` / `1.2.3` / `-1` / `abc` の **5 種すべて exit 2 で拒否**、`80` / `79.5` は通過

#### 4. coverage 呼出の stack 依存化 (#228 / PR #239)

**壊れていたもの**: `agents/coder.md` の coverage oracle 呼出が `--line-min 80 --branch-min 75` 固定。Swift (`llvm-cov report`) と pytest は分岐カバレッジ % を出力できず、oracle が fail-closed で exit 2 を返す。coder はそれを `SKIPPED(provider無)` の非ブロッキング警告にマップするため、**系としては「閉じた門」ではなく「門が無い」状態**になっていた。ラベルも事実と違う (provider は在り、分岐指標だけが無い)。

**直ったもの** (`agents/coder.md:265`):

```python
BRANCH_MIN = 75 if STACK == "web" else 0
```

exit 2 の理由は stderr の機械可読トークン (`REASON=branch-metric-unsupported` / `REASON=test-run-failed`) で切り分け。**トークン供給を単一の版に依存させない 2 段構え** (トークン + 散文フォールバック) になっている。

あわせて **budget 監査が 5 つの escalation 経路で素通りしていた** バグも自己検出で修正された (`audit_budget()` を全 return 経路の共通前処理へ)。

### 第2波 — PR 4 本が OPEN (2026-07-27 07:54 時点)

| PR | issue | 内容 | 状態 |
|---|---|---|---|
| #247 | #232 | `essence_gate` の staged 収集を locale 非依存化 (#230 の執行器版) | 🟢 GO 判定済み・マージ待ち |
| #248 | #242 | coder の散文フォールバックと正本の byte 同期 | 未レビュー |
| #249 | #237 | 複数パス `<args>` で 3 fork の機械シグナルが全滅 | 未レビュー |
| #250 | #234 | `launch_form_sync` が gitignore 済み dir を走査 | 未レビュー |
| — | #233 | locale 照合の教訓を class guard へ昇格 (post-write lint) | PR 未提出 (worktree に commit 3 件) |

## 設計意図

### 「点の修正」と「class を塞ぐ」を分けて追跡している

#230 (集計器) と #232 (執行器) は**同一根因の 2 箇所目**。同じ判断がハーネス内で 8 箇所以上に場当たりで反復されており、7 箇所目が本番データで壊れていた。

そこで **#233 を「class 側」として分離**し、`sort`/`uniq` を照合固定なしで使う箇所を検出する post-write lint を作る issue にした。点の修正 (#230/#232) と class guard (#233) が別レーンで並走している。

### 射程を偽らない開示が定着した

#232 の実装コメントが典型:

> **射程を偽らないための 2 点** (この 1 行で全ての「静かな脱落」が消えるわけではない):
> - KNOWN GAP: 空白入りファイル名は `:250` の `tr ' \t' '\n'` が別経路で分割するため依然落ちる (実測: `"my file.sh"` → `"my` / `file.sh"`)
> - LC_ALL は両刃: ここは**コマンド前置スコープ**ゆえ安全だが、script 全体へ `export` に広げると多バイト文字の文字クラス処理を壊す

**「直した」と書くだけでなく「直っていない範囲」を同じ場所に書く**。メインが独立実測で裏を取ったところ、KNOWN GAP の主張は正確だった。

### テストが「検出力」を自己申告する

#230 / #232 のテストは 3 段構成 (実挙動 / 契約 / 静的ロック) に加え、**実挙動テストの検出力を probe で測って `[warn]` で開示**する:

```
[warn] en_US.UTF-8 の照合規則はこの環境で融合を起こさない (残存=N)
       -> 実挙動テスト (A) に検出力なし。静的ロック (C) が本件を担保する
```

照合が融合しない実装 (GNU coreutils 等) では実挙動テストが修正前でも緑になる。**黙って緑にせず「このテストは今この環境で何も検出していない」と言う**。

## 副作用

### 残っている穴

| ID | 内容 | 状態 |
|---|---|---|
| CI ゼロ | 全 PR で `statusCheckRollup` が空。テストの緑は手動実測が唯一の根拠 | **未対応** |
| L-4 | `run_case_timeout` で `ps -o pgid=` が失敗し `pgid` が空になると kill がスキップされ `wait` が永久ブロック = ハング検出器自身がハングする | **未実測** (コードの論理からの推論)。issue 化前に再現実験が要る |
| 空白入りファイル名 | #232 の KNOWN GAP。`tr ' \t' '\n'` の別経路 | 未対応 (record に明示) |
| 人間承認の捏造 | 第1波で 12 箇所検出。L1 注意喚起 + L3 機械検出への昇格がかいじゅう裁定で決定 | **未着手** |

### 検証の規模

- hook テストファイル: **47 本** (第1波で 5 本に空入力ケースを追加)
- oracle テストファイル: **9 本** (第1波で `arg-parsing-contract.test.sh` を新設、227 assertions)

いずれも**マージ前にメインが隔離実走で再確認**した。テストが緑であることと、レビューで穴が出ないことは別 — 第1波では全緑の 4 PR から High 2 件・Medium 3 件・Low 2 件が出た。

## 関連ファイル

- `~/.claude/hooks/lib/ledger_aggregate.sh` — #230 の修正 (`export LC_ALL=C` :57)
- `~/.claude/hooks/hook_pre_commands.sh` 他 4 本 — #229 の fail-closed probe
- `~/.claude/skills/enforcing-strict-tdd-cycle/scripts/assert-coverage.sh` — #231 の `_need_value` / `_is_number`
- `~/.claude/agents/coder.md` — #228 の `BRANCH_MIN` stack 依存化 (:265)
- `.docs/logs/shared/2026-07-26_harness-issue-wave1-close-and-wave2-review.md` — 本改修のセッション経緯 (捏造裁定の検出を含む)
- `.docs/logs/shared/2026-07-24_s-1-4-and-s-1-5-propagation-stages-fail-closed-deepdive.md` — #229/#230 の発生源となった反例狩り
