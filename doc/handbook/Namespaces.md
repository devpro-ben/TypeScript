> **關於術語的一點說明:**
請務必注意一點，TypeScript 1.5里術語名已經發生了變化。
“內部模塊”現在稱做“命名空間”。
“外部模塊”現在則簡稱為“模塊”，這是為了與[ECMAScript 2015](http://www.ecma-international.org/ecma-262/6.0/)裡的術語保持一致，(也就是說 `module X {` 相當於現在推薦的寫法 `namespace X {`)。

# 介紹

這篇文章描述了如何在TypeScript裡使用命名空間（之前叫做“內部模塊”）來組織你的代碼。
就像我們在術語說明裡提到的那樣，“內部模塊”現在叫做“命名空間”。
另外，任何使用`module`關鍵字來聲明一個內部模塊的地方都應該使用`namespace`關鍵字來替換。
這就避免了讓新的使用者被相似的名稱所迷惑。

# 第一步

我們先來寫一段程序並將在整篇文章中都使用這個例子。
我們定義幾個簡單的字符串驗證器，假設你會使用它們來驗證表單裡的用戶輸入或驗證外部數據。

## 所有的驗證器都放在一個文件裡

```ts
interface StringValidator {
    isAcceptable(s: string): boolean;
}

let lettersRegexp = /^[A-Za-z]+$/;
let numberRegexp = /^[0-9]+$/;

class LettersOnlyValidator implements StringValidator {
    isAcceptable(s: string) {
        return lettersRegexp.test(s);
    }
}

class ZipCodeValidator implements StringValidator {
    isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s);
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: StringValidator; } = {};
validators["ZIP code"] = new ZipCodeValidator();
validators["Letters only"] = new LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        let isMatch = validators[name].isAcceptable(s);
        console.log(`'${ s }' ${ isMatch ? "matches" : "does not match" } '${ name }'.`);
    }
}
```

# 命名空間

隨著更多驗證器的加入，我們需要一種手段來組織代碼，以便於在記錄它們類型的同時還不用擔心與其它對象產生命名衝突。
因此，我們把驗證器包裹到一個命名空間內，而不是把它們放在全局命名空間下。

下面的例子裡，把所有與驗證器相關的類型都放到一個叫做`Validation`的命名空間裡。
因為我們想讓這些接口和類在命名空間之外也是可訪問的，所以需要使用`export`。
相反的，變量`lettersRegexp`和`numberRegexp`是實現的細節，不需要導出，因此它們在命名空間外是不能訪問的。
在文件末尾的測試代碼裡，由於是在命名空間之外訪問，因此需要限定類型的名稱，比如`Validation.LettersOnlyValidator`。

## 使用命名空間的驗證器

```ts
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }

    const lettersRegexp = /^[A-Za-z]+$/;
    const numberRegexp = /^[0-9]+$/;

    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }

    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}
```

# 分離到多文件

當應用變得越來越大時，我們需要將代碼分離到不同的文件中以便於維護。

## 多文件中的命名空間

現在，我們把`Validation`命名空間分割成多個文件。
儘管是不同的文件，它們仍是同一個命名空間，並且在使用的時候就如同它們在一個文件中定義的一樣。
因為不同文件之間存在依賴關係，所以我們加入了引用標籤來告訴編譯器文件之間的關聯。
我們的測試代碼保持不變。

##### Validation.ts

```ts
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}
```

##### LettersOnlyValidator.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

##### ZipCodeValidator.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
    const numberRegexp = /^[0-9]+$/;
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s);
        }
    }
}
```

##### Test.ts

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ["Hello", "98052", "101"];

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {};
validators["ZIP code"] = new Validation.ZipCodeValidator();
validators["Letters only"] = new Validation.LettersOnlyValidator();

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`);
    }
}
```

當涉及到多文件時，我們必須確保所有編譯後的代碼都被加載了。
我們有兩種方式。

第一種方式，把所有的輸入文件編譯為一個輸出文件，需要使用`--outFile`標記：

```Shell
tsc --outFile sample.js Test.ts
```

編譯器會根據源碼裡的引用標籤自動地對輸出進行排序。你也可以單獨地指定每個文件。

```Shell
tsc --outFile sample.js Validation.ts LettersOnlyValidator.ts ZipCodeValidator.ts Test.ts
```

第二種方式，我們可以編譯每一個文件（默認方式），那麼每個源文件都會對應生成一個JavaScript文件。
然後，在頁面上通過`<script>`標籤把所有生成的JavaScript文件按正確的順序引進來，比如：

##### MyTestPage.html (excerpt)

```html
    <script src="Validation.js" type="text/javascript" />
    <script src="LettersOnlyValidator.js" type="text/javascript" />
    <script src="ZipCodeValidator.js" type="text/javascript" />
    <script src="Test.js" type="text/javascript" />
```

# 別名

另一種簡化命名空間操作的方法是使用`import q = x.y.z`給常用的對象起一個短的名字。
不要與用來加載模塊的`import x = require('name')`語法弄混了，這裡的語法是為指定的符號創建一個別名。
你可以用這種方法為任意標識符創建別名，也包括導入的模塊中的對象。

```ts
namespace Shapes {
    export namespace Polygons {
        export class Triangle { }
        export class Square { }
    }
}

import polygons = Shapes.Polygons;
let sq = new polygons.Square(); // Same as "new Shapes.Polygons.Square()"
```

注意，我們並沒有使用`require`關鍵字，而是直接使用導入符號的限定名賦值。
這與使用`var`相似，但它還適用於類型和導入的具有命名空間含義的符號。
重要的是，對於值來講，`import`會生成與原始符號不同的引用，所以改變別名的`var`值並不會影響原始變量的值。

# 使用其它的JavaScript庫

為了描述不是用TypeScript編寫的類庫的類型，我們需要聲明類庫導出的API。
由於大部分程序庫只提供少數的頂級對象，命名空間是用來表示它們的一個好辦法。

我們稱其為聲明是因為它不是外部程序的具體實現。
我們通常在`.d.ts`裡寫這些聲明。
如果你熟悉C/C++，你可以把它們當做`.h`文件。
讓我們看一些例子。

## 外部命名空間

流行的程序庫D3在全局對象`d3`裡定義它的功能。
因為這個庫通過一個`<script>`標籤加載（不是通過模塊加載器），它的聲明文件使用內部模塊來定義它的類型。
為了讓TypeScript編譯器識別它的類型，我們使用外部命名空間聲明。
比如，我們可以像下面這樣寫：

##### D3.d.ts (部分摘錄)

```ts
declare namespace D3 {
    export interface Selectors {
        select: {
            (selector: string): Selection;
            (element: EventTarget): Selection;
        };
    }

    export interface Event {
        x: number;
        y: number;
    }

    export interface Base extends Selectors {
        event: Event;
    }
}

declare var d3: D3.Base;
```
