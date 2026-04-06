---
name: gas-clasp
description: Google Apps Script (GAS) プロジェクトをclaspで自動初期化し、ローカル開発環境を構築する。GASプロジェクト、clasp、スプレッドシート連携、Web API、外部API連携、GASセットアップなどのキーワードに対応。clasp 3.x に対応。
license: MIT
compatibility: Node.js 22+, npm, @google/clasp 3.0+
metadata:
  author: ikaken
  version: "3.0"
---

## Purpose

GASプロジェクトを自動で初期化し、ローカル開発環境を構築する。
実行後すぐにGASコードの開発を開始できる状態にする。

## Prerequisites

- **Node.js 22.0.0 以上**がインストールされていること
- **clasp 3.0 以上**がインストールされていること
- Apps Script API が有効化されていること（https://script.google.com/home/usersettings）
- `clasp login` で Google アカウントにログイン済みであること（`~/.clasprc.json` が存在）

## Step-by-step Procedure

このスキルが実行されたら、以下を**確認なしで順番に自動実行**する。
エラーが発生した場合のみユーザーに通知して中断する。

### 1. 環境チェック

以下のコマンドを実行し、バージョンを確認する：

```bash
node --version
clasp --version
```

- Node.js 22.0.0 未満 → エラー表示して中断（「Node.js 22以上が必要です」）
- clasp 3.0.0 未満 → `npm install -g @google/clasp@latest` を実行してリトライ
- clasp 未インストール → `npm install -g @google/clasp@latest` を実行
- `~/.clasprc.json` が存在しない → 「`clasp login` を実行してください」と案内して中断

### 2. プロジェクト情報をユーザーに質問

以下を聞く（最小限の質問で進める）：

1. **プロジェクト名**（GASエディタ上の表示名）
2. **プロジェクトタイプ**（以下から選択）:
   - `sheets` - スプレッドシート連携
   - `docs` - ドキュメント連携
   - `slides` - スライド連携
   - `forms` - フォーム連携
   - `webapp` - Webアプリ（doGet/doPost）
   - `api` - API実行
   - `standalone` - スタンドアロン
3. **新規作成 or 既存クローン**:
   - 新規 → ステップ3aへ
   - 既存 → スクリプトIDを聞いてステップ3bへ

### 3. GASプロジェクト作成

#### 3a. 新規作成の場合

```bash
mkdir -p src
npx clasp create-script --type <タイプ> --title "<プロジェクト名>" --rootDir src
```

#### 3b. 既存クローンの場合

```bash
mkdir -p src
npx clasp clone-script <スクリプトID> --rootDir src
```

### 4. 設定ファイルの自動生成

以下のファイルを**すべて自動生成**する。既に存在するファイルは上書きしない。

#### package.json

```json
{
  "name": "<プロジェクト名（小文字ケバブケース）>",
  "version": "1.0.0",
  "description": "Google Apps Script project",
  "scripts": {
    "push": "clasp push",
    "push:force": "clasp push -f",
    "pull": "clasp pull",
    "open": "clasp open-script",
    "open:web": "clasp open-web-app",
    "status": "clasp show-file-status",
    "version": "clasp create-version",
    "deploy": "clasp push -f && clasp create-deployment",
    "deployments": "clasp list-deployments",
    "logs": "clasp tail-logs"
  },
  "devDependencies": {
    "@google/clasp": "^3.0.0",
    "@types/google-apps-script": "^1.0.83"
  }
}
```

生成後、`npm install` を実行する。

#### .gitignore

```
node_modules/
.clasp.json
.clasprc.json
*.log
```

#### .claspignore

```
**/**
!src/**
!src/appsscript.json
```

#### src/appsscript.json（新規作成で存在しない場合のみ）

プロジェクトタイプに応じて生成：

**sheets/docs/slides/forms/standalone/api の場合:**
```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8"
}
```

**webapp の場合:**
```json
{
  "timeZone": "Asia/Tokyo",
  "dependencies": {},
  "exceptionLogging": "STACKDRIVER",
  "runtimeVersion": "V8",
  "webapp": {
    "executeAs": "USER_DEPLOYING",
    "access": "MYSELF"
  }
}
```

#### src/main.js（新規作成の場合のみ）

プロジェクトタイプに応じたスターターコードを生成：

**sheets の場合:**
```javascript
/**
 * スプレッドシートを開いた時に実行されるメニュー追加
 */
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu('カスタムメニュー')
    .addItem('実行', 'main')
    .addToUi();
}

/**
 * メイン処理
 */
function main() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  console.log('実行開始: ' + sheet.getName());
  // ここに処理を記述
  console.log('実行完了');
}
```

**webapp の場合:**
```javascript
/**
 * GETリクエストのハンドラ
 * @param {Object} e - イベントオブジェクト
 * @returns {TextOutput} JSONレスポンス
 */
function doGet(e) {
  const action = e.parameter.action || 'default';
  const result = { status: 'ok', action: action, timestamp: new Date().toISOString() };
  return ContentService.createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}

/**
 * POSTリクエストのハンドラ
 * @param {Object} e - イベントオブジェクト
 * @returns {TextOutput} JSONレスポンス
 */
function doPost(e) {
  const body = JSON.parse(e.postData.contents);
  const result = { status: 'ok', received: body, timestamp: new Date().toISOString() };
  return ContentService.createTextOutput(JSON.stringify(result))
    .setMimeType(ContentService.MimeType.JSON);
}
```

**standalone/api/その他の場合:**
```javascript
/**
 * メイン処理
 */
function main() {
  console.log('実行開始');
  // ここに処理を記述
  console.log('実行完了');
}
```

### 5. 初回プッシュ

```bash
npx clasp push -f
```

### 6. 完了報告

以下をユーザーに表示する：

```
GASプロジェクトの初期化が完了しました

プロジェクト名: <プロジェクト名>
タイプ: <タイプ>
ソースディレクトリ: src/

主要コマンド:
  npm run push      - GASにプッシュ
  npm run pull      - GASからプル
  npm run deploy    - プッシュ＋デプロイ
  npm run open      - GASエディタを開く

次のステップ:
  - src/ 内のコードを編集して npm run push
  - GitHub Actionsで自動デプロイしたい場合は gas-setup-cicd スキルを実行
```

## GASコード生成・編集時の必須ルール

このスキル実行後、GASコードの生成や編集を依頼された場合は以下を**必ず**遵守する。

### 言語制約

- **ES Modules非対応**: `import` / `export` は使用不可。すべての関数はグローバルスコープに配置
- **Node.js API非対応**: `require()`, `process`, `__dirname`, `fetch()` は使用不可
- **ブラウザAPI非対応**: `window`, `document` は使用不可
- **非同期処理**: GAS標準APIはすべて同期的。基本は同期処理として記述
- **名前衝突回避**: 異なるファイルで同名の関数・変数を定義しない
- **GAS固有API使用**: `fetch()` → `UrlFetchApp.fetch()`、`console.log()` → Cloud Logging

### パフォーマンス（最重要）

- **バッチ処理**: スプレッドシート操作は `getValues()` / `setValues()` で一括処理
- **API呼び出し削減**: オブジェクトを変数に保持して再利用
- **キャッシュ活用**: `CacheService` で重い処理結果を一時保存
- **排他制御**: `LockService` で同時実行時のデータ競合を防止
- **実行時間制限**: 6分（カスタム関数は30秒）。長時間処理は分割してトリガーで継続

### セキュリティ

- 秘密情報は `PropertiesService.getScriptProperties()` で管理し、コードに直接書かない
- `appsscript.json` の `oauthScopes` は必要最小限に
- Web API公開時は入力値を必ず検証

詳細は以下を参照：
- [GAS固有の言語仕様・制約](references/gas-language-constraints.md)
- [TypeScript対応](references/typescript-support.md)
- [パフォーマンス最適化](references/performance-optimization.md)
- [セキュリティベストプラクティス](references/security-bestpractices.md)

## Error Handling

- **clasp create-script が失敗**: `clasp login` でログインし直す。Apps Script APIが有効か確認
- **clasp push でエラー**: `.clasp.json` の `rootDir` が `src` になっているか確認
- **権限エラー**: `appsscript.json` の `oauthScopes` を確認
- **getActiveSpreadsheet() が null**: トリガー実行時は `SpreadsheetApp.openById()` を使用
- **実行時間超過（6分）**: 処理を分割し、`PropertiesService` で進捗を保存して次回トリガーで継続

## Supporting Resources

- references/project-templates.md — 必須ファイルのテンプレート
- references/migration-to-3x.md — clasp 2.x → 3.x 移行ガイド
- references/advanced-commands.md — 高度な clasp コマンド
- references/webapp-deployment.md — Web アプリデプロイ手順
- references/gas-language-constraints.md — GAS固有の言語仕様・制約
- references/typescript-support.md — TypeScript対応
- references/performance-optimization.md — パフォーマンス最適化
- references/testing-strategies.md — テスト手法
- references/development-bestpractices.md — 開発ベストプラクティス
- references/quotas-and-limits.md — GAS クォータ・制限値
- references/security-bestpractices.md — セキュリティベストプラクティス

## When to Apply

このスキルは以下の場合に適用される：

- ユーザーが「GASプロジェクト」「Google Apps Script」「clasp」「スプレッドシート」等のキーワードを使用
- GASの開発環境セットアップを依頼された場合
- 新規GASプロジェクトの作成を依頼された場合
- 既存GASプロジェクトのクローンを依頼された場合
