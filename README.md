# gas-clasp スキル

Google Apps Script (GAS) + clasp を使った開発を支援する AI エディタ向けスキル集です。

## 元リポジトリからの変更点

元のスキル（Fork元）は、GASコードを修正するたびにスキルを手動実行してデプロイする方式でした。

本リポジトリでは、スキルを**2つに分割**し、**GitHub Actionsによる自動デプロイ**に対応しました。

| 項目 | Fork元（旧） | 本リポジトリ（新） |
|------|-------------|-------------------|
| **スキル構成** | 1つ（セットアップ〜デプロイまで一体） | 2つに分割（初期化 + CI/CD） |
| **デプロイ方法** | コード変更のたびにスキルを手動実行 | **mainブランチへのpushで自動デプロイ** |
| **CI/CD** | なし | GitHub Actions で `clasp push` + `clasp create-deployment` を自動実行 |
| **Secrets管理** | 手動 | `gh secret set` でコマンドから自動登録 |
| **対応エディタ** | Windsurf のみ | Windsurf / Cursor / Claude Code / Antigravity / Codex |
| **clasp対応** | 3.x | 3.x（認証構造を3.x形式に対応済み） |

## スキル一覧

| スキル | 説明 | 実行タイミング |
|--------|------|---------------|
| **gas-clasp** | GASプロジェクトの自動初期化（環境チェック → プロジェクト作成 → 設定ファイル生成 → 初回プッシュ） | プロジェクト作成時に1回 |
| **gas-setup-cicd** | GitHub Actionsによる自動デプロイのセットアップ（Secrets自動登録 → ワークフロー生成） | CI/CD設定時に1回 |

初期セットアップ後は、`src/` 内のコードを変更して `git push` するだけで自動的にGASにデプロイされます。

## 対応エディタ

| エディタ | 対応状況 |
|---------|---------|
| **Windsurf** | SKILL.md 形式で対応 |
| **Cursor** | commands 形式で対応 |
| **Claude Code** | commands 形式で対応 |
| **Antigravity** | SKILL.md 形式で対応 |
| **Codex** | SKILL.md 形式で対応 |

## フォルダ構造

```
gas-clasp-skill/
├── README.md
├── gas-clasp/                          # GASプロジェクト初期化スキル
│   ├── SKILL.md                        # スキル定義（Windsurf / Antigravity 共通）
│   └── references/                     # 参照ドキュメント群
│       ├── project-templates.md
│       ├── migration-to-3x.md
│       ├── advanced-commands.md
│       ├── webapp-deployment.md
│       ├── gas-language-constraints.md
│       ├── typescript-support.md
│       ├── performance-optimization.md
│       ├── testing-strategies.md
│       ├── development-bestpractices.md
│       ├── quotas-and-limits.md
│       └── security-bestpractices.md
└── gas-clasp-cicd/                     # CI/CD セットアップスキル
    └── SKILL.md                        # スキル定義（Windsurf / Antigravity 共通）
```

## 使い方

### 全体の流れ

```
1. GitHub で空のリポジトリを作成 → ローカルにクローン

2. gas-clasp スキルを実行（初回のみ）
   → 環境チェック（Node.js, clasp バージョン）
   → プロジェクト名・タイプを質問
   → clasp create-script でGASプロジェクト作成
   → package.json, .gitignore, src/main.js 等を自動生成
   → clasp push で初回プッシュ

3. gas-setup-cicd スキルを実行（初回のみ）
   → ~/.clasprc.json から認証情報を読み取り
   → gh secret set で GitHub Secrets に自動登録
   → .github/workflows/deploy-gas.yml を自動生成

4. Webアプリの場合: 初回デプロイ後にGASエディタで手動確認（初回のみ）
   → clasp open-script でGASエディタを開く
   → 「デプロイ」→「デプロイを管理」でアクセス権限を確認・設定
   → これはGoogleの仕様上、GUIでの設定が最終決定権を持つため必須
   ※ sheets/docs 等のWebアプリ以外のタイプでは不要

5. 以降の開発（スキル実行不要）
   → src/ 内のコードを編集
   → git add → git commit → git push
   → GitHub Actions が自動で clasp push + create-deployment を実行
```

> **注意（Webアプリの場合）**: 初回デプロイ時のみ、GASエディタでアクセス権限の手動確認が必要です。`appsscript.json` の `webapp` 設定は初期値にすぎず、GASエディタのGUI設定が最終決定権を持つというGoogleの仕様があるためです。2回目以降は `git push` による自動デプロイのみで動作します。

### 各エディタでの実行方法

| エディタ | gas-clasp | gas-setup-cicd |
|---------|-----------|----------------|
| **Windsurf** | 「GASプロジェクトをセットアップして」と入力 | 「GitHub Actionsで自動デプロイを設定して」と入力 |
| **Cursor** | `/gas-clasp` と入力 | `/gas-setup-cicd` と入力 |
| **Claude Code** | `/gas-clasp` と入力 | `/gas-setup-cicd` と入力 |
| **Antigravity** | 「GASプロジェクトをセットアップして」と入力 | 「GitHub Actionsで自動デプロイを設定して」と入力 |
| **Codex** | `$gas-clasp` と入力、または「GASプロジェクトをセットアップして」 | `$gas-setup-cicd` と入力、または「CI/CDを設定して」 |

## インストール方法

### Windsurf

```bash
# グローバル（全プロジェクトで使用）
cp -r gas-clasp ~/.windsurf/skills/gas-clasp
cp -r gas-clasp-cicd ~/.windsurf/skills/gas-clasp-cicd

# プロジェクトローカル
cp -r gas-clasp .windsurf/skills/gas-clasp
cp -r gas-clasp-cicd .windsurf/skills/gas-clasp-cicd
```

### Cursor

```bash
# プロジェクトローカル
mkdir -p .cursor/commands
cp gas-clasp/SKILL.md .cursor/commands/gas-clasp.md
cp gas-clasp-cicd/SKILL.md .cursor/commands/gas-setup-cicd.md
```

> **注意**: references/ フォルダの参照ドキュメントも活用したい場合は、`.cursor/rules/` にルールとして追加するか、プロジェクトルートに `gas-clasp/references/` を配置してください。

### Claude Code

```bash
# グローバル（全プロジェクトで使用・推奨）
mkdir -p ~/.claude/commands
cp gas-clasp/SKILL.md ~/.claude/commands/gas-clasp.md
cp gas-clasp-cicd/SKILL.md ~/.claude/commands/gas-setup-cicd.md

# プロジェクトローカル
mkdir -p .claude/commands
cp gas-clasp/SKILL.md .claude/commands/gas-clasp.md
cp gas-clasp-cicd/SKILL.md .claude/commands/gas-setup-cicd.md
```

### Antigravity

```bash
# グローバル（全プロジェクトで使用）
cp -r gas-clasp ~/.gemini/antigravity/skills/gas-clasp
cp -r gas-clasp-cicd ~/.gemini/antigravity/skills/gas-clasp-cicd

# プロジェクトローカル
cp -r gas-clasp .agents/skills/gas-clasp
cp -r gas-clasp-cicd .agents/skills/gas-clasp-cicd
```

### Codex

```bash
# グローバル（全プロジェクトで使用）
mkdir -p ~/.agents/skills
cp -r gas-clasp ~/.agents/skills/gas-clasp
cp -r gas-clasp-cicd ~/.agents/skills/gas-clasp-cicd

# プロジェクトローカル
mkdir -p .agents/skills
cp -r gas-clasp .agents/skills/gas-clasp
cp -r gas-clasp-cicd .agents/skills/gas-clasp-cicd
```

## 事前準備

### 必須

- **Node.js 22.0.0 以上**
- **@google/clasp 3.0 以上** (`npm install -g @google/clasp@latest`)
- **Apps Script API** が有効化されていること（[設定ページ](https://script.google.com/home/usersettings)）
- `clasp login` で Google アカウントにログイン済み

### CI/CD セットアップ時に追加で必要

- **gh CLI** がインストール・認証済み（`gh auth login`）
- GitHub リポジトリへの書き込み権限

### clasp 認証トークンの取得方法

```bash
# 1. clasp をインストール
npm install -g @google/clasp@latest

# 2. ログイン（ブラウザが開く）
clasp login

# 3. ~/.clasprc.json が生成される（clasp 3.x 形式）
#    gas-setup-cicd スキルがこのファイルを自動で読み取り GitHub Secrets に登録する
```

clasp 3.x の `~/.clasprc.json` 構造:

```json
{
  "tokens": {
    "default": {
      "access_token": "ya29.xxx...",
      "refresh_token": "1//xxx...",
      "client_id": "xxx.apps.googleusercontent.com",
      "client_secret": "xxx",
      "token_type": "Bearer",
      "expiry_date": 1234567890000,
      "type": "authorized_user"
    }
  }
}
```

## clasp バージョンについて

**このスキルは clasp 3.x に対応しています。**

| 項目 | clasp 2.x | clasp 3.x（本スキル対応） |
|------|-----------|---------------------------|
| **Node.js 要件** | 12+ | **22.0.0 以上** |
| **TypeScript** | 自動トランスパイル | **廃止（バンドラー必須）** |
| **コマンド名** | `create`, `clone` | `create-script`, `clone-script` |
| **認証ファイル構造** | `token` + `oauth2ClientSettings` | `tokens.default` に統合 |

詳細は `gas-clasp/references/migration-to-3x.md` を参照。

## ライセンス

MIT
