# Frontend EOL Checker Plugin

フロントエンドプロジェクトの依存ライブラリのEOL（End of Life）状況を調査し、アップデート計画を作成するClaude Codeプラグインです。

## 機能

- ライブラリのリリース情報・破壊的変更の調査
- 依存関係の互換性・EOL状況の分析
- 既存コードへの影響箇所の特定
- テストカバレッジの確認と追加テストの提案
- 段階的アップグレード計画の作成

## インストール

### 方法1: マーケットプレイスからインストール（推奨）

Claude Code内で以下のコマンドを実行:

```bash
# 1. マーケットプレイスを追加
/plugin marketplace add snhrm/claude-fe-eol-plugin

# 2. プラグインをインストール
/plugin install claude-fe-eol-plugin@claude-fe-eol-plugin
```

または対話型メニューで:
```bash
/plugin
```
→ 「Browse Plugins」を選択してインストール

### 方法2: プロジェクトにコピー

プラグインの内容をプロジェクトの `.claude/` ディレクトリにコピー:

```bash
git clone https://github.com/snhrm/claude-fe-eol-plugin.git
cp -r claude-fe-eol-plugin/skills /path/to/your-project/.claude/
cp -r claude-fe-eol-plugin/agents /path/to/your-project/.claude/
cp -r claude-fe-eol-plugin/commands /path/to/your-project/.claude/
```

## アンインストール

```bash
/plugin
```
→ 「Manage Plugins」→ プラグインを選択して削除

## 使い方

### スラッシュコマンド

#### `/eol-plan` - EOL対応計画の作成

プロジェクト全体または特定のライブラリのEOL対応計画を作成し、Markdownファイルに出力します。

```bash
# プロジェクト全体を対象に計画作成
/eol-plan

# 特定のライブラリを指定
/eol-plan react@19

# 複数ライブラリを指定
/eol-plan react@19 next@15 typescript@5.5
```

**出力先の指定（オプション）**

`CLAUDE.md` に以下を記載すると、出力先をカスタマイズできます:

```markdown
EOL計画出力先: docs/eol-plan.md
```

指定がない場合は、プロジェクトルートに `eol-plan.md` が作成されます。

### サブエージェント

#### `library-update-analyzer`

特定のライブラリのアップデート影響を詳細に調査します。

```
ライブラリ: react
現在のバージョン: 18.2.0
アップグレード先バージョン: 19.0.0
```

調査内容:
- 破壊的変更と影響範囲
- 依存関係の連鎖アップデート要件
- 既存コードへの影響箇所
- テストカバレッジと追加テスト要件

### スキル

プラグインには3つのスキルが含まれており、必要に応じて自動的に活用されます。

| スキル | 説明 |
|--------|------|
| `library-release-checker` | GitHubや公式ドキュメントからリリース情報を収集 |
| `dependency-analyzer` | 依存関係の互換性とEOL状況を分析 |
| `code-impact-checker` | 既存コードへの影響とテストカバレッジを確認 |

## 出力例

### EOL対応計画

```markdown
# EOL対応計画

作成日: 2024-01-15
対象プロジェクト: my-frontend-app
パッケージマネージャー: pnpm

## 概要

| 項目 | 内容 |
|------|------|
| 対象ライブラリ数 | 5件 |
| セキュリティ脆弱性 | 2件 |
| リスクレベル | 中 |
| 推奨戦略 | 段階的 |

## 段階的アップグレード計画

### Phase 1: セキュリティ修正
- lodash: 4.17.20 → 4.17.21 (パッチ)

### Phase 2: マイナーアップデート
- typescript: 5.0.0 → 5.3.0

### Phase 3: メジャーアップデート
- react: 18.2.0 → 19.0.0
- next: 14.0.0 → 15.0.0
```

## 対応パッケージマネージャー

| パッケージマネージャー | 検出ファイル | audit対応 |
|----------------------|-------------|----------|
| npm | `package-lock.json` | ✅ |
| yarn | `yarn.lock` | ✅ |
| pnpm | `pnpm-lock.yaml` | ✅ |
| bun | `bun.lockb`, `bun.lock` | ⚠️ npm audit代用 |

## プラグイン構造

```
claude-fe-eol-plugin/
├── .claude-plugin/
│   └── plugin.json          # プラグイン設定
├── commands/
│   └── eol-plan.md          # /eol-plan コマンド
├── agents/
│   └── library-update-analyzer.md  # サブエージェント
├── skills/
│   ├── library-release-checker/
│   │   └── SKILL.md         # リリース情報調査スキル
│   ├── dependency-analyzer/
│   │   └── SKILL.md         # 依存関係分析スキル
│   └── code-impact-checker/
│       └── SKILL.md         # コード影響分析スキル
└── README.md
```

## 主要な調査対象ライブラリ

このプラグインは以下のフロントエンドライブラリのEOL調査に対応しています:

### フレームワーク
- React
- Next.js
- Vue.js
- Angular
- Svelte

### ランタイム
- Node.js

### ビルドツール
- TypeScript
- Vite
- Webpack

## 参考リンク

- [endoflife.date](https://endoflife.date/) - EOL情報データベース
- [npm-check-updates](https://www.npmjs.com/package/npm-check-updates) - パッケージ更新確認ツール

## ライセンス

MIT
