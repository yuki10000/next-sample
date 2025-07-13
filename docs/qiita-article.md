# Next.js + TypeScript + Prettier + ESLint + GitHub Actions + Vercel の完全セットアップ 2025年版

## はじめに

Next.js 15.3.5を使用した最新のフロントエンド開発環境を、ゼロから構築する完全ガイドです。個人開発からチーム開発まで対応できる、実用的な設定を網羅しています。

## 技術スタック

- Next.js 15.3.5 (App Router)
- TypeScript
- Tailwind CSS
- ESLint + Prettier
- GitHub Actions
- Vercel

## セットアップの全体像

1. Next.jsプロジェクト作成
2. コード品質ツール設定
3. 開発環境統一設定
4. CI/CD環境構築
5. 運用ルール策定

## 1. プロジェクト初期化

```bash
npx create-next-app@latest project-name --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd project-name
```

## 2. Prettierの導入と設定

### パッケージインストール

```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

### 設定ファイル

`.prettierrc.json`

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "es5",
  "printWidth": 80,
  "endOfLine": "lf"
}
```

### ESLint統合

`eslint.config.mjs`を更新

```javascript
import { dirname } from 'path';
import { fileURLToPath } from 'url';
import { FlatCompat } from '@eslint/eslintrc';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

const compat = new FlatCompat({
  baseDirectory: __dirname,
});

const eslintConfig = [
  ...compat.extends('next/core-web-vitals', 'next/typescript'),
  ...compat.extends('prettier'),
  {
    plugins: {
      prettier: (await import('eslint-plugin-prettier')).default,
    },
    rules: {
      'prettier/prettier': 'error',
    },
  },
];

export default eslintConfig;
```

### npmスクリプト追加

```json
{
  "scripts": {
    "lint:fix": "next lint --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

## 3. 開発環境統一設定

### EditorConfig

`.editorconfig`

```ini
root = true

[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

[*.md]
trim_trailing_whitespace = false
```

### Git属性設定

`.gitattributes`

```text
* text=auto
*.{js,jsx,ts,tsx} text
*.{json,md,yml,yaml} text
*.{png,jpg,jpeg,gif,ico,webp} binary
*.{lock,lockb} binary
```

## 4. GitHub Actions CI/CD

### 3段階デプロイメント設計

- **本番環境** - mainブランチ
- **開発環境** - developmentブランチ
- **プレビュー環境** - 機能ブランチ・PR

### ワークフロー設定

`.github/workflows/ci.yml`

```yaml
name: CI

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

on:
  push:
    branches:
      - main
      - development
      - 'feature-**'
      - 'bug-**'
      - 'fix-**'
  pull_request:
    branches:
      - main
      - development

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm run format:check
      - run: npm run build

  production:
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'
      - run: npm ci
      - run: npm install --global vercel@latest
      - run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}
      - run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}
      - run: vercel deploy --prod --prebuilt --token=${{ secrets.VERCEL_TOKEN }}
```

## 5. 運用ルールの策定

### ブランチ戦略

```text
main (本番) ← development (開発) ← feature-[Issue番号]-[内容]
```

### ブランチ命名規則

- `feature-10-user-auth` - 新機能
- `bug-15-login-error` - バグ修正
- `fix-20-typo` - 軽微な修正

### コミットメッセージ

```text
#Issue番号 種別: 変更内容

例:
#10 feat: ユーザー認証機能を追加
#15 fix: ログイン時のバリデーションエラーを修正
```

## 6. Vercel設定

### GitHub Secrets

- `VERCEL_TOKEN`
- `VERCEL_PROJECT_ID`
- `VERCEL_ORG_ID`

### 情報取得方法

```bash
npx vercel login
npx vercel link
cat .vercel/project.json
```

## 開発フロー

### 日常開発

1. developmentから機能ブランチ作成
2. 開発・コミット
3. developmentにPR作成
4. レビュー・マージ
5. 開発環境に自動デプロイ

### 本番リリース

1. development → mainにPR
2. 最終確認・マージ
3. 本番環境に自動デプロイ

## 実際の運用での効果

### 品質向上

- 自動フォーマット・リンティング
- PR前の自動テスト
- コードレビュープロセス

### 効率化

- 環境別自動デプロイ
- 統一されたコーディング規則
- Issue連携による追跡性

### チーム開発

- 明確な開発フロー
- 環境分離による安全性
- ドキュメント化された運用

## まとめ

このセットアップにより、個人開発からチーム開発まで対応できる、モダンで実用的な開発環境が構築できます。特に以下の点で効果を発揮します。

- コード品質の自動保証
- 段階的で安全なデプロイメント
- 効率的なチーム開発体制

Next.jsプロジェクトの本格運用を検討している方は、ぜひ参考にしてください。

## 参考リンク

- [Next.js公式ドキュメント](https://nextjs.org/docs)
- [Prettier設定ガイド](https://prettier.io/docs/en/configuration.html)
- [GitHub Actions公式ドキュメント](https://docs.github.com/en/actions)
- [Vercelデプロイガイド](https://vercel.com/docs)
