---
name: test-runner
description: lint、test、buildコマンドを実行し、結果を解析する。エラー時は原因分析と修正提案を行う
allowed-tools: Bash, Read, Grep
---

# テスト実行スキル

**役割**: lint/test/buildの実行と結果解析に特化。

**入力**: 実行コマンド、プロジェクト情報（呼び出し元から渡される）

## コマンド実行

### Lint

```bash
# npm
npm run lint

# yarn
yarn lint

# pnpm
pnpm lint

# bun
bun run lint

# 自動修正
npm run lint -- --fix
yarn lint --fix
pnpm lint --fix
bun run lint --fix
```

### Test

```bash
# npm
npm test
npm run test

# yarn
yarn test

# pnpm
pnpm test

# bun
bun test

# 特定ファイルのみ
npm test -- {path}
yarn test {path}
pnpm test {path}
bun test {path}

# watchモードを無効化
npm test -- --watchAll=false
CI=true npm test
```

### Build

```bash
# npm
npm run build

# yarn
yarn build

# pnpm
pnpm build

# bun
bun run build
```

### 型チェック

```bash
# TypeScript
npx tsc --noEmit

# または
npm run type-check
yarn type-check
pnpm type-check
```

## 結果解析

### ESLint結果

```bash
# JSON形式で出力
npm run lint -- --format json

# 出力例
[
  {
    "filePath": "/path/to/file.ts",
    "messages": [
      {
        "ruleId": "no-unused-vars",
        "severity": 2,
        "message": "'x' is defined but never used",
        "line": 10,
        "column": 5
      }
    ],
    "errorCount": 1,
    "warningCount": 0
  }
]
```

### Jest/Vitest結果

```bash
# JSON形式で出力
npm test -- --json --outputFile=test-results.json

# 出力例
{
  "numTotalTests": 50,
  "numPassedTests": 48,
  "numFailedTests": 2,
  "testResults": [
    {
      "name": "/path/to/test.ts",
      "status": "failed",
      "assertionResults": [
        {
          "title": "should work",
          "status": "failed",
          "failureMessages": ["Error: ..."]
        }
      ]
    }
  ]
}
```

### TypeScript結果

```bash
# エラー出力例
src/components/Example.tsx(15,5): error TS2345:
  Argument of type 'string' is not assignable to parameter of type 'number'.
```

### Build結果

```bash
# Next.js
# エラー出力例
Error: Build failed because of webpack errors

# Vite
# エラー出力例
error during build:
RollupError: ...
```

## エラー分類

### Lint エラー

| カテゴリ | 例 | 自動修正 |
|---------|-----|---------|
| フォーマット | indent, semi, quotes | 可能 |
| 未使用変数 | no-unused-vars | 要確認 |
| インポート順 | import/order | 可能 |
| React Hooks | react-hooks/rules-of-hooks | 不可 |

### Test エラー

| カテゴリ | 原因 | 対処 |
|---------|------|------|
| アサーション失敗 | 期待値不一致 | テストまたはコード修正 |
| タイムアウト | 非同期処理 | タイムアウト延長またはモック |
| モジュール解決 | import失敗 | パス確認、モック追加 |
| スナップショット | 不一致 | スナップショット更新 |

### Build エラー

| カテゴリ | 原因 | 対処 |
|---------|------|------|
| 型エラー | TypeScript | 型修正 |
| モジュール解決 | パッケージ不足 | インストール |
| 設定エラー | config不正 | 設定修正 |
| メモリ不足 | heap overflow | NODE_OPTIONS調整 |

## 修正コマンド

### Lintエラーの自動修正

```bash
# ESLint
npm run lint -- --fix

# Prettier
npx prettier --write .

# 両方
npm run lint -- --fix && npx prettier --write .
```

### スナップショット更新

```bash
# Jest
npm test -- -u
npm test -- --updateSnapshot

# Vitest
npx vitest --update
```

### キャッシュクリア

```bash
# Jest
npx jest --clearCache

# Next.js
rm -rf .next

# 一般
rm -rf node_modules/.cache
```

## 出力フォーマット

```json
{
  "lint": {
    "status": "pass|fail",
    "command": "{実行コマンド}",
    "errors": [
      {
        "file": "{ファイルパス}",
        "line": 10,
        "column": 5,
        "rule": "{ルール名}",
        "message": "{エラーメッセージ}",
        "severity": "error|warning",
        "fixable": true
      }
    ],
    "summary": {
      "errorCount": 5,
      "warningCount": 10,
      "fixableCount": 12
    }
  },
  "test": {
    "status": "pass|fail",
    "command": "{実行コマンド}",
    "summary": {
      "total": 50,
      "passed": 48,
      "failed": 2,
      "skipped": 0,
      "duration": "15.3s"
    },
    "failures": [
      {
        "file": "{テストファイル}",
        "test": "{テスト名}",
        "message": "{エラーメッセージ}",
        "expected": "{期待値}",
        "actual": "{実際の値}"
      }
    ]
  },
  "build": {
    "status": "pass|fail|skipped",
    "command": "{実行コマンド}",
    "errors": [
      {
        "file": "{ファイルパス}",
        "line": 15,
        "message": "{エラーメッセージ}",
        "type": "typescript|webpack|rollup"
      }
    ],
    "duration": "45.2s"
  },
  "recommendations": [
    {
      "type": "auto_fix|manual_fix|investigate",
      "description": "{推奨アクション}",
      "command": "{実行コマンド（あれば）}"
    }
  ]
}
```

## CI環境での実行

```bash
# CI環境フラグ
CI=true npm test
CI=true npm run build

# 並列実行の制限（メモリ節約）
npm test -- --maxWorkers=2

# カバレッジ出力
npm test -- --coverage
```

## 注意事項

- **watchモードを無効化**: CI環境やスクリプト実行時は必須
- **タイムアウト設定**: 長時間テストには適切なタイムアウトを設定
- **並列実行の調整**: メモリ不足時はワーカー数を減らす
- **エラー出力の保存**: 後で分析できるようにログを保存
