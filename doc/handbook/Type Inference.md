# 介紹

這節介紹TypeScript裡的類型推論。即，類型是在哪裡如何被推斷的。

# 基礎

TypeScript裡，在有些沒有明確指出類型的地方，類型推論會幫助提供類型。如下面的例子

```ts
let x = 3;
```

變量`x`的類型被推斷為數字。
這種推斷髮生在初始化變量和成員，設置默認參數值和決定函數返回值時。

大多數情況下，類型推論是直截了當地。
後面的小節，我們會瀏覽類型推論時的細微差別。

# 最佳通用類型

當需要從幾個表達式中推斷類型時候，會使用這些表達式的類型來推斷出一個最合適的通用類型。例如，

```ts
let x = [0, 1, null];
```

為了推斷`x`的類型，我們必須考慮所有元素的類型。
這裡有兩種選擇：`number`和`null`。
計算通用類型算法會考慮所有的候選類型，並給出一個兼容所有候選類型的類型。

由於最終的通用類型取自候選類型，有些時候候選類型共享相同的通用類型，但是卻沒有一個類型能做為所有候選類型的類型。例如：

```ts
let zoo = [new Rhino(), new Elephant(), new Snake()];
```

這裡，我們想讓zoo被推斷為`Animal[]`類型，但是這個數組裡沒有對象是`Animal`類型的，因此不能推斷出這個結果。
為了更正，當候選類型不能使用的時候我們需要明確的指出類型：

```ts
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

如果沒有找到最佳通用類型的話，類型推斷的結果為聯合數組類型，`(Rhino | Elephant | Snake)[]`。

# 上下文類型

TypeScript類型推論也可能按照相反的方向進行。
這被叫做“按上下文歸類”。按上下文歸類會發生在表達式的類型與所處的位置相關時。比如：

```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.clickTime);  //<- Error
};
```

這個例子會得到一個類型錯誤，TypeScript類型檢查器使用`Window.onmousedown`函數的類型來推斷右邊函數表達式的類型。
因此，就能推斷出`mouseEvent`參數的類型了。
如果函數表達式不是在上下文類型的位置，`mouseEvent`參數的類型需要指定為`any`，這樣也不會報錯了。

如果上下文類型表達式包含了明確的類型信息，上下文的類型被忽略。
重寫上面的例子：

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.clickTime);  //<- Now, no error is given
};
```

這個函數表達式有明確的參數類型註解，上下文類型被忽略。
這樣的話就不報錯了，因為這裡不會使用到上下文類型。

上下文歸類會在很多情況下使用到。
通常包含函數的參數，賦值表達式的右邊，類型斷言，對象成員和數組字面量和返回值語句。
上下文類型也會做為最佳通用類型的候選類型。比如：

```ts
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

這個例子裡，最佳通用類型有4個候選者：`Animal`，`Rhino`，`Elephant`和`Snake`。
當然，`Animal`會被做為最佳通用類型。
