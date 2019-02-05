> 這節假設你已經瞭解了模塊的一些基本知識
請閱讀[模塊](./Modules.md)文檔瞭解更多信息。

*模塊解析*是指編譯器在查找導入模塊內容時所遵循的流程。
假設有一個導入語句`import { a } from "moduleA"`;
為了去檢查任何對`a`的使用，編譯器需要準確的知道它表示什麼，並且需要檢查它的定義`moduleA`。

這時候，編譯器會有個疑問“`moduleA`的結構是怎樣的？”
這聽上去很簡單，但`moduleA`可能在你寫的某個`.ts`/`.tsx`文件裡或者在你的代碼所依賴的`.d.ts`裡。

首先，編譯器會嘗試定位表示導入模塊的文件。
編譯器會遵循以下二種策略之一：[Classic](#classic)或[Node](#node)。
這些策略會告訴編譯器到*哪裡*去查找`moduleA`。

如果上面的解析失敗了並且模塊名是非相對的（且是在`"moduleA"`的情況下），編譯器會嘗試定位一個[外部模塊聲明](./Modules.md#ambient-modules)。
我們接下來會講到非相對導入。

最後，如果編譯器還是不能解析這個模塊，它會記錄一個錯誤。
在這種情況下，錯誤可能為`error TS2307: Cannot find module 'moduleA'.`

## 相對 vs. 非相對模塊導入

根據模塊引用是相對的還是非相對的，模塊導入會以不同的方式解析。

*相對導入*是以`/`，`./`或`../`開頭的。
下面是一些例子：

* `import Entry from "./components/Entry";`
* `import { DefaultHeaders } from "../constants/http";`
* `import "/mod";`

所有其它形式的導入被當作*非相對*的。
下面是一些例子：

* `import * as $ from "jQuery";`
* `import { Component } from "@angular/core";`

相對導入在解析時是相對於導入它的文件，並且*不能*解析為一個外部模塊聲明。
你應該為你自己寫的模塊使用相對導入，這樣能確保它們在運行時的相對位置。

非相對模塊的導入可以相對於`baseUrl`或通過下文會講到的路徑映射來進行解析。
它們還可以被解析成[外部模塊聲明](./Modules.md#ambient-modules)。
使用非相對路徑來導入你的外部依賴。

## 模塊解析策略

共有兩種可用的模塊解析策略：[Node](#node)和[Classic](#classic)。
你可以使用`--moduleResolution`標記來指定使用哪種模塊解析策略。
若未指定，那麼在使用了`--module AMD | System | ES2015`時的默認值為[Classic](#classic)，其它情況時則為[Node](#node)。

### Classic

這種策略在以前是TypeScript默認的解析策略。
現在，它存在的理由主要是為了向後兼容。

相對導入的模塊是相對於導入它的文件進行解析的。
因此`/root/src/folder/A.ts`文件裡的`import { b } from "./moduleB"`會使用下面的查找流程：

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`

對於非相對模塊的導入，編譯器則會從包含導入文件的目錄開始依次向上級目錄遍歷，嘗試定位匹配的聲明文件。

比如：

有一個對`moduleB`的非相對導入`import { b } from "moduleB"`，它是在`/root/src/folder/A.ts`文件裡，會以如下的方式來定位`"moduleB"`：

1. `/root/src/folder/moduleB.ts`
2. `/root/src/folder/moduleB.d.ts`
3. `/root/src/moduleB.ts`
4. `/root/src/moduleB.d.ts`
5. `/root/moduleB.ts`
6. `/root/moduleB.d.ts`
7. `/moduleB.ts`
8. `/moduleB.d.ts`

### Node

這個解析策略試圖在運行時模仿[Node.js](https://nodejs.org/)模塊解析機制。
完整的Node.js解析算法可以在[Node.js module documentation](https://nodejs.org/api/modules.html#modules_all_together)找到。

#### Node.js如何解析模塊

為了理解TypeScript編譯依照的解析步驟，先弄明白Node.js模塊是非常重要的。
通常，在Node.js裡導入是通過`require`函數調用進行的。
Node.js會根據`require`的是相對路徑還是非相對路徑做出不同的行為。

相對路徑很簡單。
例如，假設有一個文件路徑為`/root/src/moduleA.js`，包含了一個導入`var x = require("./moduleB");`
Node.js以下面的順序解析這個導入：

1. 檢查`/root/src/moduleB.js`文件是否存在。

2. 檢查`/root/src/moduleB`目錄是否包含一個`package.json`文件，且`package.json`文件指定了一個`"main"`模塊。
   在我們的例子裡，如果Node.js發現文件`/root/src/moduleB/package.json`包含了`{ "main": "lib/mainModule.js" }`，那麼Node.js會引用`/root/src/moduleB/lib/mainModule.js`。

3. 檢查`/root/src/moduleB`目錄是否包含一個`index.js`文件。
   這個文件會被隱式地當作那個文件夾下的"main"模塊。

你可以閱讀Node.js文檔瞭解更多詳細信息：[file modules](https://nodejs.org/api/modules.html#modules_file_modules) 和 [folder modules](https://nodejs.org/api/modules.html#modules_folders_as_modules)。

但是，[非相對模塊名](#relative-vs-non-relative-module-imports)的解析是個完全不同的過程。
Node會在一個特殊的文件夾`node_modules`裡查找你的模塊。
`node_modules`可能與當前文件在同一級目錄下，或者在上層目錄裡。
Node會向上級目錄遍歷，查找每個`node_modules`直到它找到要加載的模塊。

還是用上面例子，但假設`/root/src/moduleA.js`裡使用的是非相對路徑導入`var x = require("moduleB");`。
Node則會以下面的順序去解析`moduleB`，直到有一個匹配上。

1. `/root/src/node_modules/moduleB.js`
2. `/root/src/node_modules/moduleB/package.json` (如果指定了`"main"`屬性)
3. `/root/src/node_modules/moduleB/index.js`
   <br /><br />
4. `/root/node_modules/moduleB.js`
5. `/root/node_modules/moduleB/package.json` (如果指定了`"main"`屬性)
6. `/root/node_modules/moduleB/index.js`
   <br /><br />
7. `/node_modules/moduleB.js`
8. `/node_modules/moduleB/package.json` (如果指定了`"main"`屬性)
9. `/node_modules/moduleB/index.js`

注意Node.js在步驟（4）和（7）會向上跳一級目錄。

你可以閱讀Node.js文檔瞭解更多詳細信息：[loading modules from `node_modules`](https://nodejs.org/api/modules.html#modules_loading_from_node_modules_folders)。

#### TypeScript如何解析模塊

TypeScript是模仿Node.js運行時的解析策略來在編譯階段定位模塊定義文件。
因此，TypeScript在Node解析邏輯基礎上增加了TypeScript源文件的擴展名（`.ts`，`.tsx`和`.d.ts`）。
同時，TypeScript在`package.json`裡使用字段`"types"`來表示類似`"main"`的意義 - 編譯器會使用它來找到要使用的"main"定義文件。

比如，有一個導入語句`import { b } from "./moduleB"`在`/root/src/moduleA.ts`裡，會以下面的流程來定位`"./moduleB"`：

1. `/root/src/moduleB.ts`
2. `/root/src/moduleB.tsx`
3. `/root/src/moduleB.d.ts`
4. `/root/src/moduleB/package.json` (如果指定了`"types"`屬性)
5. `/root/src/moduleB/index.ts`
6. `/root/src/moduleB/index.tsx`
7. `/root/src/moduleB/index.d.ts`

回想一下Node.js先查找`moduleB.js`文件，然後是合適的`package.json`，再之後是`index.js`。

類似地，非相對的導入會遵循Node.js的解析邏輯，首先查找文件，然後是合適的文件夾。
因此`/root/src/moduleA.ts`文件裡的`import { b } from "moduleB"`會以下面的查找順序解析：

1. `/root/src/node_modules/moduleB.ts`
2. `/root/src/node_modules/moduleB.tsx`
3. `/root/src/node_modules/moduleB.d.ts`
4. `/root/src/node_modules/moduleB/package.json` (如果指定了`"types"`屬性)
5. `/root/src/node_modules/@types/moduleB.d.ts`
6. `/root/src/node_modules/moduleB/index.ts`
7. `/root/src/node_modules/moduleB/index.tsx`
8. `/root/src/node_modules/moduleB/index.d.ts`
   <br /><br />
9. `/root/node_modules/moduleB.ts`
10. `/root/node_modules/moduleB.tsx`
11. `/root/node_modules/moduleB.d.ts`
12. `/root/node_modules/moduleB/package.json` (如果指定了`"types"`屬性)
13. `/root/node_modules/@types/moduleB.d.ts`
14. `/root/node_modules/moduleB/index.ts`
15. `/root/node_modules/moduleB/index.tsx`
16. `/root/node_modules/moduleB/index.d.ts`
    <br /><br />
17. `/node_modules/moduleB.ts`
18. `/node_modules/moduleB.tsx`
19. `/node_modules/moduleB.d.ts`
20. `/node_modules/moduleB/package.json` (如果指定了`"types"`屬性)
21. `/node_modules/@types/moduleB.d.ts`
22. `/node_modules/moduleB/index.ts`
23. `/node_modules/moduleB/index.tsx`
24. `/node_modules/moduleB/index.d.ts`

不要被這裡步驟的數量嚇到 - TypeScript只是在步驟（9）和（17）向上跳了兩次目錄。
這並不比Node.js裡的流程複雜。

## 附加的模塊解析標記

有時工程源碼結構與輸出結構不同。
通常是要經過一系統的構建步驟最後生成輸出。
它們包括將`.ts`編譯成`.js`，將不同位置的依賴拷貝至一個輸出位置。
最終結果就是運行時的模塊名與包含它們聲明的源文件裡的模塊名不同。
或者最終輸出文件裡的模塊路徑與編譯時的源文件路徑不同了。

TypeScript編譯器有一些額外的標記用來*通知*編譯器在源碼編譯成最終輸出的過程中都發生了哪個轉換。

有一點要特別注意的是編譯器*不會*進行這些轉換操作；
它只是利用這些信息來指導模塊的導入。

### Base URL

在利用AMD模塊加載器的應用裡使用`baseUrl`是常見做法，它要求在運行時模塊都被放到了一個文件夾裡。
這些模塊的源碼可以在不同的目錄下，但是構建腳本會將它們集中到一起。

設置`baseUrl`來告訴編譯器到哪裡去查找模塊。
所有非相對模塊導入都會被當做相對於`baseUrl`。

*baseUrl*的值由以下兩者之一決定：

* 命令行中*baseUrl*的值（如果給定的路徑是相對的，那麼將相對於當前路徑進行計算）
* ‘tsconfig.json’裡的*baseUrl*屬性（如果給定的路徑是相對的，那麼將相對於‘tsconfig.json’路徑進行計算）

注意相對模塊的導入不會被設置的`baseUrl`所影響，因為它們總是相對於導入它們的文件。

閱讀更多關於`baseUrl`的信息[RequireJS](http://requirejs.org/docs/api.html#config-baseUrl)和[SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/config-api.md#baseurl)。

### 路徑映射

有時模塊不是直接放在*baseUrl*下面。
比如，充分`"jquery"`模塊地導入，在運行時可能被解釋為`"node_modules/jquery/dist/jquery.slim.min.js"`。
加載器使用映射配置來將模塊名映射到運行時的文件，查看[RequireJs documentation](http://requirejs.org/docs/api.html#config-paths)和[SystemJS documentation](https://github.com/systemjs/systemjs/blob/master/docs/config-api.md#paths)。

TypeScript編譯器通過使用`tsconfig.json`文件裡的`"paths"`來支持這樣的聲明映射。
下面是一個如何指定`jquery`的`"paths"`的例子。

```json
{
  "compilerOptions": {
    "baseUrl": ".", // This must be specified if "paths" is.
    "paths": {
      "jquery": ["node_modules/jquery/dist/jquery"] // 此處映射是相對於"baseUrl"
    }
  }
}
```

請注意`"paths"`是相對於`"baseUrl"`進行解析。
如果`"baseUrl"`被設置成了除`"."`外的其它值，比如`tsconfig.json`所在的目錄，那麼映射必須要做相應的改變。
如果你在上例中設置了`"baseUrl": "./src"`，那麼jquery應該映射到`"../node_modules/jquery/dist/jquery"`。

通過`"paths"`我們還可以指定複雜的映射，包括指定多個回退位置。
假設在一個工程配置裡，有一些模塊位於一處，而其它的則在另個的位置。
構建過程會將它們集中至一處。
工程結構可能如下：

```tree
projectRoot
├── folder1
│   ├── file1.ts (imports 'folder1/file2' and 'folder2/file3')
│   └── file2.ts
├── generated
│   ├── folder1
│   └── folder2
│       └── file3.ts
└── tsconfig.json
```

相應的`tsconfig.json`文件如下：

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "*": [
        "*",
        "generated/*"
      ]
    }
  }
}
```

它告訴編譯器所有匹配`"*"`（所有的值）模式的模塊導入會在以下兩個位置查找：

 1. `"*"`： 表示名字不發生改變，所以映射為`<moduleName>` => `<baseUrl>/<moduleName>`
 2. `"generated/*"`表示模塊名添加了“generated”前綴，所以映射為`<moduleName>` => `<baseUrl>/generated/<moduleName>`

按照這個邏輯，編譯器將會如下嘗試解析這兩個導入：

* 導入'folder1/file2'
  1. 匹配'*'模式且通配符捕獲到整個名字。
  2. 嘗試列表裡的第一個替換：'*' -> `folder1/file2`。
  3. 替換結果為非相對名 - 與*baseUrl*合併 -> `projectRoot/folder1/file2.ts`。
  4. 文件存在。完成。
* 導入'folder2/file3'
  1. 匹配'*'模式且通配符捕獲到整個名字。
  2. 嘗試列表裡的第一個替換：'*' -> `folder2/file3`。
  3. 替換結果為非相對名 - 與*baseUrl*合併 -> `projectRoot/folder2/file3.ts`。
  4. 文件不存在，跳到第二個替換。
  5. 第二個替換：'generated/*' -> `generated/folder2/file3`。
  6. 替換結果為非相對名 - 與*baseUrl*合併 -> `projectRoot/generated/folder2/file3.ts`。
  7. 文件存在。完成。

### 利用`rootDirs`指定虛擬目錄

有時多個目錄下的工程源文件在編譯時會進行合併放在某個輸出目錄下。
這可以看做一些源目錄創建了一個“虛擬”目錄。

利用`rootDirs`，可以告訴編譯器生成這個虛擬目錄的*roots*；
因此編譯器可以在“虛擬”目錄下解析相對模塊導入，就*好像*它們被合併在了一起一樣。

比如，有下面的工程結構：

```tree
 src
 └── views
     └── view1.ts (imports './template1')
     └── view2.ts

 generated
 └── templates
         └── views
             └── template1.ts (imports './view2')
```

`src/views`裡的文件是用於控制UI的用戶代碼。
`generated/templates`是UI模版，在構建時通過模版生成器自動生成。
構建中的一步會將`/src/views`和`/generated/templates/views`的輸出拷貝到同一個目錄下。
在運行時，視圖可以假設它的模版與它同在一個目錄下，因此可以使用相對導入`"./template"`。

可以使用`"rootDirs"`來告訴編譯器。
`"rootDirs"`指定了一個*roots*列表，列表裡的內容會在運行時被合併。
因此，針對這個例子，`tsconfig.json`如下：

```json
{
  "compilerOptions": {
    "rootDirs": [
      "src/views",
      "generated/templates/views"
    ]
  }
}
```

每當編譯器在某一`rootDirs`的子目錄下發現了相對模塊導入，它就會嘗試從每一個`rootDirs`中導入。

`rootDirs`的靈活性不僅僅侷限於其指定了要在邏輯上合併的物理目錄列表。它提供的數組可以包含任意數量的任何名字的目錄，不論它們是否存在。這允許編譯器以類型安全的方式處理複雜捆綁(bundles)和運行時的特性，比如條件引入和工程特定的加載器插件。

設想這樣一個國際化的場景，構建工具自動插入特定的路徑記號來生成針對不同區域的捆綁，比如將`#{locale}`做為相對模塊路徑`./#{locale}/messages`的一部分。在這個假定的設置下，工具會枚舉支持的區域，將抽像的路徑映射成`./zh/messages`，`./de/messages`等。

假設每個模塊都會導出一個字符串的數組。比如`./zh/messages`可能包含：

```ts
export default [
    "您好嗎",
    "很高興認識你"
];
```

利用`rootDirs`我們可以讓編譯器瞭解這個映射關係，從而也允許編譯器能夠安全地解析`./#{locale}/messages`，就算這個目錄永遠都不存在。比如，使用下面的`tsconfig.json`：

```json
{
  "compilerOptions": {
    "rootDirs": [
      "src/zh",
      "src/de",
      "src/#{locale}"
    ]
  }
}
```

編譯器現在可以將`import messages from './#{locale}/messages'`解析為`import messages from './zh/messages'`用做工具支持的目的，並允許在開發時不必瞭解區域信息。

## 跟蹤模塊解析

如之前討論，編譯器在解析模塊時可能訪問當前文件夾外的文件。
這會導致很難診斷模塊為什麼沒有被解析，或解析到了錯誤的位置。
通過`--traceResolution`啟用編譯器的模塊解析跟蹤，它會告訴我們在模塊解析過程中發生了什麼。

假設我們有一個使用了`typescript`模塊的簡單應用。
`app.ts`裡有一個這樣的導入`import * as ts from "typescript"`。

```tree
│   tsconfig.json
├───node_modules
│   └───typescript
│       └───lib
│               typescript.d.ts
└───src
        app.ts
```

使用`--traceResolution`調用編譯器。

```shell
tsc --traceResolution
```

輸出結果如下：

```txt
======== Resolving module 'typescript' from 'src/app.ts'. ========
Module resolution kind is not specified, using 'NodeJs'.
Loading module 'typescript' from 'node_modules' folder.
File 'src/node_modules/typescript.ts' does not exist.
File 'src/node_modules/typescript.tsx' does not exist.
File 'src/node_modules/typescript.d.ts' does not exist.
File 'src/node_modules/typescript/package.json' does not exist.
File 'node_modules/typescript.ts' does not exist.
File 'node_modules/typescript.tsx' does not exist.
File 'node_modules/typescript.d.ts' does not exist.
Found 'package.json' at 'node_modules/typescript/package.json'.
'package.json' has 'types' field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.
File 'node_modules/typescript/lib/typescript.d.ts' exist - use it as a module resolution result.
======== Module name 'typescript' was successfully resolved to 'node_modules/typescript/lib/typescript.d.ts'. ========
```

#### 需要留意的地方

* 導入的名字及位置

 > ======== Resolving module **'typescript'** from **'src/app.ts'**. ========

* 編譯器使用的策略

 > Module resolution kind is not specified, using **'NodeJs'**.

* 從npm加載types

 > 'package.json' has **'types'** field './lib/typescript.d.ts' that references 'node_modules/typescript/lib/typescript.d.ts'.

* 最終結果

 > ======== Module name 'typescript' was **successfully resolved** to 'node_modules/typescript/lib/typescript.d.ts'. ========

## 使用`--noResolve`

正常來講編譯器會在開始編譯之前解析模塊導入。
每當它成功地解析了對一個文件`import`，這個文件被會加到一個文件列表裡，以供編譯器稍後處理。

`--noResolve`編譯選項告訴編譯器不要添加任何不是在命令行上傳入的文件到編譯列表。
編譯器仍然會嘗試解析模塊，但是只要沒有指定這個文件，那麼它就不會被包含在內。

比如

#### app.ts

```ts
import * as A from "moduleA" // OK, moduleA passed on the command-line
import * as B from "moduleB" // Error TS2307: Cannot find module 'moduleB'.
```

```shell
tsc app.ts moduleA.ts --noResolve
```

使用`--noResolve`編譯`app.ts`：

* 可能正確找到`moduleA`，因為它在命令行上指定了。
* 找不到`moduleB`，因為沒有在命令行上傳遞。

## 常見問題

### 為什麼在`exclude`列表裡的模塊還會被編譯器使用

`tsconfig.json`將文件夾轉變一個“工程”
如果不指定任何`“exclude”`或`“files”`，文件夾裡的所有文件包括`tsconfig.json`和所有的子目錄都會在編譯列表裡。
如果你想利用`“exclude”`排除某些文件，甚至你想指定所有要編譯的文件列表，請使用`“files”`。

有些是被`tsconfig.json`自動加入的。
它不會涉及到上面討論的模塊解析。
如果編譯器識別出一個文件是模塊導入目標，它就會加到編譯列表裡，不管它是否被排除了。

因此，要從編譯列表中排除一個文件，你需要在排除它的同時，還要排除所有對它進行`import`或使用了`/// <reference path="..." />`指令的文件。
