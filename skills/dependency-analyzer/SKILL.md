---
name: dependency-analyzer
description: ライブラリのEOL状況と互換性要件を確認する。endoflife.dateやnpmからバージョン互換性情報を取得する
allowed-tools: WebFetch, WebSearch
---

# EOL・互換性確認スキル

**役割**: EOL情報と互換性要件の取得に特化。

**入力**: ライブラリ名、バージョン（呼び出し元から渡される）

## 調査対象

1. **EOL状況**: サポート終了日、セキュリティサポート状況
2. **互換性要件**: peerDependencies、engines要件
3. **LTSスケジュール**: Node.js等のLTSサイクル

## 情報ソース

### endoflife.date
```
https://endoflife.date/{library}
https://endoflife.date/api/{library}.json
```

### npm registry
```
https://registry.npmjs.org/{package}
https://registry.npmjs.org/{package}/{version}
```

## 主要ライブラリのEOL情報

### Node.js
| バージョン | ステータス | EOL |
|-----------|----------|-----|
| 22.x | Current | 2027-04 |
| 20.x | Active LTS | 2026-04 |
| 18.x | Maintenance | 2025-04 |

### React
- 最新2メジャーバージョンがサポート対象
- 古いバージョンはセキュリティ修正のみ

### Next.js
- 最新2メジャーバージョンがアクティブサポート

## 互換性マトリクス

```
Node.js ← React ← Next.js
          ↓
        TypeScript ← @types/*
```

### 確認ポイント
| 組み合わせ | 確認事項 |
|-----------|---------|
| Node.js + Next.js | engines要件 |
| React + Next.js | peerDependencies |
| TypeScript + React | @types/reactのバージョン |

## 出力フォーマット

```json
{
  "eolStatus": {
    "library": "nodejs",
    "version": "18.x",
    "status": "maintenance",
    "eolDate": "2025-04-30",
    "securitySupport": true
  },
  "compatibility": {
    "engines": {
      "node": ">=18.17.0"
    },
    "peerDependencies": {
      "react": "^18.2.0 || ^19.0.0"
    }
  },
  "recommendations": [
    {
      "type": "upgrade",
      "reason": "EOLが近い",
      "target": "20.x"
    }
  ]
}
```
