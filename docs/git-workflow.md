# Git運用ガイド

## ブランチ戦略

### メインブランチ

- **main** 本番環境と同期するブランチ
- **development** 開発環境と同期するブランチ

### 作業ブランチ

作業ブランチは以下の命名規則に従って作成する

- **feature-[Issue番号]-** 新機能開発
- **bug-[Issue番号]-** バグ修正
- **fix-[Issue番号]-** 軽微な修正
- **improve-[Issue番号]-** 既存機能の改善
- **enhancement-[Issue番号]-** 機能拡張

### ブランチ命名例

```text
feature-10-user-authentication
bug-15-login-error
fix-20-typo-in-header
improve-25-performance
enhancement-30-search-filter
```

### ブランチ命名の詳細ルール

- **形式** `種別-Issue番号-内容説明`
- **種別** feature, bug, fix, improve, enhancement
- **Issue番号** GitHubのIssue番号（#は含めない）
- **内容説明** ハイフン区切りで内容が分かる英語表記

### 具体的な命名例

```text
feature-10-user-authentication
bug-15-login-error
fix-20-typo-in-header
improve-25-performance
enhancement-30-search-filter
```

### ブランチ作成例

```bash
# 新機能開発の場合（Issue #10の場合）
git checkout development
git pull origin development
git checkout -b feature-10-user-authentication

# バグ修正の場合（Issue #15の場合）
git checkout development
git pull origin development
git checkout -b bug-15-login-error
```

### ブランチとIssueの関連付け

- 1つのブランチは1つのIssueに対応させる
- ブランチ名にIssue番号を含める
- ブランチ名の形式は「種別-Issue番号-内容」とする
- 内容部分はハイフン区切りで分かりやすい名前にする

## 開発フロー

### 通常の開発作業

1. developmentブランチから作業ブランチを作成
2. 作業ブランチで開発・コミット
3. developmentブランチにPRを作成
4. コードレビュー後にマージ

### 本番リリース

1. developmentブランチからmainブランチにPRを作成
2. 最終確認後にマージ
3. 本番環境に自動デプロイ

### ホットフィックス

1. mainブランチから直接作業ブランチを作成
2. 修正後にmainブランチにPRを作成
3. マージ後に本番環境に自動デプロイ
4. developmentブランチにもマージして同期

## コミットルール

### コミットメッセージの形式

```text
#Issue番号 種別: 変更内容の要約

詳細な説明（必要に応じて）
```

### Issue番号の表記

- 作業に関連するGitHub Issue番号を先頭に記載する
- 番号の前に「#」を付ける
- Issue番号の後に半角スペースを入れる

### 種別の分類

- **feat** 新機能追加
- **fix** バグ修正
- **docs** ドキュメント変更
- **style** コードフォーマット
- **refactor** リファクタリング
- **test** テスト追加・修正
- **chore** その他の変更

### コミットメッセージ例

```bash
git commit -m "#10 feat: ユーザー認証機能を追加"
git commit -m "#15 fix: ログイン時のバリデーションエラーを修正"
git commit -m "#20 docs: README.mdにセットアップ手順を追加"
git commit -m "#25 style: Prettierでコードフォーマットを適用"
```

## プルリクエスト

### PRタイトルの形式

コミットメッセージと同様の形式を使用する

```text
#Issue番号 種別: 変更内容の要約
```

### PRタイトル例

```text
#10 feat: ユーザー認証機能を追加
#15 fix: ログイン時のバリデーションエラーを修正
#20 docs: API仕様書を更新
```

### PRの説明に含める内容

- 変更内容の概要
- 動作確認の方法
- 関連するIssue番号（該当する場合）
- スクリーンショット（UI変更の場合）

### レビューのポイント

- コードの品質と可読性
- テストの実装状況
- セキュリティ上の問題がないか
- パフォーマンスへの影響

## Gitの基本操作

### 初期設定

```bash
git config --global user.name "Your Name"
git config --global user.email "your.email@example.com"
```

### 日常的な操作

```bash
# 最新の変更を取得
git pull origin development

# 変更をステージング
git add .

# コミット
git commit -m "コミットメッセージ"

# リモートにプッシュ
git push origin ブランチ名

# ブランチ切り替え
git checkout ブランチ名

# 新しいブランチ作成
git checkout -b 新しいブランチ名
```

### 注意事項

- mainブランチに直接プッシュしない
- 作業前に必ず最新の変更を取得する
- コミット前にlintとフォーマットチェックを実行する
- 意味のある単位でコミットを分ける
- PRは小さな単位で作成する
