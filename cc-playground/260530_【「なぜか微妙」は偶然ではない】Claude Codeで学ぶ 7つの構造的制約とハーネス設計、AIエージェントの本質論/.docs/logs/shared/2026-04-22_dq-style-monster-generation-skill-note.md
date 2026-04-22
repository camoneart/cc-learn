---
date: 2026-04-22 17:56:00
type: work
topic: dq-style-monster-generation-skill-note
session: ハーネス改善 日次メモ (後日記録)

related_skill: [dq-style-monster-generation, logging]
related_log_ids:
  - 2026-04-22_dq-style-monster-generation-skill
---

# 2026-04-22 ハーネス改善メモ — dq-style-monster-generation skill 新規作成

> その日のハーネス改善を 1 行で残す日次メモ。

## 概要

当日ぶんの日次メモが残っていなかったため、`~/.claude/.docs/logs/local/2026-04-22_dq-style-monster-generation-skill.md` を出典に 1 行へ要約して補填した。

> **記録日: 2026-07-29 (後日記録)**。frontmatter の `date` とファイル名は記録対象日 (2026-04-22) を指す。当日の作業実体は上記ログに基づく。

## 内容

- `dq-style-monster-generation` skill を新規作成 — LLM が「汎用西洋ファンタジー生物」へ収束する癖 (distributional convergence: 学習分布の平均へ寄る現象) を剥がし、13 系統の系統辞書と鳥山明ビジュアル指標でプロンプトを前処理する構成にした。画像生成本体は `nano-banana` へ委譲し、本 skill はプロンプト前処理専任。

## 関連ファイル

- `~/.claude/.docs/logs/local/2026-04-22_dq-style-monster-generation-skill.md` — 当日の詳細ログ (本メモの出典)
- `~/.claude/skills-disabled/dq-style-monster-generation/` — 作成した skill 本体。2026-07-29 時点では `skills-disabled/` へ退避されており無効化状態
