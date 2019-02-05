# 介紹

函數是JavaScript應用程序的基礎。
它幫助你實現抽象層，模擬類，信息隱藏和模塊。
在TypeScript裡，雖然已經支持類，命名空間和模塊，但函數仍然是主要的定義*行為*的地方。
TypeScript為JavaScript函數添加了額外的功能，讓我們可以更容易地使用。

# 函數

和JavaScript一樣，TypeScript函數可以創建有名字的函數和匿名函數。
你可以隨意選擇適合應用程序的方式，不論是定義一系列API函數還是只使用一次的函數。

通過下面的例子可以迅速回想起這兩種JavaScript中的函數：

```ts
// Named function
function add(x, y) {
    return x + y;
}

// Anonymous function
let myAdd = function(x, y) { return x + y; };
```

在JavaScript裡，函數可以使用函數體外部的變量。
當函數這麼做時，我們說它‘捕獲’了這些變量。
至於為什麼可以這樣做以及其中的利弊超出了本文的範圍，但是深刻理解這個機制對學習JavaScript和TypeScript會很有幫助。

```ts
let z = 100;

function addToZ(x, y) {
    return x + y + z;
}
```

# 函數類型

## 為函數定義類型

讓我們為上面那個函數添加類型：

```ts
function add(x: number, y: number): number {
    return x + y;
}

let myAdd = function(x: number, y: number): number { return x + y; };
```

我們可以給每個參數添加類型之後再為函數本身添加返回值類型。
TypeScript能夠根據返回語句自動推斷出返回值類型，因此我們通常省略它。

## 書寫完整函數類型

現在我們已經為函數指定了類型，下面讓我們寫出函數的完整類型。

```ts
let myAdd: (x:number, y:number) => number =
    function(x: number, y: number): number { return x + y; };
```

函數類型包含兩部分：參數類型和返回值類型。
當寫出完整函數類型的時候，這兩部分都是需要的。
我們以參數列表的形式寫出參數類型，為每個參數指定一個名字和類型。
這個名字只是為了增加可讀性。
我們也可以這麼寫：

```ts
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

只要參數類型是匹配的，那麼就認為它是有效的函數類型，而不在乎參數名是否正確。

第二部分是返回值類型。
對於返回值，我們在函數和返回值類型之前使用(`=>`)符號，使之清晰明了。
如之前提到的，返回值類型是函數類型的必要部分，如果函數沒有返回任何值，你也必須指定返回值類型為`void`而不能留空。

函數的類型只是由參數類型和返回值組成的。
函數中使用的捕獲變量不會體現在類型裡。
實際上，這些變量是函數的隱藏狀態並不是組成API的一部分。

## 推斷類型

嘗試這個例子的時候，你會發現如果你在賦值語句的一邊指定了類型但是另一邊沒有類型的話，TypeScript編譯器會自動識別出類型：

```ts
// myAdd has the full function type
let myAdd = function(x: number, y: number): number { return x + y; };

// The parameters `x` and `y` have the type number
let myAdd: (baseValue: number, increment: number) => number =
    function(x, y) { return x + y; };
```

這叫做“按上下文歸類”，是類型推論的一種。
它幫助我們更好地為程序指定類型。

# 可選參數和默認參數

TypeScript裡的每個函數參數都是必須的。
這不是指不能傳遞`null`或`undefined`作為參數，而是說編譯器檢查用戶是否為每個參數都傳入了值。
編譯器還會假設只有這些參數會被傳遞進函數。
簡短地說，傳遞給一個函數的參數個數必須與函數期望的參數個數一致。

```ts
function buildName(firstName: string, lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // ah, just right
```

JavaScript裡，每個參數都是可選的，可傳可不傳。
沒傳參的時候，它的值就是undefined。
在TypeScript裡我們可以在參數名旁使用`?`實現可選參數的功能。
比如，我們想讓last name是可選的：

```ts
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // works correctly now
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");  // ah, just right
```

可選參數必須跟在必須參數後面。
如果上例我們想讓first name是可選的，那麼就必須調整它們的位置，把first name放在後面。

在TypeScript裡，我們也可以為參數提供一個默認值當用戶沒有傳遞這個參數或傳遞的值是`undefined`時。
它們叫做有默認初始化值的參數。
讓我們修改上例，把last name的默認值設置為`"Smith"`。

```ts
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // works correctly now, returns "Bob Smith"
let result2 = buildName("Bob", undefined);       // still works, also returns "Bob Smith"
let result3 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result4 = buildName("Bob", "Adams");         // ah, just right
```

在所有必須參數後面的帶默認初始化的參數都是可選的，與可選參數一樣，在調用函數的時候可以省略。
也就是說可選參數與末尾的默認參數共享參數類型。

```ts
function buildName(firstName: string, lastName?: string) {
    // ...
}
```

和

```ts
function buildName(firstName: string, lastName = "Smith") {
    // ...
}
```

共享同樣的類型`(firstName: string, lastName?: string) => string`。
默認參數的默認值消失了，只保留了它是一個可選參數的信息。

與普通可選參數不同的是，帶默認值的參數不需要放在必須參數的後面。
如果帶默認值的參數出現在必須參數前面，用戶必須明確的傳入`undefined`值來獲得默認值。
例如，我們重寫最後一個例子，讓`firstName`是帶默認值的參數：

```ts
function buildName(firstName = "Will", lastName: string) {
    return firstName + " " + lastName;
}

let result1 = buildName("Bob");                  // error, too few parameters
let result2 = buildName("Bob", "Adams", "Sr.");  // error, too many parameters
let result3 = buildName("Bob", "Adams");         // okay and returns "Bob Adams"
let result4 = buildName(undefined, "Adams");     // okay and returns "Will Adams"
```

# 剩餘參數

必要參數，默認參數和可選參數有個共同點：它們表示某一個參數。
有時，你想同時操作多個參數，或者你並不知道會有多少參數傳遞進來。
在JavaScript裡，你可以使用`arguments`來訪問所有傳入的參數。

在TypeScript裡，你可以把所有參數收集到一個變量裡：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

剩餘參數會被當做個數不限的可選參數。
可以一個都沒有，同樣也可以有任意個。
編譯器創建參數數組，名字是你在省略號（`...`）後面給定的名字，你可以在函數體內使用這個數組。

這個省略號也會在帶有剩餘參數的函數類型定義上使用到：

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let buildNameFun: (fname: string, ...rest: string[]) => string = buildName;
```

# `this`

學習如何在JavaScript裡正確使用`this`就好比一場成年禮。
由於TypeScript是JavaScript的超集，TypeScript程序員也需要弄清`this`工作機制並且當有bug的時候能夠找出錯誤所在。
幸運的是，TypeScript能通知你錯誤地使用了`this`的地方。
如果你想瞭解JavaScript裡的`this`是如何工作的，那麼首先閱讀Yehuda Katz寫的[Understanding JavaScript Function Invocation and "this"](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)。
Yehuda的文章詳細的闡述了`this`的內部工作原理，因此我們這裡只做簡單介紹。

## `this`和箭頭函數

JavaScript裡，`this`的值在函數被調用的時候才會指定。
這是個既強大又靈活的特點，但是你需要花點時間弄清楚函數調用的上下文是什麼。
但眾所周知，這不是一件很簡單的事，尤其是在返回一個函數或將函數當做參數傳遞的時候。

下面看一個例子：

```ts
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        return function() {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

可以看到`createCardPicker`是個函數，並且它又返回了一個函數。
如果我們嘗試運行這個程序，會發現它並沒有彈出對話框而是報錯了。
因為`createCardPicker`返回的函數裡的`this`被設置成了`window`而不是`deck`對象。
因為我們只是獨立的調用了`cardPicker()`。
頂級的非方法式調用會將`this`視為`window`。
（注意：在嚴格模式下，`this`為`undefined`而不是`window`）。

為瞭解決這個問題，我們可以在函數被返回時就綁好正確的`this`。
這樣的話，無論之後怎麼使用它，都會引用綁定的‘deck’對象。
我們需要改變函數表達式來使用ECMAScript 6箭頭語法。
箭頭函數能保存函數創建時的`this`值，而不是調用時的值：

```ts
let deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    createCardPicker: function() {
        // NOTE: the line below is now an arrow function, allowing us to capture 'this' right here
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

更好事情是，TypeScript會警告你犯了一個錯誤，如果你給編譯器設置了`--noImplicitThis`標記。
它會指出`this.suits[pickedSuit]`裡的`this`的類型為`any`。

## `this`參數

不幸的是，`this.suits[pickedSuit]`的類型依舊為`any`。
這是因為`this`來自對象字面量裡的函數表達式。
修改的方法是，提供一個顯式的`this`參數。
`this`參數是個假的參數，它出現在參數列表的最前面：

```ts
function f(this: void) {
    // make sure `this` is unusable in this standalone function
}
```

讓我們往例子裡添加一些接口，`Card` 和 `Deck`，讓類型重用能夠變得清晰簡單些：

```ts
interface Card {
    suit: string;
    card: number;
}
interface Deck {
    suits: string[];
    cards: number[];
    createCardPicker(this: Deck): () => Card;
}
let deck: Deck = {
    suits: ["hearts", "spades", "clubs", "diamonds"],
    cards: Array(52),
    // NOTE: The function now explicitly specifies that its callee must be of type Deck
    createCardPicker: function(this: Deck) {
        return () => {
            let pickedCard = Math.floor(Math.random() * 52);
            let pickedSuit = Math.floor(pickedCard / 13);

            return {suit: this.suits[pickedSuit], card: pickedCard % 13};
        }
    }
}

let cardPicker = deck.createCardPicker();
let pickedCard = cardPicker();

alert("card: " + pickedCard.card + " of " + pickedCard.suit);
```

現在TypeScript知道`createCardPicker`期望在某個`Deck`對象上調用。
也就是說`this`是`Deck`類型的，而非`any`，因此`--noImplicitThis`不會報錯了。

### 回調函數裡的`this`參數

當你將一個函數傳遞到某個庫函數裡在稍後被調用時，你可能也見到過回調函數裡的`this`會報錯。
因為當回調函數被調用時，它會被當成一個普通函數調用，`this`將為`undefined`。
稍做改動，你就可以通過`this`參數來避免錯誤。
首先，庫函數的作者要指定`this`的類型：

```ts
interface UIElement {
    addClickListener(onclick: (this: void, e: Event) => void): void;
}
```

`this: void`意味著`addClickListener`期望`onclick`是一個函數且它不需要一個`this`類型。
然後，為調用代碼裡的`this`添加類型註解：

```ts
class Handler {
    info: string;
    onClickBad(this: Handler, e: Event) {
        // oops, used this here. using this callback would crash at runtime
        this.info = e.message;
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickBad); // error!
```

指定了`this`類型後，你顯式聲明`onClickBad`必須在`Handler`的實例上調用。
然後TypeScript會檢測到`addClickListener`要求函數帶有`this: void`。
改變`this`類型來修復這個錯誤：

```ts
class Handler {
    info: string;
    onClickGood(this: void, e: Event) {
        // can't use this here because it's of type void!
        console.log('clicked!');
    }
}
let h = new Handler();
uiElement.addClickListener(h.onClickGood);
```

因為`onClickGood`指定了`this`類型為`void`，因此傳遞`addClickListener`是合法的。
當然了，這也意味著不能使用`this.info`.
如果你兩者都想要，你不得不使用箭頭函數了：

```ts
class Handler {
    info: string;
    onClickGood = (e: Event) => { this.info = e.message }
}
```

這是可行的因為箭頭函數不會捕獲`this`，所以你總是可以把它們傳給期望`this: void`的函數。
缺點是每個`Handler`對象都會創建一個箭頭函數。
另一方面，方法只會被創建一次，添加到`Handler`的原型鏈上。
它們在不同`Handler`對象間是共享的。

# 重載

JavaScript本身是個動態語言。
JavaScript裡函數根據傳入不同的參數而返回不同類型的數據是很常見的。

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

`pickCard`方法根據傳入參數的不同會返回兩種不同的類型。
如果傳入的是代表紙牌的對象，函數作用是從中抓一張牌。
如果用戶想抓牌，我們告訴他抓到了什麼牌。
但是這怎麼在類型系統裡表示呢。

方法是為同一個函數提供多個函數類型定義來進行函數重載。
編譯器會根據這個列表去處理函數的調用。
下面我們來重載`pickCard`函數。

```ts
let suits = ["hearts", "spades", "clubs", "diamonds"];

function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
function pickCard(x): any {
    // Check to see if we're working with an object/array
    // if so, they gave us the deck and we'll pick the card
    if (typeof x == "object") {
        let pickedCard = Math.floor(Math.random() * x.length);
        return pickedCard;
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
        let pickedSuit = Math.floor(x / 13);
        return { suit: suits[pickedSuit], card: x % 13 };
    }
}

let myDeck = [{ suit: "diamonds", card: 2 }, { suit: "spades", card: 10 }, { suit: "hearts", card: 4 }];
let pickedCard1 = myDeck[pickCard(myDeck)];
alert("card: " + pickedCard1.card + " of " + pickedCard1.suit);

let pickedCard2 = pickCard(15);
alert("card: " + pickedCard2.card + " of " + pickedCard2.suit);
```

這樣改變後，重載的`pickCard`函數在調用的時候會進行正確的類型檢查。

為了讓編譯器能夠選擇正確的檢查類型，它與JavaScript裡的處理流程相似。
它查找重載列表，嘗試使用第一個重載定義。
如果匹配的話就使用這個。
因此，在定義重載的時候，一定要把最精確的定義放在最前面。

注意，`function pickCard(x): any`並不是重載列表的一部分，因此這裡只有兩個重載：一個是接收對象另一個接收數字。
以其它參數調用`pickCard`會產生錯誤。
