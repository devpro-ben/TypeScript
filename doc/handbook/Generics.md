# 介紹

軟體工程中，我們不僅要建立一致的定義良好的API，同時也要考慮可重用性。
元件不僅能夠支援當前的資料型別，同時也能支援未來的資料型別，這在建立大型系統時為您提供了十分靈活的功能。

在像C#和Java這樣的語言中，可以使用`泛型`來建立可重用的元件，一個元件可以支援多種型別的資料。
這樣用戶就可以以自己的資料型別來使用元件。

# 泛型之Hello World

下面來建立第一個使用泛型的範例：`identity`函數。
這個函數會傳回任何傳入它的值。
您可以把這個函數當成是`echo`命令。

不用泛型的話，這個函數可能是下面這樣：

```ts
function identity(arg: number): number {
    return arg;
}
```

或者，我們使用`any`型別來定義函數：

```ts
function identity(arg: any): any {
    return arg;
}
```

使用`any`型別會導致這個函數可以接收任何型別的`arg`參數，這樣就丟失了一些訊息：傳入的型別與傳回的型別應該是相同的。
如果我們傳入一個數字，我們只知道任何型別的值都有可能被傳回。

因此，我們需要一種方法使傳回值的型別與傳入參數的型別是相同的。
這裡，我們使用了*型別變數*，它是一種特殊的變數，只用於表示型別而不是值。

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

我們給identity加入了型別變數`T`。
`T`幫助我們捕獲用戶傳入的型別(比如：`number`)，之後我們就可以使用這個型別。
之後我們再次使用了`T`當做傳回值型別。現在我們可以知道參數型別與傳回值型別是相同的了。
這允許我們跟蹤函數裡使用的型別的訊息。

我們把這個版本的`identity`函數叫做泛型，因為它可以適用於多個型別。
不同於使用`any`，它不會丟失訊息，像第一個範例那像保持準確性，傳入數值型別並傳回數值型別。

我們定義了泛型函數後，可以用兩種方法使用。
第一種是，傳入所有的參數，包含型別參數：

```ts
let output = identity<string>("myString");  // type of output will be 'string'
```

這裡我們明確的指定了`T`是`string`型別，並做為一個參數傳給函數，使用了`<>`括起來而不是`()`。

第二種方法更普遍。利用了*型別推論* -- 即編譯器會根據傳入的參數自動地幫助我們確定T的型別：

```ts
let output = identity("myString");  // type of output will be 'string'
```

注意我們沒必要使用尖括號(`<>`)來明確地傳入型別；編譯器可以查看`myString`的值，然後把`T`設置為它的型別。
型別推論幫助我們保持程式碼精簡和高可讀性。如果編譯器不能夠自動地推斷出型別的話，只能像上面那樣明確的傳入T的型別，在一些複雜的情況下，這是可能出現的。

# 使用泛型變數

使用泛型建立像`identity`這樣的泛型函數時，編譯器要求您在函數體必須正確的使用這個通用的型別。
換句話說，您必須把這些參數當做是任意或所有型別。

看下之前`identity`範例：

```ts
function identity<T>(arg: T): T {
    return arg;
}
```

如果我們想同時列印出`arg`的長度。
我們很可能會這樣做：

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

如果這麼做，編譯器會報錯說我們使用了`arg`的`.length`屬性，但是沒有地方指明`arg`具有這個屬性。
記住，這些型別變數代表的是任意型別，所以使用這個函數的人可能傳入的是個數字，而數字是沒有`.length`屬性的。

現在假設我們想操作`T`型別的陣列而不直接是`T`。由於我們操作的是陣列，所以`.length`屬性是應該存在的。
我們可以像建立其它陣列一樣建立這個陣列：

```ts
function loggingIdentity<T>(arg: T[]): T[] {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

您可以這樣理解`loggingIdentity`的型別：泛型函數`loggingIdentity`，接收型別參數`T`和參數`arg`，它是個元素型別是`T`的陣列，並傳回元素型別是`T`的陣列。
如果我們傳入數字陣列，將傳回一個數字陣列，因為此時`T`的的型別為`number`。
這可以讓我們把泛型變數T當做型別的一部分使用，而不是整個型別，增加了靈活性。

我們也可以這樣實現上面的範例：

```ts
function loggingIdentity<T>(arg: Array<T>): Array<T> {
    console.log(arg.length);  // Array has a .length, so no more error
    return arg;
}
```

使用過其它語言的話，您可能對這種語法已經很熟悉了。
在下一節，會介紹如何建立自訂泛型像`Array<T>`一樣。

# 泛型型別

上一節，我們建立了identity通用函數，可以適用於不同的型別。
在這節，我們研究一下函數本身的型別，以及如何建立泛型介面。

泛型函數的型別與非泛型函數的型別沒什麼不同，只是有一個型別參數在最前面，像函數宣告一樣：

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

我們還可以使用帶有呼叫簽名的物件字面值來定義泛型函數：

```ts
function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: {<T>(arg: T): T} = identity;
```

這引導我們去寫第一個泛型介面了。
我們把上面範例裡的物件字面值拿出來做為一個介面：

```ts
interface GenericIdentityFn {
    <T>(arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn = identity;
```

一個相似的範例，我們可能想把泛型參數當作整個介面的一個參數。
這樣我們就能清楚的知道使用的具體是哪個泛型型別(比如：`Dictionary<string>而不只是Dictionary`)。
這樣介面裡的其它成員也能知道這個參數的型別了。

```ts
interface GenericIdentityFn<T> {
    (arg: T): T;
}

function identity<T>(arg: T): T {
    return arg;
}

let myIdentity: GenericIdentityFn<number> = identity;
```

注意，我們的範例做了少許改動。
不再描述泛型函數，而是把非泛型函數簽名作為泛型型別一部分。
當我們使用`GenericIdentityFn`的時候，還得傳入一個型別參數來指定泛型型別(這裡是：`number`)，鎖定了之後程式碼裡使用的型別。
對於描述哪部分型別屬於泛型部分來說，理解何時把參數放在呼叫簽名裡和何時放在介面上是很有幫助的。

除了泛型介面，我們還可以建立泛型類別。
注意，無法建立泛型列舉和泛型命名空間。

# 泛型類別

泛型類別看上去與泛型介面差不多。
泛型類別使用(`<>`)括起泛型型別，跟在類別名稱後面。

```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
myGenericNumber.zeroValue = 0;
myGenericNumber.add = function(x, y) { return x + y; };
```

`GenericNumber`類別的使用是十分直觀的，並且您可能已經注意到了，沒有什麼去限制它只能使用`number`型別。
也可以使用字串或其它更複雜的型別。

```ts
let stringNumeric = new GenericNumber<string>();
stringNumeric.zeroValue = "";
stringNumeric.add = function(x, y) { return x + y; };

console.log(stringNumeric.add(stringNumeric.zeroValue, "test"));
```

與介面一樣，直接把泛型型別放在類別後面，可以幫助我們確認類別的所有屬性都在使用相同的型別。

我們在[類別](./Classes.md)那節說過，類別有兩部分：靜態部分和實例部分。
泛型類別指的是實例部分的型別，所以類別的靜態屬性不能使用這個泛型型別。

# 泛型約束

您應該會記得之前的一個範例，我們有時候想操作某型別的一組值，並且我們知道這組值具有什麼樣的屬性。
在`loggingIdentity`範例中，我們想存取`arg`的`length`屬性，但是編譯器並不能證明每種型別都有`length`屬性，因此它警告我們不能做出這個假設。

```ts
function loggingIdentity<T>(arg: T): T {
    console.log(arg.length);  // Error: T doesn't have .length
    return arg;
}
```

相比於操作any所有型別，我們想要限制函數去處理任意帶有`.length`屬性的所有型別。
只要傳入的型別有這個屬性，我們就允許，就是說至少包含這一屬性。
為此，我們需要列出對於`T`的約束要求。

為此，我們定義一個介面來描述約束條件。
建立一個包含`.length`屬性的介面，使用這個介面和`extends`關鍵字來實現約束：

```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    console.log(arg.length);  // Now we know it has a .length property, so no more error
    return arg;
}
```

現在這個泛型函數被定義了約束，因此它不再是適用於任意型別：

```ts
loggingIdentity(3);  // Error, number doesn't have a .length property
```

我們需要傳入符合約束型別的值，必須包含必須的屬性：

```ts
loggingIdentity({length: 10, value: 3});
```

## 在泛型約束中使用型別參數

您可以宣告一個型別參數，且它被另一個型別參數所約束。
比如，現在我們想要用屬性名從物件裡獲取這個屬性。
並且我們想要確保這個屬性存在於物件`obj`上，因此我們需要在這兩個型別之間使用約束。

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
    return obj[key];
}

let x = { a: 1, b: 2, c: 3, d: 4 };

getProperty(x, "a"); // okay
getProperty(x, "m"); // error: Argument of type 'm' isn't assignable to 'a' | 'b' | 'c' | 'd'.
```

## 在泛型裡使用類別型別

在TypeScript使用泛型建立工廠函數時，需要引用建構函數的類別型別。比如，

```ts
function create<T>(c: {new(): T; }): T {
    return new c();
}
```

一個更高級的範例，使用原型屬性推斷並約束建構函數與類別實例的關係。

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
