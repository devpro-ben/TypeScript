# 介紹

TypeScript裡的類型兼容性是基於結構子類型的。
結構類型是一種只使用其成員來描述類型的方式。
它正好與名義（nominal）類型形成對比。（譯者註：在基於名義類型的類型系統中，數據類型的兼容性或等價性是通過明確的聲明和/或類型的名稱來決定的。這與結構性類型系統不同，它是基於類型的組成結構，且不要求明確地聲明。）
看下面的例子：

```ts
interface Named {
    name: string;
}

class Person {
    name: string;
}

let p: Named;
// OK, because of structural typing
p = new Person();
```

在使用基於名義類型的語言，比如C#或Java中，這段代碼會報錯，因為Person類沒有明確說明其實現了Named接口。

TypeScript的結構性子類型是根據JavaScript代碼的典型寫法來設計的。
因為JavaScript裡廣泛地使用匿名對象，例如函數表達式和對象字面量，所以使用結構類型系統來描述這些類型比使用名義類型系統更好。

## 關於可靠性的注意事項

TypeScript的類型系統允許某些在編譯階段無法確認其安全性的操作。當一個類型系統具此屬性時，被當做是“不可靠”的。TypeScript允許這種不可靠行為的發生是經過仔細考慮的。通過這篇文章，我們會解釋什麼時候會發生這種情況和其有利的一面。

# 開始

TypeScript結構化類型系統的基本規則是，如果`x`要兼容`y`，那麼`y`至少具有與`x`相同的屬性。比如：

```ts
interface Named {
    name: string;
}

let x: Named;
// y's inferred type is { name: string; location: string; }
let y = { name: 'Alice', location: 'Seattle' };
x = y;
```

這裡要檢查`y`是否能賦值給`x`，編譯器檢查`x`中的每個屬性，看是否能在`y`中也找到對應屬性。
在這個例子中，`y`必須包含名字是`name`的`string`類型成員。`y`滿足條件，因此賦值正確。

檢查函數參數時使用相同的規則：

```ts
function greet(n: Named) {
    console.log('Hello, ' + n.name);
}
greet(y); // OK
```

注意，`y`有個額外的`location`屬性，但這不會引發錯誤。
只有目標類型（這裡是`Named`）的成員會被一一檢查是否兼容。

這個比較過程是遞歸進行的，檢查每個成員及子成員。

# 比較兩個函數

相對來講，在比較原始類型和對象類型的時候是比較容易理解的，問題是如何判斷兩個函數是兼容的。
下面我們從兩個簡單的函數入手，它們僅是參數列表略有不同：

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

要查看`x`是否能賦值給`y`，首先看它們的參數列表。
`x`的每個參數必須能在`y`裡找到對應類型的參數。
注意的是參數的名字相同與否無所謂，只看它們的類型。
這裡，`x`的每個參數在`y`中都能找到對應的參數，所以允許賦值。

第二個賦值錯誤，因為`y`有個必需的第二個參數，但是`x`並沒有，所以不允許賦值。

你可能會疑惑為什麼允許`忽略`參數，像例子`y = x`中那樣。
原因是忽略額外的參數在JavaScript裡是很常見的。
例如，`Array#forEach`給回調函數傳3個參數：數組元素，索引和整個數組。
儘管如此，傳入一個只使用第一個參數的回調函數也是很有用的：

```ts
let items = [1, 2, 3];

// Don't force these extra arguments
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach((item) => console.log(item));
```

下面來看看如何處理返回值類型，創建兩個僅是返回值類型不同的函數：

```ts
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error, because x() lacks a location property
```

類型系統強制源函數的返回值類型必須是目標函數返回值類型的子類型。

## 函數參數雙向協變

當比較函數參數類型時，只有當源函數參數能夠賦值給目標函數或者反過來時才能賦值成功。
這是不穩定的，因為調用者可能傳入了一個具有更精確類型信息的函數，但是調用這個傳入的函數的時候卻使用了不是那麼精確的類型信息。
實際上，這極少會發生錯誤，並且能夠實現很多JavaScript裡的常見模式。例如：

```ts
enum EventType { Mouse, Keyboard }

interface Event { timestamp: number; }
interface MouseEvent extends Event { x: number; y: number }
interface KeyEvent extends Event { keyCode: number }

function listenEvent(eventType: EventType, handler: (n: Event) => void) {
    /* ... */
}

// Unsound, but useful and common
listenEvent(EventType.Mouse, (e: MouseEvent) => console.log(e.x + ',' + e.y));

// Undesirable alternatives in presence of soundness
listenEvent(EventType.Mouse, (e: Event) => console.log((<MouseEvent>e).x + ',' + (<MouseEvent>e).y));
listenEvent(EventType.Mouse, <(e: Event) => void>((e: MouseEvent) => console.log(e.x + ',' + e.y)));

// Still disallowed (clear error). Type safety enforced for wholly incompatible types
listenEvent(EventType.Mouse, (e: number) => console.log(e));
```

## 可選參數及剩餘參數

比較函數兼容性的時候，可選參數與必須參數是可互換的。
源類型上有額外的可選參數不是錯誤，目標類型的可選參數在源類型裡沒有對應的參數也不是錯誤。

當一個函數有剩餘參數時，它被當做無限個可選參數。

這對於類型系統來說是不穩定的，但從運行時的角度來看，可選參數一般來說是不強制的，因為對於大多數函數來說相當於傳遞了一些`undefinded`。

有一個好的例子，常見的函數接收一個回調函數並用對於程序員來說是可預知的參數但對類型系統來說是不確定的參數來調用：

```ts
function invokeLater(args: any[], callback: (...args: any[]) => void) {
    /* ... Invoke callback with 'args' ... */
}

// Unsound - invokeLater "might" provide any number of arguments
invokeLater([1, 2], (x, y) => console.log(x + ', ' + y));

// Confusing (x and y are actually required) and undiscoverable
invokeLater([1, 2], (x?, y?) => console.log(x + ', ' + y));
```

## 函數重載

對於有重載的函數，源函數的每個重載都要在目標函數上找到對應的函數簽名。
這確保了目標函數可以在所有源函數可調用的地方調用。

# 枚舉

枚舉類型與數字類型兼容，並且數字類型與枚舉類型兼容。不同枚舉類型之間是不兼容的。比如，

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Green;  // Error
```

# 類

類與對象字面量和接口差不多，但有一點不同：類有靜態部分和實例部分的類型。
比較兩個類類型的對象時，只有實例的成員會被比較。
靜態成員和構造函數不在比較的範圍內。

```ts
class Animal {
    feet: number;
    constructor(name: string, numFeet: number) { }
}

class Size {
    feet: number;
    constructor(numFeet: number) { }
}

let a: Animal;
let s: Size;

a = s;  // OK
s = a;  // OK
```

## 類的私有成員和受保護成員

類的私有成員和受保護成員會影響兼容性。
當檢查類實例的兼容時，如果目標類型包含一個私有成員，那麼源類型必須包含來自同一個類的這個私有成員。
同樣地，這條規則也適用於包含受保護成員實例的類型檢查。
這允許子類賦值給父類，但是不能賦值給其它有同樣類型的類。

# 泛型

因為TypeScript是結構性的類型系統，類型參數隻影響使用其做為類型一部分的結果類型。比如，

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // OK, because y matches structure of x
```

上面代碼裡，`x`和`y`是兼容的，因為它們的結構使用類型參數時並沒有什麼不同。
把這個例子改變一下，增加一個成員，就能看出是如何工作的了：

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // Error, because x and y are not compatible
```

在這裡，泛型類型在使用時就好比不是一個泛型類型。

對於沒指定泛型類型的泛型參數時，會把所有泛型參數當成`any`比較。
然後用結果類型進行比較，就像上面第一個例子。

比如，

```ts
let identity = function<T>(x: T): T {
    // ...
}

let reverse = function<U>(y: U): U {
    // ...
}

identity = reverse;  // OK, because (x: any) => any matches (y: any) => any
```

# 高級主題

## 子類型與賦值

目前為止，我們使用了“兼容性”，它在語言規範裡沒有定義。
在TypeScript裡，有兩種兼容性：子類型和賦值。
它們的不同點在於，賦值擴展了子類型兼容性，增加了一些規則，允許和`any`來回賦值，以及`enum`和對應數字值之間的來回賦值。

語言裡的不同地方分別使用了它們之中的機制。
實際上，類型兼容性是由賦值兼容性來控制的，即使在`implements`和`extends`語句也不例外。

更多信息，請參閱[TypeScript語言規範](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md).
