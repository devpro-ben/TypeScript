## 編譯選項

選項                                     | 類型      | 默認值                    | 描述
----------------------------------------|-----------|--------------------------|----------------------------------------------------------------------
`--allowJs`                             | `boolean` |  `false`                 | 允許編譯javascript文件。
`--allowSyntheticDefaultImports`        | `boolean` | `module === "system"`或設置了`--esModuleInterop`且`module`不為`es2015`/`esnext` | 允許從沒有設置默認導出的模塊中默認導入。這並不影響代碼的輸出，僅為了類型檢查。
`--allowUnreachableCode`                | `boolean` | `false`                  | 不報告執行不到的代碼錯誤。
`--allowUnusedLabels`                   | `boolean` | `false`                  | 不報告未使用的標籤錯誤。
`--alwaysStrict`                        | `boolean` | `false`                  | 以嚴格模式解析並為每個源文件生成`"use strict"`語句
`--baseUrl`                             | `string`  |                          | 解析非相對模塊名的基準目錄。查看[模塊解析文檔](./Module Resolution.md#base-url)瞭解詳情。
`--charset`                             | `string`  | `"utf8"`                 | 輸入文件的字符集。
`--checkJs`                             | `boolean` | `false`                  | 在.js文件中報告錯誤。與`--allowJs`配合使用。
`--declaration`<br/>`-d`                | `boolean` | `false`                  | 生成相應的`.d.ts`文件。
`--declarationDir`                      | `string`  |                          | 生成聲明文件的輸出路徑。
`--diagnostics`                         | `boolean` | `false`                  | 顯示診斷信息。
`--disableSizeLimit`                    | `boolean` | `false`                  | 禁用JavaScript工程體積大小的限制
`--emitBOM`                             | `boolean` | `false`                  | 在輸出文件的開頭加入BOM頭（UTF-8 Byte Order Mark）。
`--emitDecoratorMetadata`<sup>[1]</sup> | `boolean` | `false`                  | 給源碼裡的裝飾器聲明加上設計類型元數據。查看[issue #2577](https://github.com/Microsoft/TypeScript/issues/2577)瞭解更多信息。
`--experimentalDecorators`<sup>[1]</sup>| `boolean` | `false`                  | 啟用實驗性的ES裝飾器。
`--extendedDiagnostics`                 | `boolean` | `false`                  | 顯示詳細的診段信息。
`--forceConsistentCasingInFileNames`    | `boolean` | `false`                  | 禁止對同一個文件的不一致的引用。
`--help`<br/>`-h`                       |           |                          | 打印幫助信息。
`--importHelpers`                       | `string`  |                          | 從[`tslib`](https://www.npmjs.com/package/tslib)導入輔助工具函數（比如`__extends`，`__rest`等）
`--inlineSourceMap`                     | `boolean` | `false`                  | 生成單個sourcemaps文件，而不是將每sourcemaps生成不同的文件。
`--inlineSources`                       | `boolean` | `false`                  | 將代碼與sourcemaps生成到一個文件中，要求同時設置了`--inlineSourceMap`或`--sourceMap`屬性。
`--init`                                |           |                          | 初始化TypeScript項目並創建一個`tsconfig.json`文件。
`--isolatedModules`                     | `boolean` | `false`                  | 將每個文件作為單獨的模塊（與“ts.transpileModule”類似）。
`--jsx`                                 | `string`  | `"Preserve"`             | 在`.tsx`文件裡支持JSX：`"React"`或`"Preserve"`。查看[JSX](./JSX.md)。
`--jsxFactory`                          | `string`  | `"React.createElement"`  | 指定生成目標為react JSX時，使用的JSX工廠函數，比如`React.createElement`或`h`。
`--lib`                                 | `string[]`|                          | 編譯過程中需要引入的庫文件的列表。<br/>可能的值為：  <br/>► `ES5` <br/>► `ES6` <br/>► `ES2015` <br/>► `ES7` <br/>► `ES2016` <br/>► `ES2017`  <br/>► `ES2018` <br/>► `ESNext` <br/>► `DOM` <br/>► `DOM.Iterable` <br/>► `WebWorker` <br/>► `ScriptHost` <br/>► `ES2015.Core` <br/>► `ES2015.Collection` <br/>► `ES2015.Generator` <br/>► `ES2015.Iterable` <br/>► `ES2015.Promise` <br/>► `ES2015.Proxy` <br/>► `ES2015.Reflect` <br/>► `ES2015.Symbol` <br/>► `ES2015.Symbol.WellKnown` <br/>► `ES2016.Array.Include` <br/>► `ES2017.object` <br/>► `ES2017.Intl` <br/>► `ES2017.SharedMemory` <br/>► `ES2017.String` <br/>► `ES2017.TypedArrays` <br/>► `ES2018.Intl` <br/>► `ES2018.Promise` <br/>► `ES2018.RegExp` <br/>► `ESNext.AsyncIterable` <br/>► `ESNext.Array` <br/>► `ESNext.Intl` <br/>► `ESNext.Symbol` <br/><br/> 注意：如果`--lib`沒有指定默認注入的庫的列表。默認注入的庫為：<br/> ► 針對於`--target ES5`：`DOM，ES5，ScriptHost`<br/>  ► 針對於`--target ES6`：`DOM，ES6，DOM.Iterable，ScriptHost`
`--listEmittedFiles`                    | `boolean` | `false`                  | 打印出編譯後生成文件的名字。
`--listFiles`                           | `boolean` | `false`                  | 編譯過程中打印文件名。
`--locale`                              | `string`  | *(platform specific)*    | 顯示錯誤信息時使用的語言，比如：en-us。
`--mapRoot`                             | `string`  |                          | 為調試器指定指定sourcemap文件的路徑，而不是使用生成時的路徑。當`.map`文件是在運行時指定的，並不同於`js`文件的地址時使用這個標記。指定的路徑會嵌入到`sourceMap`裡告訴調試器到哪裡去找它們。
`--maxNodeModuleJsDepth`                | `number`  | `0`                      | node_modules依賴的最大搜索深度並加載JavaScript文件。僅適用於`--allowJs`。
`--module`<br/>`-m`                     | `string`  | `target === "ES6" ? "ES6" : "commonjs"`                                  | 指定生成哪個模塊系統代碼：`"None"`，`"CommonJS"`，`"AMD"`，`"System"`，`"UMD"`，`"ES6"`或`"ES2015"`。<br/>► 只有`"AMD"`和`"System"`能和`--outFile`一起使用。<br/>►`"ES6"`和`"ES2015"`可使用在目標輸出為`"ES5"`或更低的情況下。
`--moduleResolution`                    | `string`  | `module === "AMD" or "System" or "ES6" ? "Classic" : "Node"`              | 決定如何處理模塊。或者是`"Node"`對於Node.js/io.js，或者是`"Classic"`（默認）。查看[模塊解析](./Module Resolution.md)瞭解詳情。
`--newLine`                             | `string`  | *(platform specific)*    | 當生成文件時指定行結束符：`"crlf"`（windows）或`"lf"`（unix）。
`--noEmit`                              | `boolean` | `false`                  | 不生成輸出文件。
`--noEmitHelpers`                       | `boolean` | `false`                  | 不在輸出文件中生成用戶自定義的幫助函數代碼，如`__extends`。
`--noEmitOnError`                       | `boolean` | `false`                  | 報錯時不生成輸出文件。
`--noErrorTruncation`                   | `boolean` | `false`                  | 不截短錯誤消息。
`--noFallthroughCasesInSwitch`          | `boolean` | `false`                  | 報告switch語句的fallthrough錯誤。（即，不允許switch的case語句貫穿）
`--noImplicitAny`                       | `boolean` | `false`                  | 在表達式和聲明上有隱含的`any`類型時報錯。
`--noImplicitReturns`                   | `boolean` | `false`                  | 不是函數的所有返回路徑都有返回值時報錯。
`--noImplicitThis`                      | `boolean` | `false`                  | 當`this`表達式的值為`any`類型的時候，生成一個錯誤。
`--noImplicitUseStrict`                 | `boolean` | `false`                  | 模塊輸出中不包含`"use strict"`指令。
`--noLib`                               | `boolean` | `false`                  | 不包含默認的庫文件（`lib.d.ts`）。
`--noResolve`                           | `boolean` | `false`                  | 不把`/// <reference``>`或模塊導入的文件加到編譯文件列表。
`--noStrictGenericChecks`               | `boolean` | `false`                  | 禁用在函數類型裡對泛型簽名進行嚴格檢查。
`--noUnusedLocals`                      | `boolean` | `false`                  | 若有未使用的局部變量則拋錯。
`--noUnusedParameters`                  | `boolean` | `false`                  | 若有未使用的參數則拋錯。
~~`--out`~~                             | `string`  |                          | 棄用。使用 `--outFile` 代替。
`--outDir`                              | `string`  |                          | 重定向輸出目錄。
`--outFile`                             | `string`  |                          | 將輸出文件合併為一個文件。合併的順序是根據傳入編譯器的文件順序和`///<reference``>`和`import`的文件順序決定的。查看輸出文件順序文件瞭解詳情。
`paths`<sup>[2]</sup>                   | `Object`  |                          | 模塊名到基於`baseUrl`的路徑映射的列表。查看[模塊解析文檔](./Module Resolution.md#path-mapping)瞭解詳情。
`--preserveConstEnums`                  | `boolean` | `false`                  | 保留`const`和`enum`聲明。查看[const enums documentation](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md#94-constant-enum-declarations)瞭解詳情。
`--preserveSymlinks`                    | `boolean` | `false`                  | 不把符號鏈接解析為其真實路徑；將符號鏈接文件視為真正的文件。
`--preserveWatchOutput`                 | `boolean` | `false`                  | 保留watch模式下過時的控制台輸出。
`--pretty`<sup>[1]</sup>                | `boolean` | `false`                  | 給錯誤和消息設置樣式，使用顏色和上下文。
`--project`<br/>`-p`                    | `string`  |                          | 編譯指定目錄下的項目。這個目錄應該包含一個`tsconfig.json`文件來管理編譯。查看[tsconfig.json](./tsconfig.json.md)文檔瞭解更多信息。
`--reactNamespace`                      | `string`  | `"React"`                | 當目標為生成`"react"` JSX時，指定`createElement`和`__spread`的調用對象
`--removeComments`                      | `boolean` | `false`                  | 刪除所有註釋，除了以`/!*`開頭的版權信息。
`--rootDir`                             | `string`  | *(common root directory is computed from the list of input files)*   | 僅用來控制輸出的目錄結構`--outDir`。
`rootDirs`<sup>[2]</sup>                | `string[]`|                          | <i>根（root）</i>文件夾列表，表示運行時組合工程結構的內容。查看[模塊解析文檔](./Module Resolution.md#virtual-directories-with-rootdirs)瞭解詳情。
`--showConfig`                          | `boolean` | `false`                  | 不真正執行build，而是顯示build使用的配置文件信息。
`--skipDefaultLibCheck`                 | `boolean` | `false`                  | 忽略[庫的默認聲明文件](./Triple-Slash Directives.md#-reference-no-default-libtrue)的類型檢查。
`--skipLibCheck`                        | `boolean` | `false`                  | 忽略所有的聲明文件（`*.d.ts`）的類型檢查。
`--sourceMap`                           | `boolean` | `false`                  | 生成相應的`.map`文件。
`--sourceRoot`                          | `string`  |                          | 指定TypeScript源文件的路徑，以便調試器定位。當TypeScript文件的位置是在運行時指定時使用此標記。路徑信息會被加到`sourceMap`裡。
`--strict`                              | `boolean` | `false`                  | 啟用所有嚴格檢查選項。 <br/>包含`--noImplicitAny`, `--noImplicitThis`, `--alwaysStrict`, `--strictBindCallApply`, `--strictNullChecks`, `--strictFunctionTypes`和`--strictPropertyInitialization`.
`--strictFunctionTypes`                 | `boolean` | `false`                  | 禁用函數參數雙向協變檢查。
`--strictPropertyInitialization`        | `boolean` | `false`                  | 確保類的非`undefined`屬性已經在構造函數裡初始化。若要令此選項生效，需要同時啟用`--strictNullChecks`。
`--strictNullChecks`                    | `boolean` | `false`                  | 在嚴格的`null`檢查模式下，`null`和`undefined`值不包含在任何類型裡，只允許用它們自己和`any`來賦值（有個例外，`undefined`可以賦值到`void`）。
`--stripInternal`<sup>[1]</sup>         | `boolean` | `false`                  | 不對具有`/** @internal */` JSDoc註解的代碼生成代碼。
`--suppressExcessPropertyErrors`<sup>[1]</sup> | `boolean` | `false`           | 阻止對對象字面量的額外屬性檢查。
`--suppressImplicitAnyIndexErrors`      | `boolean` | `false`                  | 阻止`--noImplicitAny`對缺少索引簽名的索引對象報錯。查看[issue #1232](https://github.com/Microsoft/TypeScript/issues/1232#issuecomment-64510362)瞭解詳情。
`--target`<br/>`-t`                     | `string`  | `"ES3"`                  | 指定ECMAScript目標版本`"ES3"`（默認），`"ES5"`，`"ES6"`/`"ES2015"`，`"ES2016"`，`"ES2017"`或`"ESNext"`。<br/><br/> 注意：`"ESNext"`最新的生成目標列表為[ES proposed features](https://github.com/tc39/proposals)
`--traceResolution`                     | `boolean` | `false`                  | 生成模塊解析日誌信息
`--types`                               | `string[]`|                          | 要包含的類型聲明文件名列表。查看[@types，--typeRoots和--types](./tsconfig.json.md#types-typeroots-and-types)章節瞭解詳細信息。
`--typeRoots`                           | `string[]`|                          | 要包含的類型聲明文件路徑列表。查看[@types，--typeRoots和--types](./tsconfig.json.md#types-typeroots-and-types)章節瞭解詳細信息。
`--version`<br/>`-v`                    |           |                          | 打印編譯器版本號。
`--watch`<br/>`-w`                      |           |                          | 在監視模式下運行編譯器。會監視輸出文件，在它們改變時重新編譯。監視文件和目錄的具體實現可以通過環境變量進行配置。詳情請看[配置 Watch](./Configuring%20Watch.md)。

* <sup>[1]</sup> 這些選項是試驗性的。
* <sup>[2]</sup> 這些選項只能在`tsconfig.json`裡使用，不能在命令行使用。

## 相關信息

* 在[`tsconfig.json`](./tsconfig.json.md)文件裡設置編譯器選項。
* 在[MSBuild工程](./Compiler%20Options%20in%20MSBuild.md)裡設置編譯器選項。
