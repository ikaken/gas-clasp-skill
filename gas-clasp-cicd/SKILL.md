---
name: gas-setup-cicd
description: GASプロジェクトにGitHub Actionsによる自動デプロイ（CI/CD）を設定する。コミット・プッシュ時にclaspで自動的にGASへデプロイされる仕組みを構築する。GitHub Actions、GAS自動デプロイ、CI/CD設定などのキーワードに対応。
license: MIT
compatibility: Node.js 22+, gh CLI, @google/clasp 3.0+
metadata:
  author: ikaken
  version: "1.0"
---

## Purpose

GASを管理するGitHubリポジトリに、GitHub Actionsによる自動デプロイワークフローを追加する。
コミット・プッシュ時にclaspで自動的にGASへデプロイされる仕組みを構築する。

## Prerequisites

- GASプロジェクトがGitHubリポジトリで管理されていること
- `.clasp.json` がプロジェクトに存在すること（`.gitignore` に追加済みでもOK）
- clasp認証トークンが取得済みであること（`~/.clasprc.json` が存在。なければ `clasp login` を実行）
- `gh` CLI がインストール・認証済みであること（`gh auth status` で確認）

## Step-by-step Procedure

このスキルが実行されたら、以下を順番に自動実行する。

### 1. 環境・プロジェクト構成の確認

まず以下を確認する：

```bash
gh auth status          # gh CLI の認証状態
git remote -v           # GitHubリモートリポジトリの設定
ls .clasp.json          # clasp設定の存在
ls ~/.clasprc.json      # clasp認証トークンの存在
```

- `gh` CLI が未認証 → 「`gh auth login` を実行してください」と案内して中断
- GitHubリモートが未設定 → 「`git remote add origin <URL>` を実行してください」と案内して中断
- `.clasp.json` が存在しない → 「先に `gas-clasp` スキルでプロジェクトを初期化してください」と案内して中断
- `~/.clasprc.json` が存在しない → 「`clasp login` を実行してください」と案内して中断

### 2. GitHub Secrets の自動登録

`~/.clasprc.json` と `.clasp.json` から認証情報を読み取り、`gh secret set` で自動登録する。

**実行前にユーザーに確認**: 「GitHub Secretsに認証情報を登録します。よろしいですか？」

確認が取れたら以下を実行：

```bash
CLASPRC="$HOME/.clasprc.json"

# access_token
node -e "const t=require('$CLASPRC').tokens.default; process.stdout.write(t.access_token)" | gh secret set CLASP_ACCESS_TOKEN

# refresh_token
node -e "const t=require('$CLASPRC').tokens.default; process.stdout.write(t.refresh_token)" | gh secret set CLASP_REFRESH_TOKEN

# client_id
node -e "const t=require('$CLASPRC').tokens.default; process.stdout.write(t.client_id)" | gh secret set CLASP_CLIENT_ID

# client_secret
node -e "const t=require('$CLASPRC').tokens.default; process.stdout.write(t.client_secret)" | gh secret set CLASP_CLIENT_SECRET

# expiry_date
node -e "const t=require('$CLASPRC').tokens.default; process.stdout.write(String(t.expiry_date))" | gh secret set CLASP_EXPIRY_DATE

# scriptId
node -e "const c=require('./.clasp.json'); process.stdout.write(c.scriptId)" | gh secret set SCRIPT_ID
```

登録後、`gh secret list` で登録されたことを確認する。

### 3. GitHub Actions ワークフローファイルを作成

`.github/workflows/deploy-gas.yml` を以下の内容で作成する：

```yaml
name: Deploy to Google Apps Script

on:
  push:
    branches: [main]
    paths:
      - 'src/**'

  workflow_dispatch:
    inputs:
      deploy_description:
        description: 'デプロイの説明'
        required: false
        default: 'Manual deployment'

jobs:
  deploy:
    runs-on: ubuntu-latest
    timeout-minutes: 10

    steps:
      - name: リポジトリをチェックアウト
        uses: actions/checkout@v4

      - name: Node.js セットアップ
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: 依存関係インストール
        run: npm ci

      - name: clasp インストール
        run: npm install -g @google/clasp

      - name: clasp 認証設定
        run: |
          cat > ~/.clasprc.json << 'CLASPRC'
          {
            "tokens": {
              "default": {
                "access_token": "${{ secrets.CLASP_ACCESS_TOKEN }}",
                "refresh_token": "${{ secrets.CLASP_REFRESH_TOKEN }}",
                "client_id": "${{ secrets.CLASP_CLIENT_ID }}",
                "client_secret": "${{ secrets.CLASP_CLIENT_SECRET }}",
                "token_type": "Bearer",
                "expiry_date": ${{ secrets.CLASP_EXPIRY_DATE }},
                "type": "authorized_user"
              }
            }
          }
          CLASPRC

      - name: .clasp.json 設定
        run: |
          cat > .clasp.json << 'CLASPJSON'
          {
            "scriptId": "${{ secrets.SCRIPT_ID }}",
            "rootDir": "src"
          }
          CLASPJSON

      - name: GAS にプッシュ
        run: clasp push -f

      - name: デプロイ作成
        if: github.event_name == 'push'
        run: |
          DESCRIPTION="Auto deploy: $(git log -1 --pretty=format:'%s' HEAD)"
          clasp create-deployment --description "$DESCRIPTION"

      - name: デプロイ作成（手動トリガー）
        if: github.event_name == 'workflow_dispatch'
        run: |
          clasp create-deployment --description "${{ github.event.inputs.deploy_description }}"

      - name: デプロイ一覧表示
        run: clasp list-deployments
```

### 4. .gitignore の確認・更新

以下が `.gitignore` に含まれていることを確認し、不足分を追加する：

```
.clasp.json
node_modules/
.clasprc.json
```

### 5. 完了報告

以下をユーザーに表示する：

```
GitHub Actions 自動デプロイの設定が完了しました

登録済みSecrets:
  - CLASP_ACCESS_TOKEN
  - CLASP_REFRESH_TOKEN
  - CLASP_CLIENT_ID
  - CLASP_CLIENT_SECRET
  - CLASP_EXPIRY_DATE
  - SCRIPT_ID

ワークフロー: .github/workflows/deploy-gas.yml

動作:
  - src/ 内のファイルを変更して main にプッシュ → 自動デプロイ
  - GitHub Actions タブから手動実行も可能

確認手順:
  1. src/ 内のファイルを変更
  2. git add, commit, push
  3. GitHub の Actions タブでワークフロー実行状況を確認
```

## Error Handling

- **認証エラー**: GitHub SecretsのトークンがExpiredの可能性。ローカルで `clasp login` し直して新しいトークンをSecretsに再登録
- **push失敗**: `rootDir` の設定やファイル構成を確認
- **デプロイ失敗**: GAS側の権限設定を確認（Apps Script APIが有効か）
- **gh secret set 失敗**: リポジトリへの書き込み権限があるか確認

## When to Apply

このスキルは以下の場合に適用される：

- ユーザーが「GitHub Actions」「自動デプロイ」「CI/CD」「GASの自動化」等のキーワードを使用
- GASプロジェクトの自動デプロイ設定を依頼された場合
- GitHub ActionsでGASをデプロイしたいと依頼された場合
