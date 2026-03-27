# gas-clasp スキル

Google Apps Script (GAS) + clasp を使った開発を支援する Windsurf スキルです。

AI エージェントが GAS プロジェクトのセットアップ、コード生成、デプロイを行う際に、GAS 固有の言語制約やベストプラクティスを自動的に参照し、正しいコードと開発フローを提供します。

## スキルのフォルダ構造

```
gas-clasp-skill/
├── README.md                                  # 本ファイル（スキルの説明）
└── gas-clasp/
    ├── SKILL.md                               # スキル定義ファイル（メインエントリ）
    └── references/                            # 参照ドキュメント群
        ├── project-templates.md               # プロジェクトテンプレート集
        ├── migration-to-3x.md                 # clasp 2.x → 3.x 移行ガイド
        ├── advanced-commands.md               # 高度な clasp コマンド
        ├── webapp-deployment.md               # Web アプリデプロイ手順
        ├── gas-language-constraints.md         # GAS 言語仕様・制約
        ├── typescript-support.md               # TypeScript 対応ガイド
        ├── performance-optimization.md         # パフォーマンス最適化
        ├── testing-strategies.md               # テスト手法
        ├── development-bestpractices.md        # 開発ベストプラクティス
        ├── quotas-and-limits.md                # GAS クォータ・制限値
        └── security-bestpractices.md           # セキュリティベストプラクティス
```

## 各ファイルの説明

### `gas-clasp/SKILL.md`

スキルのメインエントリファイルです。以下の情報を定義しています。

- **メタデータ** — スキル名、説明、ライセンス、互換性情報
- **前提条件** — Node.js 22+、clasp 3.0+、Apps Script API の有効化、clasp ログイン
- **手順** — プロジェクト構成の作成から開発・デプロイまでのステップバイステップガイド
- **エラーハンドリング** — よくあるエラーと対処法
- **適用条件** — どのようなユーザーリクエストでこのスキルが発動するか

### `gas-clasp/references/`

`SKILL.md` から参照される詳細ドキュメントを格納するディレクトリです。

| ファイル | 内容 |
| --- | --- |
| `project-templates.md` | `appsscript.json`、`package.json`、`.claspignore` 等の必須ファイルテンプレートと、プロジェクトタイプ別の推奨構成 |
| `migration-to-3x.md` | clasp 2.x から 3.x への移行ガイド、コマンド対照表 |
| `advanced-commands.md` | 複数ユーザー管理、`clasp run-function`、ログ表示、API 管理など高度なコマンド |
| `webapp-deployment.md` | Web アプリのデプロイ手順、GUI 確認、トラブルシューティング |
| `gas-language-constraints.md` | ES Modules 非対応、Node.js/ブラウザ API 非対応など GAS 固有の言語制約 |
| `typescript-support.md` | clasp 3.x での TypeScript 利用方法（バンドラー必須） |
| `performance-optimization.md` | バッチ処理、API 呼び出し削減、CacheService 活用など最適化パターン |
| `testing-strategies.md` | テスト用関数、アサーション、テスト用シート運用など GAS 向けテスト戦略 |
| `development-bestpractices.md` | ファイル分割、定数管理、ログ出力など GAS 開発全般のベストプラクティス |
| `quotas-and-limits.md` | スクリプト実行時間、API 呼び出し回数など GAS の制限値一覧 |
| `security-bestpractices.md` | 秘密情報の管理（PropertiesService）、OAuth スコープの最小化、入力値検証 |

## 利用方法

### 1. スキルのインストール

本スキルフォルダ (`gas-clasp/`) を Windsurf のスキルディレクトリに配置します。

### 2. スキルの発動

以下のようなリクエストを Windsurf に送ると、このスキルが自動的に適用されます。

- 「GAS プロジェクトをセットアップして」
- 「clasp で新しいスプレッドシート連携スクリプトを作って」
- 「Google Apps Script でWeb APIを作りたい」
- 「既存の GAS プロジェクトをクローンして編集したい」

### 3. スキルが提供するサポート

スキル適用時、AI エージェントは以下を自動的に行います。

1. **環境確認** — Node.js、clasp のインストール状況とログイン状態を確認
2. **プロジェクト構成の作成** — `SKILL.md` の推奨構成に従い、必要なファイルを生成（テンプレートは `references/project-templates.md` を参照）
3. **GAS コードの生成** — `references/gas-language-constraints.md` の制約を遵守したコードを出力
4. **パフォーマンス考慮** — `references/performance-optimization.md` に基づく最適化を適用
5. **セキュリティ確認** — `references/security-bestpractices.md` に従い、秘密情報の安全な管理を徹底
6. **デプロイ支援** — clasp コマンドによるバージョン作成・デプロイを実行

## 動作要件

このスキルが対象とする GAS + clasp 開発環境には以下が必要です。

- **Node.js 22.0.0 以上**
- **npm**
- **@google/clasp 3.0 以上**
- Apps Script API が有効化されていること（[設定ページ](https://script.google.com/home/usersettings)）
- `npx clasp login` で Google アカウントにログイン済みであること

## ⚠️ clasp バージョンに関する重要な注意事項

### clasp 3.x と 2.x の違い

**このスキルは clasp 3.x に対応しています。** clasp のバージョンによってコマンド名や機能が大きく異なるため、必ず環境のバージョンを確認してください。

```bash
# バージョン確認
clasp --version
```

### 主な違い

| 項目 | clasp 2.x | clasp 3.x（本スキル対応） |
|------|-----------|---------------------------|
| **Node.js 要件** | 12+ 以上 | **22.0.0 以上** |
| **TypeScript** | 自動トランスパイル対応 | **廃止（バンドラー必須）** |
| **コマンド名** | 大幅に変更 | 詳細は `references/migration-to-3x.md` 参照 |

バージョンの違いによる問題（コマンドが見つからない、TypeScript が動かない、Node.js エラー等）とその対処法については `gas-clasp/references/migration-to-3x.md` を参照してください。

## ライセンス

MIT
