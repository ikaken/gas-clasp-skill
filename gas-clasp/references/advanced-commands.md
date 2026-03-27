# 高度な clasp コマンド

このドキュメントでは、clasp の高度な使い方や特殊なユースケースについて説明します。

## 複数ユーザー・複数プロジェクトの管理

clasp では複数の Google アカウントや複数のプロジェクトを切り替えて使用できます。

### 名前付き認証情報の使用

`--user` オプションで認証情報に名前を付けて管理できます：

```bash
# デフォルト認証情報でログイン
clasp login

# 名前付き認証情報でログイン
clasp login --user work-account
clasp login --user personal-account

# 特定のユーザーでコマンド実行
clasp push --user work-account
clasp run-function myFunction --user personal-account
```

**使用例**:
- 個人アカウントと仕事用アカウントを使い分ける
- 複数のクライアントプロジェクトを管理する
- テスト用アカウントと本番用アカウントを分ける

### カスタム認証情報でのログイン

```bash
clasp login --user myproject --creds client_secret.json
```

## `clasp login` の高度なオプション

### プロジェクトスコープの使用

`appsscript.json` で定義されたスコープを使用してログインする場合：

```bash
# プロジェクトスコープのみを使用
clasp login --use-project-scopes --creds client_secret.json

# プロジェクトスコープ + clasp のデフォルトスコープ
clasp login --use-project-scopes --include-clasp-scopes --creds client_secret.json

# 追加のスコープを指定
clasp login --extra-scopes https://www.googleapis.com/auth/spreadsheets.readonly,https://www.googleapis.com/auth/drive.readonly --creds client_secret.json
```

**使用例**:
- `clasp run-function` でスクリプトを実行する際、スクリプトが必要とするスコープを事前に認証
- カスタム OAuth クライアントで特定のスコープのみを許可

### ローカルサーバーを使わないログイン

ファイアウォールやポート制限がある環境では、`--no-localhost` オプションを使用：

```bash
clasp login --no-localhost
```

認証コードを手動でコピー&ペーストする方式になります。

### カスタムリダイレクトポート

特定のポートを使用する必要がある場合：

```bash
clasp login --redirect-port 37473
```

## `clasp run-function` - リモート関数実行

GAS の関数をコマンドラインから実行できます。

### 前提条件

1. **API Executable としてデプロイ**
   - GAS エディタで「デプロイ」→「新しいデプロイ」
   - 種類: **API 実行可能ファイル**
   - アクセスできるユーザー: 自分のみ / 全員

2. **`appsscript.json` に設定を追加**

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "executionApi": {
    "access": "ANYONE"
  }
}
```

3. **カスタム認証情報でログイン**

```bash
clasp login --user myproject --creds client_secret.json
```

4. **`.clasp.json` に `projectId` を追加**

```json
{
  "scriptId": "your-script-id",
  "rootDir": "src",
  "projectId": "your-gcp-project-id"
}
```

### 使用方法

```bash
# 引数なしの関数を実行
clasp run-function myFunction

# 引数付きの関数を実行（JSON 配列形式）
clasp run-function addNumbers -p '[10, 20]'
clasp run-function sendEmail -p '["test@example.com", "Hello", "Message body"]'

# 複雑なオブジェクトを渡す
clasp run-function processData -p '[{"name": "John", "age": 30}, true]'

# 本番モードで実行（devMode を無効化）
clasp run-function myFunction --nondev
```

### 注意事項

- 実行する関数は GAS プロジェクト内に存在する必要があります
- 関数の戻り値は JSON 形式で返されます
- 実行時間は GAS の制限（6分）に従います

## ログの表示

### 前提条件

`.clasp.json` に `projectId` が設定されている必要があります。

### 基本的な使い方

```bash
# 最新のログを表示
clasp tail-logs

# JSON 形式で表示
clasp tail-logs --json

# リアルタイムでログを監視（5秒ごとに更新）
clasp tail-logs --watch

# タイムスタンプを省略して表示
clasp tail-logs --simplified
```

### ログの種類

- `console.log()` の出力が表示されます
- `Logger.log()` は表示されません（GAS エディタの「実行ログ」で確認）

### ログのセットアップ

```bash
# GCP プロジェクト ID が未設定の場合、セットアップを実行
clasp setup-logs
```

## API の管理

### API の一覧表示

```bash
# 有効化可能な API の一覧
clasp list-apis
```

### API の有効化・無効化

```bash
# API を有効化
clasp enable-api drive
clasp enable-api calendar

# API を無効化
clasp disable-api drive
```

### API コンソールを開く

```bash
# Google Cloud Console の API 管理画面を開く
clasp open-api-console
```

## その他の便利なコマンド

### スクリプト一覧の表示

```bash
# 最近使用した GAS プロジェクトの一覧
clasp list-scripts
```

### 各種エディタを開く

```bash
# GAS エディタを開く
clasp open-script

# デプロイ済み Web アプリを開く
clasp open-web-app

# コンテナ（スプレッドシート等）を開く
clasp open-container

# OAuth 認証情報設定画面を開く
clasp open-credentials-setup

# ログ画面を開く
clasp open-logs
```

### 現在のログインユーザーを確認

```bash
# ログインユーザー情報を表示
clasp show-authorized-user

# JSON 形式で表示
clasp show-authorized-user --json
```

## トラブルシューティング

### Node.js バージョンエラー

```
Error: The library requires NodeJS version >= 22.0.0
```

**解決方法**: Node.js 22.0.0 以上にアップグレード

```bash
node --version
# 22.0.0 未満の場合はアップグレード
```

### デバッグモードの有効化

問題が発生した場合、デバッグログを有効化して詳細情報を確認：

```bash
# Windows (PowerShell)
$env:DEBUG="clasp:*"
clasp push

# macOS/Linux
DEBUG=clasp:* clasp push
```

### プロジェクト ID が見つからない

`clasp run-function` や `clasp tail-logs` で「Project ID が必要」というエラーが出る場合：

1. GAS エディタを開く: `clasp open-script`
2. 「プロジェクトの設定」→「Google Cloud Platform (GCP) プロジェクト」
3. プロジェクト番号をコピー
4. `.clasp.json` に追加:

```json
{
  "scriptId": "your-script-id",
  "rootDir": "src",
  "projectId": "project-id-xxxxxxxxxxxxxxxxxxx"
}
```
