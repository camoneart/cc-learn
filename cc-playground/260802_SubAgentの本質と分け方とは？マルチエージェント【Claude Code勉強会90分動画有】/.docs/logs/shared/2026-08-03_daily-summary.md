---
date: 2026-08-03 03:30:00
type: work
topic: daily-summary
session: daily-summary-retroactive
related_skill: [logging]
---

# 2026-08-03 デイリーサマリー — ペルソナを Cursor rule へ移植

> 深夜 03:30 の 1 コミット分。Claude Code 側のペルソナ規約を Cursor の always-apply rule として playground へ移植した。

## 概要

この日ファイルとして残った作業は 1 件。`cc-playground/.cursor/rules/agent-persona.mdc` (69 行) の新規作成で、Claude Code のハーネスが持つペルソナ・禁止事項・参照規約を Cursor 側でも効かせるための移植。

**記録メタ (遡及作成の開示)**: 本ログの実記録日は 2026-08-08。ファイル名と frontmatter `date` は作業日 2026-08-03 に合わせてあり、commit の author date も同日へ遡及させている。遡及である事実をここで開示する。

**配置理由の開示**: 移植対象の `.cursor/` は playground 全体に掛かるもので、本記事ディレクトリ固有の成果物ではない。それでもここに置いたのは、作業時刻 03:30 が前日 (08-02) のサブエージェント学習セッションの地続きであり、記録を分断しないため。playground 直下に共通のログ層が無いことも理由。

## 内容

### 実測した当日の変更

`.DS_Store` を除くと、この日更新されたファイルは 1 本だけだった。

| 時刻 (JST) | ファイル | 内容 |
|---|---|---|
| 03:30 | `cc-playground/.cursor/rules/agent-persona.mdc` | 新規 69 行 / 4232 バイト |

### 移植した rule の構造

frontmatter は `description:` 空 + `alwaysApply: true`。Cursor で常時適用される形。節構成は元のハーネス規約をほぼそのまま踏襲している。

- `Username` / `Persona` (性格・口調) — キャラクタ定義
- `Response` — 応答構造の規約
- `Prohibition` — 禁止事項
- `Interview` — 質問時は `AskUserQuestion` を使う規約
- `Stack / References` — 着手時に読む参照層への導線
- `Use of a harness` — **移植先で新設した節** (元のハーネス規約には無い)

### 元規約との差分 (diff 実測)

ペルソナ節を機械 diff にかけたところ、差異は **3 箇所のみ**で、いずれも二人称の表記ゆれだった (`Dende` → `デンデ`)。内容の改変は無い。

一方、構造上の違いは 2 点ある。

- **参照パスに `@` prefix が付いている** — `@~/.claude/.docs/progressive-disclosure/...` の形。Cursor のファイル参照記法に合わせた変換で、Claude Code 側の素のパス表記とは異なる。
- **`Use of a harness` 節が新設されている** — prompt に `skill` / `skills` が出たら skills ディレクトリを、`agent` / `agents` / `subagent` / `subagents` が出たら agents ディレクトリを見に行ってから応答する、という 2 行の誘導。Claude Code はこれらをネイティブに解決するが、Cursor は解決しないため、移植先で明示的に補う必要があった。

## 学び

- **規約の移植は「コピー」では終わらない。** 本文の 96% はそのまま通ったが、ホストが変わると (a) 参照記法の変換と (b) ホストがネイティブに持っていた機能の明文化、の 2 種類の追加作業が発生した。`Use of a harness` 節はまさに後者で、移植して初めて「Claude Code が暗黙にやってくれていたこと」が可視化された形になる。
- **2 箇所に同じ規約を置いた時点でドリフトが始まる。** 現状は表記ゆれ 3 箇所の差で済んでいるが、片方だけ更新すれば静かに乖離する。同期の手立ては現時点で無い。

## 関連ファイル

- `../.cursor/rules/agent-persona.mdc` — 移植した Cursor rule 本体 (本ログ時点では未コミット)
