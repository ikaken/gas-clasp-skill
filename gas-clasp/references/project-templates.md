# プロジェクトテンプレート

## 必須ファイルの作成例

### `.clasp.json`

プロジェクトルートに配置する clasp の設定ファイル：

```json
{
  "scriptId": "your-script-id",
  "rootDir": "src",
  "projectId": "your-gcp-project-id",
  "filePushOrder": ["config.js", "utils.js", "main.js"]
}
```

#### 設定項目

- **`scriptId`** (必須): GAS プロジェクトの ID
- **`rootDir`** (オプション): ソースコードのディレクトリ（デフォルト: カレントディレクトリ）
- **`projectId`** (オプション): Google Cloud Platform プロジェクト ID（`clasp run-function` や `clasp tail-logs` で必要）
- **`filePushOrder`** (オプション): ファイルのプッシュ順序を指定

#### `filePushOrder` の使い方

GAS では**ファイルの読み込み順序**が実行に影響する場合があります。上記テンプレートのように `filePushOrder` でプッシュ順序を制御できます。

**使用例**:
- `config.js` でグローバル変数を定義
- `utils.js` でヘルパー関数を定義
- `main.js` でメイン処理を実行

**注意**:
- `filePushOrder` に指定されていないファイルは、その後にアルファベット順でプッシュされます
- ファイルパスは `rootDir` からの相対パスで指定します

### `src/appsscript.json`

Apps Script プロジェクトのマニフェストファイル。基本設定と Web アプリ設定を含みます。

#### 基本的な構成

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": []
}
```

#### Web アプリ用の設定（`webapp` エントリ）

Web アプリとして公開する場合、`webapp` エントリを追加します：

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "ANYONE_ANONYMOUS"
  },
  "oauthScopes": []
}
```

**重要**: `appsscript.json` の `webapp` 設定は**デフォルト値**です。実際のデプロイ時には GAS エディタの UI で設定を上書きできます（UI の設定が優先されます）。

##### `executeAs` パラメータ（実行ユーザー）

| 値 | 説明 | 推奨 |
|---|---|---|
| `USER_DEPLOYING` | デプロイしたユーザーとして実行 | ✅ 通常はこれ |
| `USER_ACCESSING` | アクセスしているユーザーとして実行 | - |
| `SERVICE_ACCOUNT` | サービスアカウントとして実行 | - |
| `UNKNOWN_EXECUTE_AS` | 未定義 | - |

**推奨**: `USER_DEPLOYING` を使用することで、Web アプリは常にデプロイしたユーザーの権限で実行されます。

##### `access` パラメータ（アクセス権限）

| 値 | 説明 | 推奨 |
|---|---|---|
| `MYSELF` | 自分のみアクセス可能 | - |
| `DOMAIN` | 同一 Google Workspace ドメイン内のユーザー | - |
| `ANYONE` | Google ログインが必要な全員 | - |
| `ANYONE_ANONYMOUS` | ログイン不要で誰でもアクセス可能 | ✅ 外部公開用 |
| `UNKNOWN_ACCESS` | 未定義 | - |

**推奨**: 外部に公開する Web アプリの場合は `ANYONE_ANONYMOUS` を使用します。

#### OAuth スコープの設定

`oauthScopes` はプロジェクトに必要なスコープのみ追加する：

| 用途 | スコープ |
|---|---|
| スプレッドシート（バインド） | `https://www.googleapis.com/auth/spreadsheets.currentonly` |
| スプレッドシート（任意） | `https://www.googleapis.com/auth/spreadsheets` |
| 外部API呼び出し | `https://www.googleapis.com/auth/script.external_request` |
| トリガー管理 | `https://www.googleapis.com/auth/script.scriptapp` |
| Drive | `https://www.googleapis.com/auth/drive` |
| Gmail送信 | `https://www.googleapis.com/auth/gmail.send` |
| Calendar | `https://www.googleapis.com/auth/calendar` |

### `.claspignore`

`.claspignore` は `clasp push` 時にアップロードから除外するファイルを指定します。

#### 基本的な `.claspignore` の例

```
node_modules/**
.git/**
.windsurf/**
.gitignore
package.json
package-lock.json
tsconfig.json
*.md
.env
.clasp.json
```

**重要**: `.clasp.json` は環境固有の設定（デプロイ先やスクリプトID等）を含むため、原則Git管理から除外する。

#### `.claspignore` が存在しない場合のデフォルト動作

`.claspignore` ファイルが存在しない場合、clasp は以下のデフォルトパターンを自動的に適用します：

```text
# すべてのファイルを除外…
**/**

# 以下の拡張子のみ許可…
!appsscript.json
!**/*.gs
!**/*.js
!**/*.ts
!**/*.html

# 以下のディレクトリは常に除外…
.git/**
node_modules/**
```

つまり、`.claspignore` を作成しない場合でも、`node_modules` や `.git` は自動的に除外されます。

#### `.gitignore` との違い

**重要**: `.claspignore` のパターンは [multimatch](https://github.com/sindresorhus/multimatch) で処理されるため、`.gitignore` とは異なります。

**ディレクトリを除外する場合の違い**:

```text
# .gitignore の場合
node_modules/

# .claspignore の場合（** が必要）
node_modules/**
**/node_modules/**
```

**パターンの適用基準**:
- `.claspignore` のパターンは `rootDir` からの相対パスで適用されます
- ディレクトリを除外する場合は `**/ディレクトリ名/**` の形式を推奨

### `.gitignore`

```
node_modules/
.clasp.json
*.log
.env
.windsurf/
```

### `package.json`

```json
{
  "name": "project-name",
  "version": "1.0.0",
  "description": "Google Apps Script project",
  "scripts": {
    "push": "clasp push",
    "pull": "clasp pull",
    "open": "clasp open-script",
    "open:web": "clasp open-web-app",
    "status": "clasp show-file-status",
    "version": "clasp create-version",
    "deploy": "clasp create-deployment",
    "deployments": "clasp list-deployments"
  },
  "devDependencies": {
    "@google/clasp": "^3.0.0",
    "@types/google-apps-script": "^1.0.83"
  }
}
```

## プロジェクトタイプ別の作成方法

clasp 3.x でプロジェクトを作成する際、`--type` オプションでプロジェクトタイプを指定できる：

```bash
# スタンドアロンスクリプト（デフォルト）
clasp create-script --type standalone --title "プロジェクト名" --rootDir src

# スプレッドシート連携
clasp create-script --type sheets --title "プロジェクト名" --rootDir src

# ドキュメント連携
clasp create-script --type docs --title "プロジェクト名" --rootDir src

# スライド連携
clasp create-script --type slides --title "プロジェクト名" --rootDir src

# フォーム連携
clasp create-script --type forms --title "プロジェクト名" --rootDir src

# Webアプリ
clasp create-script --type webapp --title "プロジェクト名" --rootDir src

# API実行可能スクリプト
clasp create-script --type api --title "プロジェクト名" --rootDir src

# 既存プロジェクトのクローン
clasp clone-script <スクリプトID> --rootDir src
```

### カスタム OAuth クライアントの設定

カスタム OAuth クライアントを使用する場合（セキュリティやコンプライアンス要件がある環境）、以下の手順で設定します：

1. [Google Cloud Console](https://console.cloud.google.com/) で新規プロジェクトを作成
2. OAuth クライアントを作成（クライアントタイプ: **Desktop Application**）
3. 必要な API を有効化：
   - Apps Script API (`script.googleapis.com`) - 必須
   - Service Usage API (`serviceusage.googleapis.com`) - API 管理に必要
   - Google Drive API (`drive.googleapis.com`) - スクリプト一覧、コンテナバインドスクリプト作成に必要
   - Cloud Logging API (`logging.googleapis.com`) - ログ表示に必要

#### 必要な OAuth スコープ（完全版）

外部ユーザーに公開する場合、以下のスコープを OAuth 同意画面に登録する必要があります：

```
https://www.googleapis.com/auth/script.deployments
https://www.googleapis.com/auth/script.projects
https://www.googleapis.com/auth/script.webapp.deploy
https://www.googleapis.com/auth/drive.metadata.readonly
https://www.googleapis.com/auth/drive.file
https://www.googleapis.com/auth/service.management
https://www.googleapis.com/auth/logging.read
https://www.googleapis.com/auth/userinfo.email
https://www.googleapis.com/auth/userinfo.profile
https://www.googleapis.com/auth/cloud-platform
```

**ログイン方法**:
```bash
clasp login --user myproject --creds client_secret.json
```

## プロジェクトタイプ別の推奨構成

### スプレッドシート連携

```javascript
// src/main.js
function main() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  // 処理
}

function onEdit(e) {
  // 編集時の処理
}
```

必要なスコープ: `https://www.googleapis.com/auth/spreadsheets`

### Web API（doGet / doPost）

GASをWeb APIとして公開する場合、`doGet(e)` と `doPost(e)` が HTTPリクエストのエントリーポイントになる。

#### doGet — GETリクエストの処理

```javascript
// src/main.js
function doGet(e) {
  // クエリパラメータの取得: ?action=list&limit=10
  const action = e.parameter.action;  // 'list'（単一値）
  const limit = e.parameter.limit;    // '10'（文字列として取得）

  // 同名パラメータが複数ある場合: ?tag=a&tag=b
  const tags = e.parameters.tag;      // ['a', 'b']（配列）

  const result = { status: 'ok', action: action };
  return ContentService.createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
```

#### doPost — POSTリクエストの処理

```javascript
// src/main.js
function doPost(e) {
  // リクエストボディの取得
  const body = e.postData.contents;       // 生のリクエストボディ（文字列）
  const type = e.postData.type;           // Content-Type（例: 'application/json'）

  // JSONの場合
  const data = JSON.parse(body);
  const name = data.name;

  // クエリパラメータも同時に取得可能: POST /exec?action=create
  const action = e.parameter.action;

  // レスポンスを返す
  const result = { status: 'ok', received: name };
  return ContentService.createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
```

#### イベントオブジェクト `e` の構造

| プロパティ | 説明 | 例 |
|---|---|---|
| `e.parameter` | クエリパラメータ（単一値） | `{ action: 'list' }` |
| `e.parameters` | クエリパラメータ（配列） | `{ tag: ['a', 'b'] }` |
| `e.pathInfo` | `/exec` 以降のパス | `'api/users'` |
| `e.postData.contents` | POSTボディ（文字列） | `'{"name":"test"}'` |
| `e.postData.type` | Content-Type | `'application/json'` |
| `e.contentLength` | ボディのバイト数 | `18` |

#### レスポンスの種類

```javascript
// JSON レスポンス
return ContentService.createTextOutput(JSON.stringify(data))
  .setMimeType(ContentService.MimeType.JSON);

// テキスト レスポンス
return ContentService.createTextOutput('OK');

// HTML レスポンス（Webページを返す場合）
return HtmlService.createHtmlOutput('<h1>Hello</h1>');
```

#### Web APIデプロイ時の注意

- GASのWeb APIは通常のサーバーのように柔軟なCORSヘッダー制御ができない。
- ブラウザのフロントエンドから直接 `fetch` する場合、CORSエラーやリダイレクトの制約に注意（プロキシサーバー等を検討）。
- コード変更後は、`clasp push` だけでなく、新しいバージョンを発行して `deploy` し直す必要がある（またはテストデプロイURLを使用）。
- 公開範囲（自分のみ / 組織内 / 全員）と実行ユーザー（自分 / アクセスしたユーザー）はデプロイ設定で適切に選択する。

### 外部API連携

```javascript
// src/api.js
function callExternalAPI() {
  const apiKey = PropertiesService.getScriptProperties().getProperty('API_KEY');
  const response = UrlFetchApp.fetch('https://api.example.com/data', {
    method: 'GET',
    headers: { 'Authorization': 'Bearer ' + apiKey }
  });
  return JSON.parse(response.getContentText());
}
```

必要なスコープ: `https://www.googleapis.com/auth/script.external_request`

### メール送信（MailApp vs GmailApp）

基本的には `MailApp` を推奨する。

| 項目 | MailApp | GmailApp |
|---|---|---|
| **用途** | メール送信に特化 | Gmail全般の操作（送信・受信・検索・ラベル等） |
| **必要なスコープ** | `gmail.send`（送信のみ） | `mail.google.com`（Gmailの全権限） |
| **セキュリティ** | 最小権限で安全 | 不要な権限まで承認が必要 |
| **推奨度** | **推奨** | メール検索・ラベル操作等が必要な場合のみ |

```javascript
// MailApp（推奨）: シンプルなメール送信
MailApp.sendEmail(recipient, subject, body);

// オプション付き
MailApp.sendEmail({
  to: recipient,
  subject: subject,
  body: body,                    // プレーンテキスト
  htmlBody: '<h1>Hello</h1>',   // HTML（bodyのフォールバック付き）
  attachments: [blob],           // 添付ファイル
  cc: 'cc@example.com',
  bcc: 'bcc@example.com',
  replyTo: 'reply@example.com',
  name: '送信者名'
});

// 残りの送信可能数を確認（クォータ管理）
const remaining = MailApp.getRemainingDailyQuota();
console.log('本日の残り送信可能数:', remaining);
```

```javascript
// GmailApp: Gmail固有の機能が必要な場合のみ
// スレッド検索、ラベル操作、下書き作成など
const threads = GmailApp.search('is:unread');
GmailApp.sendEmail(recipient, subject, body);
```

**選択の目安**: メールを送るだけなら `MailApp`、受信メールの検索・ラベル操作・下書き管理が必要なら `GmailApp` を使用する。
