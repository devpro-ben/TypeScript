# 介紹

TypeScript裡的型別相容性是基於結構(structural)子型別的。
結構型別是一種只使用其成員來描述型別的方式。
它正好與名義(nominal)型別形成對比。(譯者註：在基於名義型別的型別系統中，資料型別的相容性或等價性是透過明確的宣告和/或型別的名稱來決定的。這與結構性型別系統不同，它是基於型別的組成結構，且不要求明確地宣告。)
看下面的範例：

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

在使用基於名義型別的語言，比如C#或Java中，這段程式碼會報錯，因為Person類沒有明確說明其實現了Named介面。

TypeScript的結構性子型別是根據JavaScript程式碼的典型寫法來設計的。
因為JavaScript裡廣泛地使用匿名物件，例如函數運算式和物件字面量，所以使用結構型別系統來描述這些型別比使用名義型別系統更好。

## 關於可靠性的注意事項

TypeScript的型別系統允許某些在編譯階段無法確認其安全性的操作。當一個型別系統具此屬性時，被當做是「不可靠」的。TypeScript允許這種不可靠行為的發生是經過仔細考慮的。透過這篇文章，我們會解釋什麼時候會發生這種情況和其有利的一面。

# 開始

TypeScript結構化型別系統的基本規則是，如果`x`要相容`y`，那麼`y`至少具有與`x`相同的屬性。比如：

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
在這個範例中，`y`必須包含名字是`name`的`string`型別成員。`y`滿足條件，因此賦值正確。

檢查函數參數時使用相同的規則：

```ts
function greet(n: Named) {
    console.log('Hello, ' + n.name);
}
greet(y); // OK
```

注意，`y`有個額外的`location`屬性，但這不會引發錯誤。
只有目標型別(這裡是`Named`)的成員會被一一檢查是否相容。

這個比較程序是遞歸進行的，檢查每個成員及子成員。

# 比較兩個函數

相對來講，在比較原始型別和物件型別的時候是比較容易理解的，問題是如何判斷兩個函數是相容的。
下面我們從兩個簡單的函數入手，它們僅是參數列表略有不同：

```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK
x = y; // Error
```

要查看`x`是否能賦值給`y`，首先看它們的參數列表。
`x`的每個參數必須能在`y`裡找到對應型別的參數。
注意的是參數的名字相同與否無所謂，只看它們的型別。
這裡，`x`的每個參數在`y`中都能找到對應的參數，所以允許賦值。

第二個賦值錯誤，因為`y`有個必需的第二個參數，但是`x`並沒有，所以不允許賦值。

您可能會疑惑為什麼允許`忽略`參數，像範例`y = x`中那樣。
原因是忽略額外的參數在JavaScript裡是很常見的。
例如，`Array#forEach`給回呼函數傳3個參數：陣列元素，索引和整個陣列。
儘管如此，傳入一個只使用第一個參數的回呼函數也是很有用的：

```ts
let items = [1, 2, 3];

// Don't force these extra arguments
items.forEach((item, index, array) => console.log(item));

// Should be OK!
items.forEach((item) => console.log(item));
```

下面來看看如何處理傳回值型別，建立兩個僅是傳回值型別不同的函數：

```ts
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
y = x; // Error, because x() lacks a location property
```

型別系統強制源函數的傳回值型別必須是目標函數傳回值型別的子型別。

## 函數參數雙向協變

當比較函數參數型別時，只有當來源函數參數能夠賦值給目標函數或者反過來時才能賦值成功。
這是不穩定的，因為呼叫者可能傳入了一個具有更精確型別訊息的函數，但是呼叫這個傳入的函數的時候卻使用了不是那麼精確的型別訊息。
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

## 選擇性參數及剩餘參數

比較函數相容性的時候，選擇性參數與必須參數是可互換的。
來源型別上有額外的選擇性參數不是錯誤，目標型別的選擇性參數在來源型別裡沒有對應的參數也不是錯誤。

當一個函數有剩餘參數時，它被當做無限個選擇性參數。

這對於型別系統來說是不穩定的，但從執行時的角度來看，選擇性參數一般來說是不強制的，因為對於大多數函數來說相當於傳遞了一些`undefinded`。

有一個好的範例，常見的函數接收一個回呼函數並用對於程式員來說是可預知的參數但對型別系統來說是不確定的參數來呼叫：

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

對於有重載的函數，來源函數的每個重載都要在目標函數上找到對應的函數簽名。
這確保了目標函數可以在所有來源函數可呼叫的地方呼叫。

# 列舉

列舉型別與數值型別相容，並且數值型別與列舉型別相容。不同列舉型別之間是不相容的。比如，

```ts
enum Status { Ready, Waiting };
enum Color { Red, Blue, Green };

let status = Status.Ready;
status = Color.Green;  // Error
```

# 類別

類別與物件字面量和介面差不多，但有一點不同：類別有靜態部分和實例部分的型別。
比較兩個型別的物件時，只有實例的成員會被比較。
靜態成員和建構函數不在比較的範圍內。

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

## 類別的私有成員和受保護成員

類別的私有成員和受保護成員會影響相容性。
當檢查類別實例的相容時，如果目標型別包含一個私有成員，那麼來源型別必須包含來自同一個類別的這個私有成員。
同樣地，這條規則也適用於包含受保護成員實例的型別檢查。
這允許子類別賦值給父類別，但是不能賦值給其它有同樣型別的類別。

# 泛型

因為TypeScript是結構性的型別系統，型別參數只影響使用其做為型別一部分的結果型別。比如，

```ts
interface Empty<T> {
}
let x: Empty<number>;
let y: Empty<string>;

x = y;  // OK, because y matches structure of x
```

上面程式碼裡，`x`和`y`是相容的，因為它們的結構使用型別參數時並沒有什麼不同。
把這個範例改變一下，增加一個成員，就能看出是如何工作的了：

```ts
interface NotEmpty<T> {
    data: T;
}
let x: NotEmpty<number>;
let y: NotEmpty<string>;

x = y;  // Error, because x and y are not compatible
```

在這裡，泛型型別在使用時就好比不是一個泛型型別。

對於沒指定泛型型別的泛型參數時，會把所有泛型參數當成`any`比較。
然後用結果型別進行比較，就像上面第一個範例。

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

## 子型別與賦值

目前為止，我們使用了「相容性」，它在語言規範裡沒有定義。
在TypeScript裡，有兩種相容性：子型別和賦值。
它們的不同點在於，賦值擴展了子型別相容性，增加了一些規則，允許和`any`來回賦值，以及`enum`和對應數字值之間的來回賦值。

語言裡的不同地方分別使用了它們之中的機制。
實際上，型別相容性是由賦值相容性來控制的，即使在`implements`和`extends`敘述也不例外。

更多訊息，請參閱[TypeScript語言規範](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md).
