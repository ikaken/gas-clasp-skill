# Web アプリのデプロイ

GAS の Web アプリ（`doGet` / `doPost`）を外部に公開する手順を説明します。

## 前提理解（2層構造）

GAS の Web アプリ公開設定は**2層構造**になっています：

| レイヤー | 役割 | 優先度 |
|---------|------|--------|
| `appsscript.json` の `webapp` エントリ | デフォルト設定 | 低 |
| GAS エディタ（GUI） | **実際の公開設定** | **高（優先される）** |

**結論**: 最終的な公開状態は **GAS エディタの GUI 設定が決定します**。

## デプロイ手順

### ステップ1: `appsscript.json` の設定

`src/appsscript.json` に `webapp` エントリを追加（デフォルト値として）：

```json
"webapp": {
  "executeAs": "USER_DEPLOYING",
  "access": "ANYONE_ANONYMOUS"
}
```

JSON 全体のテンプレートや `executeAs` / `access` の各パラメータ詳細は `references/project-templates.md` の「Web アプリ用の設定」を参照。

### ステップ2: clasp でデプロイ

```bash
# コードをプッシュ（-f: ローカルのマニフェストでリモートを上書き）
clasp push -f

# 初回デプロイ（新しいURLが発行される）
clasp create-deployment --description "初回リリース"

# 既存デプロイの更新（URLを変えずに更新）
clasp push -f
clasp update-deployment <デプロイID> --description "更新内容"

# デプロイされたWebアプリをブラウザで開く
clasp open-web-app
```

**効率化**: `package.json` に登録しておくと便利

```json
"scripts": {
  "deploy": "clasp push -f && clasp update-deployment <デプロイID> -d 'update'"
}
```

### ステップ3: GAS エディタで最終確認（初回必須）

⚠️ **最重要**: `appsscript.json` で `ANYONE_ANONYMOUS` を指定しても、**そのままでは全員公開にならない場合があります**。

**初回デプロイ後、必ず以下の手順を実行**：

1. `clasp open-script` で GAS エディタを開く
2. 右上の「デプロイ」→「デプロイを管理」をクリック
3. 対象デプロイの ⚙️（歯車アイコン）→「デプロイを編集」をクリック
4. **アクセス権限を確認・設定**：
   - 外部公開する場合: **「全員」を選択**
   - ログイン不要にする場合: **「全員（匿名ユーザーを含む）」を選択**
5. 「デプロイ」ボタンをクリックして保存

**なぜ必要か**: Google の仕様上、`appsscript.json` は初期値であり、GUI 設定が最終決定権を持つため。

### ステップ4: 2回目以降の更新

初回の GUI 設定後は、clasp のみで更新可能：

```bash
clasp push -f
clasp update-deployment <デプロイID> --description "更新"
```

URL も公開状態も維持されます。

## デプロイIDの確認方法

```bash
clasp list-deployments
```

出力例：
```
- AKfycby... @1 - 初回リリース
- AKfycby... @2 - 更新版
```

## よくあるトラブルと対処法

| 問題 | 原因 | 対処法 |
|------|------|--------|
| 外部からアクセスできない | GUI で「全員」になっていない | GUI で「全員（匿名ユーザーを含む）」に変更 |
| ログインを求められる | `ANYONE` になっている | `appsscript.json` を `ANYONE_ANONYMOUS` に変更 + GUI 確認 |
| URL が変わってしまう | `--deploymentId` を付けずに新規 deploy | 必ず `update-deployment` で更新 |
| push しても反映されない | デプロイの更新をしていない | `push` 後に `update-deployment` も実行 |

## 運用ベストプラクティス

**初回**:
1. `clasp push -f` でコードをプッシュ
2. `clasp create-deployment` でデプロイ
3. **必ず GUI で「全員」公開を確認**

**2回目以降**:
- `clasp push -f && clasp update-deployment <ID>` のみで OK（URL も公開状態も維持）

**結論**: 「clasp で管理 + 初回だけ GUI 確定」が最も安定した運用方法
