---
date: 2026-08-02 21:31:49
type: study
topic: subagent-architecture-study
session: 
related_article: /Users/camone/dev/claude-code/claude-code-learn/cc-playground/260802_SubAgentの本質と分け方とは？マルチエージェント【Claude Code勉強会90分動画有】/.docs/references/sources/260426_SubAgentの本質と分け方とは？マルチエージェント【Claude Code勉強会90分動画有】/text.md
related_skill: [explain-in-html]
---

# サブエージェントとハーネス設計の学習ログ

> まさお氏の記事「SubAgentの本質と分け方」を元に、context: fork、subagent、memoryの連携を学習。

## 概要

Claude Codeのマルチエージェント（オーケストレーター＋サブエージェント）における設計原則と、ユーザー（デンデ）のローカルハーネス（`~/.claude/`）での実装状況を照らし合わせながら、本質的な知識を深めた。

## 内容

- **context: fork と subagent:**
  - `context: fork` はメイン会話を汚さない「隔離部屋（別スレッド）」。
  - `subagent:` は別室に送り込む「専門家（役割ごとの人格）」。
  - 単体では意味が薄く、セットで使うことで初めて「メインの脳みそを汚染せず、高度な専門タスクを並列化」できる。
  - デンデのハーネスでは「fork時は必ずsubagentを指定する」という厳格なルールが敷かれており、理想的な設計となっている。
- **使うべき場面と使わない場面:**
  - **使うべき:** TDDサイクル、監査、複数ファイルの並列処理など、試行錯誤のログがメインを汚染しやすい重厚なタスク。
  - **使わないべき:** 単一ファイル小修正、ユーザー対話が必要なタスク（サブエージェントは対話不可）。
  - 「その中間ログをメインに残したいか？」が切り分けの究極の問い。
- **Agent Teams と Subagents:**
  - **Subagents** は親子（縦構造）の一方通行の委譲。1人で完結できる作業向け。
  - **Agent Teams** はチーム（横構造）の相互協調。調整コストが高いので、まずは Subagent で完結させるのが鉄則。
- **Memoryの概念と実体:**
  - 「メモリが育つ」＝ 専門家ごとの純度の高い学習ノート（ルールや癖）が蓄積されていくこと。
  - ネイティブでは `~/.claude/` 内部のJSONに隠蔽されるが、デンデのハーネスでは `MEMORY.md` として物理ファイルに抽出し、透明性を担保している設計思想が確認できた。
  - `CLAUDE.md` は「人間が管理する絶対の憲法」、`MEMORY.md` は「AIが自動で育てる学習ノート」。
- **ロングテール知識:**
  - 出番は少ないが種類が膨大な知識。これをエージェントの人格に持たせると保守が破綻するため、Skillやドキュメントに逃がして適宜読み込ませるのがベストプラクティス。

## 関連ファイル

- `/Users/camone/dev/claude-code/claude-code-learn/cc-playground/.docs/output/explain-in-html/260802_context-fork-and-agent.html` — `explain-in-html` スキルで生成した context: fork と subagent: の解説ダッシュボード
- `/Users/camone/dev/claude-code/claude-code-learn/cc-playground/.docs/output/explain-in-html/260802_subagent-usecases-from-note.html` — `explain-in-html` スキルで生成した「使うべき場面・使わない場面」の解説ダッシュボード
