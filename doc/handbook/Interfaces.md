# 介紹

TypeScript的核心原則之一是對值所具有的*結構*進行型別檢查。
它有時被稱做「鴨式辨型法」或「結構性子型別化」。
在TypeScript裡，介面的作用就是為這些型別命名和為您的程式碼或第三方程式碼定義契約。

# 介面初探

下面透過一個簡單範例來觀察介面是如何工作的：

```ts
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

let myObj = { size: 10, label: "Size 10 Object" };
printLabel(myObj);
```

型別檢查器會查看`printLabel`的呼叫。
`printLabel`有一個參數，並要求這個物件參數有一個名為`label`型別為`string`的屬性。
需要注意的是，我們傳入的物件參數實際上會包含很多屬性，但是編譯器只會檢查那些必需的屬性是否存在，並且其型別是否匹配。
然而，有些時候TypeScript卻並不會這麼寬鬆，我們下面會稍做講解。

下面我們重寫上面的範例，這次使用介面來描述：必須包含一個`label`屬性且型別為`string`：

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

`LabelledValue`介面就好比一個名字，用來描述上面範例裡的要求。
它代表了有一個`label`屬性且型別為`string`的物件。
需要注意的是，我們在這裡並不能像在其它語言裡一樣，說傳給`printLabel`的物件實作了這個介面。我們只會去關注值的外形。
只要傳入的物件滿足上面提到的必要條件，那麼它就是被允許的。

還有一點值得提的是，型別檢查器不會去檢查屬性的順序，只要對應的屬性存在並且型別也是對的就可以。

# 選擇性屬性

介面裡的屬性不全都是必需的。
有些是只在某些條件下存在，或者根本不存在。
選擇性屬性在應用「option bags」模式時很常用，即給函數傳入的參數物件中只有部分屬性賦值了。

下面是應用了「option bags」的範例：

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

帶有選擇性屬性的介面與普通的介面定義差不多，只是在選擇性屬性名字定義的後面加一個`?`符號。

選擇性屬性的好處之一是可以對可能存在的屬性進行預定義，好處之二是可以捕獲引用了不存在的屬性時的錯誤。
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

# 唯讀屬性

一些物件屬性只能在物件剛剛建立的時候修改其值。
您可以在屬性名前用`readonly`來指定唯讀屬性:

```ts
interface Point {
    readonly x: number;
    readonly y: number;
}
```

您可以透過賦值一個物件字面值來構造一個`Point`。
賦值後，`x`和`y`再也不能被改變了。

```ts
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

TypeScript具有`ReadonlyArray<T>`型別，它與`Array<T>`相似，只是把所有可變方法去掉了，因此可以確保陣列建立後再也不能被修改：

```ts
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!
ro.length = 100; // error!
a = ro; // error!
```

上面程式碼的最後一行，可以看到就算把整個`ReadonlyArray`賦值到一個普通陣列也是不可以的。
但是您可以用型別斷言重寫：

```ts
a = ro as number[];
```

## `readonly` vs `const`

最簡單判斷該用`readonly`還是`const`的方法是看要把它做為變數使用還是做為一個屬性。
做為變數使用的話用`const`，若做為屬性則使用`readonly`。

# 額外的屬性檢查

我們在第一個範例裡使用了介面，TypeScript讓我們傳入`{ size: number; label: string; }`到僅期望得到`{ label: string; }`的函數裡。
我們已經學過了選擇性屬性，並且知道他們在「option bags」模式裡很有用。

然而，天真地將這兩者結合的話就會像在JavaScript裡那樣搬起石頭砸自己的腳。
比如，拿`createSquare`範例來說：

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

您可能會爭辯這個程式已經正確地型別化了，因為`width`屬性是相容的，不存在`color`屬性，而且額外的`colour`屬性是無意義的。

然而，TypeScript會認為這段程式碼可能存在bug。
物件字面值會被特殊對待而且會經過*額外屬性檢查*，當將它們賦值給變數或作為參數傳遞的時候。
如果一個物件字面值存在任何「目標型別」不包含的屬性時，您會得到一個錯誤。

```ts
// error: 'colour' not expected in type 'SquareConfig'
let mySquare = createSquare({ colour: "red", width: 100 });
```

繞開這些檢查非常簡單。
最簡便的方法是使用型別斷言：

```ts
let mySquare = createSquare({ width: 100, opacity: 0.5 } as SquareConfig);
```

然而，最佳的方式是能夠加入一個字串索引簽名，前提是您能夠確定這個物件可能具有某些做為特殊用途使用的額外屬性。
如果`SquareConfig`帶有上面定義的型別的`color`和`width`屬性，並且*還會*帶有任意數量的其它屬性，那麼我們可以這樣定義它：

```ts
interface SquareConfig {
    color?: string;
    width?: number;
    [propName: string]: any;
}
```

我們稍後會講到索引簽名，但在這我們要表示的是`SquareConfig`可以有任意數量的屬性，並且只要它們不是`color`和`width`，那麼就無所謂它們的型別是什麼。

還有最後一種跳過這些檢查的方式，這可能會讓您感到驚訝，它就是將這個物件賦值給一個另一個變數：
因為`squareOptions`不會經過額外屬性檢查，所以編譯器不會報錯。

```ts
let squareOptions = { colour: "red", width: 100 };
let mySquare = createSquare(squareOptions);
```

要留意，在像上面一樣的簡單程式碼裡，您可能不應該去繞開這些檢查。
對於包含方法和內部狀態的複雜物件字面值來講，您可能需要使用這些技巧，但是大部額外屬性檢查錯誤是真正的bug。
就是說您遇到了額外型別檢查出的錯誤，比如「option bags」，您應該去審查一下您的型別宣告。
在這裡，如果支援傳入`color`或`colour`屬性到`createSquare`，您應該修改`SquareConfig`定義來體現出這一點。

# 函數型別

介面能夠描述JavaScript中物件擁有的各種各樣的外形。
除了描述帶有屬性的普通物件外，介面也可以描述函數型別。

為了使用介面表示函數型別，我們需要給介面定義一個呼叫簽名。
它就像是一個只有參數列表和返回值型別的函數定義。參數列表裡的每個參數都需要名字和型別。

```ts
interface SearchFunc {
  (source: string, subString: string): boolean;
}
```

這樣定義後，我們可以像使用其它介面一樣使用這個函數型別的介面。
下例展示了如何建立一個函數型別的變數，並將一個同型別的函數賦值給這個變數。

```ts
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string) {
  let result = source.search(subString);
  return result > -1;
}
```

對於函數型別的型別檢查來說，函數的參數名不需要與介面裡定義的名字相匹配。
比如，我們使用下面的程式碼重寫上面的範例：

```ts
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string): boolean {
  let result = src.search(sub);
  return result > -1;
}
```

函數的參數會逐個進行檢查，要求對應位置上的參數型別是相容的。
如果您不想指定型別，TypeScript的型別系統會推斷出參數型別，因為函數直接賦值給了`SearchFunc`型別變數。
函數的返回值型別是透過其返回值推斷出來的(此例是`false`和`true`)。
如果讓這個函數返回數字或字串，型別檢查器會警告我們函數的返回值型別與`SearchFunc`介面中的定義不匹配。

```ts
let mySearch: SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

# 可索引的型別

與使用介面描述函數型別差不多，我們也可以描述那些能夠「透過索引得到」的型別，比如`a[10]`或`ageMap["daniel"]`。
可索引型別具有一個*索引簽名*，它描述了物件索引的型別，還有對應的索引返回值型別。
讓我們看一個範例：

```ts
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

上面範例裡，我們定義了`StringArray`介面，它具有索引簽名。
這個索引簽名表示了當用`number`去索引`StringArray`時會得到`string`型別的返回值。

Typescript支援兩種索引簽名：字串和數字。
可以同時使用兩種型別的索引，但是數字索引的返回值必須是字串索引返回值型別的子型別。
這是因為當使用`number`來索引時，JavaScript會將它轉換成`string`然後再去索引物件。
也就是說用`100`(一個`number`)去索引等同於使用`"100"`(一個`string`)去索引，因此兩者需要保持一致。

```ts
class Animal {
    name: string;
}
class Dog extends Animal {
    breed: string;
}

// 錯誤：使用數值型的字串索引，有時會得到完全不同的Animal!
interface NotOkay {
    [x: number]: Animal;
    [x: string]: Dog;
}
```

字串索引簽名能夠很好的描述`dictionary`模式，並且它們也會確保所有屬性與其返回值型別相匹配。
因為字串索引宣告了`obj.property`和`obj["property"]`兩種形式都可以。
下面的範例裡，`name`的型別與字串索引型別不匹配，所以型別檢查器給出一個錯誤提示：

```ts
interface NumberDictionary {
  [index: string]: number;
  length: number;    // 可以，length是number型別
  name: string       // 錯誤，`name`的型別與索引型別返回值的型別不匹配
}
```

最後，您可以將索引簽名設置為唯讀，這樣就防止了給索引賦值：

```ts
interface ReadonlyStringArray {
    readonly [index: number]: string;
}
let myArray: ReadonlyStringArray = ["Alice", "Bob"];
myArray[2] = "Mallory"; // error!
```

您不能設置`myArray[2]`，因為索引簽名是唯讀的。

# 類別型別

## 實作介面

與C#或Java裡介面的基本作用一樣，TypeScript也能夠用它來明確的強制一個類別去符合某種契約。

```ts
interface ClockInterface {
    currentTime: Date;
}

class Clock implements ClockInterface {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

您也可以在介面中描述一個方法，在類裡實作它，如同下面的`setTime`方法一樣：

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

介面描述了類的公共部分，而不是公共和私有兩部分。
它不會幫您檢查類是否具有某些私有成員。

## 類別靜態部分與實例部分的區別

當您操作類別和介面的時候，您要知道類別是具有兩個型別的：靜態部分的型別和實例部分的型別。
您會注意到，當您用建構子簽名去定義一個介面並試圖定義一個類別去實作這個介面時會得到一個錯誤：

```ts
interface ClockConstructor {
    new (hour: number, minute: number);
}

class Clock implements ClockConstructor {
    currentTime: Date;
    constructor(h: number, m: number) { }
}
```

這裡因為當一個類實作了一個介面時，只對其實例部分進行型別檢查。
constructor存在於類別的靜態部分，所以不在檢查的範圍內。

因此，我們應該直接操作類別的靜態部分。
看下面的範例，我們定義了兩個介面，`ClockConstructor`為建構函數所用和`ClockInterface`為實例方法所用。
為了方便我們定義一個建構函數`createClock`，它用傳入的型別建立實例。

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

因為`createClock`的第一個參數是`ClockConstructor`型別，在`createClock(AnalogClock, 7, 32)`裡，會檢查`AnalogClock`是否符合建構函數簽名。

# 繼承介面

和類別一樣，介面也可以相互繼承。
這讓我們能夠從一個介面裡複製成員到另一個介面裡，可以更靈活地將介面分割到可重用的模組裡。

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

一個介面可以繼承多個介面，建立出多個介面的合成介面。

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

# 混合型別

先前我們提過，介面能夠描述JavaScript裡豐富的型別。
因為JavaScript其動態靈活的特點，有時您會希望一個物件可以同時具有上面提到的多種型別。

一個範例就是，一個物件可以同時做為函數和物件使用，並帶有額外的屬性。

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

在使用JavaScript第三方庫的時候，您可能需要像上面那樣去完整地定義型別。

# 介面繼承類別

當介面繼承了一個型別時，它會繼承類別的成員但不包括其實作。
就好像介面宣告了所有類別中存在的成員，但並沒有提供具體實作一樣。
介面同樣會繼承到類別的private和protected成員。
這意味著當您建立了一個介面繼承了一個擁有私有或受保護的成員的類別時，這個介面型別只能被這個類別或其子類別所實作(implement)。

當您有一個龐大的繼承結構時這很有用，但要指出的是您的程式碼只在子類別擁有特定屬性時起作用。
除了繼承自基礎類別，子類別之間不必相關聯。
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

在上面的範例裡，`SelectableControl`包含了`Control`的所有成員，包括私有成員`state`。
因為`state`是私有成員，所以只能夠是`Control`的子類別們才能實作`SelectableControl`介面。
因為只有`Control`的子類別才能夠擁有一個宣告於`Control`的私有成員`state`，這對私有成員的相容性是必需的。

在`Control`類別內部，是允許透過`SelectableControl`的實例來存取私有成員`state`的。
實際上，`SelectableControl`就像`Control`一樣，並擁有一個`select`方法。
`Button`和`TextBox`類別是`SelectableControl`的子類別(因為它們都繼承自`Control`並有`select`方法)，但`Image`和`Location`類別並不是這樣的。
