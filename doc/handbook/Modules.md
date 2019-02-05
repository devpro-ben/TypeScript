> **關於術語的一點說明:**
請務必注意一點，TypeScript 1.5里術語名已經發生了變化。
“內部模塊”現在稱做“命名空間”。
“外部模塊”現在則簡稱為“模塊”，這是為了與[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/)裡的術語保持一致，(也就是說 `module X {` 相當於現在推薦的寫法 `namespace X {`)。

# 介紹

從ECMAScript 2015開始，JavaScript引入了模塊的概念。TypeScript也沿用這個概念。

模塊在其自身的作用域裡執行，而不是在全局作用域裡；這意味著定義在一個模塊裡的變量，函數，類等等在模塊外部是不可見的，除非你明確地使用[`export`形式](#export)之一導出它們。
相反，如果想使用其它模塊導出的變量，函數，類，接口等的時候，你必須要導入它們，可以使用[`import`形式](#import)之一。

模塊是自聲明的；兩個模塊之間的關係是通過在文件級別上使用imports和exports建立的。

模塊使用模塊加載器去導入其它的模塊。
在運行時，模塊加載器的作用是在執行此模塊代碼前去查找並執行這個模塊的所有依賴。
大家最熟知的JavaScript模塊加載器是服務於Node.js的[CommonJS](https://en.wikipedia.org/wiki/CommonJS)和服務於Web應用的[Require.js](http://requirejs.org/)。

TypeScript與ECMAScript 2015一樣，任何包含頂級`import`或者`export`的文件都被當成一個模塊。
相反地，如果一個文件不帶有頂級的`import`或者`export`聲明，那麼它的內容被視為全局可見的（因此對模塊也是可見的）。

# <a name="export"></a>導出

## 導出聲明

任何聲明（比如變量，函數，類，類型別名或接口）都能夠通過添加`export`關鍵字來導出。

##### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

##### ZipCodeValidator.ts

```ts
export const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

## 導出語句

導出語句很便利，因為我們可能需要對導出的部分重命名，所以上面的例子可以這樣改寫：

```ts
class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export { ZipCodeValidator };
export { ZipCodeValidator as mainValidator };
```

## 重新導出

我們經常會去擴展其它模塊，並且只導出那個模塊的部分內容。
重新導出功能並不會在當前模塊導入那個模塊或定義一個新的局部變量。

##### ParseIntBasedZipCodeValidator.ts

```ts
export class ParseIntBasedZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && parseInt(s).toString() === s;
    }
}

// 導出原先的驗證器但做了重命名
export {ZipCodeValidator as RegExpBasedZipCodeValidator} from "./ZipCodeValidator";
```

或者一個模塊可以包裹多個模塊，並把他們導出的內容聯合在一起通過語法：`export * from "module"`。

##### AllValidators.ts

```ts
export * from "./StringValidator"; // exports interface StringValidator
export * from "./LettersOnlyValidator"; // exports class LettersOnlyValidator
export * from "./ZipCodeValidator";  // exports class ZipCodeValidator
```

# <a name="import"></a>導入

模塊的導入操作與導出一樣簡單。
可以使用以下`import`形式之一來導入其它模塊中的導出內容。

## 導入一個模塊中的某個導出內容

```ts
import { ZipCodeValidator } from "./ZipCodeValidator";

let myValidator = new ZipCodeValidator();
```

可以對導入內容重命名

```ts
import { ZipCodeValidator as ZCV } from "./ZipCodeValidator";
let myValidator = new ZCV();
```

## 將整個模塊導入到一個變量，並通過它來訪問模塊的導出部分

```ts
import * as validator from "./ZipCodeValidator";
let myValidator = new validator.ZipCodeValidator();
```

## 具有副作用的導入模塊

儘管不推薦這麼做，一些模塊會設置一些全局狀態供其它模塊使用。
這些模塊可能沒有任何的導出或用戶根本就不關注它的導出。
使用下面的方法來導入這類模塊：

```ts
import "./my-module.js";
```

# 默認導出

每個模塊都可以有一個`default`導出。
默認導出使用`default`關鍵字標記；並且一個模塊只能夠有一個`default`導出。
需要使用一種特殊的導入形式來導入`default`導出。

`default`導出十分便利。
比如，像JQuery這樣的類庫可能有一個默認導出`jQuery`或`$`，並且我們基本上也會使用同樣的名字`jQuery`或`$`導出JQuery。

##### JQuery.d.ts

```ts
declare let $: JQuery;
export default $;
```

##### App.ts

```ts
import $ from "JQuery";

$("button.continue").html( "Next Step..." );
```

類和函數聲明可以直接被標記為默認導出。
標記為默認導出的類和函數的名字是可以省略的。

##### ZipCodeValidator.ts

```ts
export default class ZipCodeValidator {
    static numberRegexp = /^[0-9]+$/;
    isAcceptable(s: string) {
        return s.length === 5 && ZipCodeValidator.numberRegexp.test(s);
    }
}
```

##### Test.ts

```ts
import validator from "./ZipCodeValidator";

let myValidator = new validator();
```

或者

##### StaticZipCodeValidator.ts

```ts
const numberRegexp = /^[0-9]+$/;

export default function (s: string) {
    return s.length === 5 && numberRegexp.test(s);
}
```

##### Test.ts

```ts
import validate from "./StaticZipCodeValidator";

let strings = ["Hello", "98052", "101"];

// Use function validate
strings.forEach(s => {
  console.log(`"${s}" ${validate(s) ? " matches" : " does not match"}`);
});
```

`default`導出也可以是一個值

##### OneTwoThree.ts

```ts
export default "123";
```

##### Log.ts

```ts
import num from "./OneTwoThree";

console.log(num); // "123"
```

# `export =` 和 `import = require()`

CommonJS和AMD的環境裡都有一個`exports`變量，這個變量包含了一個模塊的所有導出內容。

CommonJS和AMD的`exports`都可以被賦值為一個`對象`, 這種情況下其作用就類似於 es6 語法裡的默認導出，即 `export default`語法了。雖然作用相似，但是 `export default` 語法並不能兼容CommonJS和AMD的`exports`。

為了支持CommonJS和AMD的`exports`, TypeScript提供了`export =`語法。

`export =`語法定義一個模塊的導出`對象`。
這裡的`對象`一詞指的是類，接口，命名空間，函數或枚舉。

若使用`export =`導出一個模塊，則必須使用TypeScript的特定語法`import module = require("module")`來導入此模塊。

##### ZipCodeValidator.ts

```ts
let numberRegexp = /^[0-9]+$/;
class ZipCodeValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
export = ZipCodeValidator;
```

##### Test.ts

```ts
import zip = require("./ZipCodeValidator");

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validator = new zip();

// Show whether each string passed each validator
strings.forEach(s => {
  console.log(`"${ s }" - ${ validator.isAcceptable(s) ? "matches" : "does not match" }`);
});
```

# 生成模塊代碼

根據編譯時指定的模塊目標參數，編譯器會生成相應的供Node.js ([CommonJS](http://wiki.commonjs.org/wiki/CommonJS))，Require.js ([AMD](https://github.com/amdjs/amdjs-api/wiki/AMD))，[UMD](https://github.com/umdjs/umd), [SystemJS](https://github.com/systemjs/systemjs)或[ECMAScript 2015 native modules](http://www.ecma-international.org/ecma-262/6.0/#sec-modules) (ES6)模塊加載系統使用的代碼。
想要瞭解生成代碼中`define`，`require` 和 `register`的意義，請參考相應模塊加載器的文檔。

下面的例子說明了導入導出語句裡使用的名字是怎麼轉換為相應的模塊加載器代碼的。

##### SimpleModule.ts

```ts
import m = require("mod");
export let t = m.something + 1;
```

##### AMD / RequireJS SimpleModule.js

```js
define(["require", "exports", "./mod"], function (require, exports, mod_1) {
    exports.t = mod_1.something + 1;
});
```

##### CommonJS / Node SimpleModule.js

```js
let mod_1 = require("./mod");
exports.t = mod_1.something + 1;
```

##### UMD SimpleModule.js

```js
(function (factory) {
    if (typeof module === "object" && typeof module.exports === "object") {
        let v = factory(require, exports); if (v !== undefined) module.exports = v;
    }
    else if (typeof define === "function" && define.amd) {
        define(["require", "exports", "./mod"], factory);
    }
})(function (require, exports) {
    let mod_1 = require("./mod");
    exports.t = mod_1.something + 1;
});
```

##### System SimpleModule.js

```js
System.register(["./mod"], function(exports_1) {
    let mod_1;
    let t;
    return {
        setters:[
            function (mod_1_1) {
                mod_1 = mod_1_1;
            }],
        execute: function() {
            exports_1("t", t = mod_1.something + 1);
        }
    }
});
```

##### Native ECMAScript 2015 modules SimpleModule.js

```js
import { something } from "./mod";
export let t = something + 1;
```

# 簡單示例

下面我們來整理一下前面的驗證器實現，每個模塊只有一個命名的導出。

為了編譯，我們必需要在命令行上指定一個模塊目標。對於Node.js來說，使用`--module commonjs`；
對於Require.js來說，使用`--module amd`。比如：

```Shell
tsc --module commonjs Test.ts
```

編譯完成後，每個模塊會生成一個單獨的`.js`文件。
好比使用了reference標籤，編譯器會根據`import`語句編譯相應的文件。

##### Validation.ts

```ts
export interface StringValidator {
    isAcceptable(s: string): boolean;
}
```

##### LettersOnlyValidator.ts

```ts
import { StringValidator } from "./Validation";

const lettersRegexp = /^[A-Za-z]+$/;

export class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}
```

##### ZipCodeValidator.ts

```ts
import { StringValidator } from "./Validation";

const numberRegexp = /^[0-9]+$/;

export class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}
```

##### Test.ts

```ts
import { StringValidator } from "./Validation";
import { ZipCodeValidator } from "./ZipCodeValidator";
import { LettersOnlyValidator } from "./LettersOnlyValidator";

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
strings.forEach(s => {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
});
```

# 可選的模塊加載和其它高級加載場景

有時候，你只想在某種條件下才加載某個模塊。
在TypeScript裡，使用下面的方式來實現它和其它的高級加載場景，我們可以直接調用模塊加載器並且可以保證類型完全。

編譯器會檢測是否每個模塊都會在生成的JavaScript中用到。
如果一個模塊標識符只在類型註解部分使用，並且完全沒有在表達式中使用時，就不會生成`require`這個模塊的代碼。
省略掉沒有用到的引用對性能提升是很有益的，並同時提供了選擇性加載模塊的能力。

這種模式的核心是`import id = require("...")`語句可以讓我們訪問模塊導出的類型。
模塊加載器會被動態調用（通過`require`），就像下面`if`代碼塊裡那樣。
它利用了省略引用的優化，所以模塊只在被需要時加載。
為了讓這個模塊工作，一定要注意`import`定義的標識符只能在表示類型處使用（不能在會轉換成JavaScript的地方）。

為了確保類型安全性，我們可以使用`typeof`關鍵字。
`typeof`關鍵字，當在表示類型的地方使用時，會得出一個類型值，這裡就表示模塊的類型。

##### 示例：Node.js裡的動態模塊加載

```ts
declare function require(moduleName: string): any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    let ZipCodeValidator: typeof Zip = require("./ZipCodeValidator");
    let validator = new ZipCodeValidator();
    if (validator.isAcceptable("...")) { /* ... */ }
}
```

##### 示例：require.js裡的動態模塊加載

```ts
declare function require(moduleNames: string[], onLoad: (...args: any[]) => void): void;

import * as Zip from "./ZipCodeValidator";

if (needZipValidation) {
    require(["./ZipCodeValidator"], (ZipCodeValidator: typeof Zip) => {
        let validator = new ZipCodeValidator.ZipCodeValidator();
        if (validator.isAcceptable("...")) { /* ... */ }
    });
}
```

##### 示例：System.js裡的動態模塊加載

```ts
declare const System: any;

import { ZipCodeValidator as Zip } from "./ZipCodeValidator";

if (needZipValidation) {
    System.import("./ZipCodeValidator").then((ZipCodeValidator: typeof Zip) => {
        var x = new ZipCodeValidator();
        if (x.isAcceptable("...")) { /* ... */ }
    });
}
```

# 使用其它的JavaScript庫

要想描述非TypeScript編寫的類庫的類型，我們需要聲明類庫所暴露出的API。

我們叫它聲明因為它不是“外部程序”的具體實現。
它們通常是在`.d.ts`文件裡定義的。
如果你熟悉C/C++，你可以把它們當做`.h`文件。
讓我們看一些例子。

## 外部模塊

在Node.js裡大部分工作是通過加載一個或多個模塊實現的。
我們可以使用頂級的`export`聲明來為每個模塊都定義一個`.d.ts`文件，但最好還是寫在一個大的`.d.ts`文件裡。
我們使用與構造一個外部命名空間相似的方法，但是這裡使用`module`關鍵字並且把名字用引號括起來，方便之後`import`。
例如：

##### node.d.ts (simplified excerpt)

```ts
declare module "url" {
    export interface Url {
        protocol?: string;
        hostname?: string;
        pathname?: string;
    }

    export function parse(urlStr: string, parseQueryString?, slashesDenoteHost?): Url;
}

declare module "path" {
    export function normalize(p: string): string;
    export function join(...paths: any[]): string;
    export let sep: string;
}
```

現在我們可以`/// <reference>` `node.d.ts`並且使用`import url = require("url");`或`import * as URL from "url"`加載模塊。

```ts
/// <reference path="node.d.ts"/>
import * as URL from "url";
let myUrl = URL.parse("http://www.typescriptlang.org");
```

### 外部模塊簡寫

假如你不想在使用一個新模塊之前花時間去編寫聲明，你可以採用聲明的簡寫形式以便能夠快速使用它。

##### declarations.d.ts

```ts
declare module "hot-new-module";
```

簡寫模塊裡所有導出的類型將是`any`。

```ts
import x, {y} from "hot-new-module";
x(y);
```

### 模塊聲明通配符

某些模塊加載器如[SystemJS](https://github.com/systemjs/systemjs/blob/master/docs/overview.md#plugin-syntax)
和[AMD](https://github.com/amdjs/amdjs-api/blob/master/LoaderPlugins.md)支持導入非JavaScript內容。
它們通常會使用一個前綴或後綴來表示特殊的加載語法。
模塊聲明通配符可以用來表示這些情況。

```ts
declare module "*!text" {
    const content: string;
    export default content;
}
// Some do it the other way around.
declare module "json!*" {
    const value: any;
    export default value;
}
```

現在你可以就導入匹配`"*!text"`或`"json!*"`的內容了。

```ts
import fileContent from "./xyz.txt!text";
import data from "json!http://example.com/data.json";
console.log(data, fileContent);
```

### UMD模塊

有些模塊被設計成兼容多個模塊加載器，或者不使用模塊加載器（全局變量）。
它們以[UMD](https://github.com/umdjs/umd)模塊為代表。
這些庫可以通過導入的形式或全局變量的形式訪問。
例如：

##### math-lib.d.ts

```ts
export function isPrime(x: number): boolean;
export as namespace mathLib;
```

之後，這個庫可以在某個模塊裡通過導入來使用：

```ts
import { isPrime } from "math-lib";
isPrime(2);
mathLib.isPrime(2); // ERROR: can't use the global definition from inside a module
```

它同樣可以通過全局變量的形式使用，但只能在某個腳本裡。
（腳本是指一個不帶有導入或導出的文件。）

```ts
mathLib.isPrime(2);
```

# 創建模塊結構指導

## 儘可能地在頂層導出

用戶應該更容易地使用你模塊導出的內容。
嵌套層次過多會變得難以處理，因此仔細考慮一下如何組織你的代碼。

從你的模塊中導出一個命名空間就是一個增加嵌套的例子。
雖然命名空間有時候有它們的用處，在使用模塊的時候它們額外地增加了一層。
這對用戶來說是很不便的並且通常是多餘的。

導出類的靜態方法也有同樣的問題 - 這個類本身就增加了一層嵌套。
除非它能方便表述或便於清晰使用，否則請考慮直接導出一個輔助方法。

### 如果僅導出單個 `class` 或 `function`，使用 `export default`

就像“在頂層上導出”幫助減少用戶使用的難度，一個默認的導出也能起到這個效果。
如果一個模塊就是為了導出特定的內容，那麼你應該考慮使用一個默認導出。
這會令模塊的導入和使用變得些許簡單。
比如：

#### MyClass.ts

```ts
export default class SomeType {
  constructor() { ... }
}
```

#### MyFunc.ts

```ts
export default function getThing() { return 'thing'; }
```

#### Consumer.ts

```ts
import t from "./MyClass";
import f from "./MyFunc";
let x = new t();
console.log(f());
```

對用戶來說這是最理想的。他們可以隨意命名導入模塊的類型（本例為`t`）並且不需要多餘的（.）來找到相關對象。

### 如果要導出多個對象，把它們放在頂層裡導出

#### MyThings.ts

```ts
export class SomeType { /* ... */ }
export function someFunc() { /* ... */ }
```

相反地，當導入的時候：

### 明確地列出導入的名字

#### Consumer.ts

```ts
import { SomeType, SomeFunc } from "./MyThings";
let x = new SomeType();
let y = someFunc();
```

### 使用命名空間導入模式當你要導出大量內容的時候

#### MyLargeModule.ts

```ts
export class Dog { ... }
export class Cat { ... }
export class Tree { ... }
export class Flower { ... }
```

#### Consumer.ts

```ts
import * as myLargeModule from "./MyLargeModule.ts";
let x = new myLargeModule.Dog();
```

## 使用重新導出進行擴展

你可能經常需要去擴展一個模塊的功能。
JS裡常用的一個模式是JQuery那樣去擴展原對象。
如我們之前提到的，模塊不會像全局命名空間對象那樣去*合併*。
推薦的方案是*不要*去改變原來的對象，而是導出一個新的實體來提供新的功能。

假設`Calculator.ts`模塊裡定義了一個簡單的計算器實現。
這個模塊同樣提供了一個輔助函數來測試計算器的功能，通過傳入一系列輸入的字符串並在最後給出結果。

#### Calculator.ts

```ts
export class Calculator {
    private current = 0;
    private memory = 0;
    private operator: string;

    protected processDigit(digit: string, currentValue: number) {
        if (digit >= "0" && digit <= "9") {
            return currentValue * 10 + (digit.charCodeAt(0) - "0".charCodeAt(0));
        }
    }

    protected processOperator(operator: string) {
        if (["+", "-", "*", "/"].indexOf(operator) >= 0) {
            return operator;
        }
    }

    protected evaluateOperator(operator: string, left: number, right: number): number {
        switch (this.operator) {
            case "+": return left + right;
            case "-": return left - right;
            case "*": return left * right;
            case "/": return left / right;
        }
    }

    private evaluate() {
        if (this.operator) {
            this.memory = this.evaluateOperator(this.operator, this.memory, this.current);
        }
        else {
            this.memory = this.current;
        }
        this.current = 0;
    }

    public handleChar(char: string) {
        if (char === "=") {
            this.evaluate();
            return;
        }
        else {
            let value = this.processDigit(char, this.current);
            if (value !== undefined) {
                this.current = value;
                return;
            }
            else {
                let value = this.processOperator(char);
                if (value !== undefined) {
                    this.evaluate();
                    this.operator = value;
                    return;
                }
            }
        }
        throw new Error(`Unsupported input: '${char}'`);
    }

    public getResult() {
        return this.memory;
    }
}

export function test(c: Calculator, input: string) {
    for (let i = 0; i < input.length; i++) {
        c.handleChar(input[i]);
    }

    console.log(`result of '${input}' is '${c.getResult()}'`);
}
```

下面使用導出的`test`函數來測試計算器。

#### TestCalculator.ts

```ts
import { Calculator, test } from "./Calculator";


let c = new Calculator();
test(c, "1+2*33/11="); // prints 9
```

現在擴展它，添加支持輸入其它進制（十進制以外），讓我們來創建`ProgrammerCalculator.ts`。

#### ProgrammerCalculator.ts

```ts
import { Calculator } from "./Calculator";

class ProgrammerCalculator extends Calculator {
    static digits = ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9", "A", "B", "C", "D", "E", "F"];

    constructor(public base: number) {
        super();
        const maxBase = ProgrammerCalculator.digits.length;
        if (base <= 0 || base > maxBase) {
            throw new Error(`base has to be within 0 to ${maxBase} inclusive.`);
        }
    }

    protected processDigit(digit: string, currentValue: number) {
        if (ProgrammerCalculator.digits.indexOf(digit) >= 0) {
            return currentValue * this.base + ProgrammerCalculator.digits.indexOf(digit);
        }
    }
}

// Export the new extended calculator as Calculator
export { ProgrammerCalculator as Calculator };

// Also, export the helper function
export { test } from "./Calculator";
```

新的`ProgrammerCalculator`模塊導出的API與原先的`Calculator`模塊很相似，但卻沒有改變原模塊裡的對象。
下面是測試ProgrammerCalculator類的代碼：

#### TestProgrammerCalculator.ts

```ts
import { Calculator, test } from "./ProgrammerCalculator";

let c = new Calculator(2);
test(c, "001+010="); // prints 3
```

## 模塊裡不要使用命名空間

當初次進入基於模塊的開發模式時，可能總會控制不住要將導出包裹在一個命名空間裡。
模塊具有其自己的作用域，並且只有導出的聲明才會在模塊外部可見。
記住這點，命名空間在使用模塊時幾乎沒什麼價值。

在組織方面，命名空間對於在全局作用域內對邏輯上相關的對象和類型進行分組是很便利的。
例如，在C#裡，你會從`System.Collections`裡找到所有集合的類型。
通過將類型有層次地組織在命名空間裡，可以方便用戶找到與使用那些類型。
然而，模塊本身已經存在於文件系統之中，這是必須的。
我們必須通過路徑和文件名找到它們，這已經提供了一種邏輯上的組織形式。
我們可以創建`/collections/generic/`文件夾，把相應模塊放在這裡面。

命名空間對解決全局作用域裡命名衝突來說是很重要的。
比如，你可以有一個`My.Application.Customer.AddForm`和`My.Application.Order.AddForm` -- 兩個類型的名字相同，但命名空間不同。
然而，這對於模塊來說卻不是一個問題。
在一個模塊裡，沒有理由兩個對象擁有同一個名字。
從模塊的使用角度來說，使用者會挑出他們用來引用模塊的名字，所以也沒有理由發生重名的情況。

> 更多關於模塊和命名空間的資料查看[命名空間和模塊](./Namespaces and Modules.md)

## 危險信號

以下均為模塊結構上的危險信號。重新檢查以確保你沒有在對模塊使用命名空間：

* 文件的頂層聲明是`export namespace Foo { ... }` （刪除`Foo`並把所有內容向上層移動一層）
* 文件只有一個`export class`或`export function` （考慮使用`export default`）
* 多個文件的頂層具有同樣的`export namespace Foo {` （不要以為這些會合併到一個`Foo`中！）
