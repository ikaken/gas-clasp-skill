# セキュリティベストプラクティス

## 1. 秘密情報の管理

```javascript
// スクリプトプロパティから取得（推奨）
const apiKey = PropertiesService.getScriptProperties().getProperty('API_KEY');

// コードに直接書かない（NG）
// const apiKey = 'abc123xyz'; // 絶対にしない
```

## 2. 入力値の検証

```javascript
function doPost(e) {
  const data = JSON.parse(e.postData.contents);
  
  // 入力値を検証
  if (!data.email || !data.email.match(/^[^\s@]+@[^\s@]+\.[^\s@]+$/)) {
    return ContentService.createTextOutput(JSON.stringify({error: 'Invalid email'}))
      .setMimeType(ContentService.MimeType.JSON);
  }
  
  // 処理
}
```

## 3. スコープの最小化

`appsscript.json` で必要最小限のスコープのみ指定する：

```json
{
  "oauthScopes": [
    "https://www.googleapis.com/auth/spreadsheets.currentonly"
  ]
}
```

## 4. Web APIデプロイ時のセキュリティ

- **公開範囲を最小限に**: デプロイ設定で「自分のみ」または「組織内」を選択し、不要な公開を避ける
- **入力値を必ず検証**: `doPost(e)` で受け取るデータは信頼できない前提で検証する
- **実行ユーザーの選択に注意**: 「自分として実行」にすると、呼び出し元に自分の権限が付与される

## 5. セキュリティチェックリスト

1. **秘密情報**: スクリプトプロパティで管理、コードに直接書かない
2. **スコープ最小化**: `appsscript.json` の `oauthScopes` は必要最小限に
3. **入力値検証**: 外部から受け取るデータは信頼できない前提で検証する
4. **公開範囲**: デプロイ設定で「自分のみ」または「組織内」を選択し、不要な公開を避ける
5. **実行ユーザー**: 「自分として実行」にすると呼び出し元に自分の権限が付与されるため注意
6. **ソースコード管理**: `.clasp.json` や `.env` をリポジトリに含めない（`.gitignore` で除外）
7. **ログ出力**: 秘密情報やユーザーの個人情報をログに出力しない
