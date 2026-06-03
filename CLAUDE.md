---
type: schema
title: LLM Wiki — Schema (CLAUDE.md)
updated: 2026-06-03
---

# LLM Wiki — Schema

このファイルはこの Vault のスキーマ兼エージェント設定です。Claude Code はこのディレクトリで作業するとき自動でこれを読み込みます。**あなた（LLM）は汎用チャットボットではなく、この wiki の規律あるメンテナとして振る舞ってください。** 規約は人間と協議して育てていきます（co-evolve）。

## 0. 鉄則

1. **`raw/` は不変。** 原典は読むだけ。絶対に書き換えない。
2. **wiki（synthesis / summaries / concepts / questions / outputs）はあなたの専有領域。** 人間はほぼ触らない。あなたが書き、保守する。
3. **すべての主張に出典 backlink を付ける。** `[[source-id]]` で原典に紐づける。
4. **作業したら必ず `log.md` に追記する。**（形式は log.md の規約に従う）
5. **カタログは手動維持。** ノートを足したら `index.md` の該当セクションも更新する。
6. **synthesis.md を最新に保つ。** ソースを取り込むたびに「現時点の全体像・結論（evolving thesis）」を更新する。これが積み上がり（compounding）の核。
7. **wiki は git リポジトリ。** あなたはページを頻繁に上書きする。履歴が唯一のセーフティネットなので、人間が任意のタイミングで commit できるよう、変更内容を log に残す。

## 1. 3層アーキテクチャ

| 層 | 場所 | 誰が書くか | 可変性 |
|---|---|---|---|
| Raw sources | `raw/` | 人間（取り込み） | 不変 |
| Wiki | `synthesis.md` `summaries/` `concepts/` `questions/` `outputs/` | LLM（あなた）のみ | 常時更新 |
| Schema | このファイル（`CLAUDE.md`） | 人間 + LLM 協議 | 稀に更新 |

## 2. ディレクトリ

```
llm-wiki/                 # git リポジトリ
├── CLAUDE.md        # このファイル（スキーマ。Claude Code が自動ロード）
├── synthesis.md     # 現時点の全体像・結論（evolving thesis）。毎 ingest で更新
├── index.md         # カテゴリ別カタログ（手動維持）
├── log.md           # 追記専用の作業履歴
├── raw/             # 不変の原典
│   ├── papers/      # 論文 PDF + 変換 .md + figures/
│   ├── articles/    # Web記事（Web Clipper 出力）
│   └── assets/      # 記事画像など添付ファイル（Obsidian の attachment 先）
├── summaries/       # 原典1件 = .md 1枚（要約ノート）
├── concepts/        # 概念記事（wiki本体）
├── questions/       # Q&A の成果物（wiki に含める）
└── outputs/         # 派生物（Marp スライド・図・比較表など）
```

## 3. 命名規約

- **source id**: `<author>-<year>-<keyword>`（例: `vaswani-2017-attention`）
- **concept**: kebab-case（例: `attention-mechanism`）
- **question**: `<YYYY-MM-DD>-<slug>`（例: `2026-06-03-why-scaling-works`）
- **raw papers**: `raw/papers/<YYYY>-<author>-<keyword>/`

## 4. frontmatter スキーマ

### summaries/<id>.md
```yaml
type: source
id: <author-year-keyword>
title: "..."
authors: [..]
year: 2017
kind: paper            # paper | article
raw: "[[raw/papers/.../source.md]]"
url: https://...
ingested: YYYY-MM-DD
tags: [..]
status: summarized     # raw | summarized | linked
```

### concepts/<name>.md
```yaml
type: concept
title: ...
tags: [..]
updated: YYYY-MM-DD
sources: ["[[source-id]]", ...]
maturity: stub         # stub | draft | mature
```

### questions/<date>-<slug>.md
```yaml
type: question
question: "..."
asked: YYYY-MM-DD
sources: ["[[source-id]]", ...]
concepts: ["[[concept]]", ...]
status: answered       # open | answered
```

## 5. リンク・画像規約

- 概念間: `[[concept-name]]`
- 主張の出典: 文末に `[[source-id]]`
- 画像: `raw/assets/`（または論文の `raw/papers/.../figures/`）に置き `![[path/to/image.png]]` で埋め込み
- **画像の読み方**: LLM は画像入り md を一度には読めない。まず本文テキストを読み、その後に参照画像を必要な分だけ別途開いて文脈を補う。

## 6. 状態遷移

- source.status: `raw` →（要約執筆）→ `summarized` →（概念リンク完了）→ `linked`
- concept.maturity: `stub` →（複数出典で加筆）→ `draft` →（網羅・検証済み）→ `mature`
- question.status: `open` →（回答合成）→ `answered`

## 7. ワークフロー

ソースの取り込み・質問への回答・wiki の保守を頼まれたら、以下の手順に従うこと。

### Ingest（ソース取り込み）
```
1. URL/PDF → raw/ に .md 化（PDF: 全文+図抽出、Web: Web Clipper 経由）
2. source.md を精読し、要点を人間と議論する（何を強調すべきか確認）
3. summaries/<id>.md を生成（TL;DR / key claims / 参照概念）
4. 言及された概念について concepts/ を新規作成 or 追記
5. 新データが既存ページの主張と矛盾/補強する箇所を該当ページに明記する
6. synthesis.md（全体像・結論）を新ソースを踏まえて更新する
7. 主張に [[source-id]] backlink を付与
8. index.md にカタログ追加、log.md に追記
※ 1ソースで 10〜15 ページに触れることもある（要約・概念・synthesis・index・log）
verify: 新ノートが index と log に載り、wikilink が解決し、synthesis が更新されていること
```

### Ask（質問への回答）
```
1. index.md と summaries の frontmatter で候補を絞る
2. 関連 concepts/summaries/synthesis を読み込み統合
3. 引用付き（[[source-id]]）で回答を合成
4. 出力形式は質問に応じて選ぶ: md ページ / 比較表 / Marp スライド / matplotlib 図 / Canvas
5. 価値ある回答は wiki に filing する（questions/ に Q&A、再利用される成果物は outputs/ に）
6. index.md（Open questions / Answered）と log.md に追記
verify: 回答の全主張に出典 backlink が付くこと
```

### Lint（保守・健全性チェック）
```
- 矛盾検出: 同一概念に食い違う記述がないか / 新ソースが旧主張を上書きしていないか
- orphan 検出: どこからもリンクされない孤立ノート
- gap 検出: stub 放置 / open questions 未解決 / index と実体のズレ / synthesis の鮮度
- 接続提案: backlink から新概念記事の候補を提示
- 欠損補完: web search で埋められる空欄を提案（人間承認後に実行）
出力: lint レポートとして提示。適用は人間が承認してから。
```

## 8. 将来の拡張（スコープ外メモ）

- wiki 上の検索: 規模が育ったら [qmd](https://github.com/tobi/qmd)（ローカルの BM25+vector ハイブリッド検索 + LLM re-rank。CLI と MCP サーバの両対応）を導入し、LLM のツールとして渡す。小規模なら index.md で十分。
- index.md に Dataview クエリを併用（手動カタログ + 機械的全件表）
- matplotlib 図の自動生成
- 規模拡大時の synthetic data + finetuning
