---
name: gas-clasp
description: Google Apps Script (GAS) プロジェクトでclaspを使った開発環境のセットアップ、コード生成、デプロイを支援する。GASプロジェクト、clasp、スプレッドシート連携、Web API、外部API連携などのキーワードに対応。
license: MIT
compatibility: Node.js 18+, npm, @google/clasp 2.4+
metadata:
  author: ikaken
  version: "1.0"
---

## Purpose

Google Apps Script (GAS) プロジェクトにおいて、claspを使ったローカル開発環境の構築・コード生成・デプロイを支援する。
GAS固有の言語制約を考慮した正しいコード生成と、ベストプラクティスに基づく開発フローを提供する。

## Prerequisites

- Node.js 18+ がインストールされていること
- Apps Script API が有効化されていること（https://script.google.com/home/usersettings）
- `npx clasp login` で Google アカウントにログイン済みであること

## Step-by-step Procedure

### 1. プロジェクト構成の確認・作成

以下のフォルダ構造を推奨する：

```
project-root/
├── src/                    # GASソースコード配置ディレクトリ
│   ├── appsscript.json    # GASマニフェスト
│   └── *.js               # GASスクリプトファイル
├── .clasp.json            # clasp設定（gitignore対象）
├── .claspignore           # clasp除外ファイル設定
├── .gitignore             # Git除外設定
├── package.json           # npm設定
└── README.md              # プロジェクト説明
```

各ファイルのテンプレートは references/project-templates.md を参照。

### 2. 初期セットアップ

```bash
# 依存関係インストール
npm install

# 新規GASプロジェクトを作成（typeは sheets, docs, slides, forms, webapp, api 等）
npx clasp create --type sheets --title "プロジェクト名" --rootDir src

# 既存のGASプロジェクトをクローン
npx clasp clone <スクリプトID> --rootDir src
```

`.clasp.json` の `rootDir` が `src` になっていることを確認する。

### 3. GASコード（JavaScript/TypeScript）の生成・編集

GASコードを生成・編集する際は、以下のGAS固有の制約を**必ず**遵守する：

- **ES Modules非対応**: `import` / `export` は使用不可。すべての関数はグローバルスコープに配置
- **Node.js API非対応**: `require()`, `process`, `__dirname`, `fetch()` は使用不可
- **ブラウザAPI非対応**: `window`, `document` は使用不可
- **非同期処理**: GAS標準APIはすべて同期的。基本は同期処理として記述
- **名前衝突回避**: 異なるファイルで同名の関数・変数を定義しない

TypeScriptを使用する場合は、トランスパイルや型の扱いについて制約があるため注意する。

詳細は以下を参照：
- [GAS固有の言語仕様・制約](references/gas-language-constraints.md)
- [TypeScript対応](references/typescript-support.md)

### 4. 基本的な開発コマンド

```bash
# ローカル → GAS にプッシュ
npm run push

# GAS → ローカル にプル
npm run pull

# GASエディタを開く
npm run open

# デプロイ済みWebアプリをブラウザで開く
npx clasp open-web-app

# プッシュ対象ファイルの確認
npx clasp status
```

### 5. GASエディタでの設定

`clasp open` でエディタを開き、以下を手動で設定する：

- **トリガー設定**: 左メニュー「トリガー」（時計アイコン）から追加
- **スクリプトプロパティ**: 「プロジェクトの設定」→「スクリプトプロパティ」でAPIキー等を設定
- **ライブラリ追加**: 必要に応じて外部ライブラリを追加

### 6. デプロイ（WebアプリやAPIとして公開する場合）

#### 基本的なデプロイコマンド

```bash
# 新規デプロイ（新しいバージョンとデプロイを作成）
clasp create-deployment --description "v1.0.0 - 説明"

# 特定バージョンで新規デプロイ
clasp create-deployment --versionNumber 4

# デプロイ一覧の確認
clasp list-deployments

# バージョン一覧の確認
clasp list-versions

# バージョン作成のみ
clasp create-version "バージョンの説明"
```

#### Webアプリのデプロイ

**重要**: Webアプリの場合、各デプロイには一意のURLが割り当てられる。

```bash
# 新規Webアプリデプロイ（新しいURLが発行される）
clasp create-deployment --description "初回リリース"

# 既存デプロイの更新（URLを変えずに更新）
clasp create-deployment --deploymentId <デプロイID> --description "更新"
# または
clasp update-deployment <デプロイID> --description "更新"

# デプロイされたWebアプリをブラウザで開く
clasp open-web-app
```

#### デプロイIDの確認方法

```bash
clasp list-deployments
```

出力例：
```
- AKfyc... @1 - 初回リリース
- AKfyc... @2 - 更新版
```

#### 注意事項

- コード変更後は `clasp push` だけでなく、新しいバージョンを発行して `deploy` し直す必要がある
- Webアプリで既存のURLを維持したい場合は、必ず `--deploymentId` を指定して更新する
- 新規デプロイを作成すると新しいURLが発行されるため、URLを共有している場合は注意

### 7. パフォーマンス最適化

GASコードを生成する際は、パフォーマンスを考慮する：

- **バッチ処理**: スプレッドシート操作は `getValues()` / `setValues()` で一括処理（最重要）
- **API呼び出し削減**: オブジェクトを変数に保持して再利用
- **キャッシュ活用**: `CacheService` で重い処理結果を一時保存
- **排他制御**: `LockService` で同時実行時のデータ競合を防止

詳細は references/performance-optimization.md を参照。

### 8. セキュリティ確認

- 秘密情報は `PropertiesService.getScriptProperties()` で管理し、コードに直接書かない
- `appsscript.json` の `oauthScopes` は必要最小限に
- Web API公開時は入力値を必ず検証

詳細は references/security-bestpractices.md を参照。

## Error Handling

- **clasp push でエラー**: `clasp login` でログインし直す
- **権限エラー**: `appsscript.json` の `oauthScopes` を確認
- **ファイルが見つからない**: `.clasp.json` の `rootDir` が `src` になっているか確認
- **スクリプトID不明**: GASエディタのURL `https://script.google.com/home/projects/<スクリプトID>/edit` から確認
- **getActiveSpreadsheet() が null**: 時間主導型トリガーからの実行では `openById()` を使用
- **関数が見つからない**: ファイル名の読み込み順序を確認、同名関数の衝突がないか確認
- **実行時間超過**: 処理を分割し、PropertiesServiceで進捗を保存して次回トリガーで継続
- **clasp push で不要ファイルがアップロードされる**: `.claspignore` に除外パターンを追加

## Supporting Resources

- references/project-templates.md — 必須ファイルのテンプレートとプロジェクトタイプ別推奨構成
- references/gas-language-constraints.md — GAS固有の言語仕様・制約
- references/typescript-support.md — TypeScript対応のプロジェクト構成と制約事項
- references/performance-optimization.md — パフォーマンス最適化パターン
- references/testing-strategies.md — テスト手法（テスト用シート、アサーションなど）
- references/development-bestpractices.md — GAS開発全般のベストプラクティスまとめ
- references/quotas-and-limits.md — GAS実行時の制約事項（クォータ・制限値）
- references/security-bestpractices.md — セキュリティベストプラクティス

## When to Apply

このスキルは以下の場合に適用される：

- ユーザーが「GASプロジェクト」「Google Apps Script」「clasp」「スプレッドシート」等のキーワードを使用
- claspコマンドの実行が必要な場合
- GASの開発環境セットアップを依頼された場合
- GASコードの生成・編集を依頼された場合

**自動作成の条件**:
- ユーザーが明示的にセットアップを依頼した場合
- 既存プロジェクト構成を確認後、不足分がある場合のみ提案または作成
- 既に適切な構成がある場合は、既存構成を尊重し変更しない
