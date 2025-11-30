# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## プロジェクト概要

Claude Code用プラグインのマーケットプレイスリポジトリ。現在は `fe-eol-checker` プラグインを提供。

## リポジトリ構造

```
claude-plugin/
├── .claude-plugin/marketplace.json  # マーケットプレイス設定
├── plugin/                          # fe-eol-checker プラグイン
│   ├── .claude-plugin/plugin.json   # プラグインメタデータ
│   ├── commands/
│   │   ├── eol-plan.md              # /eol-plan 計画作成コマンド
│   │   └── eol-update.md            # /eol-update 実行コマンド
│   ├── agents/
│   │   ├── library-update-analyzer.md  # 調査用エージェント
│   │   └── phase-executor.md           # 実行用エージェント
│   └── skills/
│       ├── library-release-checker/    # リリース情報調査
│       ├── dependency-analyzer/        # EOL・互換性分析
│       ├── code-impact-checker/        # コード影響分析
│       ├── package-updater/            # パッケージ更新実行
│       ├── code-migrator/              # コード移行実行
│       └── test-runner/                # lint/test実行
└── README.md
```

## アーキテクチャ

### コマンド → サブエージェント → スキル の階層構造

#### 計画作成フロー（/eol-plan）

1. **`/eol-plan` コマンド** → 計画書作成
2. **`library-update-analyzer` エージェント** → 調査実行
3. **調査系スキル**:
   - `library-release-checker`: Web調査（GitHub Releases、マイグレーションガイド）
   - `dependency-analyzer`: EOL・互換性情報取得（endoflife.date、npm registry）
   - `code-impact-checker`: コード影響分析（Grep/Glob検索、テストカバレッジ）

#### 実行フロー（/eol-update）

1. **`/eol-update` コマンド** → 計画書に基づいて実行
2. **`phase-executor` エージェント** → フェーズ単位で実行
3. **実行系スキル**:
   - `package-updater`: パッケージ更新（npm/yarn/pnpm/bun、mise/nvm/asdf対応）
   - `code-migrator`: 破壊的変更に対応したコード修正
   - `test-runner`: lint/test/build実行と結果解析

### 段階的アップグレード戦略

Phase 0〜6の順序でアップグレードを計画:
- Phase 0: 事前準備（テスト追加、ベースライン確立）
- Phase 1: ランタイム・言語基盤（Node.js, TypeScript）
- Phase 2: 品質担保ツール（ESLint, Jest/Vitest）
- Phase 3: 主要フレームワーク（Next.js, React）
- Phase 4: 状態管理・データ取得（TanStack Query, Jotai）
- Phase 5: UIライブラリ・その他
- Phase 6: ライブラリ置換

複数メジャーバージョンを上げる場合は、Phase 1〜6を複数サイクル繰り返す。

## 開発時の注意

- プラグインのMarkdownファイルはClaude Codeのプロンプトとして解釈される
- YAML frontmatter（`---`で囲まれた部分）でメタデータを定義
- スキルの`allowed-tools`でそのスキルが使用できるツールを制限
