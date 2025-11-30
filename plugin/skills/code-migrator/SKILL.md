---
name: code-migrator
description: 破壊的変更に対応したコード修正を行う。import文の変更、API呼び出しの修正、設定ファイルの更新を実行
allowed-tools: Read, Edit, Grep, Glob
---

# コード移行スキル

**役割**: 破壊的変更に基づくコード修正に特化。

**入力**: 破壊的変更リスト、修正パターン、対象ディレクトリ（呼び出し元から渡される）

## 修正パターン

### 1. import文の変更

#### パッケージ名の変更

```typescript
// Before
import { something } from 'old-package';

// After
import { something } from 'new-package';
```

検索パターン:
```regex
import\s+.*\s+from\s+['"]old-package['"]
```

#### エクスポート名の変更

```typescript
// Before
import { oldName } from 'package';

// After
import { newName } from 'package';
```

#### デフォルトエクスポートへの変更

```typescript
// Before
import { Component } from 'package';

// After
import Component from 'package';
```

### 2. API呼び出しの変更

#### 関数名の変更

```typescript
// Before
oldFunction(arg1, arg2);

// After
newFunction(arg1, arg2);
```

#### 引数の変更

```typescript
// Before
func(arg1, arg2, arg3);

// After
func({ param1: arg1, param2: arg2, param3: arg3 });
```

#### メソッドチェーンの変更

```typescript
// Before
obj.oldMethod().chain();

// After
obj.newMethod().chain();
```

### 3. React固有の変更

#### Hooksの変更

```typescript
// Before (React 18)
import { useEffect } from 'react';

// After (React 19 - use() API)
import { use } from 'react';
```

#### コンポーネントPropsの変更

```typescript
// Before
<Component oldProp={value} />

// After
<Component newProp={value} />
```

#### forwardRefの廃止（React 19）

```typescript
// Before
const Component = forwardRef((props, ref) => {
  return <div ref={ref} />;
});

// After
const Component = ({ ref, ...props }) => {
  return <div ref={ref} />;
};
```

### 4. Next.js固有の変更

#### Pages Router → App Router

```typescript
// Before (pages/)
export async function getServerSideProps() {}

// After (app/)
// Server Componentとして実装
async function Page() {
  const data = await fetchData();
  return <div>{data}</div>;
}
```

#### next/router → next/navigation

```typescript
// Before
import { useRouter } from 'next/router';
const router = useRouter();
router.push('/path');

// After
import { useRouter } from 'next/navigation';
const router = useRouter();
router.push('/path');
```

#### next/image の変更

```typescript
// Before (Next.js 12)
import Image from 'next/image';
<Image layout="fill" objectFit="cover" />

// After (Next.js 13+)
import Image from 'next/image';
<Image fill style={{ objectFit: 'cover' }} />
```

### 5. 設定ファイルの変更

#### next.config.js

```javascript
// Before (Next.js 14)
module.exports = {
  experimental: {
    appDir: true,
  },
};

// After (Next.js 15)
module.exports = {
  // appDirはデフォルトで有効
};
```

#### tsconfig.json

```json
// Before (TypeScript 4.x)
{
  "compilerOptions": {
    "target": "ES2020"
  }
}

// After (TypeScript 5.x)
{
  "compilerOptions": {
    "target": "ES2022"
  }
}
```

#### ESLint設定

```javascript
// Before (ESLint 8)
module.exports = {
  extends: ['next/core-web-vitals'],
};

// After (ESLint 9 - Flat Config)
import nextConfig from 'eslint-config-next';
export default [...nextConfig];
```

## 検索・置換戦略

### 安全な置換（自動実行可能）

- 単純な文字列置換
- import文のパッケージ名変更
- 明確なAPI名の変更

### 確認が必要な置換

- 複数の解釈が可能な変更
- コンテキストに依存する変更
- 削除を伴う変更

### 手動対応が必要

- ロジックの変更を伴う修正
- 複雑な型の変更
- アーキテクチャの変更

## 出力フォーマット

```json
{
  "migrations": [
    {
      "pattern": "{変更パターン}",
      "description": "{変更内容}",
      "files": [
        {
          "path": "{ファイルパス}",
          "changes": [
            {
              "line": 15,
              "before": "{変更前コード}",
              "after": "{変更後コード}",
              "status": "applied|skipped|manual_required"
            }
          ]
        }
      ],
      "status": "success|partial|failed"
    }
  ],
  "summary": {
    "totalFiles": 10,
    "totalChanges": 25,
    "applied": 20,
    "skipped": 3,
    "manualRequired": 2
  },
  "manualTasks": [
    {
      "file": "{ファイルパス}",
      "line": 42,
      "description": "{手動で行う必要がある作業}",
      "reason": "{自動化できない理由}"
    }
  ]
}
```

## 検索コマンド

```bash
# 特定のimportを検索
grep -rn "from 'old-package'" --include="*.ts" --include="*.tsx" src/

# 特定の関数呼び出しを検索
grep -rn "oldFunction\s*(" --include="*.ts" --include="*.tsx" src/

# 特定のJSX属性を検索
grep -rn "oldProp=" --include="*.tsx" src/
```

## 注意事項

- **バックアップを取ってから実行**: git stash や git commit で保存
- **一度に大量の変更をしない**: ファイル単位で確認しながら進める
- **テストが通ることを確認**: 変更後に必ずテスト実行
- **コメントやドキュメントも更新**: 古いAPI名が残らないように
