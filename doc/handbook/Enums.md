# 列舉

使用列舉我們可以定義一些帶名字的常數。
使用列舉可以清晰地表達意圖或建立一組有區別的案例。
TypeScript支援數字的和基於字串的列舉。

## 數字列舉

首先我們看看數字列舉，如果您使用過其它編程語言應該會很熟悉。

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

如上，我們定義了一個數字列舉，`Up`使用初始化為`1`。
其餘的成員會從`1`開始自動增長。
換句話說，`Direction.Up`的值為`1`，`Down`為`2`，`Left`為`3`，`Right`為`4`。

我們還可以完全不使用初始化器：

```ts
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

現在，`Up`的值為`0`，`Down`的值為`1`等等。
當我們不在乎成員的值的時候，這種自動增長的行為是很有用處的，但是要注意每個列舉成員的值都是不同的。

使用列舉很簡單：透過列舉的屬性來存取列舉成員，和使用列舉的名稱宣告型別：

```ts
enum Response {
    No = 0,
    Yes = 1,
}

function respond(recipient: string, message: Response): void {
    // ...
}

respond("Princess Caroline", Response.Yes)
```

數字列舉可以被混入到[計算過的和常數成員(如下所示)](#計算的和常數成員)。
簡短地說，不帶初始化器的列舉必須被放在第一個，或者被放在使用了數字常數初始化或其它常數列舉成員後面。
換句話說，下面的情況是不被允許的：

```ts
enum E {
    A = getSomeValue(),
    B, // error! 'A' is not constant-initialized, so 'B' needs an initializer
}
```

## 字串列舉

字串列舉的概念很簡單，但是有細微的[執行時的差別](#執行時的列舉)。
在一個字串列舉裡，每個成員都必須用字串字面值，或另外一個字串列舉成員進行初始化。

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

由於字串列舉沒有自動增長的行為，字串列舉可以很好的序列化。
換句話說，如果您正在偵錯並且必須要讀一個數字列舉的執行時期的值，這個值通常是很難讀的 - 它並不能表達有用的訊息(儘管[反向映射](#執行時的列舉)會有所幫助)，字串列舉允許您提供一個執行時期有意義的並且可讀的值，獨立於列舉成員的名字。

## 異質列舉(Heterogeneous enums)

從技術的角度來說，列舉可以混合字串和數字成員，但是似乎您並不會這麼做：

```ts
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

除非您真的想要利用JavaScript執行時期的行為，否則我們不建議這樣做。

## 計算的和常數成員

每個列舉成員都帶有一個值，它可以是*常數*或*計算出來的*。
當滿足如下條件時，列舉成員被當作是常數：

* 它是列舉的第一個成員且沒有初始化器，這種情況下它被賦予值`0`：

  ```ts
  // E.X is constant:
  enum E { X }
  ```

* 它不帶有初始化器且它之前的列舉成員是一個*數字*常數。
  這種情況下，當前列舉成員的值為它上一個列舉成員的值加1。

  ```ts
  // All enum members in 'E1' and 'E2' are constant.

  enum E1 { X, Y, Z }

  enum E2 {
      A = 1, B, C
  }
  ```

* 列舉成員使用*常數列舉運算式*初始化。
  常數列舉運算式是TypeScript運算式的子集，它可以在編譯階段求值。
  當一個運算式滿足下面條件之一時，它就是一個常數列舉運算式：
  * 一個列舉運算式字面值(主要是字串字面值或數字字面值)
  * 一個對之前定義的常數列舉成員的引用(可以是在不同的列舉型別中定義的)
  * 帶括號的常數列舉運算式
  * 一元運算符`+`, `-`, `~`其中之一應用在了常數列舉運算式
  * 常數列舉運算式做為二元運算符`+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^`的操作物件。
  若常數列舉運算式求值後為`NaN`或`Infinity`，則會在編譯階段報錯。

所有其它情況的列舉成員被當作是需要計算得出的值。

```ts
enum FileAccess {
    // constant members
    None,
    Read    = 1 << 1,
    Write   = 1 << 2,
    ReadWrite  = Read | Write,
    // computed member
    G = "123".length
}
```

## 聯合列舉與列舉成員的型別

存在一種特殊的非計算的常數列舉成員的子集：字面值列舉成員。
字面值列舉成員是指不帶有初始值的常數列舉成員，或者是值被初始化為

* 任何字串字面值(例如：`"foo"`，`"bar"`，`"baz"`)
* 任何數字字面值(例如：`1`, `100`)
* 應用了一元`-`符號的數字字面值(例如：`-1`, `-100`)

當所有列舉成員都擁有字面值列舉值時，它就帶有了一種特殊的語義。

首先，列舉成員成為了型別！
例如，我們可以說某些成員*只能*是列舉成員的值：

```ts
enum ShapeKind {
    Circle,
    Square,
}

interface Circle {
    kind: ShapeKind.Circle;
    radius: number;
}

interface Square {
    kind: ShapeKind.Square;
    sideLength: number;
}

let c: Circle = {
    kind: ShapeKind.Square,
    //    ~~~~~~~~~~~~~~~~ Error!
    radius: 100,
}
```

另一個變化是列舉型別本身變成了每個列舉成員的*聯合*。
雖然我們還沒有討論[聯合型別](./Advanced Types.md#union-types)，但您只要知道透過聯合列舉，型別系統能夠利用這樣一個事實，它可以知道列舉裡的值的集合。
因此，TypeScript能夠捕獲在比較值的時候犯的愚蠢的錯誤。
例如：

```ts
enum E {
    Foo,
    Bar,
}

function f(x: E) {
    if (x !== E.Foo || x !== E.Bar) {
        //             ~~~~~~~~~~~
        // Error! Operator '!==' cannot be applied to types 'E.Foo' and 'E.Bar'.
    }
}
```

這個範例裡，我們先檢查`x`是否不是`E.Foo`。
如果透過了這個檢查，然後`||`會發生短路效果，`if`敘述本體裡的內容會被執行。
然而，這個檢查沒有通過，那麼`x`則*只能*為`E.Foo`，因此沒理由再去檢查它是否為`E.Bar`。

## 執行時的列舉

列舉是在執行時真正存在的物件。
例如下面的列舉：

```ts
enum E {
    X, Y, Z
}
```

實際上可以傳遞給函數

```ts
function f(obj: { X: number }) {
    return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
```

### 反向映射

除了建立一個以屬性名稱做為物件成員的物件之外，數字列舉成員還具有了*反向映射*，從列舉值到列舉名字。
例如，在下面的範例中：

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

TypeScript可能會將這段程式碼編譯為下面的JavaScript：

```js
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[a]; // "A"
```

產生的程式碼中，列舉型別被編譯成一個物件，它包含了正向映射(`name` -> `value`)和反向映射(`value` -> `name`)。
引用列舉成員總會產生為對屬性存取並且永遠也不會產生內聯(inline)程式碼。

要注意的是*不會*為字串列舉成員產生反向映射。

### `const`列舉

大多數情況下，列舉是十分有效的方案。
然而在某些情況下需求很嚴格。
為了避免在額外產生的程式碼上的開銷和額外的非直接的對列舉成員的存取，我們可以使用`const`列舉。
常數列舉透過在列舉上使用`const`修飾符來定義。

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

常數列舉只能使用常數列舉運算式，並且不同於常規的列舉，它們在編譯階段會被刪除。
常數列舉成員在使用的地方會被內聯(inline)進來。
之所以可以這麼做是因為，常數列舉不允許包含計算成員。

```ts
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

產生後的程式碼為：

```js
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

# 外部列舉

外部列舉用來描述已經存在的列舉型別的形狀。

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

外部列舉和非外部列舉之間有一個重要的區別，在正常的列舉裡，沒有初始化方法的成員被當成常數成員。
對於非常數的外部列舉而言，沒有初始化的列舉成員總是被當做需要經過計算的。
