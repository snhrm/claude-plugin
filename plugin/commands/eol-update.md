---
description: EOL対応計画に基づいてライブラリのアップデート作業を実行する
argument-hint: [フェーズ指定] (省略可。phase0, phase1, all等を指定)
allowed-tools: Read, Write, Edit, Grep, Glob, Bash, Task, AskUserQuestion
---

# EOLアップデート実行コマンド

`/eol-plan` で作成した計画書に基づいて、実際のアップデート作業を実行します。

## 引数

- `$ARGUMENTS`: 実行するフェーズを指定
  - 省略時: 次に実行すべきフェーズを自動判定
  - `phase0`, `phase1`, ... `phase6`: 特定フェーズを実行
  - `all`: 全フェーズを順次実行（各フェーズ完了時に確認）

### 引数の指定パターン

```
/eol-update              # 次のフェーズを実行
/eol-update phase0       # Phase 0（事前準備）を実行
/eol-update phase3       # Phase 3（主要フレームワーク）を実行
/eol-update all          # 全フェーズを順次実行
```

---

## 実行フロー

```
[Step 1] 計画書の読み込み・状態確認
    ↓
[Step 2] 実行フェーズの決定
    ↓
[Step 3] フェーズ実行（サブエージェント呼び出し）
    ↓
[Step 4] lint/test実行・結果確認
    ↓
[Step 5] 計画書の進捗更新
    ↓
[Step 6] 次のアクション提案
```

---

## Step 1: 計画書の読み込み

### 1.1 計画書の特定

```bash
# CLAUDE.mdから出力先を確認
grep -i "EOL計画出力先" CLAUDE.md

# デフォルト: プロジェクトルートのeol-plan.md
cat eol-plan.md
```

計画書が存在しない場合:
→ ユーザーに `/eol-plan` の実行を促す

### 1.2 現在の進捗確認

計画書内のチェックボックス状態を確認:
- `- [x]`: 完了
- `- [ ]`: 未完了

### 1.3 プロジェクト情報の取得

```bash
# パッケージマネージャーの特定
ls bun.lockb bun.lock pnpm-lock.yaml yarn.lock package-lock.json 2>/dev/null | head -1

# Node.jsバージョン管理ツールの特定
ls .nvmrc .node-version .tool-versions mise.toml .mise.toml 2>/dev/null | head -1

# lint/testコマンドの特定
cat package.json | grep -A5 '"scripts"'
```

---

## Step 2: 実行フェーズの決定

### 引数がある場合
指定されたフェーズを実行

### 引数がない場合
計画書の進捗から次のフェーズを自動判定:

1. Phase 0が未完了 → Phase 0を実行
2. Phase 0完了、Phase 1未完了 → Phase 1を実行
3. ...以降同様

### 実行前確認

ユーザーに確認:
```
次のフェーズを実行します:

Phase {N}: {フェーズ名}
対象:
  - {ライブラリ1}: {現在} → {目標}
  - {ライブラリ2}: {現在} → {目標}

実行してよろしいですか？
```

---

## Step 3: フェーズ実行

### Phase 0: 事前準備

1. **不足テストの確認**
   - 計画書の「テスト追加が必要な箇所」を確認
   - テストファイルの作成をユーザーに提案

2. **ベースライン確立**
   ```bash
   # lint実行
   {lint コマンド}

   # test実行
   {test コマンド}
   ```

3. **結果確認**
   - すべてパスした場合 → Phase 0完了
   - 失敗がある場合 → 修正を提案

### Phase 1〜5: ライブラリアップデート

**phase-executor サブエージェントを呼び出し**

```
実行フェーズ: Phase {N}
フェーズ名: {フェーズ名}

対象ライブラリ:
  - {ライブラリ1}: {現在} → {目標}
  - {ライブラリ2}: {現在} → {目標}

破壊的変更への対応:
  - {変更1}: {対応方法}
  - {変更2}: {対応方法}

プロジェクト情報:
  パッケージマネージャー: {npm/yarn/pnpm/bun}
  Node.jsバージョン管理: {nvm/mise/asdf/なし}
```

### Phase 6: ライブラリ置換

**migration-executor サブエージェントを呼び出し**

```
置換対象:
  旧ライブラリ: {旧}
  新ライブラリ: {新}

API対応表:
  - {旧API} → {新API}

プロジェクト情報:
  ...
```

---

## Step 4: lint/test実行・結果確認

各フェーズ完了後に必ず実行:

```bash
# lint
{lint コマンド}

# test
{test コマンド}

# build（オプション）
{build コマンド}
```

### 結果の判定

| 結果 | アクション |
|------|----------|
| すべてパス | 次のステップへ |
| lint失敗 | 自動修正を試行（--fix）、手動修正を提案 |
| test失敗 | 失敗テストを分析、修正を提案 |
| build失敗 | エラーを分析、修正を提案 |

---

## Step 5: 計画書の進捗更新

フェーズ完了時に計画書を更新:

```markdown
### Phase {N}: {フェーズ名}

**確認項目**:
- [x] `npm run lint` 成功  ← 更新
- [x] `npm run test` 成功  ← 更新
- [x] ビルド成功           ← 更新
- [ ] 動作確認             ← ユーザーに確認を促す
```

---

## Step 6: 次のアクション提案

### フェーズ成功時

```
✅ Phase {N} が完了しました。

次のステップ:
1. [ ] 動作確認を行ってください
2. [ ] 問題なければ `/eol-update phase{N+1}` で次のフェーズへ

または `/eol-update all` で残りのフェーズを連続実行
```

### フェーズ失敗時

```
❌ Phase {N} でエラーが発生しました。

エラー内容:
{エラー詳細}

推奨アクション:
1. {修正提案1}
2. {修正提案2}

修正後、再度 `/eol-update phase{N}` を実行してください。
```

---

## 注意事項

- **各フェーズは独立して実行可能**: 途中からの再開が可能
- **lint/testは必ず実行**: スキップ不可
- **動作確認はユーザー責任**: 自動化できない部分
- **失敗時はロールバック可能**: git stashやgit checkout での復元を案内
- **Node.jsアップデート時はバージョン管理ツールを使用**: mise, nvm等

## ロールバック手順

問題発生時:

```bash
# 変更を一時退避
git stash

# または特定ファイルを復元
git checkout -- package.json package-lock.json

# 依存関係を再インストール
{install コマンド}
```
