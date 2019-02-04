# 介紹

為了讓程式有價值，我們需要能夠處理最簡單的資料單元：數字，字串，結構體，布林值等。
TypeScript支持與JavaScript幾乎相同的資料型別，此外還提供了實用的列舉型別方便我們使用。

# 布林值

最基本的資料型別就是簡單的true/false值，在JavaScript和TypeScript裡叫做`boolean`（其它語言中也一樣）。

```ts
let isDone: boolean = false;
```

# 數字

和JavaScript一樣，TypeScript裡的所有數字都是浮點數。
這些浮點數的型別是`number`。
除了支持十進位和十六進位字面值，TypeScript還支持ECMAScript 2015中引入的二進位和八進位字面值。

```ts
let decLiteral: number = 6;
let hexLiteral: number = 0xf00d;
let binaryLiteral: number = 0b1010;
let octalLiteral: number = 0o744;
```

# 字串

JavaScript程式的另一項基本操作是處理網頁或服務器端的文字資料。
像其它語言裡一樣，我們使用`string`表示文字資料型別。
和JavaScript一樣，可以使用雙引號（`"`）或單引號（`'`）表示字串。

```ts
let name: string = "bob";
name = "smith";
```

你還可以使用*模版字串*，它可以定義多行文字和內嵌運算式。
這種字串是被反引號包圍（`` ` ``），並且以`${ expr }`這種形式嵌入運算式

```ts
let name: string = `Gene`;
let age: number = 37;
let sentence: string = `Hello, my name is ${ name }.

I'll be ${ age + 1 } years old next month.`;
```

這與下面定義`sentence`的方式效果相同：

```ts
let sentence: string = "Hello, my name is " + name + ".\n\n" +
    "I'll be " + (age + 1) + " years old next month.";
```

# 陣列

TypeScript像JavaScript一樣可以操作陣列元素。
有兩種方式可以定義陣列。
第一種，可以在元素型別後面接上`[]`，表示由此型別元素組成的一個陣列：

```ts
let list: number[] = [1, 2, 3];
```

第二種方式是使用陣列泛型，`Array<元素型別>`：

```ts
let list: Array<number> = [1, 2, 3];
```

# 元組 Tuple

元組型別允許表示一個已知元素數量和型別的陣列，各元素的型別不必相同。
比如，你可以定義一對值分別為`string`和`number`型別的元組。

```ts
// Declare a tuple type
let x: [string, number];
// Initialize it
x = ['hello', 10]; // OK
// Initialize it incorrectly
x = [10, 'hello']; // Error
```

當存取一個已知索引的元素，會得到正確的型別：

```ts
console.log(x[0].substr(1)); // OK
console.log(x[1].substr(1)); // Error, 'number' does not have 'substr'
```

當存取一個越界的元素，會使用聯合型別替代：

```ts
x[3] = 'world'; // OK, 字串可以賦值給(string | number)型別

console.log(x[5].toString()); // OK, 'string' 和 'number' 都有 toString

x[6] = true; // Error, 布爾不是(string | number)型別
```

聯合型別是進階主題，我們會在以後的章節裡討論它。

# 列舉

`enum`型別是對JavaScript標準資料型別的一個補充。
像C#等其它語言一樣，使用列舉型別可以為一組數值賦予友好的名字。

```ts
enum Color {Red, Green, Blue}
let c: Color = Color.Green;
```

預設情況下，從`0`開始為元素編號。
你也可以手動的指定成員的數值。
例如，我們將上面的例子改成從`1`開始編號：

```ts
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green;
```

或者，全部都採用手動賦值：

```ts
enum Color {Red = 1, Green = 2, Blue = 4}
let c: Color = Color.Green;
```

列舉型別提供的一個便利是你可以由列舉的值得到它的名字。
例如，我們知道數值為2，但是不確定它映射到Color裡的哪個名字，我們可以查找相應的名字：

```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2];

console.log(colorName);  // 顯示'Green'因為上面代碼裡它的值是2
```

# 任意值

有時候，我們會想要為那些在編程階段還不清楚型別的變數指定一個型別。
這些值可能來自於動態的內容，比如來自用戶輸入或第三方代碼庫。
這種情況下，我們不希望型別檢查器對這些值進行檢查而是直接讓它們通過編譯階段的檢查。
那麼我們可以使用`any`型別來標記這些變數：

```ts
let notSure: any = 4;
notSure = "maybe a string instead";
notSure = false; // okay, definitely a boolean
```

在對現有代碼進行改寫的時候，`any`型別是十分有用的，它允許你在編譯時可選擇地包含或移除型別檢查。
你可能認為`Object`有相似的作用，就像它在其它語言中那樣。
但是`Object`型別的變數只是允許你給它賦任意值 - 但是卻不能夠在它上面呼叫任意的方法，即便它真的有這些方法：

```ts
let notSure: any = 4;
notSure.ifItExists(); // okay, ifItExists might exist at runtime
notSure.toFixed(); // okay, toFixed exists (but the compiler doesn't check)

let prettySure: Object = 4;
prettySure.toFixed(); // Error: Property 'toFixed' doesn't exist on type 'Object'.
```

當你只知道一部分資料的型別時，`any`型別也是有用的。
比如，你有一個陣列，它包含了不同的型別的資料：

```ts
let list: any[] = [1, true, "free"];

list[1] = 100;
```

# 空值

某種程度上來說，`void`型別像是與`any`型別相反，它表示沒有任何型別。
當一個函數沒有傳回值時，你通常會見到其傳回值型別是`void`：

```ts
function warnUser(): void {
    console.log("This is my warning message");
}
```

聲明一個`void`型別的變數沒有什麼大用，因為你只能為它賦予`undefined`和`null`：

```ts
let unusable: void = undefined;
```

# Null 和 Undefined

TypeScript裡，`undefined`和`null`兩者各自有自己的型別分別叫做`undefined`和`null`。
和`void`相似，它們的本身的型別用處不是很大：

```ts
// Not much else we can assign to these variables!
let u: undefined = undefined;
let n: null = null;
```

預設情況下`null`和`undefined`是所有型別的子型別。
就是說你可以把`null`和`undefined`賦值給`number`型別的變數。

然而，當你指定了`--strictNullChecks`標記，`null`和`undefined`只能賦值給`void`和它們各自。
這能避免*很多*常見的問題。
也許在某處你想傳入一個`string`或`null`或`undefined`，你可以使用聯合型別`string | null | undefined`。
再次說明，稍後我們會介紹聯合型別。

> 注意：我們鼓勵儘可能地使用`--strictNullChecks`，但在本手冊裡我們假設這個標記是關閉的。

# Never

`never`型別表示的是那些永不存在的值的型別。
例如，`never`型別是那些總是會拋出異常或根本就不會有傳回值的函數運算式或箭頭函數運算式的傳回值型別；
變數也可能是`never`型別，當它們被永不為真的型別保護所約束時。

`never`型別是任何型別的子型別，也可以賦值給任何型別；然而，*沒有*型別是`never`的子型別或可以賦值給`never`型別（除了`never`本身之外）。
即使`any`也不可以賦值給`never`。

下面是一些傳回`never`型別的函數：

```ts
// 傳回never的函數必須存在無法達到的終點
function error(message: string): never {
    throw new Error(message);
}

// 推斷的傳回值型別為never
function fail() {
    return error("Something failed");
}

// 傳回never的函數必須存在無法達到的終點
function infiniteLoop(): never {
    while (true) {
    }
}
```

# Object

`object`表示非原始型別，也就是除`number`，`string`，`boolean`，`symbol`，`null`或`undefined`之外的型別。

使用`object`型別，就可以更好的表示像`Object.create`這樣的API。例如：

```ts
declare function create(o: object | null): void;

create({ prop: 0 }); // OK
create(null); // OK

create(42); // Error
create("string"); // Error
create(false); // Error
create(undefined); // Error
```

# 型別斷言

有時候你會遇到這樣的情況，你會比TypeScript更瞭解某個值的詳細信息。
通常這會發生在你清楚地知道一個實體具有比它現有型別更確切的型別。

通過*型別斷言*這種方式可以告訴編譯器，“相信我，我知道自己在幹什麼”。
型別斷言好比其它語言裡的型別轉換，但是不進行特殊的資料檢查和解構。
它沒有運行時的影響，只是在編譯階段起作用。
TypeScript會假設你，程式員，已經進行了必須的檢查。

型別斷言有兩種形式。
其一是“尖括號”語法：

```ts
let someValue: any = "this is a string";

let strLength: number = (<string>someValue).length;
```

另一個為`as`語法：

```ts
let someValue: any = "this is a string";

let strLength: number = (someValue as string).length;
```

兩種形式是等價的。
至於使用哪個大多數情況下是憑個人喜好；然而，當你在TypeScript裡使用JSX時，只有`as`語法斷言是被允許的。

# 關於`let`

你可能已經注意到了，我們使用`let`關鍵字來代替大家所熟悉的JavaScript關鍵字`var`。
`let`關鍵字是JavaScript的一個新概念，TypeScript實現了它。
我們會在以後詳細介紹它，很多常見的問題都可以通過使用`let`來解決，所以儘可能地使用`let`來代替`var`吧。
