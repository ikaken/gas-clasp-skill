# GAS固有の言語仕様・制約

GASはブラウザのJavaScriptやNode.jsとは異なる制約がある。コード生成時は必ず以下を考慮すること。

## ES Modules非対応

```javascript
// NG: import/export は使えない
import { func } from './utils.js';
export function main() {}

// OK: すべての関数・変数はグローバルスコープに配置される
function main() {}
function helperFunc() {}
```

## グローバルスコープと初期化順序

- GASは複数のファイルを1つのグローバルスコープに結合して実行する。
- **異なるファイルで同名の関数・変数を定義すると衝突する**。プレフィックスや命名規則で回避すること。
- トップレベルの変数の初期化はファイルの読み込み順（名前順等）に依存するため、**初期化順序に依存する設計は避けること**。定数は関数内で取得するか、設定専用のオブジェクトにまとめるのが安全。

## 使用不可なAPI・機能

- `require()` / `import` — 使用不可
- `process`, `__dirname`, `__filename` — Node.js固有のため使用不可
- `window`, `document` — ブラウザ固有のため使用不可
- `fetch()` — 使用不可、代わりに `UrlFetchApp.fetch()` を使用

## 非同期処理（Promise / async / await）の扱い

- V8ランタイムでは文法上 `async/await` や `Promise` が使えるが、**GASの標準API（SpreadsheetApp, UrlFetchAppなど）はすべて同期的**。
- 無理に非同期設計を取り入れると、GASの実行時間制限やロック制御と相性が悪くなるため、基本は**同期的な処理として記述することを強く推奨**する。

## トリガー実行時の注意

```javascript
// 時間主導型トリガーから実行する場合、getActiveSpreadsheet() は null になる
// → スプレッドシートIDやURLを直接指定する

// NG: トリガー実行時に null になる可能性あり
const ss = SpreadsheetApp.getActiveSpreadsheet();

// OK: IDで明示的に取得
const ss = SpreadsheetApp.openById('スプレッドシートID');

// OK: URLで取得
const ss = SpreadsheetApp.openByUrl('https://docs.google.com/spreadsheets/d/...');
```

## フォーム送信トリガーのイベントオブジェクト

`onFormSubmit` トリガーは、**フォーム側とスプレッドシート側でイベントオブジェクト `e` の中身が全く異なる**。サンプルコードをコピペする際は、どちら側のスクリプトか必ず確認すること。

**フォーム側（イベントソース: フォーム）**:
```javascript
function onFormSubmit(e) {
  // e.response: FormResponseオブジェクト
  const itemResponses = e.response.getItemResponses();
  const email = e.response.getRespondentEmail();
  const timestamp = e.response.getTimestamp();

  itemResponses.forEach(item => {
    const question = item.getItem().getTitle();
    const answer = item.getResponse();
    console.log(question + ': ' + answer);
  });
}
```

**スプレッドシート側（イベントソース: スプレッドシート）**:
```javascript
function onFormSubmit(e) {
  // e.namedValues: { '質問タイトル': ['回答'], ... }
  // e.values: ['タイムスタンプ', '回答1', '回答2', ...]
  // e.range: 書き込まれたセル範囲

  const name = e.namedValues['お名前'][0];
  const email = e.namedValues['メールアドレス'][0];
  console.log(name + ': ' + email);
}
```

| プロパティ | フォーム側 | スプレッドシート側 |
|---|---|---|
| 回答データの取得 | `e.response.getItemResponses()` | `e.namedValues` / `e.values` |
| メールアドレス | `e.response.getRespondentEmail()` | `e.namedValues['メールアドレス'][0]` |
| タイムスタンプ | `e.response.getTimestamp()` | `e.values[0]`（先頭列） |
| スプレッドシート参照 | `FormApp.getActiveForm().getDestinationId()` で取得 | `SpreadsheetApp.getActiveSpreadsheet()` |

## トリガー実行順序の注意

- フォームとスプレッドシートの両方にトリガーを設定した場合、**実行順序は保証されない**（非同期で呼ばれる）。
- 処理の流れ: フォーム送信 → スプレッドシートに書き込み → フォーム側/スプレッドシート側のGASが**非同期で**実行
- **一方の処理結果に依存する設計は避けること**。どちらが先に実行されても正しく動くように設計する必要がある。

## デバッグとログ出力

### ログ出力方法

```javascript
// Cloud Logging（推奨）
console.log('デバッグ情報:', value);
console.error('エラー:', error);
console.warn('警告:', warning);

// 従来のLogger（GASエディタで確認）
Logger.log('ログ: ' + value);
```

### エラーハンドリングパターン

```javascript
function main() {
  try {
    // メイン処理
    const result = processData();
    console.log('処理成功:', result);
  } catch (error) {
    console.error('エラー発生:', error.message);
    console.error('スタックトレース:', error.stack);
    
    // エラーをスプレッドシートに記録
    const ss = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = ss ? ss.getSheetByName('エラーログ') : null;
    if (sheet) {
      sheet.appendRow([new Date(), error.message, error.stack]);
    }
    
    // 必要に応じてメール通知
    // MailApp.sendEmail('admin@example.com', 'GASエラー', error.message);
  }
}
```

### デバッグ用関数

```javascript
// 実行時間計測
function measureTime(func) {
  const start = new Date();
  func();
  const end = new Date();
  console.log('実行時間:', (end - start) / 1000, '秒');
}
```
