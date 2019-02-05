> **關於術語的一點說明:**
請務必注意一點，TypeScript 1.5里術語名已經發生了變化。
“內部模塊”現在稱做“命名空間”。
“外部模塊”現在則簡稱為“模塊”，這是為了與[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/)裡的術語保持一致，(也就是說 `module X {` 相當於現在推薦的寫法 `namespace X {`)。

# 介紹

這篇文章將概括介紹在TypeScript裡使用模塊與命名空間來組織代碼的方法。
我們也會談及命名空間和模塊的高級使用場景，和在使用它們的過程中常見的陷阱。

查看[模塊](./Modules.md)章節瞭解關於模塊的更多信息。
查看[命名空間](./Namespaces.md)章節瞭解關於命名空間的更多信息。

# 使用命名空間

命名空間是位於全局命名空間下的一個普通的帶有名字的JavaScript對象。
這令命名空間十分容易使用。
它們可以在多文件中同時使用，並通過`--outFile`結合在一起。
命名空間是幫你組織Web應用不錯的方式，你可以把所有依賴都放在HTML頁面的`<script>`標籤裡。

但就像其它的全局命名空間污染一樣，它很難去識別組件之間的依賴關係，尤其是在大型的應用中。

# 使用模塊

像命名空間一樣，模塊可以包含代碼和聲明。
不同的是模塊可以*聲明*它的依賴。

模塊會把依賴添加到模塊加載器上（例如CommonJs / Require.js）。
對於小型的JS應用來說可能沒必要，但是對於大型應用，這一點點的花費會帶來長久的模塊化和可維護性上的便利。
模塊也提供了更好的代碼重用，更強的封閉性以及更好的使用工具進行優化。

對於Node.js應用來說，模塊是默認並推薦的組織代碼的方式。

從ECMAScript 2015開始，模塊成為了語言內置的部分，應該會被所有正常的解釋引擎所支持。
因此，對於新項目來說推薦使用模塊做為組織代碼的方式。

# 命名空間和模塊的陷阱

這部分我們會描述常見的命名空間和模塊的使用陷阱和如何去避免它們。

## 對模塊使用`/// <reference>`

一個常見的錯誤是使用`/// <reference>`引用模塊文件，應該使用`import`。
要理解這之間的區別，我們首先應該弄清編譯器是如何根據`import`路徑（例如，`import x from "...";`或`import x = require("...")`裡面的`...`，等等）來定位模塊的類型信息的。

編譯器首先嘗試去查找相應路徑下的`.ts`，`.tsx`再或者`.d.ts`。
如果這些文件都找不到，編譯器會查找*外部模塊聲明*。
回想一下，它們是在`.d.ts`文件裡聲明的。

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

這裡的引用標籤指定了外來模塊的位置。
這就是一些TypeScript例子中引用`node.d.ts`的方法。

## 不必要的命名空間

如果你想把命名空間轉換為模塊，它可能會像下面這個文件一件：

* `shapes.ts`

```ts
export namespace Shapes {
    export class Triangle { /* ... */ }
    export class Square { /* ... */ }
}
```

頂層的模塊`Shapes`包裹了`Triangle`和`Square`。
對於使用它的人來說這是令人迷惑和討厭的：

* `shapeConsumer.ts`

```ts
import * as shapes from "./shapes";
let t = new shapes.Shapes.Triangle(); // shapes.Shapes?
```

TypeScript裡模塊的一個特點是不同的模塊永遠也不會在相同的作用域內使用相同的名字。
因為使用模塊的人會為它們命名，所以完全沒有必要把導出的符號包裹在一個命名空間裡。

再次重申，不應該對模塊使用命名空間，使用命名空間是為了提供邏輯分組和避免命名衝突。
模塊文件本身已經是一個邏輯分組，並且它的名字是由導入這個模塊的代碼指定，所以沒有必要為導出的對象增加額外的模塊層。

下面是改進的例子：

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

## 模塊的取捨

就像每個JS文件對應一個模塊一樣，TypeScript裡模塊文件與生成的JS文件也是一一對應的。
這會產生一種影響，根據你指定的目標模塊系統的不同，你可能無法連接多個模塊源文件。
例如當目標模塊系統為`commonjs`或`umd`時，無法使用`outFile`選項，但是在TypeScript 1.8以上的版本[能夠](./release%20notes/TypeScript%201.8.md#concatenate-amd-and-system-modules-with---outfile)使用`outFile`當目標為`amd`或`system`。
