# 交叉型別(Intersection Types)

交叉型別是將多個型別合併為一個型別。
這讓我們可以把現有的多種型別疊加到一起成為一種型別，它包含了所需的所有型別的特性。
例如，`Person & Serializable & Loggable`同時是`Person`*和*`Serializable`*和*`Loggable`。
就是說這個型別的物件同時擁有了這三種型別的成員。

我們大多是在混入(mixins)或其它不適合典型物件導向模型的地方看到交叉型別的使用。
(在JavaScript裡發生這種情況的場合很多！)
下面是如何建立混入的一個簡單範例("target": "es5")：

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

# 聯合型別(Union Types)

聯合型別與交叉型別很有關聯，但是使用上卻完全不同。
偶爾您會遇到這種情況，一個程式碼庫希望傳入`number`或`string`型別的參數。
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

`padLeft`存在一個問題，`padding`參數的型別指定成了`any`。
這就是說我們可以傳入一個既不是`number`也不是`string`型別的參數，但是TypeScript卻不報錯。

```ts
let indentedString = padLeft("Hello world", true); // 編譯階段透過，執行時報錯
```

在傳統的物件導向語言裡，我們可能會將這兩種型別抽象成有層級的型別。
這麼做顯然是非常清晰的，但同時也存在了過度設計。
`padLeft`原始版本的好處之一是允許我們傳入原始型別。
這樣做的話使用起來既簡單又方便。
如果我們就是想使用已經存在的函數的話，這種新的方式就不適用了。

代替`any`， 我們可以使用*聯合型別*做為`padding`的參數：

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

聯合型別表示一個值可以是幾種型別之一。
我們用豎線(`|`)分隔每個型別，所以`number | string | boolean`表示一個值可以是`number`，`string`，或`boolean`。

如果一個值是聯合型別，我們只能存取此聯合型別的所有型別裡共有的成員。

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

這裡的聯合型別可能有點複雜，但是您很容易就習慣了。
如果一個值的型別是`A | B`，我們能夠*確定*的是它包含了`A`*和*`B`中共有的成員。
這個範例裡，`Bird`具有一個`fly`成員。
我們不能確定一個`Bird | Fish`型別的變數是否有`fly`方法。
如果變數在執行時是`Fish`型別，那麼呼叫`pet.fly()`就出錯了。

# 型別守衛與型別區分(Type Guards and Differentiating Types)

聯合型別適合於那些值可以為不同型別的情況。
但當我們想確切地瞭解是否為`Fish`時怎麼辦？
JavaScript裡常用來區分2個可能值的方法是檢查成員是否存在。
如之前提及的，我們只能存取聯合型別中共同擁有的成員。

```ts
let pet = getSmallPet();

// 每一個成員存取都會報錯
if (pet.swim) {
    pet.swim();
}
else if (pet.fly) {
    pet.fly();
}
```

為了讓這段程式碼工作，我們要使用型別斷言：

```ts
let pet = getSmallPet();

if ((<Fish>pet).swim) {
    (<Fish>pet).swim();
}
else {
    (<Bird>pet).fly();
}
```

## 用戶自訂的型別守衛

這裡可以注意到我們不得不多次使用型別斷言。
假若我們一旦檢查過型別，就能在之後的每個分支裡清楚地知道`pet`的型別的話就好了。

TypeScript裡的*型別守衛*機制讓它成為了現實。
型別守衛就是一些運算式，它們會在執行時檢查以確保在某個作用域裡的型別。
要定義一個型別守衛，我們只要簡單地定義一個函數，它的返回值是一個*型別謂詞*：

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim !== undefined;
}
```

在這個範例裡，`pet is Fish`就是型別謂詞。
謂詞為`parameterName is Type`這種形式，`parameterName`必須是來自於當前函數簽名裡的一個參數名。

每當使用一些變數呼叫`isFish`時，TypeScript會將變數縮減為那個具體的型別，只要這個型別與變數的原始型別是相容的。

```ts
// 'swim' 和 'fly' 呼叫都沒有問題了

if (isFish(pet)) {
    pet.swim();
}
else {
    pet.fly();
}
```

注意TypeScript不僅知道在`if`分支裡`pet`是`Fish`型別；
它還清楚在`else`分支裡，一定*不是*`Fish`型別，一定是`Bird`型別。

## `typeof`型別守衛

現在我們回過頭來看看怎麼使用聯合型別書寫`padLeft`程式碼。
我們可以像下面這樣利用型別斷言來寫：

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

然而，必須要定義一個函數來判斷型別是否是原始型別，這太痛苦了。
幸運的是，現在我們不必將`typeof x === "number"`抽象成一個函數，因為TypeScript可以將它識別為一個型別守衛。
也就是說我們可以直接在程式碼裡檢查型別了。

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

這些*`typeof`型別守衛*只有兩種形式能被識別：`typeof v === "typename"`和`typeof v !== "typename"`，`"typename"`必須是`"number"`，`"string"`，`"boolean"`或`"symbol"`。
但是TypeScript並不會阻止您與其它字串比較，語言不會把那些運算式識別為型別守衛。

## `instanceof`型別守衛

如果您已經閱讀了`typeof`型別守衛並且對JavaScript裡的`instanceof`操作符熟悉的話，您可能已經猜到了這節要講的內容。

*`instanceof`型別守衛*是透過建構函數來細化型別的一種方式。
比如，我們借鑑一下之前字串填充的範例：

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

// 型別為SpaceRepeatingPadder | StringPadder
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 型別細化為'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // 型別細化為'StringPadder'
}
```

`instanceof`的右側要求是一個建構函數，TypeScript將細化為：

1. 此建構函數的`prototype`屬性的型別，如果它的型別不為`any`的話
2. 構造簽名所返回的型別的聯合

以此順序。

# 可以為`null`的型別

TypeScript具有兩種特殊的型別，`null`和`undefined`，它們分別具有值`null`和`undefined`.
我們在[基礎型別](./Basic%20Types.md)一節裡已經做過簡要說明。
預設情況下，型別檢查器認為`null`與`undefined`可以賦值給任何型別。
`null`與`undefined`是所有其它型別的一個有效值。
這也意味著，您阻止不了將它們賦值給其它型別，就算是您想要阻止這種情況也不行。
`null`的發明者，Tony Hoare，稱它為[價值億萬美金的錯誤](https://en.wikipedia.org/wiki/Null_pointer#History)。

`--strictNullChecks`標記可以解決此錯誤：當您宣告一個變數時，它不會自動地包含`null`或`undefined`。
您可以使用聯合型別明確的包含它們：

```ts
let s = "foo";
s = null; // 錯誤, 'null'不能賦值給'string'
let sn: string | null = "bar";
sn = null; // 可以

sn = undefined; // error, 'undefined'不能賦值給'string | null'
```

注意，按照JavaScript的語義，TypeScript會把`null`和`undefined`區別對待。
`string | null`，`string | undefined`和`string | undefined | null`是不同的型別。

## 選擇性參數和選擇性屬性

使用了`--strictNullChecks`，選擇性參數會被自動地加上`| undefined`:

```ts
function f(x: number, y?: number) {
    return x + (y || 0);
}
f(1, 2);
f(1);
f(1, undefined);
f(1, null); // error, 'null' is not assignable to 'number | undefined'
```

選擇性屬性也會有同樣的處理：

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

## 型別守衛和型別斷言

由於可以為`null`的型別是透過聯合型別實現，那麼您需要使用型別守衛來去除`null`。
幸運地是這與在JavaScript裡寫的程式碼一致：

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

這裡很明顯地去除了`null`，您也可以使用短路運算符：

```ts
function f(sn: string | null): string {
    return sn || "default";
}
```

如果編譯器不能夠去除`null`或`undefined`，您可以使用型別斷言手動去除。
語法是加入`!`後綴：`identifier!`從`identifier`的型別裡去除了`null`和`undefined`：

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

本例使用了嵌套函數，因為編譯器無法去除嵌套函數的`null`(除非是立即呼叫的函數運算式)。
因為它無法跟蹤所有對嵌套函數的呼叫，尤其是您將內層函數做為外層函數的返回值。
如果無法知道函數在哪裡被呼叫，就無法知道呼叫時`name`的型別。

# 型別別名

型別別名會給一個型別起個新名字。
型別別名有時和介面很像，但是可以作用於原始值，聯合型別，元組以及其它任何您需要手寫的型別。

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

起別名不會新建一個型別 - 它建立了一個新*名字*來引用那個型別。
給原始型別起別名通常沒什麼用，儘管可以做為文件的一種形式使用。

同介面一樣，型別別名也可以是泛型 - 我們可以加入型別參數並且在別名宣告的右側傳入：

```ts
type Container<T> = { value: T };
```

我們也可以使用型別別名來在屬性裡引用自己：

```ts
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

與交叉型別一起使用，我們可以建立出一些十分稀奇古怪的型別。

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

然而，型別別名不能出現在宣告右側的任何地方。

```ts
type Yikes = Array<Yikes>; // error
```

## 介面 vs. 型別別名

像我們提到的，型別別名可以像介面一樣；然而，仍有一些細微差別。

其一，介面建立了一個新的名字，可以在其它任何地方使用。
型別別名並不建立新名字&mdash;比如，錯誤訊息就不會使用別名。
在下面的範例程式碼裡，在編譯器中將滑鼠懸停在`interfaced`上，顯示它返回的是`Interface`，但懸停在`aliased`上時，顯示的卻是物件字面量型別。

```ts
type Alias = { num: number }
interface Interface {
    num: number;
}
declare function aliased(arg: Alias): Alias;
declare function interfaced(arg: Interface): Interface;
```

另一個重要區別是型別別名不能被`extends`和`implements`(自己也不能`extends`和`implements`其它型別)。
因為[軟體中的物件應該對於擴展是開放的，但是對於修改是封閉的](https://en.wikipedia.org/wiki/Open/closed_principle)，您應該儘量去使用介面代替型別別名。

另一方面，如果您無法透過介面來描述一個型別並且需要使用聯合型別或元組型別，這時通常會使用型別別名。

# 字串字面量型別

字串字面量型別允許您指定字串必須的固定值。
在實際應用中，字串字面量型別可以與聯合型別，型別守衛和型別別名很好的配合。
透過結合使用這些特性，您可以實現類似列舉型別的字串。

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

您只能從三種允許的字符中選擇其一來做為參數傳遞，傳入其它值則會產生錯誤。

```text
Argument of type '"uneasy"' is not assignable to parameter of type '"ease-in" | "ease-out" | "ease-in-out"'
```

字串字面量型別還可以用於區分函數重載：

```ts
function createElement(tagName: "img"): HTMLImageElement;
function createElement(tagName: "input"): HTMLInputElement;
// ... more overloads ...
function createElement(tagName: string): Element {
    // ... code goes here ...
}
```

# 數字字面量型別

TypeScript還具有數字字面量型別。

```ts
function rollDie(): 1 | 2 | 3 | 4 | 5 | 6 {
    // ...
}
```

我們很少直接這樣使用，但它們可以用在縮小範圍偵錯bug的時候：

```ts
function foo(x: number) {
    if (x !== 1 || x !== 2) {
        //         ~~~~~~~
        // Operator '!==' cannot be applied to types '1' and '2'.
    }
}
```

換句話說，當`x`與`2`進行比較的時候，它的值必須為`1`，這就意味著上面的比較檢查是非法的。

# 列舉成員型別

如我們在[列舉](./Enums.md#union-enums-and-enum-member-types)一節裡提到的，當每個列舉成員都是用字面量初始化的時候列舉成員是具有型別的。

在我們談及「單例型別」的時候，多數是指列舉成員型別和數字/字串字面量型別，儘管大多數用戶會互換使用「單例型別」和「字面量型別」。

# 可辨識聯合(Discriminated Unions)

您可以合併單例型別，聯合型別，型別守衛和型別別名來建立一個叫做*可辨識聯合*的高級模式，它也稱做*標籤聯合*或*代數資料型別*。
可辨識聯合在函數式編程裡很有用處。
一些語言會自動地為您辨識聯合；而TypeScript則基於已有的JavaScript模式。
它具有3個要素：

1. 具有普通的單例型別屬性&mdash;*可辨識的特性*。
2. 一個型別別名包含了那些型別的聯合&mdash;*聯合*。
3. 此屬性上的型別守衛。

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

首先我們宣告了將要聯合的介面。
每個介面都有`kind`屬性但有不同的字串字面量型別。
`kind`屬性稱做*可辨識的特性*或*標籤*。
其它的屬性則特定於各個介面。
注意，目前各個介面間是沒有聯繫的。
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
比如，如果我們加入了`Triangle`到`Shape`，我們同時還需要更新`area`:

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
首先是啟用`--strictNullChecks`並且指定一個返回值型別：

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
如果您明確地指定了返回值型別為`number`，那麼您會看到一個錯誤，因為實際上返回值的型別為`number | undefined`。
然而，這種方法存在些微妙之處且`--strictNullChecks`對舊程式碼支援不好。

第二種方法使用`never`型別，編譯器用它來進行完整性檢查：

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

這裡，`assertNever`檢查`s`是否為`never`型別&mdash;即為除去所有可能情況後剩下的型別。
如果您忘記了某個case，那麼`s`將具有一個真實的型別並且您會得到一個錯誤。
這種方式需要您定義一個額外的函數，但是在您忘記某個case的時候也更加明顯。

# 多態的`this`型別

多態的`this`型別表示的是某個包含類或介面的*子型別*。
這被稱做*F*-bounded多態性。
它能很容易的表現連貫介面間的繼承，比如。
在計算器的範例裡，在每個操作之後都返回`this`型別：

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

由於這個類使用了`this`型別，您可以繼承它，新的類可以直接使用之前的方法，不需要做任何的改變。

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

如果沒有`this`型別，`ScientificCalculator`就不能夠在繼承`BasicCalculator`的同時還保持介面的連貫性。
`multiply`將會返回`BasicCalculator`，它並沒有`sin`方法。
然而，使用`this`型別，`multiply`會返回`this`，在這裡就是`ScientificCalculator`。

# 索引型別(Index types)

使用索引型別，編譯器就能夠檢查使用了動態屬性名的程式碼。
例如，一個常見的JavaScript模式是從物件中選取屬性的子集。

```js
function pluck(o, names) {
    return names.map(n => o[n]);
}
```

下面是如何在TypeScript裡使用此函數，透過**索引型別查詢**和**索引存取**操作符：

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
本例還引入了幾個新的型別操作符。
首先是`keyof T`，**索引型別查詢操作符**。
對於任何型別`T`，`keyof T`的結果為`T`上已知的公共屬性名的聯合。
例如：

```ts
let personProps: keyof Person; // 'name' | 'age'
```

`keyof Person`是完全可以與`'name' | 'age'`互相替換的。
不同的是如果您加入了其它的屬性到`Person`，例如`address: string`，那麼`keyof Person`會自動變為`'name' | 'age' | 'address'`。
您可以在像`pluck`函數這類上下文裡使用`keyof`，因為在使用之前您並不清楚可能出現的屬性名。
但編譯器會檢查您是否傳入了正確的屬性名給`pluck`：

```ts
pluck(person, ['age', 'unknown']); // error, 'unknown' is not in 'name' | 'age'
```

第二個操作符是`T[K]`，**索引存取操作符**。
在這裡，型別語法反映了運算式語法。
這意味著`person['name']`具有型別`Person['name']` &mdash; 在我們的範例裡則為`string`型別。
然而，就像索引型別查詢一樣，您可以在普通的上下文裡使用`T[K]`，這正是它的強大所在。
您只要確保型別變數`K extends keyof T`就可以了。
例如下面`getProperty`函數的範例：

```ts
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
```

`getProperty`裡的`o: T`和`name: K`，意味著`o[name]: T[K]`。
當您返回`T[K]`的結果，編譯器會實例化鍵的真實型別，因此`getProperty`的返回值型別會隨著您需要的屬性改變。

```ts
let name: string = getProperty(person, 'name');
let age: number = getProperty(person, 'age');
let unknown = getProperty(person, 'unknown'); // error, 'unknown' is not in 'name' | 'age'
```

## 索引型別和字串索引簽名

`keyof`和`T[K]`與字串索引簽名進行交互。
如果您有一個帶有字串索引簽名的型別，那麼`keyof T`會是`string`。
並且`T[string]`為索引簽名的型別：

```ts
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```

# 映射型別

一個常見的任務是將一個已知的型別每個屬性都變為選擇性的：

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

這在JavaScript裡經常出現，TypeScript提供了從舊型別中建立新型別的一種方式 &mdash; **映射型別**。
在映射型別裡，新型別以相同的形式去轉換舊型別裡每個屬性。
例如，您可以令每個屬性成為`readonly`型別或選擇性的。
下面是一些範例：

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

下面來看看最簡單的映射型別和它的組成部分：

```ts
type Keys = 'option1' | 'option2';
type Flags = { [K in Keys]: boolean };
```

它的語法與索引簽名的語法型別，內部使用了`for .. in`。
具有三個部分：

1. 型別變數`K`，它會依次繫結到每個屬性。
2. 字串字面量聯合的`Keys`，它包含了要迭代的屬性名的集合。
3. 屬性的結果型別。

在個簡單的範例裡，`Keys`是硬編碼的屬性名列表並且屬性型別永遠是`boolean`，因此這個映射型別等同於：

```ts
type Flags = {
    option1: boolean;
    option2: boolean;
}
```

在真正的應用裡，可能不同於上面的`Readonly`或`Partial`。
它們會基於一些已存在的型別，且按照一定的方式轉換欄位。
這就是`keyof`和索引存取型別要做的事情：

```ts
type NullablePerson = { [P in keyof Person]: Person[P] | null }
type PartialPerson = { [P in keyof Person]?: Person[P] }
```

但它更有用的地方是可以有一些通用版本。

```ts
type Nullable<T> = { [P in keyof T]: T[P] | null }
type Partial<T> = { [P in keyof T]?: T[P] }
```

在這些範例裡，屬性列表是`keyof T`且結果型別是`T[P]`的變體。
這是使用通用映射型別的一個好模版。
因為這類轉換是[同態](https://en.wikipedia.org/wiki/Homomorphism)的，映射只作用於`T`的屬性而沒有其它的。
編譯器知道在加入任何新屬性之前可以拷貝所有存在的屬性修飾符。
例如，假設`Person.name`是只讀的，那麼`Partial<Person>.name`也將是只讀的且為選擇性的。

下面是另一個範例，`T[P]`被包裝在`Proxy<T>`類裡：

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
因為`Record`並不需要輸入型別來拷貝屬性，所以它不屬於同態：

```ts
type ThreeStringProps = Record<'prop1' | 'prop2' | 'prop3', string>
```

非同態型別本質上會建立新的屬性，因此它們不會從它處拷貝屬性修飾符。

## 由映射型別進行推斷

現在您瞭解了如何包裝一個型別的屬性，那麼接下來就是如何拆包。
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

注意這個拆包推斷只適用於同態的映射型別。
如果映射型別不是同態的，那麼需要給拆包涵數一個明確的型別參數。

## 有條件型別

TypeScript 2.8引入了*有條件型別*，它能夠表示非統一的型別。
有條件的型別會以一個條件運算式進行型別關係檢測，從而在兩種型別中選擇其一：

```ts
T extends U ? X : Y
```

上面的型別意思是，若`T`能夠賦值給`U`，那麼型別是`X`，否則為`Y`。

有條件的型別`T extends U ? X : Y`或者*解析*為`X`，或者*解析*為`Y`，再或者*延遲*解析，因為它可能依賴一個或多個型別變數。
若`T`或`U`包含型別參數，那麼是否解析為`X`或`Y`或推遲，取決於型別系統是否有足夠的訊息來確定`T`總是可以賦值給`U`。

下面是一些型別可以被立即解析的範例：

```ts
declare function f<T extends boolean>(x: T): T extends true ? string : number;

// Type is 'string | number
let x = f(Math.random() < 0.5)

```

另外一個範例涉及`TypeName`型別別名，它使用了嵌套了有條件型別：

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

下面是一個有條件型別被推遲解析的範例:

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

這裡，`a`變數含有未確定的有條件型別。
當有另一段程式碼呼叫`foo`，它會用其它型別替換`U`，TypeScript將重新計算有條件型別，決定它是否可以選擇一個分支。

與此同時，我們可以將有條件型別賦值給其它型別，只要有條件型別的每個分支都可以賦值給目標型別。
因此在我們的範例裡，我們可以將`U extends Foo ? string : number`賦值給`string | number`，因為不管這個有條件型別最終結果是什麼，它只能是`string`或`number`。

### 分佈式有條件型別

如果有條件型別裡待檢查的型別是`naked type parameter`，那麼它也被稱為「分佈式有條件型別」。
分佈式有條件型別在實例化時會自動分發成聯合型別。
例如，實例化`T extends U ? X : Y`，`T`的型別為`A | B | C`，會被解析為`(A extends U ? X : Y) | (B extends U ? X : Y) | (C extends U ? X : Y)`。

#### 範例

```ts
type T10 = TypeName<string | (() => void)>;  // "string" | "function"
type T12 = TypeName<string | string[] | undefined>;  // "string" | "object" | "undefined"
type T11 = TypeName<string[] | number[]>;  // "object"
```

在`T extends U ? X : Y`的實例化裡，對`T`的引用被解析為聯合型別的一部分(比如，`T`指向某一單個部分，在有條件型別分佈到聯合型別之後)。
此外，在`X`內對`T`的引用有一個附加的型別參數約束`U`(例如，`T`被當成在`X`內可賦值給`U`)。

#### 範例

```ts
type BoxedValue<T> = { value: T };
type BoxedArray<T> = { array: T[] };
type Boxed<T> = T extends any[] ? BoxedArray<T[number]> : BoxedValue<T>;

type T20 = Boxed<string>;  // BoxedValue<string>;
type T21 = Boxed<number[]>;  // BoxedArray<number>;
type T22 = Boxed<string | number[]>;  // BoxedValue<string> | BoxedArray<number>;
```

注意在`Boxed<T>`的`true`分支裡，`T`有個額外的約束`any[]`，因此它適用於`T[number]`陣列元素型別。同時也注意一下有條件型別是如何分佈成聯合型別的。

有條件型別的分佈式的屬性可以方便地用來*過濾*聯合型別：

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

有條件型別與映射型別結合時特別有用：

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

與聯合型別和交叉型別相似，有條件型別不允許遞歸地引用自己。比如下面的錯誤。

#### 範例

```ts
type ElementType<T> = T extends any[] ? ElementType<T[number]> : T;  // Error
```

### 有條件型別中的型別推斷

現在在有條件型別的`extends`子敘述中，允許出現`infer`宣告，它會引入一個待推斷的型別變數。
這個推斷的型別變數可以在有條件型別的true分支中被引用。
允許出現多個同型別變數的`infer`。

例如，下面程式碼會提取函數型別的返回值型別：

```ts
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : any;
```

有條件型別可以嵌套來構成一系列的匹配模式，按順序進行求值：

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

下面的範例解釋了在協變位置上，同一個型別變數的多個候選型別會被推斷為聯合型別：

```ts
type Foo<T> = T extends { a: infer U, b: infer U } ? U : never;
type T10 = Foo<{ a: string, b: string }>;  // string
type T11 = Foo<{ a: string, b: number }>;  // string | number
```

相似地，在抗變位置上，同一個型別變數的多個候選型別會被推斷為交叉型別：

```ts
type Bar<T> = T extends { a: (x: infer U) => void, b: (x: infer U) => void } ? U : never;
type T20 = Bar<{ a: (x: string) => void, b: (x: string) => void }>;  // string
type T21 = Bar<{ a: (x: string) => void, b: (x: number) => void }>;  // string & number
```

當推斷具有多個呼叫簽名(例如函數重載型別)的型別時，用*最後*的簽名(大概是最自由的包含所有情況的簽名)進行推斷。
無法根據參數型別列表來解析重載。

```ts
declare function foo(x: string): number;
declare function foo(x: number): string;
declare function foo(x: string | number): string | number;
type T30 = ReturnType<typeof foo>;  // string | number
```

無法在正常型別參數的約束子敘述中使用`infer`宣告：

```ts
type ReturnType<T extends (...args: any[]) => infer R> = R;  // 錯誤，不支援
```

但是，可以這樣達到同樣的效果，在約束裡刪掉型別變數，用有條件型別替換：

```ts
type AnyFunction = (...args: any[]) => any;
type ReturnType<T extends AnyFunction> = T extends (...args: any[]) => infer R ? R : any;
```

### 預定義的有條件型別

TypeScript 2.8在`lib.d.ts`裡增加了一些預定義的有條件型別：

* `Exclude<T, U>` -- 從`T`中剔除可以賦值給`U`的型別。
* `Extract<T, U>` -- 提取`T`中可以賦值給`U`的型別。
* `NonNullable<T>` -- 從`T`中剔除`null`和`undefined`。
* `ReturnType<T>` -- 獲取函數返回值型別。
* `InstanceType<T>` -- 獲取建構函數型別的實例型別。

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

> 注意：`Exclude`型別是[建議的](https://github.com/Microsoft/TypeScript/issues/12215#issuecomment-307871458)`Diff`型別的一種實現。我們使用`Exclude`這個名字是為了避免破壞已經定義了`Diff`的程式碼，並且我們感覺這個名字能更好地表達型別的語義。我們沒有增加`Omit<T, K>`型別，因為它可以很容易的用`Pick<T, Exclude<keyof T, K>>`來表示。
