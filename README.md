# LLM Wiki (Template)

> **Claude Code を「規律あるリサーチ wiki のメンテナ」として運用するための Obsidian Vault 雛形。**

論文・記事を読み込ませると、Claude Code が要約・概念記事・全体結論を**出典付きで書き溜め・更新**していく。読み込むたびに知識が積み上がる（compounding）ことを狙った構成です。この雛形はスキーマと空のディレクトリ構造だけを含み、コンテンツは入っていません。

---

## これは何か

人間が原典を放り込み、LLM が wiki を育て、スキーマは両者で協議して進化させる——という **3層アーキテクチャ**で動きます。

| 層 | 場所 | 誰が書くか | 可変性 |
|---|---|---|---|
| Raw sources | `raw/` | 人間（取り込み） | **不変**（原典は読むだけ） |
| Wiki | `synthesis.md` `summaries/` `concepts/` `questions/` `outputs/` | **LLM（Claude Code）のみ** | 常時更新 |
| Schema | `CLAUDE.md` | 人間 + LLM 協議 | 稀に更新 |

ポイント:

- Claude Code はこのディレクトリで作業を始めると [`CLAUDE.md`](CLAUDE.md)（スキーマ兼エージェント設定）を**自動で読み込み**、その規約に従って汎用チャットボットではなく「この wiki のメンテナ」として振る舞います。
- 成果物はすべて Markdown。Obsidian の `[[wikilink]]`・graph view・backlink でブラウズできます。
- 全主張には `[[source-id]]` で原典への backlink が付き、作業履歴は [`log.md`](log.md) に追記されます。

詳細な規約・ワークフロー・frontmatter スキーマは [`CLAUDE.md`](CLAUDE.md) が唯一の正本です。

---

## ディレクトリ構成

```
llm-wiki/                 # git リポジトリ / Obsidian Vault
├── CLAUDE.md        # スキーマ兼エージェント設定（Claude Code が自動ロード）
├── synthesis.md     # 現時点の全体像・結論（evolving thesis）。毎 ingest で更新
├── index.md         # カテゴリ別カタログ（手動維持）
├── log.md           # 追記専用の作業履歴
├── raw/             # 不変の原典
│   ├── papers/      # 論文 PDF + 変換 .md + figures/
│   ├── articles/    # Web 記事（Obsidian Web Clipper 出力）
│   └── assets/      # 添付画像など（Obsidian の attachment 先）
├── summaries/       # 原典1件 = .md 1枚（要約ノート）
├── concepts/        # 概念記事（wiki 本体）
├── questions/       # Q&A の成果物
└── outputs/         # 派生物（Marp スライド・図・比較表など）
```

---

## セットアップ

### 0. この雛形から新しい wiki を始める

```bash
# このリポジトリを取得し、自分の wiki として git を貼り直す
git clone git@github.com:mpg-hibiki-suzuki/llm-wiki-template.git my-llm-wiki
cd my-llm-wiki
rm -rf .git && git init -b main
```

> `synthesis.md` / `index.md` / `log.md` の `YYYY-MM-DD` プレースホルダは、最初の作業時に Claude Code が埋めます（手で消す必要はありません）。

### 1. Claude Code

1. インストール（最新手順は[公式ドキュメント](https://docs.claude.com/en/docs/claude-code)参照）:
   ```bash
   npm install -g @anthropic-ai/claude-code
   ```
   ターミナル・VS Code 拡張・JetBrains 拡張のいずれでも利用できます。
2. wiki ディレクトリで起動:
   ```bash
   cd my-llm-wiki
   claude
   ```
3. 起動すると同ディレクトリの [`CLAUDE.md`](CLAUDE.md) が**自動で読み込まれます**。動作確認は「この wiki のスキーマを要約して」などと尋ねれば OK。

### 2. Obsidian

1. [Obsidian](https://obsidian.md/) をインストール。
2. **Open folder as vault** で wiki ディレクトリを選択。
3. Vault 設定は `.obsidian/` に同梱済みです:
   - `[[wikilink]]` 形式（`useMarkdownLinks: false` / `newLinkFormat: shortest`）
   - 添付ファイルの保存先 = `raw/assets`
   - graph / backlink / outgoing-link / tag-pane / outline などの core plugin を有効化済み
4. （任意）Web 記事を取り込むなら **Obsidian Web Clipper** を導入し、出力先を `raw/articles/`、添付先を `raw/assets/` に設定。

> per-machine 状態（`.obsidian/workspace.json` など）は `.gitignore` 済みです。

---

## 使い方（基本ワークフロー）

Claude Code に自然言語で頼むだけで、[`CLAUDE.md`](CLAUDE.md) の手順に従って複数ファイルを横断更新します。

### Ingest（ソース取り込み）

```
この PDF（or URL）を ingest して
```

→ `raw/` に原典を格納 → `summaries/<id>.md` を生成 → 関連 `concepts/` を新規作成/加筆 → `synthesis.md`（全体像）を更新 → `index.md` と `log.md` に追記。

### Ask（質問への回答）

```
〇〇について、これまでのソースを踏まえて教えて
```

→ 関連ノートを統合し**出典付き（`[[source-id]]`）**で回答 → 価値ある回答は `questions/` に filing。

### Lint（保守・健全性チェック）

```
wiki を lint して
```

→ 矛盾検出 / orphan ノート検出 / stub・未解決 question の gap 検出 / 接続提案をレポート（適用は承認後）。

---

## 鉄則（抜粋）

1. **`raw/` は不変。** 原典は読むだけ。書き換えない。
2. **wiki は LLM の専有領域。** Claude Code が書き、保守する。
3. **すべての主張に出典 backlink（`[[source-id]]`）を付ける。**
4. **作業したら必ず `log.md` に追記する。**
5. **`synthesis.md` を常に最新に保つ。** これが積み上がり（compounding）の核。

> 全文は [`CLAUDE.md`](CLAUDE.md) を参照。
