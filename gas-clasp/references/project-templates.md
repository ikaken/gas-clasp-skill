# プロジェクトテンプレート

## 必須ファイルの作成例

### `src/appsscript.json`

```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "oauthScopes": []
}
```

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
    "open": "clasp open",
    "version": "clasp version",
    "deploy": "clasp deploy"
  },
  "devDependencies": {
    "@google/clasp": "^2.4.2",
    "@types/google-apps-script": "^1.0.83"
  }
}
```

## プロジェクトタイプ別の作成方法

clasp でプロジェクトを作成する際、`--type` オプションでプロジェクトタイプを指定できる：

```bash
# スタンドアロンスクリプト（デフォルト）
clasp create-script --type standalone

# スプレッドシート連携
clasp create-script --type sheets

# ドキュメント連携
clasp create-script --type docs

# スライド連携
clasp create-script --type slides

# フォーム連携
clasp create-script --type forms

# Webアプリ
clasp create-script --type webapp

# API実行可能スクリプト
clasp create-script --type api
```

### Webアプリプロジェクトの OAuth スコープ

Webアプリとしてデプロイする場合、カスタム OAuth クライアントを使用する際は以下のスコープが必要：

```
https://www.googleapis.com/auth/script.webapp.deploy
```

その他の clasp 関連スコープ：

```
https://www.googleapis.com/auth/script.deployments
https://www.googleapis.com/auth/script.projects
https://www.googleapis.com/auth/drive.metadata.readonly
https://www.googleapis.com/auth/drive.file
```

詳細は clasp の公式ドキュメントを参照。

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
