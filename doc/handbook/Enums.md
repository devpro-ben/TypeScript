# 枚舉

使用枚舉我們可以定義一些帶名字的常量。
使用枚舉可以清晰地表達意圖或創建一組有區別的用例。
TypeScript支持數字的和基於字符串的枚舉。

## 數字枚舉

首先我們看看數字枚舉，如果你使用過其它編程語言應該會很熟悉。

```ts
enum Direction {
    Up = 1,
    Down,
    Left,
    Right
}
```

如上，我們定義了一個數字枚舉，`Up`使用初始化為`1`。
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
當我們不在乎成員的值的時候，這種自增長的行為是很有用處的，但是要注意每個枚舉成員的值都是不同的。

使用枚舉很簡單：通過枚舉的屬性來訪問枚舉成員，和枚舉的名字來訪問枚舉類型：

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

數字枚舉可以被混入到[計算過的和常量成員（如下所示）](#computed-and-constant-members)。
簡短地說，不帶初始化器的枚舉或者被放在第一的位置，或者被放在使用了數字常量或其它常量初始化了的枚舉後面。
換句話說，下面的情況是不被允許的：

```ts
enum E {
    A = getSomeValue(),
    B, // error! 'A' is not constant-initialized, so 'B' needs an initializer
}
```

## 字符串枚舉

字符串枚舉的概念很簡單，但是有細微的[運行時的差別](#enums-at-runtime)。
在一個字符串枚舉裡，每個成員都必須用字符串字面量，或另外一個字符串枚舉成員進行初始化。

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

由於字符串枚舉沒有自增長的行為，字符串枚舉可以很好的序列化。
換句話說，如果你正在調試並且必須要讀一個數字枚舉的運行時的值，這個值通常是很難讀的 - 它並不能表達有用的信息（儘管[反向映射](#enums-at-runtime)會有所幫助），字符串枚舉允許你提供一個運行時有意義的並且可讀的值，獨立於枚舉成員的名字。

## 異構枚舉（Heterogeneous enums）

從技術的角度來說，枚舉可以混合字符串和數字成員，但是似乎你並不會這麼做：

```ts
enum BooleanLikeHeterogeneousEnum {
    No = 0,
    Yes = "YES",
}
```

除非你真的想要利用JavaScript運行時的行為，否則我們不建議這樣做。

## 計算的和常量成員

每個枚舉成員都帶有一個值，它可以是*常量*或*計算出來的*。
當滿足如下條件時，枚舉成員被當作是常量：

* 它是枚舉的第一個成員且沒有初始化器，這種情況下它被賦予值`0`：

  ```ts
  // E.X is constant:
  enum E { X }
  ```

* 它不帶有初始化器且它之前的枚舉成員是一個*數字*常量。
  這種情況下，當前枚舉成員的值為它上一個枚舉成員的值加1。

  ```ts
  // All enum members in 'E1' and 'E2' are constant.

  enum E1 { X, Y, Z }

  enum E2 {
      A = 1, B, C
  }
  ```

* 枚舉成員使用*常量枚舉表達式*初始化。
  常量枚舉表達式是TypeScript表達式的子集，它可以在編譯階段求值。
  當一個表達式滿足下面條件之一時，它就是一個常量枚舉表達式：
  * 一個枚舉表達式字面量（主要是字符串字面量或數字字面量）
  * 一個對之前定義的常量枚舉成員的引用（可以是在不同的枚舉類型中定義的）
  * 帶括號的常量枚舉表達式
  * 一元運算符`+`, `-`, `~`其中之一應用在了常量枚舉表達式
  * 常量枚舉表達式做為二元運算符`+`, `-`, `*`, `/`, `%`, `<<`, `>>`, `>>>`, `&`, `|`, `^`的操作對象。
  若常量枚舉表達式求值後為`NaN`或`Infinity`，則會在編譯階段報錯。

所有其它情況的枚舉成員被當作是需要計算得出的值。

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

## 聯合枚舉與枚舉成員的類型

存在一種特殊的非計算的常量枚舉成員的子集：字面量枚舉成員。
字面量枚舉成員是指不帶有初始值的常量枚舉成員，或者是值被初始化為

* 任何字符串字面量（例如：`"foo"`，`"bar"`，`"baz"`）
* 任何數字字面量（例如：`1`, `100`）
* 應用了一元`-`符號的數字字面量（例如：`-1`, `-100`）

當所有枚舉成員都擁有字面量枚舉值時，它就帶有了一種特殊的語義。

首先，枚舉成員成為了類型！
例如，我們可以說某些成員*只能*是枚舉成員的值：

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

另一個變化是枚舉類型本身變成了每個枚舉成員的*聯合*。
雖然我們還沒有討論[聯合類型](./Advanced Types.md#union-types)，但你只要知道通過聯合枚舉，類型系統能夠利用這樣一個事實，它可以知道枚舉裡的值的集合。
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

這個例子裡，我們先檢查`x`是否不是`E.Foo`。
如果通過了這個檢查，然後`||`會發生短路效果，`if`語句體裡的內容會被執行。
然而，這個檢查沒有通過，那麼`x`則*只能*為`E.Foo`，因此沒理由再去檢查它是否為`E.Bar`。

## 運行時的枚舉

枚舉是在運行時真正存在的對象。
例如下面的枚舉：

```ts
enum E {
    X, Y, Z
}
```

can actually be passed around to functions

```ts
function f(obj: { X: number }) {
    return obj.X;
}

// Works, since 'E' has a property named 'X' which is a number.
f(E);
```

### 反向映射

除了創建一個以屬性名做為對象成員的對象之外，數字枚舉成員還具有了*反向映射*，從枚舉值到枚舉名字。
例如，在下面的例子中：

```ts
enum Enum {
    A
}
let a = Enum.A;
let nameOfA = Enum[a]; // "A"
```

TypeScript可能會將這段代碼編譯為下面的JavaScript：

```js
var Enum;
(function (Enum) {
    Enum[Enum["A"] = 0] = "A";
})(Enum || (Enum = {}));
var a = Enum.A;
var nameOfA = Enum[a]; // "A"
```

生成的代碼中，枚舉類型被編譯成一個對象，它包含了正向映射（`name` -> `value`）和反向映射（`value` -> `name`）。
引用枚舉成員總會生成為對屬性訪問並且永遠也不會內聯代碼。

要注意的是*不會*為字符串枚舉成員生成反向映射。

### `const`枚舉

大多數情況下，枚舉是十分有效的方案。
然而在某些情況下需求很嚴格。
為了避免在額外生成的代碼上的開銷和額外的非直接的對枚舉成員的訪問，我們可以使用`const`枚舉。
常量枚舉通過在枚舉上使用`const`修飾符來定義。

```ts
const enum Enum {
    A = 1,
    B = A * 2
}
```

常量枚舉只能使用常量枚舉表達式，並且不同於常規的枚舉，它們在編譯階段會被刪除。
常量枚舉成員在使用的地方會被內聯進來。
之所以可以這麼做是因為，常量枚舉不允許包含計算成員。

```ts
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]
```

生成後的代碼為：

```js
var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */];
```

# 外部枚舉

外部枚舉用來描述已經存在的枚舉類型的形狀。

```ts
declare enum Enum {
    A = 1,
    B,
    C = 2
}
```

外部枚舉和非外部枚舉之間有一個重要的區別，在正常的枚舉裡，沒有初始化方法的成員被當成常量成員。
對於非常量的外部枚舉而言，沒有初始化方法時被當做需要經過計算的。
