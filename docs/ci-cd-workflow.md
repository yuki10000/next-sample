# CI/CD運用ガイド

## 概要

GitHub ActionsとVercelを使用した自動ビルド・デプロイシステムです。

## 環境構成

### 本番環境 (Production)

- **ブランチ** main
- **トリガー** mainブランチへのpush
- **デプロイ先** Vercel Production環境
- **URL** 固定の本番URL

### 開発環境 (Development)

- **ブランチ** development  
- **トリガー** developmentブランチへのpush
- **デプロイ先** Vercel Preview環境
- **URL** 固定の開発URL

### プレビュー環境 (Preview)

- **ブランチ** 作業ブランチ（feature-, bug-, fix-, improve-, enhancement-）
- **トリガー** 作業ブランチへのpushまたはPR作成
- **デプロイ先** Vercel Preview環境
- **URL** ブランチごとの一意なURL

## CI/CDパイプライン

### 実行されるジョブ

全ての環境で以下のテストが実行される

1. **Lint check** ESLintによるコード品質チェック
2. **Format check** Prettierによるフォーマットチェック
3. **Build test** Next.jsのビルドテスト

### デプロイフロー

テストが成功した場合のみデプロイが実行される

1. **Test job** コード品質とビルドの確認
2. **Deploy job** 対象環境へのデプロイ

## 実行タイミング

### 自動実行されるケース

- mainブランチへのpush
- developmentブランチへのpush
- 作業ブランチへのpush
- mainまたはdevelopmentブランチへのPR作成・更新

### 実行されないケース

- 対象外のブランチへのpush
- mainとdevelopment以外のブランチへのPR

## 環境変数の設定

GitHub Secretsに以下の値を設定する必要がある

- **VERCEL_TOKEN** Vercelのアクセストークン
- **VERCEL_PROJECT_ID** VercelのプロジェクトID
- **VERCEL_ORG_ID** Vercelの組織ID

### 設定手順

1. GitHubリポジトリのSettings > Secrets and variablesに移動
2. 上記の環境変数を追加
3. Vercelダッシュボードから各値を取得して設定

## 開発時の注意事項

### コード品質の確保

pushする前に以下のコマンドを実行して問題がないことを確認する

```bash
# Lintチェック
npm run lint

# フォーマットチェック  
npm run format:check

# ビルドテスト
npm run build
```

### 自動修正

問題がある場合は以下のコマンドで自動修正する

```bash
# Lint自動修正
npm run lint:fix

# フォーマット自動修正
npm run format
```

## トラブルシューティング

### CIが失敗する場合

1. **Lint errors** コードの品質に問題がある
   - `npm run lint:fix`で自動修正
   - 手動で修正が必要な場合もある

2. **Format errors** コードフォーマットが統一されていない
   - `npm run format`で自動修正

3. **Build errors** ビルドに失敗している
   - TypeScriptの型エラーを確認
   - 依存関係の問題を確認

### デプロイが失敗する場合

1. **Environment variables** 環境変数が設定されていない
   - GitHub Secretsの設定を確認

2. **Vercel connection** Vercelとの接続に問題がある
   - Vercelアクセストークンの有効性を確認
   - プロジェクトIDと組織IDが正しいか確認

## モニタリング

### ビルド状況の確認

- GitHub ActionsのWorkflowsタブで実行状況を確認
- 失敗した場合はログを確認して原因を特定

### デプロイ状況の確認

- Vercelダッシュボードでデプロイ状況を確認
- プレビューURLで動作確認

## ベストプラクティス

### 効率的な開発

- 小さな単位でコミットしてCI実行時間を短縮
- PRを作成する前にローカルでテストを実行
- 並行作業時はブランチを適切に分ける

### 品質管理

- CI失敗時は速やかに修正
- コードレビュー時にプレビュー環境で動作確認
- 本番デプロイ前に開発環境で十分にテスト
