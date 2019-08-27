> **關於術語的一點說明:**
請務必注意一點，TypeScript 1.5裡的術語名稱已經發生了變化。
「內部模組」現在稱做「命名空間」。
「外部模組」現在則簡稱為「模組」，這是為了與[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/)裡的術語保持一致，(也就是說 `module X {` 相當於現在推薦的寫法 `namespace X {`)。

# 介紹

這篇文章將概括介紹在TypeScript裡使用模組與命名空間來組織程式碼的方法。
我們也會談及命名空間和模組的高級使用場景，和在使用它們的程序中常見的陷阱。

查看[模組](./Modules.md)章節瞭解關於模組的更多訊息。
查看[命名空間](./Namespaces.md)章節瞭解關於命名空間的更多訊息。

# 使用命名空間

命名空間是位於全局命名空間下的一個普通的帶有名字的JavaScript物件。
這令命名空間十分容易使用。
它們可以在多檔案中同時使用，並透過`--outFile`結合在一起。
命名空間是幫您組織Web應用不錯的方式，您可以把所有依賴都放在HTML頁面的`<script>`標籤裡。

但就像其它的全局命名空間污染一樣，它很難去識別組件之間的依賴關係，尤其是在大型的應用中。

# 使用模組

像命名空間一樣，模組可以包含程式碼和宣告。
不同的是模組可以*宣告*它的依賴。

模組會把依賴加入到模組載入器上(例如CommonJs / Require.js)。
對於小型的JS應用來說可能沒必要，但是對於大型應用，這一點點的花費會帶來長久的模組化和可維護性上的便利。
模組也提供了更好的程式碼重用，更強的封閉性以及更好的使用工具進行優化。

對於Node.js應用來說，模組是預設並推薦的組織程式碼的方式。

從ECMAScript 2015開始，模組成為了語言內建的部分，應該會被所有正常的解釋引擎所支援。
因此，對於新專案來說推薦使用模組做為組織程式碼的方式。

# 命名空間和模組的陷阱

這部分我們會描述常見的命名空間和模組的使用陷阱和如何去避免它們。

## 對模組使用`/// <reference>`

一個常見的錯誤是使用`/// <reference>`引用模組檔案，應該使用`import`。
要理解這之間的區別，我們首先應該弄清編譯器是如何根據`import`路徑(例如，`import x from "...";`或`import x = require("...")`裡面的`...`，等等)來定位模組的類型訊息的。

編譯器首先嘗試去尋找對應路徑下的`.ts`，`.tsx`再或者`.d.ts`。
如果這些檔案都找不到，編譯器會尋找*外部模組宣告*。
回想一下，它們是在`.d.ts`檔案裡宣告的。

* `myModules.d.ts`

```ts
// In a .d.ts file or .ts file that is not a module:
declare module "SomeModule" {
    export function fn(): string;
}
```

* `myOtherModule.ts`

```ts
/// <reference path="myModules.d.ts" />
import * as m from "SomeModule";
```

這裡的引用標籤指定了外來模組的位置。
這就是一些TypeScript範例中引用`node.d.ts`的方法。

## 不必要的命名空間

如果您想把命名空間轉換為模組，它可能會像下面這個檔案一件：

* `shapes.ts`

```ts
export namespace Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```

頂層的模組`Shapes`包裹了`Triangle`和`Square`。
對於使用它的人來說這是令人迷惑和討厭的：

* `shapeConsumer.ts`

```ts
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

TypeScript裡模組的一個特點是不同的模組永遠也不會在相同的作用域內使用相同的名字。
因為使用模組的人會為它們命名，所以完全沒有必要把導出的符號包裹在一個命名空間裡。

再次重申，不應該對模組使用命名空間，使用命名空間是為了提供邏輯分組和避免命名衝突。
模組檔案本身已經是一個邏輯分組，並且它的名字是由導入這個模組的程式碼指定，所以沒有必要為導出的物件增加額外的模組層。

下面是改進的範例：

* `shapes.ts`

```ts
export class Triangle { /* ... */ }
export class Square { /* ... */ }
```

* `shapeConsumer.ts`

```ts
import * as shapes from "./shapes";
let t = new shapes.Triangle();
```

## 模組的取捨

就像每個JS檔案對應一個模組一樣，TypeScript裡模組檔案與產生的JS檔案也是一一對應的。
這會產生一種影響，根據您指定的目標模組系統的不同，您可能無法連接多個模組源檔案。
例如當目標模組系統為`commonjs`或`umd`時，無法使用`outFile`選項，但是在TypeScript 1.8以上的版本[能夠](./release%20notes/TypeScript%201.8.md#concatenate-amd-and-system-modules-with---outfile)使用`outFile`當目標為`amd`或`system`。
