# clasp 2.x から 3.x への移行ガイド

clasp 3.x は 2.x から大きな破壊的変更があります。このガイドでは主な変更点と移行手順を説明します。

## 主な変更点

### 1. TypeScript 自動トランスパイル機能の廃止

**clasp 2.x**: TypeScript ファイル（`.ts`）を自動的にトランスパイルして push

**clasp 3.x**: TypeScript のトランスパイル機能が完全に廃止。バンドラー（Rollup、Webpack、esbuild など）を使った事前ビルドが必須

**対応方法**:
- 推奨テンプレートプロジェクトを使用（`references/typescript-support.md` 参照）
- または手動でバンドラーを設定

### 2. コマンド名の変更

clasp 3.x では多くのコマンドが改名されました。互換性のためエイリアスは残っていますが、新しい名前の使用が推奨されます。

| clasp 2.x | clasp 3.x | 説明 |
|-----------|-----------|------|
| `clasp create` | `clasp create-script` | プロジェクト作成 |
| `clasp clone` | `clasp clone-script` | プロジェクトクローン |
| `clasp open` | `clasp open-script` | GAS エディタを開く |
| `clasp open --web` | `clasp open-web-app` | Web アプリを開く |
| `clasp open --addon` | `clasp open-container` | コンテナを開く |
| `clasp open --creds` | `clasp open-credentials-setup` | 認証情報設定を開く |
| `clasp status` | `clasp show-file-status` | ファイルステータス表示 |
| `clasp deployments` | `clasp list-deployments` | デプロイ一覧 |
| `clasp versions` | `clasp list-versions` | バージョン一覧 |
| `clasp deploy` | `clasp create-deployment` | 新規デプロイ作成 |
| `clasp deploy -i <id>` | `clasp update-deployment <id>` | デプロイ更新 |
| `clasp version` | `clasp create-version` | バージョン作成 |
| `clasp logs --open` | `clasp open-logs` | ログを開く |
| `clasp apis --open` | `clasp open-api-console` | API コンソールを開く |
| `clasp apis enable <api>` | `clasp enable-api <api>` | API 有効化 |
| `clasp apis disable <api>` | `clasp disable-api <api>` | API 無効化 |

### 3. Node.js バージョン要件の変更

**clasp 2.x**: Node.js 12+ 以上

**clasp 3.x**: Node.js 22.0.0 以上

```bash
# Node.js バージョン確認
node --version

# 必要に応じてアップグレード
# nvm を使用している場合
nvm install 22
nvm use 22
```

### 4. ログイン時の認証情報オプションの変更

**clasp 2.x**:
```bash
clasp login --creds client_secret.json
```

**clasp 3.x**:
```bash
clasp login --user <name> --creds client_secret.json
```

`--user` オプションで認証情報に名前を付けることが推奨されます。

## 移行手順

### ステップ 1: Node.js のバージョンを確認・アップグレード

```bash
node --version
# 22.0.0 未満の場合はアップグレード
```

### ステップ 2: clasp 3.x へアップグレード

```bash
npm install -g @google/clasp@latest
clasp --version
# 3.0.0 以上であることを確認
```

### ステップ 3: プロジェクトの package.json を更新

```json
{
  "devDependencies": {
    "@google/clasp": "^3.0.0"
  },
  "scripts": {
    "open": "clasp open-script",
    "status": "clasp show-file-status",
    "deploy": "clasp create-deployment",
    "deployments": "clasp list-deployments"
  }
}
```

### ステップ 4: TypeScript プロジェクトの場合

TypeScript を使用している場合は、バンドラーの設定が必要です。

**オプション A**: 推奨テンプレートを使用
- [apps-script-engine-template](https://github.com/WildH0g/apps-script-engine-template)
- [clasp-typescript-template](https://github.com/tomoyanakano/clasp-typescript-template)
- [aside](https://github.com/google/aside)
- [apps-script-typescript-rollup-starter](https://github.com/sqrrrl/apps-script-typescript-rollup-starter)

**オプション B**: 手動でバンドラーを設定
詳細は `references/typescript-support.md` を参照。

### ステップ 5: コマンドの更新

既存のスクリプトやドキュメントで使用しているコマンドを新しい名前に更新します。

```bash
# 旧
clasp create --type sheets
clasp open
clasp status

# 新
clasp create-script --type sheets
clasp open-script
clasp show-file-status
```

### ステップ 6: 動作確認

```bash
# バージョン確認
clasp --version

# ファイルステータス確認
clasp show-file-status

# プッシュテスト
clasp push
```

## トラブルシューティング

### Node.js バージョンエラー

```
Error: The library requires NodeJS version >= 22.0.0
```

**解決方法**: Node.js 22.0.0 以上にアップグレード

### TypeScript ファイルがトランスパイルされない

clasp 3.x では TypeScript の自動トランスパイルが廃止されています。バンドラーを使用してください。

### コマンドが見つからない

古いコマンド名を使用している可能性があります。新しいコマンド名に更新してください。

## 参考リンク

- [clasp 公式リポジトリ](https://github.com/google/clasp)
- [clasp 3.x 移行ガイド（公式）](https://github.com/google/clasp#migrating-from-2x-to-3x)
- TypeScript テンプレートプロジェクト集（`references/typescript-support.md` 参照）
