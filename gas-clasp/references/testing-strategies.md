# テスト手法

GASには標準のテストフレームワークがないため、以下の手法を組み合わせてテストを行う。

## 1. テスト用関数による手動テスト

GASエディタまたは `clasp run-function` でテスト用関数を直接実行する。

```javascript
// src/tests.js — テスト用関数をまとめたファイル

function test_main() {
  console.log('=== test_main 開始 ===');

  // テスト用シートを準備
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const testSheet = ss.getSheetByName('テスト用') || ss.insertSheet('テスト用');
  testSheet.clear();

  // テストデータを投入
  testSheet.getRange(1, 1, 3, 2).setValues([
    ['名前', 'スコア'],
    ['Alice', 80],
    ['Bob', 95],
  ]);

  // テスト対象関数を実行
  const result = processSheet(testSheet);

  // 結果をログで確認
  console.log('結果:', JSON.stringify(result));
  console.log('=== test_main 終了 ===');
}
```

> **注意**: テスト用関数は `.claspignore` で除外するか、本番デプロイ前に削除する運用でも良い。命名規則（`test_` プレフィックス）で区別すると管理しやすい。

## 2. アサーション用ヘルパー

簡易的なアサーション関数を用意すると、テスト結果の確認が楽になる。

```javascript
// src/testHelper.js

function assertEqual(actual, expected, label) {
  if (actual === expected) {
    console.log('✅ PASS: ' + label);
  } else {
    console.error('❌ FAIL: ' + label + ' | expected: ' + expected + ' | actual: ' + actual);
  }
}

function assertArrayEqual(actual, expected, label) {
  const actualStr = JSON.stringify(actual);
  const expectedStr = JSON.stringify(expected);
  if (actualStr === expectedStr) {
    console.log('✅ PASS: ' + label);
  } else {
    console.error('❌ FAIL: ' + label + ' | expected: ' + expectedStr + ' | actual: ' + actualStr);
  }
}

function assertThrows(func, label) {
  try {
    func();
    console.error('❌ FAIL: ' + label + ' | 例外が発生しなかった');
  } catch (e) {
    console.log('✅ PASS: ' + label + ' | 例外: ' + e.message);
  }
}
```

使用例：

```javascript
function test_calculateTotal() {
  assertEqual(calculateTotal([10, 20, 30]), 60, 'calculateTotal: 合計値');
  assertEqual(calculateTotal([]), 0, 'calculateTotal: 空配列');
  assertThrows(() => calculateTotal(null), 'calculateTotal: nullで例外');
}
```

## 3. テスト用シートの運用

本番データを直接使わず、テスト専用のシートで検証する。

```javascript
function setupTestSheet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  let testSheet = ss.getSheetByName('テスト用');

  if (testSheet) {
    testSheet.clear();
  } else {
    testSheet = ss.insertSheet('テスト用');
  }

  // テストデータの投入
  const testData = [
    ['ID', '名前', 'メール', 'ステータス'],
    [1, 'テスト太郎', 'test1@example.com', '有効'],
    [2, 'テスト花子', 'test2@example.com', '無効'],
    [3, '', 'invalid-email', '有効'],  // 異常データ
  ];
  testSheet.getRange(1, 1, testData.length, testData[0].length).setValues(testData);

  return testSheet;
}

function cleanupTestSheet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const testSheet = ss.getSheetByName('テスト用');
  if (testSheet) {
    ss.deleteSheet(testSheet);
  }
}
```

## 4. Web API（doGet / doPost）のテスト

Web APIとして公開した関数は、テスト用関数でイベントオブジェクトを模擬して検証できる。

```javascript
function test_doPost() {
  // イベントオブジェクトを模擬
  const mockEvent = {
    postData: {
      contents: JSON.stringify({ name: 'テスト', email: 'test@example.com' }),
      type: 'application/json',
    },
    parameter: { action: 'create' },
    parameters: { action: ['create'] },
  };

  const response = doPost(mockEvent);
  const result = JSON.parse(response.getContent());

  assertEqual(result.status, 'ok', 'doPost: ステータスがok');
  assertEqual(result.received, 'テスト', 'doPost: nameが正しい');
}

function test_doGet() {
  const mockEvent = {
    parameter: { action: 'list', limit: '10' },
    parameters: { action: ['list'], limit: ['10'] },
  };

  const response = doGet(mockEvent);
  const result = JSON.parse(response.getContent());

  assertEqual(result.status, 'ok', 'doGet: ステータスがok');
}
```

## 5. テスト運用のベストプラクティス

1. **テスト関数の命名**: `test_` プレフィックスを付ける（`test_main`, `test_doPost` 等）
2. **テストデータの分離**: 本番シートではなくテスト用シートを使う
3. **セットアップ/クリーンアップ**: テストの前後でデータを初期化・削除する
4. **境界値テスト**: 空配列、null、不正な入力値もテストする
5. **ログ確認**: GASエディタの「実行ログ」またはCloud Loggingで結果を確認
6. **本番除外**: テスト用ファイルは `.claspignore` で除外するか、デプロイ前に確認する
