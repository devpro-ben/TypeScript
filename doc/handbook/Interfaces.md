# 介紹

TypeScript的核心原則之一是對值所具有的*結構*進行類型檢查。
它有時被稱做“鴨式辨型法”或“結構性子類型化”。
在TypeScript裡，接口的作用就是為這些類型命名和為你的代碼或第三方代碼定義契約。

# 接口初探

下面通過一個簡單示例來觀察接口是如何工作的：

```ts
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

類型檢查器會查看`printLabel`的調用。
`printLabel`有一個參數，並要求這個對象參數有一個名為`label`類型為`string`的屬性。
需要注意的是，我們傳入的對象參數實際上會包含很多屬性，但是編譯器只會檢查那些必需的屬性是否存在，並且其類型是否匹配。
然而，有些時候TypeScript卻並不會這麼寬鬆，我們下面會稍做講解。

下面我們重寫上面的例子，這次使用接口來描述：必須包含一個`label`屬性且類型為`string`：

```ts
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

`LabelledValue`接口就好比一個名字，用來描述上面例子裡的要求。
它代表了有一個`label`屬性且類型為`string`的對象。
需要注意的是，我們在這裡並不能像在其它語言裡一樣，說傳給`printLabel`的對象實現了這個接口。我們只會去關注值的外形。
只要傳入的對象滿足上面提到的必要條件，那麼它就是被允許的。

還有一點值得提的是，類型檢查器不會去檢查屬性的順序，只要相應的屬性存在並且類型也是對的就可以。

# 可選屬性

接口裡的屬性不全都是必需的。
有些是只在某些條件下存在，或者根本不存在。
可選屬性在應用“option bags”模式時很常用，即給函數傳入的參數對象中只有部分屬性賦值了。

下面是應用了“option bags”的例子：

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): {color: string; area: number} {
  let newSquare = {color: "white", area: 100};
  if (config.color) {
    newSquare.color = config.color;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"});
```

帶有可選屬性的接口與普通的接口定義差不多，只是在可選屬性名字定義的後面加一個`?`符號。

可選屬性的好處之一是可以對可能存在的屬性進行預定義，好處之二是可以捕獲引用了不存在的屬性時的錯誤。
比如，我們故意將`createSquare`裡的`color`屬性名拼錯，就會得到一個錯誤提示：

```ts
interface SquareConfig {
  color?: string;
  width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = {color: "white", area: 100};
  if (config.clor) {
    // Error: Property 'clor' does not exist on type 'SquareConfig'
    newSquare.color = config.clor;
  }
  if (config.width) {
    newSquare.area = config.width * config.width;
  }
  return newSquare;
}

let mySquare = createSquare({color: "black"});
```

# 只讀屬性

一些對象屬性只能在對象剛剛創建的時候修改其值。
你可以在屬性名前用`readonly`來指定只讀屬性:

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}
```

你可以通過賦值一個對象字面量來構造一個`Point`。
賦值後，`x`和`y`再也不能被改變了。

```ts
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript具有`ReadonlyArray<T>`類型，它與`Array<T>`相似，只是把所有可變方法去掉了，因此可以確保數組創建後再也不能被修改：

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

上面代碼的最後一行，可以看到就算把整個`ReadonlyArray`賦值到一個普通數組也是不可以的。
但是你可以用類型斷言重寫：

```ts
a = ro as number[];
```

## `readonly` vs `const`

最簡單判斷該用`readonly`還是`const`的方法是看要把它做為變量使用還是做為一個屬性。
做為變量使用的話用`const`，若做為屬性則使用`readonly`。

# 額外的屬性檢查

我們在第一個例子裡使用了接口，TypeScript讓我們傳入`{ size: number; label: string; }`到僅期望得到`{ label: string; }`的函數裡。
我們已經學過了可選屬性，並且知道他們在“option bags”模式裡很有用。

然而，天真地將這兩者結合的話就會像在JavaScript裡那樣搬起石頭砸自己的腳。
比如，拿`createSquare`例子來說：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
}

function createSquare(config: SquareConfig): { color: string; area: number } {
    // ...
}

let mySquare = createSquare({ colour: "red", width: 100 });
```

注意傳入`createSquare`的參數拼寫為`colour`而不是`color`。
在JavaScript裡，這會默默地失敗。

你可能會爭辯這個程序已經正確地類型化了，因為`width`屬性是兼容的，不存在`color`屬性，而且額外的`colour`屬性是無意義的。

然而，TypeScript會認為這段代碼可能存在bug。
對象字面量會被特殊對待而且會經過*額外屬性檢查*，當將它們賦值給變量或作為參數傳遞的時候。
如果一個對象字面量存在任何“目標類型”不包含的屬性時，你會得到一個錯誤。

```ts
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```

繞開這些檢查非常簡單。
最簡便的方法是使用類型斷言：

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

然而，最佳的方式是能夠添加一個字符串索引簽名，前提是你能夠確定這個對象可能具有某些做為特殊用途使用的額外屬性。
如果`SquareConfig`帶有上面定義的類型的`color`和`width`屬性，並且*還會*帶有任意數量的其它屬性，那麼我們可以這樣定義它：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

我們稍後會講到索引簽名，但在這我們要表示的是`SquareConfig`可以有任意數量的屬性，並且只要它們不是`color`和`width`，那麼就無所謂它們的類型是什麼。

還有最後一種跳過這些檢查的方式，這可能會讓你感到驚訝，它就是將這個對象賦值給一個另一個變量：
因為`squareOptions`不會經過額外屬性檢查，所以編譯器不會報錯。

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

要留意，在像上面一樣的簡單代碼裡，你可能不應該去繞開這些檢查。
對於包含方法和內部狀態的複雜對象字面量來講，你可能需要使用這些技巧，但是大部額外屬性檢查錯誤是真正的bug。
就是說你遇到了額外類型檢查出的錯誤，比如“option bags”，你應該去審查一下你的類型聲明。
在這裡，如果支持傳入`color`或`colour`屬性到`createSquare`，你應該修改`SquareConfig`定義來體現出這一點。

# 函數類型

接口能夠描述JavaScript中對象擁有的各種各樣的外形。
除了描述帶有屬性的普通對象外，接口也可以描述函數類型。

為了使用接口表示函數類型，我們需要給接口定義一個調用簽名。
它就像是一個只有參數列表和返回值類型的函數定義。參數列表裡的每個參數都需要名字和類型。

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

這樣定義後，我們可以像使用其它接口一樣使用這個函數類型的接口。
下例展示了如何創建一個函數類型的變量，並將一個同類型的函數賦值給這個變量。

```ts
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

對於函數類型的類型檢查來說，函數的參數名不需要與接口裡定義的名字相匹配。
比如，我們使用下面的代碼重寫上面的例子：

```ts
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  let result = src.search(sub);
  return result > -1;
}
```

函數的參數會逐個進行檢查，要求對應位置上的參數類型是兼容的。
如果你不想指定類型，TypeScript的類型系統會推斷出參數類型，因為函數直接賦值給了`SearchFunc`類型變量。
函數的返回值類型是通過其返回值推斷出來的（此例是`false`和`true`）。
如果讓這個函數返回數字或字符串，類型檢查器會警告我們函數的返回值類型與`SearchFunc`接口中的定義不匹配。

```ts
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

# 可索引的類型

與使用接口描述函數類型差不多，我們也可以描述那些能夠“通過索引得到”的類型，比如`a[10]`或`ageMap["daniel"]`。
可索引類型具有一個*索引簽名*，它描述了對象索引的類型，還有相應的索引返回值類型。
讓我們看一個例子：

```ts
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

上面例子裡，我們定義了`StringArray`接口，它具有索引簽名。
這個索引簽名表示了當用`number`去索引`StringArray`時會得到`string`類型的返回值。

Typescript支持兩種索引簽名：字符串和數字。
可以同時使用兩種類型的索引，但是數字索引的返回值必須是字符串索引返回值類型的子類型。
這是因為當使用`number`來索引時，JavaScript會將它轉換成`string`然後再去索引對象。
也就是說用`100`（一個`number`）去索引等同於使用`"100"`（一個`string`）去索引，因此兩者需要保持一致。

```ts
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// 錯誤：使用數值型的字符串索引，有時會得到完全不同的Animal!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

字符串索引簽名能夠很好的描述`dictionary`模式，並且它們也會確保所有屬性與其返回值類型相匹配。
因為字符串索引聲明了`obj.property`和`obj["property"]`兩種形式都可以。
下面的例子裡，`name`的類型與字符串索引類型不匹配，所以類型檢查器給出一個錯誤提示：

```ts
interface NumberDictionary {
  [index: string]: number;
  length: number;    // 可以，length是number類型
  name: string       // 錯誤，`name`的類型與索引類型返回值的類型不匹配
}
```

最後，你可以將索引簽名設置為只讀，這樣就防止了給索引賦值：

```ts
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

你不能設置`myArray[2]`，因為索引簽名是只讀的。

# 類類型

## 實現接口

與C#或Java裡接口的基本作用一樣，TypeScript也能夠用它來明確的強制一個類去符合某種契約。

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

你也可以在接口中描述一個方法，在類裡實現它，如同下面的`setTime`方法一樣：

```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class Clock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number) { }
}
```

接口描述了類的公共部分，而不是公共和私有兩部分。
它不會幫你檢查類是否具有某些私有成員。

## 類靜態部分與實例部分的區別

當你操作類和接口的時候，你要知道類是具有兩個類型的：靜態部分的類型和實例的類型。
你會注意到，當你用構造器簽名去定義一個接口並試圖定義一個類去實現這個接口時會得到一個錯誤：

```ts
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

這裡因為當一個類實現了一個接口時，只對其實例部分進行類型檢查。
constructor存在於類的靜態部分，所以不在檢查的範圍內。

因此，我們應該直接操作類的靜態部分。
看下面的例子，我們定義了兩個接口，`ClockConstructor`為構造函數所用和`ClockInterface`為實例方法所用。
為了方便我們定義一個構造函數`createClock`，它用傳入的類型創建實例。

```ts
interface ClockConstructor {
    new (hour: number, minute: number): ClockInterface;
}
interface ClockInterface {
    tick();
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
    return new ctor(hour, minute);
}

class DigitalClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("beep beep");
    }
}
class AnalogClock implements ClockInterface {
    constructor(h: number, m: number) { }
    tick() {
        console.log("tick tock");
    }
}

let digital = createClock(DigitalClock, 12, 17);
let analog = createClock(AnalogClock, 7, 32);
```

因為`createClock`的第一個參數是`ClockConstructor`類型，在`createClock(AnalogClock, 7, 32)`裡，會檢查`AnalogClock`是否符合構造函數簽名。

# 繼承接口

和類一樣，接口也可以相互繼承。
這讓我們能夠從一個接口裡複製成員到另一個接口裡，可以更靈活地將接口分割到可重用的模塊裡。

```ts
interface Shape {
    color: string;
}

interface Square extends Shape {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
```

一個接口可以繼承多個接口，創建出多個接口的合成接口。

```ts
interface Shape {
    color: string;
}

interface PenStroke {
    penWidth: number;
}

interface Square extends Shape, PenStroke {
    sideLength: number;
}

let square = <Square>{};
square.color = "blue";
square.sideLength = 10;
square.penWidth = 5.0;
```

# 混合類型

先前我們提過，接口能夠描述JavaScript裡豐富的類型。
因為JavaScript其動態靈活的特點，有時你會希望一個對象可以同時具有上面提到的多種類型。

一個例子就是，一個對象可以同時做為函數和對象使用，並帶有額外的屬性。

```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = <Counter>function (start: number): string { return '' };
    counter.interval = 123;
    counter.reset = function () { };
    return counter;
}

let c = getCounter();
c(10);
c.reset();
c.interval = 5.0;
```

在使用JavaScript第三方庫的時候，你可能需要像上面那樣去完整地定義類型。

# 接口繼承類

當接口繼承了一個類類型時，它會繼承類的成員但不包括其實現。
就好像接口聲明了所有類中存在的成員，但並沒有提供具體實現一樣。
接口同樣會繼承到類的private和protected成員。
這意味著當你創建了一個接口繼承了一個擁有私有或受保護的成員的類時，這個接口類型只能被這個類或其子類所實現（implement）。

當你有一個龐大的繼承結構時這很有用，但要指出的是你的代碼只在子類擁有特定屬性時起作用。
除了繼承自基類，子類之間不必相關聯。
例：

```ts
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

class Button extends Control implements SelectableControl {
    select() { }
}

class TextBox extends Control {
    select() { }
}

// Error: Property 'state' is missing in type 'Image'.
class Image implements SelectableControl {
    select() { }
}

class Location {

}
```

在上面的例子裡，`SelectableControl`包含了`Control`的所有成員，包括私有成員`state`。
因為`state`是私有成員，所以只能夠是`Control`的子類們才能實現`SelectableControl`接口。
因為只有`Control`的子類才能夠擁有一個聲明於`Control`的私有成員`state`，這對私有成員的兼容性是必需的。

在`Control`類內部，是允許通過`SelectableControl`的實例來訪問私有成員`state`的。
實際上，`SelectableControl`就像`Control`一樣，並擁有一個`select`方法。
`Button`和`TextBox`類是`SelectableControl`的子類（因為它們都繼承自`Control`並有`select`方法），但`Image`和`Location`類並不是這樣的。
