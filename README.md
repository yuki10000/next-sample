# Next.js Sample Project

Next.jsを使用したサンプルプロジェクトです。

## 技術構成

- **フレームワーク** Next.js 15.3.5
- **言語** TypeScript
- **スタイリング** Tailwind CSS
- **リンター** ESLint + Prettier
- **デプロイ** Vercel
- **CI/CD** GitHub Actions

## 開発環境のセットアップ

```bash
# 依存関係のインストール
npm ci

# 開発サーバーの起動
npm run dev
```

開発サーバーが起動したら [http://localhost:3000](http://localhost:3000) でアプリケーションを確認できます。

## 利用可能なコマンド

```bash
# 開発サーバー起動
npm run dev

# プロダクションビルド
npm run build

# プロダクションサーバー起動
npm start

# コード品質チェック
npm run lint
npm run lint:fix

# コードフォーマット
npm run format
npm run format:check
```

## プロジェクト構成

```text
├── .github/workflows/    # GitHub Actions設定
├── docs/                # プロジェクトドキュメント
├── public/              # 静的ファイル
├── src/
│   └── app/            # Next.js App Router
├── .editorconfig       # エディタ設定
├── .gitattributes      # Git属性設定
├── .prettierrc.json    # Prettier設定
├── eslint.config.mjs   # ESLint設定
├── next.config.ts      # Next.js設定
├── tailwind.config.ts  # Tailwind CSS設定
└── vercel.json         # Vercel設定
```

## 開発フロー

詳細な開発・運用手順については以下のドキュメントを参照してください。

- [完全セットアップガイド](./docs/complete-setup-guide.md) - 全体的なセットアップ手順
- [Git運用ガイド](./docs/git-workflow.md) - ブランチ戦略とコミットルール
- [CI/CD運用ガイド](./docs/ci-cd-workflow.md) - 自動デプロイとパイプライン設定
- [Qiita記事版](./docs/qiita-article.md) - 要点をまとめた記事形式

## 環境

- **本番環境** mainブランチから自動デプロイ
- **開発環境** developmentブランチから自動デプロイ
- **プレビュー環境** PRまたは機能ブランチから自動生成

## ライセンス

MIT
