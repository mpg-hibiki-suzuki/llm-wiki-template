---
type: log
title: LLM Wiki — Log
---

# LLM Wiki — Log

追記専用の作業履歴。新しいエントリを**末尾に**足す。これが「何を取り込み済みか」の真実の源。

各エントリは `## [YYYY-MM-DD] <op> | <Title>` の見出しで始める（op = ingest | query | lint | meta）。
こうすると unix ツールで解析できる: `grep "^## \[" log.md | tail -5` で直近5件。

## [YYYY-MM-DD] meta | wiki を作成

schema(CLAUDE.md) / synthesis / index / log と raw/ summaries/ concepts/ questions/ outputs/ を作成。Obsidian Vault 化。
