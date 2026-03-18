# TypeScript対応

claspはTypeScript（`.ts`）ファイルをサポートしており、push時に自動的にJavaScriptへトランスパイルされる。

## プロジェクト構成

```
project-root/
├── src/
│   ├── appsscript.json    # GASマニフェスト
│   ├── main.ts            # TypeScriptファイル
│   └── utils.ts
├── .clasp.json
├── .claspignore
├── package.json
└── tsconfig.json           # TypeScript設定
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "lib": ["esnext"],
    "target": "esnext",
    "strict": true,
    "noImplicitAny": true,
    "skipLibCheck": true
  }
}
```

> **注意**: claspが内部でトランスパイルを行うため、`module` や `outDir` の指定は不要。

## 型定義の活用

`@types/google-apps-script` をインストールすると、GAS APIの型補完が有効になる。

```bash
npm install --save-dev @types/google-apps-script
```

```typescript
// 型補完が効く例
function getSheetData(): string[][] {
  const ss: GoogleAppsScript.Spreadsheet.Spreadsheet =
    SpreadsheetApp.getActiveSpreadsheet();
  const sheet: GoogleAppsScript.Spreadsheet.Sheet =
    ss.getSheetByName('データ')!;
  const values: string[][] = sheet.getDataRange().getValues() as string[][];
  return values;
}
```

ただし、型注釈を省略しても型推論が効くため、簡潔に書くことも可能：

```typescript
function getSheetData() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheetByName('データ')!;
  return sheet.getDataRange().getValues();
}
```

## TypeScript使用時の注意点

### import / export は使えない

GASはES Modulesに対応していないため、TypeScriptでも `import` / `export` は使用不可。claspのトランスパイルはモジュール解決を行わない。

```typescript
// NG: import/export はトランスパイル後にエラーになる
import { helper } from './utils';
export function main() {}

// OK: すべての関数をグローバルスコープに配置
function main() {
  const result = helper();
}

function helper() {
  return 'ok';
}
```

### 名前空間を活用した衝突回避

`import` / `export` が使えない代わりに、`namespace` を使ってファイル間の名前衝突を回避できる。

```typescript
// src/utils.ts
namespace Utils {
  export function formatDate(date: Date): string {
    return Utilities.formatDate(date, 'Asia/Tokyo', 'yyyy-MM-dd');
  }

  export function isBlank(value: unknown): boolean {
    return value === null || value === undefined || value === '';
  }
}

// src/main.ts（別ファイルから利用）
function main() {
  const today = Utils.formatDate(new Date());
  console.log(today);
}
```

### 型アサーションの注意

GAS APIの戻り値は `Object[][]` 型の場合が多い。必要に応じて型アサーションを使う：

```typescript
interface UserRow {
  name: string;
  email: string;
  age: number;
}

function getUsers(): UserRow[] {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getSheetByName('ユーザー')!;
  const [header, ...rows] = sheet.getDataRange().getValues();
  return rows.map(row => ({
    name: row[0] as string,
    email: row[1] as string,
    age: row[2] as number,
  }));
}
```

## JavaScriptとTypeScriptの選択基準

| 条件 | 推奨 |
|---|---|
| 小規模なスクリプト（1〜2ファイル） | JavaScript |
| 複数ファイル・複雑なデータ構造 | TypeScript |
| チーム開発 | TypeScript |
| GAS初心者 | JavaScript（制約の理解を優先） |
