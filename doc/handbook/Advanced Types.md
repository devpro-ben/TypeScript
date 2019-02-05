# 交叉類型（Intersection Types）

交叉類型是將多個類型合併為一個類型。
這讓我們可以把現有的多種類型疊加到一起成為一種類型，它包含了所需的所有類型的特性。
例如，`Person & Serializable & Loggable`同時是`Person`*和*`Serializable`*和*`Loggable`。
就是說這個類型的對象同時擁有了這三種類型的成員。

我們大多是在混入（mixins）或其它不適合典型面向對象模型的地方看到交叉類型的使用。
（在JavaScript裡發生這種情況的場合很多！）
下面是如何創建混入的一個簡單例子("target": "es5")：

```ts
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    for (let id in first) {
        (<any>result)[id] = (<any>first)[id];
    }
    for (let id in second) {
        if (!result.hasOwnProperty(id)) {
            (<any>result)[id] = (<any>second)[id];
        }
    }
    return result;
}

class Person {
    constructor(public name: string) { }
}
interface Loggable {
    log(): void;
}
class ConsoleLogger implements Loggable {
    log() {
        // ...
    }
}
var jim = extend(new Person("Jim"), new ConsoleLogger());
var n = jim.name;
jim.log();
```

# 聯合類型（Union Types）

聯合類型與交叉類型很有關聯，但是使用上卻完全不同。
偶爾你會遇到這種情況，一個代碼庫希望傳入`number`或`string`類型的參數。
例如下面的函數：

```ts
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: any) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}

padLeft("Hello world", 4); // returns "    Hello world"
```

`padLeft`存在一個問題，`padding`參數的類型指定成了`any`。
這就是說我們可以傳入一個既不是`number`也不是`string`類型的參數，但是TypeScript卻不報錯。

```ts
let indentedString = padLeft("Hello world", true); // 編譯階段通過，運行時報錯
```

在傳統的面向對象語言裡，我們可能會將這兩種類型抽象成有層級的類型。
這麼做顯然是非常清晰的，但同時也存在了過度設計。
`padLeft`原始版本的好處之一是允許我們傳入原始類型。
這樣做的話使用起來既簡單又方便。
如果我們就是想使用已經存在的函數的話，這種新的方式就不適用了。

代替`any`， 我們可以使用*聯合類型*做為`padding`的參數：

```ts
/**
 * Takes a string and adds "padding" to the left.
 * If 'padding' is a string, then 'padding' is appended to the left side.
 * If 'padding' is a number, then that number of spaces is added to the left side.
 */
function padLeft(value: string, padding: string | number) {
    // ...
}

let indentedString = padLeft("Hello world", true); // errors during compilation
```

聯合類型表示一個值可以是幾種類型之一。
我們用豎線（`|`）分隔每個類型，所以`number | string | boolean`表示一個值可以是`number`，`string`，或`boolean`。

如果一個值是聯合類型，我們只能訪問此聯合類型的所有類型裡共有的成員。

```ts
interface Bird {
    fly();
    layEggs();
}

interface Fish {
    swim();
    layEggs();
}

function getSmallPet(): Fish | Bird {
    // ...
}

let pet = getSmallPet();
pet.layEggs(); // okay
pet.swim();    // errors
```

這裡的聯合類型可能有點複雜，但是你很容易就習慣了。
如果一個值的類型是`A | B`，我們能夠*確定*的是它包含了`A`*和*`B`中共有的成員。
這個例子裡，`Bird`具有一個`fly`成員。
我們不能確定一個`Bird | Fish`類型的變量是否有`fly`方法。
如果變量在運行時是`Fish`類型，那麼調用`pet.fly()`就出錯了。

# 類型守衛與類型區分（Type Guards and Differentiating Types）

聯合類型適合於那些值可以為不同類型的情況。
但當我們想確切地瞭解是否為`Fish`時怎麼辦？
JavaScript裡常用來區分2個可能值的方法是檢查成員是否存在。
如之前提及的，我們只能訪問聯合類型中共同擁有的成員。

```ts
let pet = getSmallPet();

// 每一個成員訪問都會報錯
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

為了讓這段代碼工作，我們要使用類型斷言：

```ts
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
```

## 用戶自定義的類型守衛

這裡可以注意到我們不得不多次使用類型斷言。
假若我們一旦檢查過類型，就能在之後的每個分支裡清楚地知道`pet`的類型的話就好了。

TypeScript裡的*類型守衛*機制讓它成為了現實。
類型守衛就是一些表達式，它們會在運行時檢查以確保在某個作用域裡的類型。
要定義一個類型守衛，我們只要簡單地定義一個函數，它的返回值是一個*類型謂詞*：

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```

在這個例子裡，`pet is Fish`就是類型謂詞。
謂詞為`parameterName is Type`這種形式，`parameterName`必須是來自於當前函數簽名裡的一個參數名。

每當使用一些變量調用`isFish`時，TypeScript會將變量縮減為那個具體的類型，只要這個類型與變量的原始類型是兼容的。

```ts
// 'swim' 和 'fly' 調用都沒有問題了

if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```

注意TypeScript不僅知道在`if`分支裡`pet`是`Fish`類型；
它還清楚在`else`分支裡，一定*不是*`Fish`類型，一定是`Bird`類型。

## `typeof`類型守衛

現在我們回過頭來看看怎麼使用聯合類型書寫`padLeft`代碼。
我們可以像下面這樣利用類型斷言來寫：

```ts
function isNumber(x: any): x is number {
    return typeof x === "number";
}

function isString(x: any): x is string {
    return typeof x === "string";
}

function padLeft(value: string, padding: string | number) {
    if (isNumber(padding)) {
        return Array(padding + 1).join(" ") + value;
    }
    if (isString(padding)) {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

然而，必須要定義一個函數來判斷類型是否是原始類型，這太痛苦了。
幸運的是，現在我們不必將`typeof x === "number"`抽象成一個函數，因為TypeScript可以將它識別為一個類型守衛。
也就是說我們可以直接在代碼裡檢查類型了。

```ts
function padLeft(value: string, padding: string | number) {
    if (typeof padding === "number") {
        return Array(padding + 1).join(" ") + value;
    }
    if (typeof padding === "string") {
        return padding + value;
    }
    throw new Error(`Expected string or number, got '${padding}'.`);
}
```

這些*`typeof`類型守衛*只有兩種形式能被識別：`typeof v === "typename"`和`typeof v !== "typename"`，`"typename"`必須是`"number"`，`"string"`，`"boolean"`或`"symbol"`。
但是TypeScript並不會阻止你與其它字符串比較，語言不會把那些表達式識別為類型守衛。

## `instanceof`類型守衛

如果你已經閱讀了`typeof`類型守衛並且對JavaScript裡的`instanceof`操作符熟悉的話，你可能已經猜到了這節要講的內容。

*`instanceof`類型守衛*是通過構造函數來細化類型的一種方式。
比如，我們借鑑一下之前字符串填充的例子：

```ts
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

function getRandomPadder() {
    return Math.random() < 0.5 ?
        new SpaceRepeatingPadder(4) :
        new StringPadder("  ");
}

// 類型為SpaceRepeatingPadder | StringPadder
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 類型細化為'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // 類型細化為'StringPadder'
}
```

`instanceof`的右側要求是一個構造函數，TypeScript將細化為：

1. 此構造函數的`prototype`屬性的類型，如果它的類型不為`any`的話
2. 構造簽名所返回的類型的聯合

以此順序。

# 可以為`null`的類型

TypeScript具有兩種特殊的類型，`null`和`undefined`，它們分別具有值`null`和`undefined`.
我們在[基礎類型](./Basic%20Types.md)一節裡已經做過簡要說明。
默認情況下，類型檢查器認為`null`與`undefined`可以賦值給任何類型。
`null`與`undefined`是所有其它類型的一個有效值。
這也意味著，你阻止不了將它們賦值給其它類型，就算是你想要阻止這種情況也不行。
`null`的發明者，Tony Hoare，稱它為[價值億萬美金的錯誤](https://en.wikipedia.org/wiki/Null_pointer#History)。

`--strictNullChecks`標記可以解決此錯誤：當你聲明一個變量時，它不會自動地包含`null`或`undefined`。
你可以使用聯合類型明確的包含它們：

```ts
let s = "foo";
s = null; // 錯誤, 'null'不能賦值給'string'
let sn: string | null = "bar";
sn = null; // 可以

sn = undefined; // error, 'undefined'不能賦值給'string | null'
```

注意，按照JavaScript的語義，TypeScript會把`null`和`undefined`區別對待。
`string | null`，`string | undefined`和`string | undefined | null`是不同的類型。

## 可選參數和可選屬性

使用了`--strictNullChecks`，可選參數會被自動地加上`| undefined`:

```ts
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // error, 'null' is not assignable to 'number | undefined'
```

可選屬性也會有同樣的處理：

```ts
class C {
    a: number;
    b?: number;
}
let c = new C();
c.a = 12;
c.a = undefined; // error, 'undefined' is not assignable to 'number'
c.b = 13;
c.b = undefined; // ok
c.b = null; // error, 'null' is not assignable to 'number | undefined'
```

## 類型守衛和類型斷言

由於可以為`null`的類型是通過聯合類型實現，那麼你需要使用類型守衛來去除`null`。
幸運地是這與在JavaScript裡寫的代碼一致：

```ts
function f(sn: string | null): string {
    if (sn == null) {
        return "default";
    }
    else {
        return sn;
    }
}
```

這裡很明顯地去除了`null`，你也可以使用短路運算符：

```ts
function f(sn: string | null): string {
    return sn || "default";
}
```

如果編譯器不能夠去除`null`或`undefined`，你可以使用類型斷言手動去除。
語法是添加`!`後綴：`identifier!`從`identifier`的類型裡去除了`null`和`undefined`：

```ts
function broken(name: string | null): string {
  function postfix(epithet: string) {
    return name.charAt(0) + '.  the ' + epithet; // error, 'name' is possibly null
  }
  name = name || "Bob";
  return postfix("great");
}

function fixed(name: string | null): string {
  function postfix(epithet: string) {
    return name!.charAt(0) + '.  the ' + epithet; // ok
  }
  name = name || "Bob";
  return postfix("great");
}
```

本例使用了嵌套函數，因為編譯器無法去除嵌套函數的`null`（除非是立即調用的函數表達式）。
因為它無法跟蹤所有對嵌套函數的調用，尤其是你將內層函數做為外層函數的返回值。
如果無法知道函數在哪裡被調用，就無法知道調用時`name`的類型。

# 類型別名

類型別名會給一個類型起個新名字。
類型別名有時和接口很像，但是可以作用於原始值，聯合類型，元組以及其它任何你需要手寫的類型。

```ts
type Name = string;
type NameResolver = () => string;
type NameOrResolver = Name | NameResolver;
function getName(n: NameOrResolver): Name {
    if (typeof n === 'string') {
        return n;
    }
    else {
        return n();
    }
}
```

起別名不會新建一個類型 - 它創建了一個新*名字*來引用那個類型。
給原始類型起別名通常沒什麼用，儘管可以做為文檔的一種形式使用。

同接口一樣，類型別名也可以是泛型 - 我們可以添加類型參數並且在別名聲明的右側傳入：

```ts
type Container<T> = { value: T };
```

我們也可以使用類型別名來在屬性裡引用自己：

```ts
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

與交叉類型一起使用，我們可以創建出一些十分稀奇古怪的類型。

```ts
type LinkedList<T> = T & { next: LinkedList<T> };

interface Person {
    name: string;
}

var people: LinkedList<Person>;
var s = people.name;
var s = people.next.name;
var s = people.next.next.name;
var s = people.next.next.next.name;
```

然而，類型別名不能出現在聲明右側的任何地方。

```ts
type Yikes = Array<Yikes>; // error
```

## 接口 vs. 類型別名

像我們提到的，類型別名可以像接口一樣；然而，仍有一些細微差別。

其一，接口創建了一個新的名字，可以在其它任何地方使用。
類型別名並不創建新名字&mdash;比如，錯誤信息就不會使用別名。
在下面的示例代碼裡，在編譯器中將鼠標懸停在`interfaced`上，顯示它返回的是`Interface`，但懸停在`aliased`上時，顯示的卻是對象字面量類型。

```ts
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```

另一個重要區別是類型別名不能被`extends`和`implements`（自己也不能`extends`和`implements`其它類型）。
因為[軟件中的對象應該對於擴展是開放的，但是對於修改是封閉的](https://en.wikipedia.org/wiki/Open/closed_principle)，你應該儘量去使用接口代替類型別名。

另一方面，如果你無法通過接口來描述一個類型並且需要使用聯合類型或元組類型，這時通常會使用類型別名。

# 字符串字面量類型

字符串字面量類型允許你指定字符串必須的固定值。
在實際應用中，字符串字面量類型可以與聯合類型，類型守衛和類型別名很好的配合。
通過結合使用這些特性，你可以實現類似枚舉類型的字符串。

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out";
class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement();
button.animate(0, 0, "ease-in");
button.animate(0, 0, "uneasy"); // error: "uneasy" is not allowed here
```

你只能從三種允許的字符中選擇其一來做為參數傳遞，傳入其它值則會產生錯誤。

```text
Argument of type '"uneasy"' is not assignable to parameter of type '"ease-in" | "ease-out" | "ease-in-out"'
```

字符串字面量類型還可以用於區分函數重載：

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
    // ... code goes here ...
}
```

# 數字字面量類型

TypeScript還具有數字字面量類型。

```ts
function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {
    // ...
}
```

我們很少直接這樣使用，但它們可以用在縮小範圍調試bug的時候：

```ts
function foo(x: number) {
    if (x !== 1 || x !== 2) {
        //         ~~~~~~~
        // Operator '!==' cannot be applied to types '1' and '2'.
    }
}
```

換句話說，當`x`與`2`進行比較的時候，它的值必須為`1`，這就意味著上面的比較檢查是非法的。

# 枚舉成員類型

如我們在[枚舉](./Enums.md#union-enums-and-enum-member-types)一節裡提到的，當每個枚舉成員都是用字面量初始化的時候枚舉成員是具有類型的。

在我們談及“單例類型”的時候，多數是指枚舉成員類型和數字/字符串字面量類型，儘管大多數用戶會互換使用“單例類型”和“字面量類型”。

# 可辨識聯合（Discriminated Unions）

你可以合併單例類型，聯合類型，類型守衛和類型別名來創建一個叫做*可辨識聯合*的高級模式，它也稱做*標籤聯合*或*代數數據類型*。
可辨識聯合在函數式編程裡很有用處。
一些語言會自動地為你辨識聯合；而TypeScript則基於已有的JavaScript模式。
它具有3個要素：

1. 具有普通的單例類型屬性&mdash;*可辨識的特徵*。
2. 一個類型別名包含了那些類型的聯合&mdash;*聯合*。
3. 此屬性上的類型守衛。

```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}
```

首先我們聲明了將要聯合的接口。
每個接口都有`kind`屬性但有不同的字符串字面量類型。
`kind`屬性稱做*可辨識的特徵*或*標籤*。
其它的屬性則特定於各個接口。
注意，目前各個接口間是沒有聯繫的。
下面我們把它們聯合到一起：

```ts
type Shape = Square | Rectangle | Circle;
```

現在我們使用可辨識聯合:

```ts
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

## 完整性檢查

當沒有涵蓋所有可辨識聯合的變化時，我們想讓編譯器可以通知我們。
比如，如果我們添加了`Triangle`到`Shape`，我們同時還需要更新`area`:

```ts
type Shape = Square | Rectangle | Circle | Triangle;
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
    // should error here - we didn't handle case "triangle"
}
```

有兩種方式可以實現。
首先是啟用`--strictNullChecks`並且指定一個返回值類型：

```ts
function area(s: Shape): number { // error: returns number | undefined
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
    }
}
```

因為`switch`沒有包含所有情況，所以TypeScript認為這個函數有時候會返回`undefined`。
如果你明確地指定了返回值類型為`number`，那麼你會看到一個錯誤，因為實際上返回值的類型為`number | undefined`。
然而，這種方法存在些微妙之處且`--strictNullChecks`對舊代碼支持不好。

第二種方法使用`never`類型，編譯器用它來進行完整性檢查：

```ts
function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}
function area(s: Shape) {
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
        default: return assertNever(s); // error here if there are missing cases
    }
}
```

這裡，`assertNever`檢查`s`是否為`never`類型&mdash;即為除去所有可能情況後剩下的類型。
如果你忘記了某個case，那麼`s`將具有一個真實的類型並且你會得到一個錯誤。
這種方式需要你定義一個額外的函數，但是在你忘記某個case的時候也更加明顯。

# 多態的`this`類型

多態的`this`類型表示的是某個包含類或接口的*子類型*。
這被稱做*F*-bounded多態性。
它能很容易的表現連貫接口間的繼承，比如。
在計算器的例子裡，在每個操作之後都返回`this`類型：

```ts
class BasicCalculator {
    public constructor(protected value: number = 0) { }
    public currentValue(): number {
        return this.value;
    }
    public add(operand: number): this {
        this.value += operand;
        return this;
    }
    public multiply(operand: number): this {
        this.value *= operand;
        return this;
    }
    // ... other operations go here ...
}

let v = new BasicCalculator(2)
            .multiply(5)
            .add(1)
            .currentValue();
```

由於這個類使用了`this`類型，你可以繼承它，新的類可以直接使用之前的方法，不需要做任何的改變。

```ts
class ScientificCalculator extends BasicCalculator {
    public constructor(value = 0) {
        super(value);
    }
    public sin() {
        this.value = Math.sin(this.value);
        return this;
    }
    // ... other operations go here ...
}

let v = new ScientificCalculator(2)
        .multiply(5)
        .sin()
        .add(1)
        .currentValue();
```

如果沒有`this`類型，`ScientificCalculator`就不能夠在繼承`BasicCalculator`的同時還保持接口的連貫性。
`multiply`將會返回`BasicCalculator`，它並沒有`sin`方法。
然而，使用`this`類型，`multiply`會返回`this`，在這裡就是`ScientificCalculator`。

# 索引類型（Index types）

使用索引類型，編譯器就能夠檢查使用了動態屬性名的代碼。
例如，一個常見的JavaScript模式是從對象中選取屬性的子集。

```js
function pluck(o, names) {
    return names.map(n => o[n]);
}
```

下面是如何在TypeScript裡使用此函數，通過**索引類型查詢**和**索引訪問**操作符：

```ts
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n]);
}

interface Person {
    name: string;
    age: number;
}
let person: Person = {
    name: 'Jarid',
    age: 35
};
let strings: string[] = pluck(person, ['name']); // ok, string[]
```

編譯器會檢查`name`是否真的是`Person`的一個屬性。
本例還引入了幾個新的類型操作符。
首先是`keyof T`，**索引類型查詢操作符**。
對於任何類型`T`，`keyof T`的結果為`T`上已知的公共屬性名的聯合。
例如：

```ts
let personProps: keyof Person; // 'name' | 'age'
```

`keyof Person`是完全可以與`'name' | 'age'`互相替換的。
不同的是如果你添加了其它的屬性到`Person`，例如`address: string`，那麼`keyof Person`會自動變為`'name' | 'age' | 'address'`。
你可以在像`pluck`函數這類上下文裡使用`keyof`，因為在使用之前你並不清楚可能出現的屬性名。
但編譯器會檢查你是否傳入了正確的屬性名給`pluck`：

```ts
pluck(person, ['age', 'unknown']); // error, 'unknown' is not in 'name' | 'age'
```

第二個操作符是`T[K]`，**索引訪問操作符**。
在這裡，類型語法反映了表達式語法。
這意味著`person['name']`具有類型`Person['name']` &mdash; 在我們的例子裡則為`string`類型。
然而，就像索引類型查詢一樣，你可以在普通的上下文裡使用`T[K]`，這正是它的強大所在。
你只要確保類型變量`K extends keyof T`就可以了。
例如下面`getProperty`函數的例子：

```ts
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
```

`getProperty`裡的`o: T`和`name: K`，意味著`o[name]: T[K]`。
當你返回`T[K]`的結果，編譯器會實例化鍵的真實類型，因此`getProperty`的返回值類型會隨著你需要的屬性改變。

```ts
let name: string = getProperty(person, 'name');
let age: number = getProperty(person, 'age');
let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```

## 索引類型和字符串索引簽名

`keyof`和`T[K]`與字符串索引簽名進行交互。
如果你有一個帶有字符串索引簽名的類型，那麼`keyof T`會是`string`。
並且`T[string]`為索引簽名的類型：

```ts
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```

# 映射類型

一個常見的任務是將一個已知的類型每個屬性都變為可選的：

```ts
interface PersonPartial {
    name?: string;
    age?: number;
}
```

或者我們想要一個只讀版本：

```ts
interface PersonReadonly {
    readonly name: string;
    readonly age: number;
}
```

這在JavaScript裡經常出現，TypeScript提供了從舊類型中創建新類型的一種方式 &mdash; **映射類型**。
在映射類型裡，新類型以相同的形式去轉換舊類型裡每個屬性。
例如，你可以令每個屬性成為`readonly`類型或可選的。
下面是一些例子：

```ts
type Readonly<T> = {
    readonly [P in keyof T]: T[P];
}
type Partial<T> = {
    [P in keyof T]?: T[P];
}
```

像下面這樣使用：

```ts
type PersonPartial = Partial<Person>;
type ReadonlyPerson = Readonly<Person>;
```

下面來看看最簡單的映射類型和它的組成部分：

```ts
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
```

它的語法與索引簽名的語法類型，內部使用了`for .. in`。
具有三個部分：

1. 類型變量`K`，它會依次綁定到每個屬性。
2. 字符串字面量聯合的`Keys`，它包含了要迭代的屬性名的集合。
3. 屬性的結果類型。

在個簡單的例子裡，`Keys`是硬編碼的屬性名列表並且屬性類型永遠是`boolean`，因此這個映射類型等同於：

```ts
type Flags = {
    option1: boolean;
    option2: boolean;
}
```

在真正的應用裡，可能不同於上面的`Readonly`或`Partial`。
它們會基於一些已存在的類型，且按照一定的方式轉換字段。
這就是`keyof`和索引訪問類型要做的事情：

```ts
type NullablePerson = { [P in keyof Person]: Person[P] | null }
type PartialPerson = { [P in keyof Person]?: Person[P] }
```

但它更有用的地方是可以有一些通用版本。

```ts
type Nullable<T> = { [P in keyof T]: T[P] | null }
type Partial<T> = { [P in keyof T]?: T[P] }
```

在這些例子裡，屬性列表是`keyof T`且結果類型是`T[P]`的變體。
這是使用通用映射類型的一個好模版。
因為這類轉換是[同態](https://en.wikipedia.org/wiki/Homomorphism)的，映射只作用於`T`的屬性而沒有其它的。
編譯器知道在添加任何新屬性之前可以拷貝所有存在的屬性修飾符。
例如，假設`Person.name`是只讀的，那麼`Partial<Person>.name`也將是只讀的且為可選的。

下面是另一個例子，`T[P]`被包裝在`Proxy<T>`類裡：

```ts
type Proxy<T> = {
    get(): T;
    set(value: T): void;
}
type Proxify<T> = {
    [P in keyof T]: Proxy<T[P]>;
}
function proxify<T>(o: T): Proxify<T> {
   // ... wrap proxies ...
}
let proxyProps = proxify(props);
```

注意`Readonly<T>`和`Partial<T>`用處不小，因此它們與`Pick`和`Record`一同被包含進了TypeScript的標準庫裡：

```ts
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
type Record<K extends string, T> = {
    [P in K]: T;
}
```

`Readonly`，`Partial`和`Pick`是同態的，但`Record`不是。
因為`Record`並不需要輸入類型來拷貝屬性，所以它不屬於同態：

```ts
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```

非同態類型本質上會創建新的屬性，因此它們不會從它處拷貝屬性修飾符。

## 由映射類型進行推斷

現在你瞭解了如何包裝一個類型的屬性，那麼接下來就是如何拆包。
其實這也非常容易：

```ts
function unproxify<T>(t: Proxify<T>): T {
    let result = {} as T;
    for (const k in t) {
        result[k] = t[k].get();
    }
    return result;
}

let originalProps = unproxify(proxyProps);
```

注意這個拆包推斷只適用於同態的映射類型。
如果映射類型不是同態的，那麼需要給拆包涵數一個明確的類型參數。

## 有條件類型

TypeScript 2.8引入了*有條件類型*，它能夠表示非統一的類型。
有條件的類型會以一個條件表達式進行類型關係檢測，從而在兩種類型中選擇其一：

```ts
T extends U ? X : Y
```

上面的類型意思是，若`T`能夠賦值給`U`，那麼類型是`X`，否則為`Y`。

有條件的類型`T extends U ? X : Y`或者*解析*為`X`，或者*解析*為`Y`，再或者*延遲*解析，因為它可能依賴一個或多個類型變量。
若`T`或`U`包含類型參數，那麼是否解析為`X`或`Y`或推遲，取決於類型系統是否有足夠的信息來確定`T`總是可以賦值給`U`。

下面是一些類型可以被立即解析的例子：

```ts
declare function f<T extends boolean>(x: T): T extends true ? string : number;

// Type is 'string | number
let x = f(Math.random() < 0.5)

```

另外一個例子涉及`TypeName`類型別名，它使用了嵌套了有條件類型：

```ts
type TypeName<T> =
    T extends string ? "string" :
    T extends number ? "number" :
    T extends boolean ? "boolean" :
    T extends undefined ? "undefined" :
    T extends Function ? "function" :
    "object";

type T0 = TypeName<string>;  // "string"
type T1 = TypeName<"a">;  // "string"
type T2 = TypeName<true>;  // "boolean"
type T3 = TypeName<() => void>;  // "function"
type T4 = TypeName<string[]>;  // "object"
```

下面是一個有條件類型被推遲解析的例子:

```ts
interface Foo {
    propA: boolean;
    propB: boolean;
}

declare function f<T>(x: T): T extends Foo ? string : number;

function foo<U>(x: U) {
    // Has type 'U extends Foo ? string : number'
    let a = f(x);

    // This assignment is allowed though!
    let b: string | number = a;
}
```

這裡，`a`變量含有未確定的有條件類型。
當有另一段代碼調用`foo`，它會用其它類型替換`U`，TypeScript將重新計算有條件類型，決定它是否可以選擇一個分支。

與此同時，我們可以將有條件類型賦值給其它類型，只要有條件類型的每個分支都可以賦值給目標類型。
因此在我們的例子裡，我們可以將`U extends Foo ? string : number`賦值給`string | number`，因為不管這個有條件類型最終結果是什麼，它只能是`string`或`number`。

### 分佈式有條件類型

如果有條件類型裡待檢查的類型是`naked type parameter`，那麼它也被稱為“分佈式有條件類型”。
分佈式有條件類型在實例化時會自動分發成聯合類型。
例如，實例化`T extends U ? X : Y`，`T`的類型為`A | B | C`，會被解析為`(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`。

#### 例子

```ts
type T10 = TypeName<string | (() => void)>;  // "string" | "function"
type T12 = TypeName<string | string[] | undefined>;  // "string" | "object" | "undefined"
type T11 = TypeName<string[] | number[]>;  // "object"
```

在`T extends U ? X : Y`的實例化裡，對`T`的引用被解析為聯合類型的一部分（比如，`T`指向某一單個部分，在有條件類型分佈到聯合類型之後）。
此外，在`X`內對`T`的引用有一個附加的類型參數約束`U`（例如，`T`被當成在`X`內可賦值給`U`）。

#### 例子

```ts
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T20 = Boxed<string>;  // BoxedValue<string>;
type T21 = Boxed<number[]>;  // BoxedArray<number>;
type T22 = Boxed<string | number[]>;  // BoxedValue<string> | BoxedArray<number>;
```

注意在`Boxed<T>`的`true`分支裡，`T`有個額外的約束`any[]`，因此它適用於`T[number]`數組元素類型。同時也注意一下有條件類型是如何分佈成聯合類型的。

有條件類型的分佈式的屬性可以方便地用來*過濾*聯合類型：

```ts
type Diff<T, U> = T extends U ? never : T;  // Remove types from T that are assignable to U
type Filter<T, U> = T extends U ? T : never;  // Remove types from T that are not assignable to U

type T30 = Diff<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
type T31 = Filter<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"
type T32 = Diff<string | number | (() => void), Function>;  // string | number
type T33 = Filter<string | number | (() => void), Function>;  // () => void

type NonNullable<T> = Diff<T, null | undefined>;  // Remove null and undefined from T

type T34 = NonNullable<string | number | undefined>;  // string | number
type T35 = NonNullable<string | string[] | null | undefined>;  // string | string[]

function f1<T>(x: T, y: NonNullable<T>) {
    x = y;  // Ok
    y = x;  // Error
}

function f2<T extends string | undefined>(x: T, y: NonNullable<T>) {
    x = y;  // Ok
    y = x;  // Error
    let s1: string = x;  // Error
    let s2: string = y;  // Ok
}
```

有條件類型與映射類型結合時特別有用：

```ts
type FunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? K : never }[keyof T];
type FunctionProperties<T> = Pick<T, FunctionPropertyNames<T>>;

type NonFunctionPropertyNames<T> = { [K in keyof T]: T[K] extends Function ? never : K }[keyof T];
type NonFunctionProperties<T> = Pick<T, NonFunctionPropertyNames<T>>;

interface Part {
    id: number;
    name: string;
    subparts: Part[];
    updatePart(newName: string): void;
}

type T40 = FunctionPropertyNames<Part>;  // "updatePart"
type T41 = NonFunctionPropertyNames<Part>;  // "id" | "name" | "subparts"
type T42 = FunctionProperties<Part>;  // { updatePart(newName: string): void }
type T43 = NonFunctionProperties<Part>;  // { id: number, name: string, subparts: Part[] }
```

與聯合類型和交叉類型相似，有條件類型不允許遞歸地引用自己。比如下面的錯誤。

#### 例子

```ts
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T;  // Error
```

### 有條件類型中的類型推斷

現在在有條件類型的`extends`子語句中，允許出現`infer`聲明，它會引入一個待推斷的類型變量。
這個推斷的類型變量可以在有條件類型的true分支中被引用。
允許出現多個同類型變量的`infer`。

例如，下面代碼會提取函數類型的返回值類型：

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

有條件類型可以嵌套來構成一系列的匹配模式，按順序進行求值：

```ts
type Unpacked<T> =
    T extends (infer U)[] ? U :
    T extends (...args: any[]) => infer U ? U :
    T extends Promise<infer U> ? U :
    T;

type T0 = Unpacked<string>;  // string
type T1 = Unpacked<string[]>;  // string
type T2 = Unpacked<() => string>;  // string
type T3 = Unpacked<Promise<string>>;  // string
type T4 = Unpacked<Promise<string>[]>;  // Promise<string>
type T5 = Unpacked<Unpacked<Promise<string>[]>>;  // string
```

下面的例子解釋了在協變位置上，同一個類型變量的多個候選類型會被推斷為聯合類型：

```ts
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number
```

相似地，在抗變位置上，同一個類型變量的多個候選類型會被推斷為交叉類型：

```ts
type Bar<T> = T extends { a: (x: infer U) => void, b: (x: infer U) => void } ? U : never;
type T20 = Bar<{ a: (x: string) => void, b: (x: string) => void }>;  // string
type T21 = Bar<{ a: (x: string) => void, b: (x: number) => void }>;  // string & number
```

當推斷具有多個調用簽名（例如函數重載類型）的類型時，用*最後*的簽名（大概是最自由的包含所有情況的簽名）進行推斷。
無法根據參數類型列表來解析重載。

```ts
declare function foo(x: string): number;
declare function foo(x: number): string;
declare function foo(x: string | number): string | number;
type T30 = ReturnType<typeof foo>;  // string | number
```

無法在正常類型參數的約束子語句中使用`infer`聲明：

```ts
type ReturnType<T extends (...args: any[]) => infer R> = R;  // 錯誤，不支持
```

但是，可以這樣達到同樣的效果，在約束裡刪掉類型變量，用有條件類型替換：

```ts
type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R ? R : any;
```

### 預定義的有條件類型

TypeScript 2.8在`lib.d.ts`裡增加了一些預定義的有條件類型：

* `Exclude<T, U>` -- 從`T`中剔除可以賦值給`U`的類型。
* `Extract<T, U>` -- 提取`T`中可以賦值給`U`的類型。
* `NonNullable<T>` -- 從`T`中剔除`null`和`undefined`。
* `ReturnType<T>` -- 獲取函數返回值類型。
* `InstanceType<T>` -- 獲取構造函數類型的實例類型。

#### Example

```ts
type T00 = Exclude<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "b" | "d"
type T01 = Extract<"a" | "b" | "c" | "d", "a" | "c" | "f">;  // "a" | "c"

type T02 = Exclude<string | number | (() => void), Function>;  // string | number
type T03 = Extract<string | number | (() => void), Function>;  // () => void

type T04 = NonNullable<string | number | undefined>;  // string | number
type T05 = NonNullable<(() => string) | string[] | null | undefined>;  // (() => string) | string[]

function f1(s: string) {
    return { a: 1, b: s };
}

class C {
    x = 0;
    y = 0;
}

type T10 = ReturnType<() => string>;  // string
type T11 = ReturnType<(s: string) => void>;  // void
type T12 = ReturnType<(<T>() => T)>;  // {}
type T13 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T14 = ReturnType<typeof f1>;  // { a: number, b: string }
type T15 = ReturnType<any>;  // any
type T16 = ReturnType<never>;  // any
type T17 = ReturnType<string>;  // Error
type T18 = ReturnType<Function>;  // Error

type T20 = InstanceType<typeof C>;  // C
type T21 = InstanceType<any>;  // any
type T22 = InstanceType<never>;  // any
type T23 = InstanceType<string>;  // Error
type T24 = InstanceType<Function>;  // Error
```

> 注意：`Exclude`類型是[建議的](https://github.com/Microsoft/TypeScript/issues/12215#issuecomment-307871458)`Diff`類型的一種實現。我們使用`Exclude`這個名字是為了避免破壞已經定義了`Diff`的代碼，並且我們感覺這個名字能更好地表達類型的語義。我們沒有增加`Omit<T, K>`類型，因為它可以很容易的用`Pick<T, Exclude<keyof T, K>>`來表示。