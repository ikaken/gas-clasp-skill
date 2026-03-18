# パフォーマンス最適化

## 1. バッチ処理（最重要）

スプレッドシート操作で最も効果が大きい最適化。API呼び出し回数を劇的に削減できる。

**悪い例（1セルずつ読み書き = N回のAPI呼び出し）**:
```javascript
// NG: 100行 × 2回 = 200回のAPI呼び出し（非常に遅い）
for (let i = 1; i <= 100; i++) {
  const value = sheet.getRange(i, 1).getValue();
  sheet.getRange(i, 2).setValue(value * 2);
}
```

**良い例（一括読み書き = 2回のAPI呼び出し）**:
```javascript
// OK: getValues + setValues = 2回のAPI呼び出し（高速）
const values = sheet.getRange(1, 1, 100, 1).getValues();
const newValues = values.map(row => [row[0] * 2]);
sheet.getRange(1, 2, 100, 1).setValues(newValues);
```

**appendRow の注意点**:
```javascript
// NG: ループ内で appendRow を繰り返す（毎回API呼び出し）
data.forEach(row => sheet.appendRow(row));

// OK: setValues で一括書き込み
const startRow = sheet.getLastRow() + 1;
sheet.getRange(startRow, 1, data.length, data[0].length).setValues(data);
```

## 2. キャッシュの活用

`CacheService` を使うと、重いAPI呼び出しや計算結果を一時保存できる。

```javascript
function getDataWithCache(key, fetchFunction, ttlSeconds) {
  const cache = CacheService.getScriptCache();
  const cached = cache.get(key);

  if (cached) {
    return JSON.parse(cached);
  }

  const data = fetchFunction();
  // CacheService の値サイズ上限は 100KB、TTL は最大 6時間（21600秒）
  cache.put(key, JSON.stringify(data), ttlSeconds || 3600);
  return data;
}

// 使用例
const apiData = getDataWithCache('externalApiResult', () => {
  const response = UrlFetchApp.fetch('https://api.example.com/data');
  return JSON.parse(response.getContentText());
}, 1800); // 30分キャッシュ
```

**CacheService の制限**:
- 1つの値の最大サイズ: **100 KB**
- 最大 TTL: **21,600秒（6時間）**
- `putAll()` で複数キーを一括設定可能（API呼び出し削減）

## 3. SpreadsheetApp.flush() の活用

```javascript
// 通常、スプレッドシートへの変更はスクリプト終了時にまとめて反映される
// flush() を呼ぶと、その時点で即座に反映される

// 用途1: 長時間処理の進捗表示
function processLargeData() {
  const statusCell = sheet.getRange('A1');
  const data = sheet.getDataRange().getValues();

  for (let i = 0; i < data.length; i += 100) {
    // 100行ごとに進捗を更新
    statusCell.setValue('処理中: ' + i + ' / ' + data.length);
    SpreadsheetApp.flush(); // ここで即座にシートに反映

    processChunk(data.slice(i, i + 100));
  }
  statusCell.setValue('完了');
}

// 用途2: 書き込み後すぐに読み取りが必要な場合
sheet.getRange('A1').setValue('新しい値');
SpreadsheetApp.flush(); // 反映を確定
const confirmed = sheet.getRange('A1').getValue(); // '新しい値' が取得できる
```

**注意**: `flush()` を頻繁に呼ぶとパフォーマンスが低下する。必要な場面でのみ使用すること。

## 4. LockService（排他制御）

トリガーが重複実行された場合や、複数ユーザーが同時にスクリプトを実行する場合のデータ競合を防ぐ。

```javascript
function processWithLock() {
  const lock = LockService.getScriptLock();
  try {
    // 最大10秒待機してロック取得を試みる
    lock.waitLock(10000);

    // --- ここから排他処理 ---
    const sheet = SpreadsheetApp.openById('ID').getSheetByName('データ');
    const lastRow = sheet.getLastRow();
    // データの読み書き（他の実行と競合しない）
    sheet.getRange(lastRow + 1, 1).setValue(new Date());
    // --- 排他処理ここまで ---

  } catch (e) {
    console.error('ロック取得失敗（別の実行が進行中）:', e.message);
  } finally {
    // 必ず finally でロックを解放する
    lock.releaseLock();
  }
}
```

**ロックの種類**:
- `LockService.getScriptLock()` — スクリプト全体で1つのロック（全ユーザー共有）
- `LockService.getUserLock()` — ユーザーごとのロック
- `LockService.getDocumentLock()` — ドキュメント（スプレッドシート等）ごとのロック

**使い分けの目安**:
- **ScriptLock**: トリガーの重複実行防止、共有リソースへのアクセス制御
- **UserLock**: ユーザーごとの処理が重複しないようにする場合
- **DocumentLock**: 同一スプレッドシートへの同時書き込みを制御する場合

## 5. 不要なAPI呼び出しの削減

```javascript
// NG: 毎回 getActiveSpreadsheet() を呼ぶ
function bad() {
  const sheet1 = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('シート1');
  const sheet2 = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('シート2');
  // SpreadsheetApp.getActiveSpreadsheet() が2回呼ばれる
}

// OK: オブジェクトを変数に保持して再利用
function good() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet1 = ss.getSheetByName('シート1');
  const sheet2 = ss.getSheetByName('シート2');
}
```

## 6. 長時間処理の分割実行パターン

6分の実行時間制限を超える処理は、トリガーチェーンで分割実行する。

```javascript
function processInChunks() {
  const props = PropertiesService.getScriptProperties();
  const startIndex = parseInt(props.getProperty('lastProcessedIndex') || '0');
  const sheet = SpreadsheetApp.openById('ID').getSheetByName('データ');
  const data = sheet.getDataRange().getValues();
  const MAX_RUNTIME_MS = 5 * 60 * 1000; // 5分（余裕を持たせる）
  const startTime = new Date().getTime();

  for (let i = startIndex; i < data.length; i++) {
    // 実行時間チェック
    if (new Date().getTime() - startTime > MAX_RUNTIME_MS) {
      props.setProperty('lastProcessedIndex', String(i));
      // 1分後に自分自身を再実行するトリガーを作成
      ScriptApp.newTrigger('processInChunks')
        .timeBased()
        .after(1 * 60 * 1000)
        .create();
      console.log('中断: ' + i + ' / ' + data.length + ' まで処理完了');
      return;
    }
    processRow(data[i]);
  }

  // 完了: 進捗をリセットし、一時トリガーを削除
  props.deleteProperty('lastProcessedIndex');
  cleanupTriggers('processInChunks');
  console.log('全件処理完了');
}

function cleanupTriggers(functionName) {
  ScriptApp.getProjectTriggers()
    .filter(t => t.getHandlerFunction() === functionName)
    .forEach(t => ScriptApp.deleteTrigger(t));
}
```
