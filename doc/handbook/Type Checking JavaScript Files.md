TypeScript 2.3以後的版本支持使用`--checkJs`對`.js`文件進行類型檢查和錯誤提示。

你可以通過添加`// @ts-nocheck`註釋來忽略類型檢查；相反，你可以通過去掉`--checkJs`設置並添加一個`// @ts-check`註釋來選則檢查某些`.js`文件。
你還可以使用`// @ts-ignore`來忽略本行的錯誤。
如果你使用了`tsconfig.json`，JS檢查將遵照一些嚴格檢查標記，如`noImplicitAny`，`strictNullChecks`等。
但因為JS檢查是相對寬鬆的，在使用嚴格標記時可能會有些出乎意料的情況。

對比`.js`文件和`.ts`文件在類型檢查上的差異，有如下幾點需要注意：

## 用JSDoc類型表示類型信息

`.js`文件裡，類型可以和在`.ts`文件裡一樣被推斷出來。
同樣地，當類型不能被推斷時，它們可以通過JSDoc來指定，就好比在`.ts`文件裡那樣。
如同TypeScript，`--noImplicitAny`會在編譯器無法推斷類型的位置報錯。
（除了對象字面量的情況；後面會詳細介紹）

JSDoc註解修飾的聲明會被設置為這個聲明的類型。比如：

```js
/** @type {number} */
var x;

x = 0;      // OK
x = false;  // Error: boolean is not assignable to number
```

你可以在這裡找到所有JSDoc支持的模式，[JSDoc文檔](https://github.com/Microsoft/TypeScript/wiki/JSDoc-support-in-JavaScript)。

## 屬性的推斷來自於類內的賦值語句

ES2015沒提供聲明類屬性的方法。屬性是動態賦值的，就像對象字面量一樣。

在`.js`文件裡，編譯器從類內部的屬性賦值語句來推斷屬性類型。
屬性的類型是在構造函數裡賦的值的類型，除非它沒在構造函數裡定義或者在構造函數裡是`undefined`或`null`。
若是這種情況，類型將會是所有賦的值的類型的聯合類型。
在構造函數裡定義的屬性會被認為是一直存在的，然而那些在方法，存取器裡定義的屬性被當成可選的。

```js
class C {
    constructor() {
        this.constructorOnly = 0
        this.constructorUnknown = undefined
    }
    method() {
        this.constructorOnly = false // error, constructorOnly is a number
        this.constructorUnknown = "plunkbat" // ok, constructorUnknown is string | undefined
        this.methodOnly = 'ok'  // ok, but y could also be undefined
    }
    method2() {
        this.methodOnly = true  // also, ok, y's type is string | boolean | undefined
    }
}
```

如果一個屬性從沒在類內設置過，它們會被當成未知的。

如果類的屬性只是讀取用的，那麼就在構造函數裡用JSDoc聲明它的類型。
如果它稍後會被初始化，你甚至都不需要在構造函數裡給它賦值：

```js
class C {
    constructor() {
        /** @type {number | undefined} */
        this.prop = undefined;
        /** @type {number | undefined} */
        this.count;
    }
}

let c = new C();
c.prop = 0;          // OK
c.count = "string";  // Error: string is not assignable to number|undefined
```

## 構造函數等同於類

ES2015以前，Javascript使用構造函數代替類。
編譯器支持這種模式並能夠將構造函數識別為ES2015的類。
屬性類型推斷機制和上面介紹的一致。

```js
function C() {
    this.constructorOnly = 0
    this.constructorUnknown = undefined
}
C.prototype.method = function() {
    this.constructorOnly = false // error
    this.constructorUnknown = "plunkbat" // OK, the type is string | undefined
}
```

## 支持CommonJS模塊

在`.js`文件裡，TypeScript能識別出CommonJS模塊。
對`exports`和`module.exports`的賦值被識別為導出聲明。
相似地，`require`函數調用被識別為模塊導入。例如：

```js
// same as `import module "fs"`
const fs = require("fs");

// same as `export function readFile`
module.exports.readFile = function(f) {
  return fs.readFileSync(f);
}
```

對JavaScript文件裡模塊語法的支持比在TypeScript裡寬泛多了。
大部分的賦值和聲明方式都是允許的。

## 類，函數和對象字面量是命名空間

`.js`文件裡的類是命名空間。
它可以用於嵌套類，比如：

```js
class C {
}
C.D = class {
}
```

ES2015之前的代碼，它可以用來模擬靜態方法：

```js
function Outer() {
  this.y = 2
}
Outer.Inner = function() {
  this.yy = 2
}
```

它還可以用於創建簡單的命名空間：

```js
var ns = {}
ns.C = class {
}
ns.func = function() {
}
```

同時還支持其它的變化：

```js
// 立即調用的函數表達式
var ns = (function (n) {
  return n || {};
})();
ns.CONST = 1

// defaulting to global
var assign = assign || function() {
  // code goes here
}
assign.extra = 1
```

## 對象字面量是開放的

`.ts`文件裡，用對象字面量初始化一個變量的同時也給它聲明了類型。
新的成員不能再被添加到對象字面量中。
這個規則在`.js`文件裡被放寬了；對象字面量具有開放的類型，允許添加並訪問原先沒有定義的屬性。例如：

```js
var obj = { a: 1 };
obj.b = 2;  // Allowed
```

對象字面量的表現就好比具有一個默認的索引簽名`[x:string]: any`，它們可以被當成開放的映射而不是封閉的對象。

與其它JS檢查行為相似，這種行為可以通過指定JSDoc類型來改變，例如：

```js
/** @type {{a: number}} */
var obj = { a: 1 };
obj.b = 2;  // Error, type {a: number} does not have property b
```

## null，undefined，和空數組的類型是any或any[]

任何用`null`，`undefined`初始化的變量，參數或屬性，它們的類型是`any`，就算是在嚴格`null`檢查模式下。
任何用`[]`初始化的變量，參數或屬性，它們的類型是`any[]`，就算是在嚴格`null`檢查模式下。
唯一的例外是像上面那樣有多個初始化器的屬性。

```js
function Foo(i = null) {
    if (!i) i = 1;
    var j = undefined;
    j = 2;
    this.l = [];
}
var foo = new Foo();
foo.l.push(foo.i);
foo.l.push("end");
```

## 函數參數是默認可選的

由於在ES2015之前無法指定可選參數，因此`.js`文件裡所有函數參數都被當做是可選的。
使用比預期少的參數調用函數是允許的。

需要注意的一點是，使用過多的參數調用函數會得到一個錯誤。

例如：

```js
function bar(a, b) {
  console.log(a + " " + b);
}

bar(1);       // OK, second argument considered optional
bar(1, 2);
bar(1, 2, 3); // Error, too many arguments
```

使用JSDoc註解的函數會被從這條規則裡移除。
使用JSDoc可選參數語法來表示可選性。比如：

```js
/**
 * @param {string} [somebody] - Somebody's name.
 */
function sayHello(somebody) {
    if (!somebody) {
        somebody = 'John Doe';
    }
    console.log('Hello ' + somebody);
}

sayHello();
```

## 由`arguments`推斷出的var-args參數聲明

如果一個函數的函數體內有對`arguments`的引用，那麼這個函數會隱式地被認為具有一個var-arg參數（比如:`(...arg: any[]) => any`)）。使用JSDoc的var-arg語法來指定`arguments`的類型。

```js
/** @param {...number} args */
function sum(/* numbers */) {
    var total = 0
    for (var i = 0; i < arguments.length; i++) {
      total += arguments[i]
    }
    return total
}
```

## 未指定的類型參數默認為`any`

由於JavaScript裡沒有一種自然的語法來指定泛型參數，因此未指定的參數類型默認為`any`。

### 在extends語句中：

例如，`React.Component`被定義成具有兩個類型參數，`Props`和`State`。
在一個`.js`文件裡，沒有一個合法的方式在extends語句裡指定它們。默認地參數類型為`any`：

```js
import { Component } from "react";

class MyComponent extends Component {
    render() {
        this.props.b; // Allowed, since this.props is of type any
    }
}
```

使用JSDoc的`@augments`來明確地指定類型。例如：

```js
import { Component } from "react";

/**
 * @augments {Component<{a: number}, State>}
 */
class MyComponent extends Component {
    render() {
        this.props.b; // Error: b does not exist on {a:number}
    }
}
```

### 在JSDoc引用中：

JSDoc裡未指定的類型參數默認為`any`：

```js
/** @type{Array} */
var x = [];

x.push(1);        // OK
x.push("string"); // OK, x is of type Array<any>

/** @type{Array.<number>} */
var y = [];

y.push(1);        // OK
y.push("string"); // Error, string is not assignable to number
```

### 在函數調用中

泛型函數的調用使用`arguments`來推斷泛型參數。有時候，這個流程不能夠推斷出類型，大多是因為缺少推斷的源；在這種情況下，類型參數類型默認為`any`。例如：

```js
var p = new Promise((resolve, reject) => { reject() });

p; // Promise<any>;
```

# 支持的JSDoc

下面的列表列出了當前所支持的JSDoc註解，你可以用它們在JavaScript文件裡添加類型信息。

注意，沒有在下面列出的標記（例如`@async`）都是還不支持的。

* `@type`
* `@param` (or `@arg` or `@argument`)
* `@returns` (or `@return`)
* `@typedef`
* `@callback`
* `@template`
* `@class` (or `@constructor`)
* `@this`
* `@extends` (or `@augments`)
* `@enum`

它們代表的意義與usejsdoc.org上面給出的通常是一致的或者是它的超集。
下面的代碼描述了它們的區別並給出了一些示例。

## `@type`

可以使用`@type`標記並引用一個類型名稱（原始類型，TypeScript裡聲明的類型，或在JSDoc裡`@typedef`標記指定的）
可以使用任何TypeScript類型和大多數JSDoc類型。

```js
/**
 * @type {string}
 */
var s;

/** @type {Window} */
var win;

/** @type {PromiseLike<string>} */
var promisedString;

// You can specify an HTML Element with DOM properties
/** @type {HTMLElement} */
var myElement = document.querySelector(selector);
element.dataset.myData = '';

```

`@type`可以指定聯合類型&mdash;例如，`string`和`boolean`類型的聯合。

```js
/**
 * @type {(string | boolean)}
 */
var sb;
```

注意，括號是可選的。

```js
/**
 * @type {string | boolean}
 */
var sb;
```

有多種方式來指定數組類型：

```js
/** @type {number[]} */
var ns;
/** @type {Array.<number>} */
var nds;
/** @type {Array<number>} */
var nas;
```

還可以指定對象字面量類型。
例如，一個帶有`a`（字符串）和`b`（數字）屬性的對象，使用下面的語法：

```js
/** @type {{ a: string, b: number }} */
var var9;
```

可以使用字符串和數字索引簽名來指定`map-like`和`array-like`的對象，使用標準的JSDoc語法或者TypeScript語法。

```js
/**
 * A map-like object that maps arbitrary `string` properties to `number`s.
 *
 * @type {Object.<string, number>}
 */
var stringToNumber;

/** @type {Object.<number, object>} */
var arrayLike;
```

這兩個類型與TypeScript裡的`{ [x: string]: number }`和`{ [x: number]: any }`是等同的。編譯器能識別出這兩種語法。

可以使用TypeScript或Closure語法指定函數類型。

```js
/** @type {function(string, boolean): number} Closure syntax */
var sbn;
/** @type {(s: string, b: boolean) => number} Typescript syntax */
var sbn2;
```

或者直接使用未指定的`Function`類型：

```js
/** @type {Function} */
var fn7;
/** @type {function} */
var fn6;
```

Closure的其它類型也可以使用：

```js
/**
 * @type {*} - can be 'any' type
 */
var star;
/**
 * @type {?} - unknown type (same as 'any')
 */
var question;
```

### 轉換

TypeScript借鑑了Closure裡的轉換語法。
在括號表達式前面使用`@type`標記，可以將一種類型轉換成另一種類型

```js
/**
 * @type {number | string}
 */
var numberOrString = Math.random() < 0.5 ? "hello" : 100;
var typeAssertedNumber = /** @type {number} */ (numberOrString)
```

### 導入類型

可以使用導入類型從其它文件中導入聲明。
這個語法是TypeScript特有的，與JSDoc標準不同：

```js
/**
 * @param p { import("./a").Pet }
 */
function walk(p) {
    console.log(`Walking ${p.name}...`);
}
```

導入類型也可以使用在類型別名聲明中：

```js
/**
 * @typedef Pet { import("./a").Pet }
 */

/**
 * @type {Pet}
 */
var myPet;
myPet.name;
```

導入類型可以用在從模塊中得到一個值的類型。

```js
/**
 * @type {typeof import("./a").x }
 */
var x = require("./a").x;
```

## `@param`和`@returns`

`@param`語法和`@type`相同，但增加了一個參數名。
使用`[]`可以把參數聲明為可選的：

```js
// Parameters may be declared in a variety of syntactic forms
/**
 * @param {string}  p1 - A string param.
 * @param {string=} p2 - An optional param (Closure syntax)
 * @param {string} [p3] - Another optional param (JSDoc syntax).
 * @param {string} [p4="test"] - An optional param with a default value
 * @return {string} This is the result
 */
function stringsStringStrings(p1, p2, p3, p4){
  // TODO
}
```

函數的返回值類型也是類似的：

```js
/**
 * @return {PromiseLike<string>}
 */
function ps(){}

/**
 * @returns {{ a: string, b: number }} - May use '@returns' as well as '@return'
 */
function ab(){}
```

## `@typedef`, `@callback`, 和 `@param`

`@typedef`可以用來聲明複雜類型。
和`@param`類似的語法。

```js
/**
 * @typedef {Object} SpecialType - creates a new type named 'SpecialType'
 * @property {string} prop1 - a string property of SpecialType
 * @property {number} prop2 - a number property of SpecialType
 * @property {number=} prop3 - an optional number property of SpecialType
 * @prop {number} [prop4] - an optional number property of SpecialType
 * @prop {number} [prop5=42] - an optional number property of SpecialType with default
 */
/** @type {SpecialType} */
var specialTypeObject;
```

可以在第一行上使用`object`或`Object`。

```js
/**
 * @typedef {object} SpecialType1 - creates a new type named 'SpecialType'
 * @property {string} prop1 - a string property of SpecialType
 * @property {number} prop2 - a number property of SpecialType
 * @property {number=} prop3 - an optional number property of SpecialType
 */
/** @type {SpecialType1} */
var specialTypeObject1;
```

`@param`允許使用相似的語法。
注意，嵌套的屬性名必須使用參數名做為前綴：

```js
/**
 * @param {Object} options - The shape is the same as SpecialType above
 * @param {string} options.prop1
 * @param {number} options.prop2
 * @param {number=} options.prop3
 * @param {number} [options.prop4]
 * @param {number} [options.prop5=42]
 */
function special(options) {
  return (options.prop4 || 1001) + options.prop5;
}
```

`@callback`與`@typedef`相似，但它指定函數類型而不是對象類型：

```js
/**
 * @callback Predicate
 * @param {string} data
 * @param {number} [index]
 * @returns {boolean}
 */
/** @type {Predicate} */
const ok = s => !(s.length % 2);
```

當然，所有這些類型都可以使用TypeScript的語法`@typedef`在一行上聲明：

```js
/** @typedef {{ prop1: string, prop2: string, prop3?: number }} SpecialType */
/** @typedef {(data: string, index?: number) => boolean} Predicate */
```

## `@template`

使用`@template`聲明泛型：

```js
/**
 * @template T
 * @param {T} p1 - A generic parameter that flows through to the return type
 * @return {T}
 */
function id(x){ return x }
```

用逗號或多個標記來聲明多個類型參數：

```js
/**
 * @template T,U,V
 * @template W,X
 */
```

還可以在參數名前指定類型約束。
只有列表的第一項類型參數會被約束：

```js
/**
 * @template {string} K - K must be a string or string literal
 * @template {{ serious(): string }} Seriousalizable - must have a serious method
 * @param {K} key
 * @param {Seriousalizable} object
 */
function seriousalize(key, object) {
  // ????
}
```

## `@constructor`

編譯器通過`this`屬性的賦值來推斷構造函數，但你可以讓檢查更嚴格提示更友好，你可以添加一個`@constructor`標記：

```js
/**
 * @constructor
 * @param {number} data
 */
function C(data) {
  this.size = 0;
  this.initialize(data); // Should error, initializer expects a string
}
/**
 * @param {string} s
 */
C.prototype.initialize = function (s) {
  this.size = s.length
}

var c = new C(0);
var result = C(1); // C should only be called with new
```

通過`@constructor`，`this`將在構造函數`C`裡被檢查，因此你在`initialize`方法裡得到一個提示，如果你傳入一個數字你還將得到一個錯誤提示。如果你直接調用`C`而不是構造它，也會得到一個錯誤。

不幸的是，這意味著那些既能構造也能直接調用的構造函數不能使用`@constructor`。

## `@this`

編譯器通常可以通過上下文來推斷出`this`的類型。但你可以使用`@this`來明確指定它的類型：

```js
/**
 * @this {HTMLElement}
 * @param {*} e
 */
function callbackForLater(e) {
    this.clientHeight = parseInt(e) // should be fine!
}
```

## `@extends`

當JavaScript類繼承了一個基類，無處指定類型參數的類型。而`@extends`標記提供了這樣一種方式：

```js
/**
 * @template T
 * @extends {Set<T>}
 */
class SortableSet extends Set {
  // ...
}
```

注意`@extends`只作用於類。當前，無法實現構造函數繼承類的情況。

## `@enum`

`@enum`標記允許你創建一個對象字面量，它的成員都有確定的類型。不同於JavaScript裡大多數的對象字面量，它不允許添加額外成員。

```js
/** @enum {number} */
const JSDocState = {
  BeginningOfLine: 0,
  SawAsterisk: 1,
  SavingComments: 2,
}
```

注意`@enum`與TypeScript的`@enum`大不相同，它更加簡單。然而，不同於TypeScript的枚舉，`@enum`可以是任何類型：

```js
/** @enum {function(number): number} */
const Math = {
  add1: n => n + 1,
  id: n => -n,
  sub1: n => n - 1,
}
```

## 更多示例

```js
var someObj = {
  /**
   * @param {string} param1 - Docs on property assignments work
   */
  x: function(param1){}
};

/**
 * As do docs on variable assignments
 * @return {Window}
 */
let someFunc = function(){};

/**
 * And class methods
 * @param {string} greeting The greeting to use
 */
Foo.prototype.sayHi = (greeting) => console.log("Hi!");

/**
 * And arrow functions expressions
 * @param {number} x - A multiplier
 */
let myArrow = x => x * x;

/**
 * Which means it works for stateless function components in JSX too
 * @param {{a: string, b: number}} test - Some param
 */
var sfc = (test) => <div>{test.a.charAt(0)}</div>;

/**
 * A parameter can be a class constructor, using Closure syntax.
 *
 * @param {{new(...args: any[]): object}} C - The class to register
 */
function registerClass(C) {}

/**
 * @param {...string} p1 - A 'rest' arg (array) of strings. (treated as 'any')
 */
function fn10(p1){}

/**
 * @param {...string} p1 - A 'rest' arg (array) of strings. (treated as 'any')
 */
function fn9(p1) {
  return p1.join();
}
```

## 已知不支持的模式

在值空間中將對象視為類型是不可以的，除非對象創建了類型，如構造函數。

```js
function aNormalFunction() {

}
/**
 * @type {aNormalFunction}
 */
var wrong;
/**
 * Use 'typeof' instead:
 * @type {typeof aNormalFunction}
 */
var right;
```

對象字面量屬性上的`=`後綴不能指定這個屬性是可選的：

```js
/**
 * @type {{ a: string, b: number= }}
 */
var wrong;
/**
 * Use postfix question on the property name instead:
 * @type {{ a: string, b?: number }}
 */
var right;
```

`Nullable`類型只在啟用了`strictNullChecks`檢查時才啟作用：

```js
/**
 * @type {?number}
 * With strictNullChecks: true -- number | null
 * With strictNullChecks: off  -- number
 */
var nullable;
```

`Non-nullable`類型沒有意義，以其原類型對待：

```js
/**
 * @type {!number}
 * Just has type number
 */
var normal;
```

不同於JSDoc類型系統，TypeScript只允許將類型標記為包不包含`null`。
沒有明確的`Non-nullable` -- 如果啟用了`strictNullChecks`，那麼`number`是非`null`的。
如果沒有啟用，那麼`number`是可以為`null`的。
