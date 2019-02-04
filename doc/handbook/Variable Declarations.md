# 變數宣告

`let`和`const`是JavaScript裡相對較新的變數宣告方式。
像我們之前提到過的，`let`在很多方面與`var`是相似的，但是可以幫助大家避免在JavaScript裡常見一些問題。
`const`是對`let`的一個增強，它能阻止對一個變數再次賦值。

因為TypeScript是JavaScript的超集，所以它本身就支援`let`和`const`。
下面我們會詳細說明這些新的宣告方式以及為什麼推薦使用它們來代替`var`。

如果你之前使用JavaScript時沒有特別在意，那麼這節內容會喚起你的回憶。
如果你已經對`var`宣告的怪異之處瞭如指掌，那麼你可以輕鬆地略過這節。

# `var` 宣告

一直以來我們都是通過`var`關鍵字定義JavaScript變數。

```ts
var a = 10;
```

大家都能理解，這裡定義了一個名為`a`值為`10`的變數。

我們也可以在函數內部定義變數：

```ts
function f() {
    var message = "Hello, world!";

    return message;
}
```

並且我們也可以在其它函數內部存取相同的變數。

```ts
function f() {
    var a = 10;
    return function g() {
        var b = a + 1;
        return b;
    }
}

var g = f();
g(); // returns 11;
```

上面的例子裡，`g`可以獲取到`f`函數裡定義的`a`變數。
每當`g`被呼叫時，它都可以存取到`f`裡的`a`變數。
即使當`g`在`f`已經執行完後才被呼叫，它仍然可以存取及修改`a`。

```ts
function f() {
    var a = 1;

    a = 2;
    var b = g();
    a = 3;

    return b;

    function g() {
        return a;
    }
}

f(); // returns 2
```

## 作用域規則

對於熟悉其它語言的人來說，`var`宣告有些奇怪的作用域規則。
看下面的例子：

```ts
function f(shouldInitialize: boolean) {
    if (shouldInitialize) {
        var x = 10;
    }

    return x;
}

f(true);  // returns '10'
f(false); // returns 'undefined'
```

有些讀者可能要多看幾遍這個例子。
變數`x`是定義在*`if`敘述裡面*，但是我們卻可以在敘述的外面存取它。
這是因為`var`宣告可以在包含它的函數，模組，命名空間或全域作用域內部任何位置被存取（我們後面會詳細介紹），包含它的代碼區塊對此沒有什麼影響。
有些人稱此為 *`var`作用域* 或 *函數作用域*。
函數參數也使用函數作用域。

這些作用域規則可能會引發一些錯誤。
其中之一就是，多次宣告同一個變數並不會報錯：

```ts
function sumMatrix(matrix: number[][]) {
    var sum = 0;
    for (var i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (var i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

這裡很容易看出一些問題，裡層的`for`迴圈會覆蓋變數`i`，因為所有`i`都引用相同的函數作用域內的變數。
有經驗的開發者們很清楚，這些問題可能在代碼審查時漏掉，引發無窮的麻煩。

## 捕獲變數怪異之處

快速的猜一下下面的代碼會返回什麼：

```ts
for (var i = 0; i < 10; i++) {
    setTimeout(function() { console.log(i); }, 100 * i);
}
```

介紹一下，`setTimeout`會在若干毫秒的延時後執行一個函數（等待其它代碼執行完畢）。

好吧，看一下結果：

```text
10
10
10
10
10
10
10
10
10
10
```

很多JavaScript程序員對這種行為已經很熟悉了，但如果你很不解，你並不是一個人。
大多數人期望輸出結果是這樣：

```text
0
1
2
3
4
5
6
7
8
9
```

還記得我們上面提到的捕獲變數嗎？
我們傳給`setTimeout`的每一個函數表達式實際上都引用了相同作用域裡的同一個`i`。

讓我們花點時間思考一下這是為什麼。
`setTimeout`在若干毫秒後執行一個函數，並且是在`for`迴圈結束後。
`for`迴圈結束後，`i`的值為`10`。
所以當函數被呼叫的時候，它會打印出`10`！

一個通常的解決方法是使用立即執行的函數表達式（IIFE）來捕獲每次迭代時`i`的值：

```ts
for (var i = 0; i < 10; i++) {
    // capture the current state of 'i'
    // by invoking a function with its current value
    (function(i) {
        setTimeout(function() { console.log(i); }, 100 * i);
    })(i);
}
```

這種奇怪的形式我們已經司空見慣了。
參數`i`會覆蓋`for`迴圈裡的`i`，但是因為我們起了同樣的名字，所以我們不用怎麼改`for`迴圈本體裡的代碼。

# `let` 宣告

現在你已經知道了`var`存在一些問題，這恰好說明了為什麼用`let`敘述來宣告變數。
除了名字不同外，`let`與`var`的寫法一致。

```ts
let hello = "Hello!";
```

主要的區別不在語法上，而是語義，我們接下來會深入研究。

## 區塊作用域

當用`let`宣告一個變數，它使用的是 *詞法作用域* 或 *區塊作用域*。
不同於使用`var`宣告的變數那樣可以在包含它們的函數外存取，區塊作用域變數在包含它們的區塊或`for`迴圈之外是不能存取的。

```ts
function f(input: boolean) {
    let a = 100;

    if (input) {
        // Still okay to reference 'a'
        let b = a + 1;
        return b;
    }

    // Error: 'b' doesn't exist here
    return b;
}
```

這裡我們定義了2個變數`a`和`b`。
`a`的作用域是`f`函數本體內，而`b`的作用域是`if`敘述區塊裡。

在`catch`敘述裡宣告的變數也具有同樣的作用域規則。

```ts
try {
    throw "oh no!";
}
catch (e) {
    console.log("Oh well.");
}

// Error: 'e' doesn't exist here
console.log(e);
```

擁有區塊層級作用域的變數的另一個特點是，它們不能在被宣告之前讀或寫。
雖然這些變數始終“存在”於它們的作用域裡，但在直到宣告它的代碼之前的區域都屬於*暫時性死區*。
它只是用來說明我們不能在`let`敘述之前存取它們，幸運的是TypeScript可以告訴我們這些信息。

```ts
a++; // illegal to use 'a' before it's declared;
let a;
```

注意一點，我們仍然可以在一個擁有區塊作用域變數被宣告前*獲取*它。
只是我們不能在變數宣告前去呼叫那個函數。
如果生成代碼目標為ES2015，現代的運行時會拋出一個錯誤；然而，現今TypeScript是不會報錯的。

```ts
function foo() {
    // okay to capture 'a'
    return a;
}

// 不能在'a'被宣告前呼叫'foo'
// 運行時應該拋出錯誤
foo();

let a;
```

關於*暫時性死區*的更多信息，查看這裡[Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_dead_zone_and_errors_with_let).

## 重新宣告及遮蔽

我們提過使用`var`宣告時，它不在乎你宣告多少次；你只會得到1個。

```ts
function f(x) {
    var x;
    var x;

    if (true) {
        var x;
    }
}
```

在上面的例子裡，所有`x`的宣告實際上都引用一個*相同*的`x`，並且這是完全有效的代碼。
這經常會成為bug的來源。
好的是，`let`宣告就不會這麼寬鬆了。

```ts
let x = 10;
let x = 20; // 錯誤，不能在1個作用域裡多次宣告`x`
```

並不是要求兩個均是區塊層級作用域的宣告TypeScript才會給出一個錯誤的警告。

```ts
function f(x) {
    let x = 100; // error: interferes with parameter declaration
}

function g() {
    let x = 100;
    var x = 100; // error: can't have both declarations of 'x'
}
```

並不是說區塊作用域變數不能用函數作用域變數來宣告。
而是區塊作用域變數需要在明顯不同的區塊裡宣告。

```ts
function f(condition, x) {
    if (condition) {
        let x = 100;
        return x;
    }

    return x;
}

f(false, 0); // returns 0
f(true, 0);  // returns 100
```

在一個巢狀作用域裡引入一個新名字的行為稱做*遮蔽*。
它是一把雙刃劍，它可能會不小心地引入新問題，同時也可能會解決一些錯誤。
例如，假設我們現在用`let`重寫之前的`sumMatrix`函數。

```ts
function sumMatrix(matrix: number[][]) {
    let sum = 0;
    for (let i = 0; i < matrix.length; i++) {
        var currentRow = matrix[i];
        for (let i = 0; i < currentRow.length; i++) {
            sum += currentRow[i];
        }
    }

    return sum;
}
```

這個版本的迴圈能得到正確的結果，因為內層迴圈的`i`可以遮蔽掉外層迴圈的`i`。

*通常*來講應該避免使用遮蔽，因為我們需要寫出清晰的代碼。
同時也有些場景適合利用它，你需要好好打算一下。

## 區塊作用域變數的獲取

在我們最初談及獲取用`var`宣告的變數時，我們簡略地探究了一下在獲取到了變數之後它的行為是怎樣的。
直觀地講，每次進入一個作用域時，它創建了一個變數的*環境*。
就算作用域內代碼已經執行完畢，這個環境與其捕獲的變數依然存在。

```ts
function theCityThatAlwaysSleeps() {
    let getCity;

    if (true) {
        let city = "Seattle";
        getCity = function() {
            return city;
        }
    }

    return getCity();
}
```

因為我們已經在`city`的環境裡獲取到了`city`，所以就算`if`敘述執行結束後我們仍然可以存取它。

回想一下前面`setTimeout`的例子，我們最後需要使用立即執行的函數表達式來獲取每次`for`迴圈迭代裡的狀態。
實際上，我們做的是為獲取到的變數創建了一個新的變數環境。
這樣做挺痛苦的，但是幸運的是，你不必在TypeScript裡這樣做了。

當`let`宣告出現在迴圈體裡時擁有完全不同的行為。
不僅是在迴圈裡引入了一個新的變數環境，而是針對*每次迭代*都會創建這樣一個新作用域。
這就是我們在使用立即執行的函數表達式時做的事，所以在`setTimeout`例子裡我們僅使用`let`宣告就可以了。

```ts
for (let i = 0; i < 10 ; i++) {
    setTimeout(function() {console.log(i); }, 100 * i);
}
```

會輸出與預料一致的結果：

```text
0
1
2
3
4
5
6
7
8
9
```

# `const` 宣告

`const` 宣告是宣告變數的另一種方式。

```ts
const numLivesForCat = 9;
```

它們與`let`宣告相似，但是就像它的名字所表達的，它們被賦值後不能再改變。
換句話說，它們擁有與`let`相同的作用域規則，但是不能對它們重新賦值。

這很好理解，它們引用的值是*不可變的*。

```ts
const numLivesForCat = 9;
const kitty = {
    name: "Aurora",
    numLives: numLivesForCat,
}

// Error
kitty = {
    name: "Danielle",
    numLives: numLivesForCat
};

// all "okay"
kitty.name = "Rory";
kitty.name = "Kitty";
kitty.name = "Cat";
kitty.numLives--;
```

除非你使用特殊的方法去避免，實際上`const`變數的內部狀態是可修改的。
幸運的是，TypeScript允許你將物件的成員設置成唯讀的。
[介面](./Interfaces.md)一章有詳細說明。

# `let` vs. `const`

現在我們有兩種作用域相似的宣告方式，我們自然會問到底應該使用哪個。
與大多數泛泛的問題一樣，答案是：依情況而定。

使用[最小特權原則](https://en.wikipedia.org/wiki/Principle_of_least_privilege)，所有變數除了你計畫去修改的都應該使用`const`。
基本原則就是如果一個變數不需要對它寫入，那麼其它使用這些代碼的人也不能夠寫入它們，並且要思考為什麼會需要對這些變數重新賦值。
使用`const`也可以讓我們更容易的推測資料的流動。

跟據你的自己判斷，如果合適的話，與團隊成員商議一下。

這個手冊大部分地方都使用了`let`宣告。

# 解構

另一個TypeScript已經有的ECMAScript 2015 特性是解構(destructuring)。
完整列表請參見 [the article on the Mozilla Developer Network](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)。
本章，我們將給出一個簡短的概述。

## 解構陣列

最簡單的解構莫過於陣列的解構賦值了：

```ts
let input = [1, 2];
let [first, second] = input;
console.log(first); // outputs 1
console.log(second); // outputs 2
```

這創建了2個命名變數 `first` 和 `second`。
相當於使用了索引，但更為方便：

```ts
first = input[0];
second = input[1];
```

解構作用於已宣告的變數會更好：

```ts
// swap variables
[first, second] = [second, first];
```

作用於函數參數：

```ts
function f([first, second]: [number, number]) {
    console.log(first);
    console.log(second);
}
f(input);
```

你可以在陣列裡使用`...`語法創建剩餘變數：

```ts
let [first, ...rest] = [1, 2, 3, 4];
console.log(first); // outputs 1
console.log(rest); // outputs [ 2, 3, 4 ]
```

當然，由於是JavaScript, 你可以忽略你不關心的尾隨元素：

```ts
let [first] = [1, 2, 3, 4];
console.log(first); // outputs 1
```

或其它元素：

```ts
let [, second, , fourth] = [1, 2, 3, 4];
```

## 物件解構

你也可以解構物件：

```ts
let o = {
    a: "foo",
    b: 12,
    c: "bar"
};
let { a, b } = o;
```

這通過 `o.a` and `o.b` 創建了 `a` 和 `b` 。
注意，如果你不需要 `c` 你可以忽略它。

就像陣列解構，你可以用沒有宣告的賦值：

```ts
({ a, b } = { a: "baz", b: 101 });
```

> 譯註：`a` 和 `b` 必須先用 `let` 宣告。所以，這有點像是重賦值，而且型別要相同才能重新賦值。

注意，我們需要用括號將它括起來，因為Javascript通常會將以 `{` 起始的敘述解析為一個塊。

你可以在物件裡使用`...`語法創建剩餘變數：

```ts
let { a, ...passthrough } = o;
let total = passthrough.b + passthrough.c.length;

```

### 屬性重命名

你也可以給屬性以不同的名字：

```ts
let { a: newName1, b: newName2 } = o;
```

這裡的語法開始變得混亂。
你可以將 `a: newName1` 讀做 "`a` 作為 `newName1`"。
方向是從左到右，好像你寫成了以下樣子：

```ts
let newName1 = o.a;
let newName2 = o.b;
```

令人困惑的是，這裡的冒號 *不是* 指示型別的。
如果你想指定它的型別， 仍然需要在其後寫上完整的模式。

```ts
let {a, b}: {a: string, b: number} = o;
```

### 預設值

預設值可以讓你在屬性為 `undefined` 時使用預設值：

```ts
function keepWholeObject(wholeObject: { a: string, b?: number }) {
    let { a, b = 1001 } = wholeObject;
}
```

現在，即使 `b` 為 undefined ， `keepWholeObject` 函數的變數 `wholeObject` 的屬性 `a` 和 `b` 都會有值。

## 函數宣告

解構也能用於函數宣告。
看以下簡單的情況：

```ts
type C = { a: string, b?: number }
function f({ a, b }: C): void {
    // ...
}
```

但是，通常情況下更多的是指定預設值，解構預設值有些棘手。
首先，你需要在預設值之前設置其格式。

```ts
function f({ a="", b=0 } = {}): void {
    // ...
}
f();
```

> 上面的代碼是一個型別推斷的例子，將在本手冊後文介紹。

其次，你需要知道在解構屬性上給予一個預設或可選的屬性用來替換主初始化列表。
要知道 `C` 的定義有一個 `b` 可選屬性：

```ts
function f({ a, b = 0 } = { a: "" }): void {
    // ...
}
f({ a: "yes" }); // ok, default b = 0
f(); // ok, default to {a: ""}, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```

要小心使用解構。
從前面的例子可以看出，就算是最簡單的解構表達式也是難以理解的。
尤其當存在深層巢狀解構的時候，就算這時沒有堆疊在一起的重命名，預設值和類型註解，也是令人難以理解的。
解構表達式要儘量保持小而簡單。
你自己也可以直接使用解構將會生成的賦值表達式。

## 展開

展開操作符正與解構相反。
它允許你將一個陣列展開為另一個陣列，或將一個物件展開為另一個物件。
例如：

```ts
let first = [1, 2];
let second = [3, 4];
let bothPlus = [0, ...first, ...second, 5];
```

這會令`bothPlus`的值為`[0, 1, 2, 3, 4, 5]`。
展開操作創建了`first`和`second`的一份淺拷貝。
它們不會被展開操作所改變。

你還可以展開物件：

```ts
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { ...defaults, food: "rich" };
```

`search`的值為`{ food: "rich", price: "$$", ambiance: "noisy" }`。
物件的展開比陣列的展開要複雜的多。
像陣列展開一樣，它是從左至右進行處理，但結果仍為物件。
這就意味著出現在展開物件後面的屬性會覆蓋前面的屬性。
因此，如果我們修改上面的例子，在結尾處進行展開的話：

```ts
let defaults = { food: "spicy", price: "$$", ambiance: "noisy" };
let search = { food: "rich", ...defaults };
```

那麼，`defaults`裡的`food`屬性會重寫`food: "rich"`，在這裡這並不是我們想要的結果。

物件展開還有其它一些意想不到的限制。
首先，它僅包含物件
[自身的可列舉屬性](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)。
大體上是說當你展開一個物件實例時，你會丟失其方法：

```ts
class C {
  p = 12;
  m() {
  }
}
let c = new C();
let clone = { ...c };
clone.p; // ok
clone.m(); // error!
```

其次，TypeScript編譯器不允許展開泛型函數上的型別參數。
這個特性會在TypeScript的未來版本中考慮實現。
