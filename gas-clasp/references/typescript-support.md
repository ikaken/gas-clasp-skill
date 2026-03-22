# TypeScript対応（clasp 3.x）

**重要**: clasp 3.x では TypeScript の自動トランスパイル機能が廃止されました。

TypeScript を使用する場合は、**バンドラー（Rollup、Webpack、esbuild など）を使って事前にトランスパイルする必要があります**。

## clasp 3.x での TypeScript 利用方法

### オプション1: 推奨テンプレートプロジェクトを使用

以下の公式推奨テンプレートを利用すると、TypeScript + バンドラーの環境が整っています：

- [apps-script-engine-template](https://github.com/WildH0g/apps-script-engine-template)
- [clasp-typescript-template](https://github.com/tomoyanakano/clasp-typescript-template)
- [aside](https://github.com/google/aside)
- [apps-script-typescript-rollup-starter](https://github.com/sqrrrl/apps-script-typescript-rollup-starter)

### オプション2: 手動でバンドラーを設定

#### プロジェクト構成例（Rollup使用）

```
project-root/
├── src/
│   ├── main.ts            # TypeScriptソース
│   └── utils.ts
├── dist/                  # ビルド出力先（clasp push対象）
│   ├── appsscript.json
│   └── Code.js            # トランスパイル済み
├── .clasp.json
├── .claspignore
├── package.json
├── tsconfig.json
└── rollup.config.js       # Rollup設定
```

#### tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2019",
    "module": "ESNext",
    "lib": ["ES2019"],
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "moduleResolution": "node",
    "outDir": "./dist"
  },
  "include": ["src/**/*"]
}
```

#### rollup.config.js（例）

```javascript
import typescript from '@rollup/plugin-typescript';
import { nodeResolve } from '@rollup/plugin-node-resolve';

export default {
  input: 'src/main.ts',
  output: {
    file: 'dist/Code.js',
    format: 'iife',
    name: 'GasApp'
  },
  plugins: [
    nodeResolve(),
    typescript()
  ]
};
```

#### package.json

```json
{
  "scripts": {
    "build": "rollup -c",
    "push": "npm run build && clasp push",
    "deploy": "npm run build && clasp create-deployment"
  },
  "devDependencies": {
    "@google/clasp": "^3.0.0",
    "@types/google-apps-script": "^1.0.83",
    "@rollup/plugin-node-resolve": "^15.0.0",
    "@rollup/plugin-typescript": "^11.0.0",
    "rollup": "^4.0.0",
    "typescript": "^5.0.0"
  }
}
```

#### .clasp.json

```json
{
  "scriptId": "your-script-id",
  "rootDir": "dist"
}
```

**重要**: `rootDir` を `dist` に設定し、ビルド済みファイルのみを push する。

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
