# 介紹

軟件工程中，我們不僅要創建一致的定義良好的API，同時也要考慮可重用性。
組件不僅能夠支持當前的數據類型，同時也能支持未來的數據類型，這在創建大型系統時為你提供了十分靈活的功能。

在像C#和Java這樣的語言中，可以使用`泛型`來創建可重用的組件，一個組件可以支持多種類型的數據。
這樣用戶就可以以自己的數據類型來使用組件。

# 泛型之Hello World

下面來創建第一個使用泛型的例子：identity函數。
這個函數會返回任何傳入它的值。
你可以把這個函數當成是`echo`命令。

不用泛型的話，這個函數可能是下面這樣：

```ts
function identity(arg: number): number {
    return arg;
}
```

或者，我們使用`any`類型來定義函數：

```ts
function identity(arg: any): any {
    return arg;
}
```

使用`any`類型會導致這個函數可以接收任何類型的`arg`參數，這樣就丟失了一些信息：傳入的類型與返回的類型應該是相同的。
如果我們傳入一個數字，我們只知道任何類型的值都有可能被返回。

因此，我們需要一種方法使返回值的類型與傳入參數的類型是相同的。
這裡，我們使用了*類型變量*，它是一種特殊的變量，只用於表示類型而不是值。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

我們給identity添加了類型變量`T`。
`T`幫助我們捕獲用戶傳入的類型（比如：`number`），之後我們就可以使用這個類型。
之後我們再次使用了`T`當做返回值類型。現在我們可以知道參數類型與返回值類型是相同的了。
這允許我們跟蹤函數裡使用的類型的信息。

我們把這個版本的`identity`函數叫做泛型，因為它可以適用於多個類型。
不同於使用`any`，它不會丟失信息，像第一個例子那像保持準確性，傳入數值類型並返回數值類型。

我們定義了泛型函數後，可以用兩種方法使用。
第一種是，傳入所有的參數，包含類型參數：

```ts
let output = identity<string>("myString");  // type of output will be 'string'
```

這裡我們明確的指定了`T`是`string`類型，並做為一個參數傳給函數，使用了`<>`括起來而不是`()`。

第二種方法更普遍。利用了*類型推論* -- 即編譯器會根據傳入的參數自動地幫助我們確定T的類型：

```ts
let output = identity("myString");  // type of output will be 'string'
```

注意我們沒必要使用尖括號（`<>`）來明確地傳入類型；編譯器可以查看`myString`的值，然後把`T`設置為它的類型。
類型推論幫助我們保持代碼精簡和高可讀性。如果編譯器不能夠自動地推斷出類型的話，只能像上面那樣明確的傳入T的類型，在一些複雜的情況下，這是可能出現的。

# 使用泛型變量

使用泛型創建像`identity`這樣的泛型函數時，編譯器要求你在函數體必須正確的使用這個通用的類型。
換句話說，你必須把這些參數當做是任意或所有類型。

看下之前`identity`例子：

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

如果我們想同時打印出`arg`的長度。
我們很可能會這樣做：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

如果這麼做，編譯器會報錯說我們使用了`arg`的`.length`屬性，但是沒有地方指明`arg`具有這個屬性。
記住，這些類型變量代表的是任意類型，所以使用這個函數的人可能傳入的是個數字，而數字是沒有`.length`屬性的。

現在假設我們想操作`T`類型的數組而不直接是`T`。由於我們操作的是數組，所以`.length`屬性是應該存在的。
我們可以像創建其它數組一樣創建這個數組：

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

你可以這樣理解`loggingIdentity`的類型：泛型函數`loggingIdentity`，接收類型參數`T`和參數`arg`，它是個元素類型是`T`的數組，並返回元素類型是`T`的數組。
如果我們傳入數字數組，將返回一個數字數組，因為此時`T`的的類型為`number`。
這可以讓我們把泛型變量T當做類型的一部分使用，而不是整個類型，增加了靈活性。

我們也可以這樣實現上面的例子：

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

使用過其它語言的話，你可能對這種語法已經很熟悉了。
在下一節，會介紹如何創建自定義泛型像`Array<T>`一樣。

# 泛型類型

上一節，我們創建了identity通用函數，可以適用於不同的類型。
在這節，我們研究一下函數本身的類型，以及如何創建泛型接口。

泛型函數的類型與非泛型函數的類型沒什麼不同，只是有一個類型參數在最前面，像函數聲明一樣：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <T>(arg: T) => T = identity;
```

我們也可以使用不同的泛型參數名，只要在數量上和使用方式上能對應上就可以。

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: <U>(arg: U) => U = identity;
```

我們還可以使用帶有調用簽名的對象字面量來定義泛型函數：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

這引導我們去寫第一個泛型接口了。
我們把上面例子裡的對象字面量拿出來做為一個接口：

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

一個相似的例子，我們可能想把泛型參數當作整個接口的一個參數。
這樣我們就能清楚的知道使用的具體是哪個泛型類型（比如：`Dictionary<string>而不只是Dictionary`）。
這樣接口裡的其它成員也能知道這個參數的類型了。

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

注意，我們的示例做了少許改動。
不再描述泛型函數，而是把非泛型函數簽名作為泛型類型一部分。
當我們使用`GenericIdentityFn`的時候，還得傳入一個類型參數來指定泛型類型（這裡是：`number`），鎖定了之後代碼裡使用的類型。
對於描述哪部分類型屬於泛型部分來說，理解何時把參數放在調用簽名裡和何時放在接口上是很有幫助的。

除了泛型接口，我們還可以創建泛型類。
注意，無法創建泛型枚舉和泛型命名空間。

# 泛型類

泛型類看上去與泛型接口差不多。
泛型類使用（`<>`）括起泛型類型，跟在類名後面。

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

`GenericNumber`類的使用是十分直觀的，並且你可能已經注意到了，沒有什麼去限制它只能使用`number`類型。
也可以使用字符串或其它更複雜的類型。

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

與接口一樣，直接把泛型類型放在類後面，可以幫助我們確認類的所有屬性都在使用相同的類型。

我們在[類](./Classes.md)那節說過，類有兩部分：靜態部分和實例部分。
泛型類指的是實例部分的類型，所以類的靜態屬性不能使用這個泛型類型。

# 泛型約束

你應該會記得之前的一個例子，我們有時候想操作某類型的一組值，並且我們知道這組值具有什麼樣的屬性。
在`loggingIdentity`例子中，我們想訪問`arg`的`length`屬性，但是編譯器並不能證明每種類型都有`length`屬性，所以就報錯了。

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

相比於操作any所有類型，我們想要限制函數去處理任意帶有`.length`屬性的所有類型。
只要傳入的類型有這個屬性，我們就允許，就是說至少包含這一屬性。
為此，我們需要列出對於T的約束要求。

為此，我們定義一個接口來描述約束條件。
創建一個包含`.length`屬性的接口，使用這個接口和`extends`關鍵字來實現約束：

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

現在這個泛型函數被定義了約束，因此它不再是適用於任意類型：

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property
```

我們需要傳入符合約束類型的值，必須包含必須的屬性：

```ts
loggingIdentity({length: 10, value: 3});
```

## 在泛型約束中使用類型參數

你可以聲明一個類型參數，且它被另一個類型參數所約束。
比如，現在我們想要用屬性名從對象裡獲取這個屬性。
並且我們想要確保這個屬性存在於對象`obj`上，因此我們需要在這兩個類型之間使用約束。

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```

## 在泛型裡使用類類型

在TypeScript使用泛型創建工廠函數時，需要引用構造函數的類類型。比如，

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

一個更高級的例子，使用原型屬性推斷並約束構造函數與類實例的關係。

```ts
class BeeKeeper {
    hasMask: boolean;
}

class ZooKeeper {
    nametag: string;
}

class Animal {
    numLegs: number;
}

class Bee extends Animal {
    keeper: BeeKeeper;
}

class Lion extends Animal {
    keeper: ZooKeeper;
}

function createInstance<A extends Animal>(c: new () => A): A {
    return new c();
}

createInstance(Lion).keeper.nametag;  // typechecks!
createInstance(Bee).keeper.hasMask;   // typechecks!
```
