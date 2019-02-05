三斜線指令是包含單個XML標籤的單行註釋。
註釋的內容會做為編譯器指令使用。

三斜線指令*僅*可放在包含它的文件的最頂端。
一個三斜線指令的前面只能出現單行或多行註釋，這包括其它的三斜線指令。
如果它們出現在一個語句或聲明之後，那麼它們會被當做普通的單行註釋，並且不具有特殊的涵義。

## `/// <reference path="..." />`

`/// <reference path="..." />`指令是三斜線指令中最常見的一種。
它用於聲明文件間的*依賴*。

三斜線引用告訴編譯器在編譯過程中要引入的額外的文件。

當使用`--out`或`--outFile`時，它也可以做為調整輸出內容順序的一種方法。
文件在輸出文件內容中的位置與經過預處理後的輸入順序一致。

### 預處理輸入文件

編譯器會對輸入文件進行預處理來解析所有三斜線引用指令。
在這個過程中，額外的文件會加到編譯過程中。

這個過程會以一些*根文件*開始；
它們是在命令行中指定的文件或是在`tsconfig.json`中的`"files"`列表裡的文件。
這些根文件按指定的順序進行預處理。
在一個文件被加入列表前，它包含的所有三斜線引用都要被處理，還有它們包含的目標。
三斜線引用以它們在文件裡出現的順序，使用深度優先的方式解析。

一個三斜線引用路徑是相對於包含它的文件的，如果不是根文件。

### 錯誤

引用不存在的文件會報錯。
一個文件用三斜線指令引用自己會報錯。

### 使用 `--noResolve`

如果指定了`--noResolve`編譯選項，三斜線引用會被忽略；它們不會增加新文件，也不會改變給定文件的順序。

## `/// <reference types="..." />`

與`/// <reference path="..." />`指令相似，這個指令是用來聲明*依賴*的；
一個`/// <reference types="..." />`指令則聲明了對某個包的依賴。

對這些包的名字的解析與在`import`語句裡對模塊名的解析類似。
可以簡單地把三斜線類型引用指令當做`import`聲明的包。

例如，把`/// <reference types="node" />`引入到聲明文件，表明這個文件使用了`@types/node/index.d.ts`裡面聲明的名字；
並且，這個包需要在編譯階段與聲明文件一起被包含進來。

僅當在你需要寫一個`d.ts`文件時才使用這個指令。

對於那些在編譯階段生成的聲明文件，編譯器會自動地添加`/// <reference types="..." />`；
*當且僅當*結果文件中使用了引用的包裡的聲明時才會在生成的聲明文件裡添加`/// <reference types="..." />`語句。

若要在`.ts`文件裡聲明一個對`@types`包的依賴，使用`--types`命令行選項或在`tsconfig.json`裡指定。
查看[在`tsconfig.json`裡使用`@types`，`typeRoots`和`types`](./tsconfig.json.md#types-typeroots-and-types)瞭解詳情。

## `/// <reference no-default-lib="true"/>`

這個指令把一個文件標記成*默認庫*。
你會在`lib.d.ts`文件和它不同的變體的頂端看到這個註釋。

這個指令告訴編譯器在編譯過程中*不要*包含這個默認庫（比如，`lib.d.ts`）。
這與在命令行上使用`--noLib`相似。

還要注意，當傳遞了`--skipDefaultLibCheck`時，編譯器只會忽略檢查帶有`/// <reference no-default-lib="true"/>`的文件。

## `/// <amd-module />`

默認情況下生成的AMD模塊都是匿名的。
但是，當一些工具需要處理生成的模塊時會產生問題，比如`r.js`。

`amd-module`指令允許給編譯器傳入一個可選的模塊名：

##### amdModule.ts

```ts
///<amd-module name='NamedModule'/>
export class C {
}
```

這會將`NamedModule`傳入到AMD `define`函數裡：

##### amdModule.js

```js
define("NamedModule", ["require", "exports"], function (require, exports) {
    var C = (function () {
        function C() {
        }
        return C;
    })();
    exports.C = C;
});
```

## `/// <amd-dependency />`

> **注意**：這個指令被廢棄了。使用`import "moduleName";`語句代替。

`/// <amd-dependency path="x" />`告訴編譯器有一個非TypeScript模塊依賴需要被注入，做為目標模塊`require`調用的一部分。

`amd-dependency`指令也可以帶一個可選的`name`屬性；它允許我們為amd-dependency傳入一個可選名字：

```ts
/// <amd-dependency path="legacy/moduleA" name="moduleA"/>
declare var moduleA:MyType
moduleA.callStuff()
```

生成的JavaScript代碼：

```js
define(["require", "exports", "legacy/moduleA"], function (require, exports, moduleA) {
    moduleA.callStuff()
});
```
