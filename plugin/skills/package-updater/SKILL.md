---
name: package-updater
description: npm/yarn/pnpm/bunに対応したパッケージ更新を実行する。Node.jsバージョン管理ツール（mise/nvm/asdf）にも対応
allowed-tools: Bash, Read, Edit
---

# パッケージ更新スキル

**役割**: パッケージマネージャーとバージョン管理ツールを使用した更新作業に特化。

**入力**: パッケージ名、バージョン、パッケージマネージャー種別（呼び出し元から渡される）

## パッケージマネージャー別コマンド

### npm

```bash
# 単一パッケージ
npm install {package}@{version}

# 複数パッケージ
npm install {pkg1}@{v1} {pkg2}@{v2}

# devDependencies
npm install -D {package}@{version}

# peer依存関係の競合を無視（非推奨だが必要な場合）
npm install {package}@{version} --legacy-peer-deps
```

### yarn (v1)

```bash
# 単一パッケージ
yarn add {package}@{version}

# 複数パッケージ
yarn add {pkg1}@{v1} {pkg2}@{v2}

# devDependencies
yarn add -D {package}@{version}
```

### yarn (berry/v2+)

```bash
# 単一パッケージ
yarn add {package}@{version}

# インタラクティブアップグレード（使用しない）
# yarn upgrade-interactive
```

### pnpm

```bash
# 単一パッケージ
pnpm add {package}@{version}

# 複数パッケージ
pnpm add {pkg1}@{v1} {pkg2}@{v2}

# devDependencies
pnpm add -D {package}@{version}

# ワークスペース全体
pnpm add {package}@{version} -w
```

### bun

```bash
# 単一パッケージ
bun add {package}@{version}

# 複数パッケージ
bun add {pkg1}@{v1} {pkg2}@{v2}

# devDependencies
bun add -d {package}@{version}
```

## Node.jsバージョン管理

### mise

```bash
# バージョンインストール
mise install node@{version}

# プロジェクトで使用
mise use node@{version}

# グローバル設定
mise use -g node@{version}

# .tool-versions更新（自動）
# mise use で自動更新される
```

### nvm

```bash
# バージョンインストール
nvm install {version}

# 使用
nvm use {version}

# .nvmrc更新
echo "{version}" > .nvmrc

# デフォルト設定
nvm alias default {version}
```

### asdf

```bash
# プラグイン確認
asdf plugin list | grep nodejs

# バージョンインストール
asdf install nodejs {version}

# ローカル設定
asdf local nodejs {version}

# .tool-versions更新（自動）
```

### nodenv

```bash
# バージョンインストール
nodenv install {version}

# ローカル設定
nodenv local {version}

# .node-version更新（自動）
```

### fnm

```bash
# バージョンインストール
fnm install {version}

# 使用
fnm use {version}

# .node-version更新
echo "{version}" > .node-version
```

## パッケージ削除

```bash
# npm
npm uninstall {package}

# yarn
yarn remove {package}

# pnpm
pnpm remove {package}

# bun
bun remove {package}
```

## 依存関係の再インストール

```bash
# npm
rm -rf node_modules package-lock.json && npm install

# yarn
rm -rf node_modules yarn.lock && yarn

# pnpm
rm -rf node_modules pnpm-lock.yaml && pnpm install

# bun
rm -rf node_modules bun.lockb && bun install
```

## 出力フォーマット

```json
{
  "action": "update|install|remove",
  "packages": [
    {
      "name": "{パッケージ名}",
      "from": "{旧バージョン}",
      "to": "{新バージョン}",
      "status": "success|failed",
      "command": "{実行コマンド}",
      "output": "{コマンド出力}",
      "error": "{エラー（失敗時）}"
    }
  ],
  "nodeVersion": {
    "from": "{旧バージョン}",
    "to": "{新バージョン}",
    "tool": "mise|nvm|asdf|nodenv|fnm",
    "status": "success|failed|skipped"
  }
}
```

## エラーハンドリング

### ERESOLVE（依存関係の競合）

```bash
# エラー例
npm ERR! ERESOLVE unable to resolve dependency tree

# 対処1: --legacy-peer-deps
npm install {package}@{version} --legacy-peer-deps

# 対処2: --force（非推奨）
npm install {package}@{version} --force
```

### バージョンが見つからない

```bash
# 利用可能なバージョン確認
npm view {package} versions --json

# 最新バージョン確認
npm view {package} version
```

### 権限エラー

```bash
# グローバルインストールの場合
# sudoは使用しない、パーミッション修正を案内
```

## 確認コマンド

```bash
# インストール済みバージョン確認
npm list {package}
yarn list {package}
pnpm list {package}
bun pm ls {package}

# 最新バージョン確認
npm view {package} version

# アウトデートパッケージ確認
npm outdated
yarn outdated
pnpm outdated
```
