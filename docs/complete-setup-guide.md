# Next.js プロジェクト完全セットアップガイド

## 概要

このドキュメントでは、Next.js 15.3.5を使用したプロジェクトのゼロからの完全セットアップ手順を記載しています。ESLint、Prettier、GitHub Actions、Vercelデプロイまでの包括的な開発環境構築を網羅しています。

## 前提条件

- Node.js 20.x以上
- npm 10.x以上
- Gitアカウント
- GitHubアカウント
- Vercelアカウント

## プロジェクト初期化

### Next.jsプロジェクト作成

```bash
npx create-next-app@latest project-name --typescript --tailwind --eslint --app --src-dir --import-alias "@/*"
cd project-name
```

### 初期ファイル構成確認

```text
├── .next/
├── node_modules/
├── public/
├── src/
│   └── app/
├── .gitignore
├── eslint.config.mjs
├── next.config.ts
├── package.json
├── postcss.config.mjs
├── README.md
├── tailwind.config.ts
└── tsconfig.json
```

## コード品質ツールの設定

### Prettierの導入

#### パッケージインストール

```bash
npm install --save-dev prettier eslint-config-prettier eslint-plugin-prettier
```

#### Prettier設定ファイル作成

`.prettierrc.json`を作成

```json
{
  "semi": true,
  "singleQuote": true,
  "tabWidth": 2,
  "useTabs": false,
  "trailingComma": "es5",
  "printWidth": 80,
  "bracketSpacing": true,
  "bracketSameLine": false,
  "arrowParens": "avoid",
  "endOfLine": "lf",
  "quoteProps": "as-needed",
  "embeddedLanguageFormatting": "auto",
  "proseWrap": "preserve",
  "htmlWhitespaceSensitivity": "css",
  "vueIndentScriptAndStyle": false,
  "insertPragma": false,
  "requirePragma": false
}
```

#### Prettierignoreファイル作成

`.prettierignore`を作成

```text
build/
coverage/
dist/
node_modules/
.next/
.vercel/
*.html
*.log
*.lock
package-lock.json
yarn.lock
pnpm-lock.yaml
.env*
.gitignore
.gitattributes
node_modules/**/*.md
```

### ESLintとPrettierの統合

#### ESLint設定更新

`eslint.config.mjs`を以下のように更新

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

#### package.jsonにスクリプト追加

```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "lint:fix": "next lint --fix",
    "format": "prettier --write .",
    "format:check": "prettier --check ."
  }
}
```

## 開発環境統一設定

### EditorConfig設定

`.editorconfig`を作成

```ini
# EditorConfig is awesome: https://EditorConfig.org

# top-most EditorConfig file
root = true

# Unix-style newlines with a newline ending every file
[*]
charset = utf-8
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true
indent_style = space
indent_size = 2

# TypeScript/JavaScript files
[*.{js,jsx,ts,tsx}]
indent_size = 2

# JSON files
[*.json]
indent_size = 2

# YAML files
[*.{yml,yaml}]
indent_size = 2

# Markdown files
[*.md]
trim_trailing_whitespace = false

# Package.json
[package.json]
indent_size = 2
```

### Git属性設定

`.gitattributes`を作成

```text
# Set default behavior to automatically normalize line endings.
* text=auto

# Force batch scripts to always use CRLF line endings so that if a repo is accessed
# in Windows via a file share from Linux, the scripts will work.
*.{cmd,[Cc][Mm][Dd]} text eol=crlf
*.{bat,[Bb][Aa][Tt]} text eol=crlf

# Force bash scripts to always use LF line endings so that if a repo is accessed
# in Windows, the scripts will work.
*.sh text eol=lf

# Explicitly declare text files you want to always be normalized and converted
# to native line endings on checkout.
*.{js,jsx,ts,tsx} text
*.{json,md,yml,yaml} text
*.{css,scss,sass} text
*.{html,htm} text
*.{svg,xml} text

# Denote all files that are truly binary and should not be modified.
*.{png,jpg,jpeg,gif,ico,webp} binary
*.{woff,woff2,eot,ttf,otf} binary
*.{pdf,zip,tar,gz,bz2,7z} binary
*.{mp3,mp4,avi,mov,wmv,flv} binary
*.{exe,dll,so,dylib} binary

# Language specific
*.{lock,lockb} binary
```

## CI/CD環境構築

### GitHub Actions設定

`.github/workflows/ci.yml`を作成

```yaml
name: CI

env:
  VERCEL_ORG_ID: ${{ secrets.VERCEL_ORG_ID }}
  VERCEL_PROJECT_ID: ${{ secrets.VERCEL_PROJECT_ID }}

on:
  push:
    branches:
      - main # 本番環境
      - development # 開発環境
      - 'feature-**' # 機能ブランチ
      - 'bug-**' # バグ修正ブランチ
      - 'fix-**' # 修正ブランチ
      - 'improve-**' # 改善ブランチ
      - 'enhancement-**' # 機能拡張ブランチ
  pull_request:
    branches:
      - main # 本番環境へのPR
      - development # 開発環境へのPR

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          check-latest: true
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Run linting
        run: npm run lint

      - name: Check formatting
        run: npm run format:check

      - name: Build project
        run: npm run build

  # 本番環境デプロイ (mainブランチのみ)
  production:
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/main' }}
    needs: test
    runs-on: ubuntu-latest
    environment:
      name: production
    steps:
      - uses: actions/checkout@v4

      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          check-latest: true
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=production --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --prod --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy Project Artifacts to Vercel
        run: vercel deploy --prod --prebuilt --token=${{ secrets.VERCEL_TOKEN }}

  # 開発環境デプロイ (developmentブランチのみ)
  development:
    if: ${{ github.event_name == 'push' && github.ref == 'refs/heads/development' }}
    needs: test
    runs-on: ubuntu-latest
    environment:
      name: development
    steps:
      - uses: actions/checkout@v4

      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          check-latest: true
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=preview --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy Project Artifacts to Vercel
        run: vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }}

  # プレビュー環境デプロイ (feature-/bug-/fix-/improve-/enhancement-ブランチ、または PR)
  preview:
    if: ${{ github.event_name == 'pull_request' || (github.event_name == 'push' && github.ref != 'refs/heads/main' && github.ref != 'refs/heads/development') }}
    needs: test
    runs-on: ubuntu-latest
    environment:
      name: preview
      url: ${{ steps.deploy.outputs.url }}
    steps:
      - uses: actions/checkout@v4

      - name: Setup node
        uses: actions/setup-node@v4
        with:
          node-version: 20
          check-latest: true
          cache: 'npm'

      - name: Install dependencies
        run: npm ci

      - name: Install Vercel CLI
        run: npm install --global vercel@latest

      - name: Pull Vercel Environment Information
        run: vercel pull --yes --environment=preview --token=${{ secrets.VERCEL_TOKEN }}

      - name: Build Project Artifacts
        run: vercel build --token=${{ secrets.VERCEL_TOKEN }}

      - name: Deploy Project Artifacts to Vercel
        id: deploy
        run: echo "url=$(vercel deploy --prebuilt --token=${{ secrets.VERCEL_TOKEN }})" >> $GITHUB_OUTPUT
```

### Vercel設定

`vercel.json`を作成

```json
{
  "version": 2,
  "builds": [
    {
      "src": "package.json",
      "use": "@vercel/next"
    }
  ]
}
```

## GitHub設定

### リポジトリ作成とリモート追加

```bash
# GitHubでリポジトリを作成後
git remote add origin https://github.com/username/repository-name.git
git branch -M main
git push -u origin main
```

### GitHub Secrets設定

リポジトリのSettings > Secrets and variablesで以下を設定

- `VERCEL_TOKEN` - Vercelアクセストークン
- `VERCEL_PROJECT_ID` - VercelプロジェクトID
- `VERCEL_ORG_ID` - Vercel組織ID

#### Vercel情報の取得方法

```bash
# Vercel CLIでログイン
npx vercel login

# プロジェクト情報を取得
npx vercel link

# 環境情報を確認
cat .vercel/project.json
```

## 開発運用ルール

### ブランチ戦略

#### メインブランチ

- `main` - 本番環境
- `development` - 開発環境

#### 作業ブランチ命名規則

- `feature-[Issue番号]-[内容]` - 新機能開発
- `bug-[Issue番号]-[内容]` - バグ修正
- `fix-[Issue番号]-[内容]` - 軽微な修正
- `improve-[Issue番号]-[内容]` - 改善
- `enhancement-[Issue番号]-[内容]` - 機能拡張

#### ブランチ例

```text
feature-10-user-authentication
bug-15-login-error
fix-20-typo-in-header
improve-25-performance
enhancement-30-search-filter
```

### コミットルール

#### コミットメッセージ形式

```text
#Issue番号 種別: 変更内容の要約

詳細な説明（必要に応じて）
```

#### 種別一覧

- `feat` - 新機能追加
- `fix` - バグ修正
- `docs` - ドキュメント変更
- `style` - コードフォーマット
- `refactor` - リファクタリング
- `test` - テスト追加・修正
- `chore` - その他の変更

#### コミット例

```bash
git commit -m "#10 feat: ユーザー認証機能を追加"
git commit -m "#15 fix: ログイン時のバリデーションエラーを修正"
git commit -m "#20 docs: README.mdにセットアップ手順を追加"
```

### プルリクエストルール

#### PRタイトル形式

```text
#Issue番号 種別: 変更内容の要約
```

#### PR説明内容

- 変更内容の概要
- 動作確認の方法
- 関連するIssue番号
- スクリーンショット（UI変更の場合）

## 開発フロー

### 日常の開発作業

1. developmentブランチから作業ブランチを作成
2. Issue番号を含むブランチ名で作成
3. 作業ブランチで開発・コミット
4. developmentブランチにPRを作成
5. コードレビュー後にマージ
6. 開発環境に自動デプロイ

### 本番リリース

1. developmentブランチからmainブランチにPRを作成
2. 最終確認後にマージ
3. 本番環境に自動デプロイ

### ホットフィックス

1. mainブランチから直接作業ブランチを作成
2. 修正後にmainブランチにPRを作成
3. マージ後に本番環境に自動デプロイ
4. developmentブランチにもマージして同期

## 環境構成

### デプロイ環境

- **本番環境** - mainブランチから自動デプロイ
- **開発環境** - developmentブランチから自動デプロイ
- **プレビュー環境** - PRまたは機能ブランチから自動生成

### CI/CDパイプライン

#### 実行タイミング

- pushイベント（指定ブランチ）
- pull_requestイベント（main、developmentへのPR）

#### 実行ジョブ

1. **test** - 全ブランチで実行
   - 依存関係インストール
   - ESLintチェック
   - Prettierフォーマットチェック
   - ビルドテスト

2. **production** - mainブランチのみ
   - testジョブ成功後に実行
   - Vercel本番環境デプロイ

3. **development** - developmentブランチのみ
   - testジョブ成功後に実行
   - Vercel開発環境デプロイ

4. **preview** - その他ブランチまたはPR
   - testジョブ成功後に実行
   - Vercelプレビュー環境デプロイ

## トラブルシューティング

### よくある問題と解決法

#### Prettierフォーマットエラー

```bash
# フォーマット修正
npm run format

# フォーマットチェック
npm run format:check
```

#### ESLintエラー

```bash
# 自動修正可能なエラーを修正
npm run lint:fix

# エラー確認
npm run lint
```

#### Vercelデプロイエラー

- GitHub Secretsの設定確認
- Vercel CLIでローカル確認
- 環境変数の設定確認

#### ビルドエラー

```bash
# ローカルでビルド確認
npm run build

# 依存関係の再インストール
rm -rf node_modules package-lock.json
npm install
```

## まとめ

このセットアップにより以下が実現されます

### 開発効率の向上

- 自動フォーマット・リンティング
- 統一されたコードスタイル
- エディタ設定の共有

### 品質保証

- PR作成時の自動テスト
- コードレビュープロセス
- 段階的デプロイメント

### 運用の自動化

- ブランチ戦略に基づく自動デプロイ
- 環境別のデプロイメント
- Issue連携による追跡可能性

### チーム開発の円滑化

- 明確な開発フロー
- 統一されたコミット規則
- ドキュメント化された運用ルール

このセットアップを基盤として、チーム開発における効率性と品質を両立した開発環境を構築できます。
