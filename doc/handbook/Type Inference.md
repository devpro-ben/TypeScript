# 介紹

這節介紹TypeScript裡的型別推論。即，型別是在哪裡如何被推斷的。

# 基礎

TypeScript裡，在有些沒有明確指出型別的地方，型別推論會幫助提供型別。如下面的範例

```ts
let x = 3;
```

變數`x`的型別被推斷為數字。
這種推斷發生在初始化變數和成員，設置預設參數值和決定函數傳回值時。

大多數情況下，型別推論是直截了當地。
後面的小節，我們會瀏覽型別推論時的細微差別。

# 最佳通用型別

當需要從幾個運算式中推斷型別時候，會使用這些運算式的型別來推斷出一個最合適的通用型別(common type)。例如，

```ts
let x = [0, 1, null];
```

為了推斷`x`的型別，我們必須考慮所有元素的型別。
這裡有兩種選擇：`number`和`null`。
計算通用型別算法會考慮所有的候選型別，並給出一個相容所有候選型別的型別。

由於最終的通用型別取自候選型別，有些時候候選型別共享相同的通用型別，但是卻沒有一個型別能做為所有候選型別的型別。例如：

```ts
let zoo = [new Rhino(), new Elephant(), new Snake()];
```

這裡，我們想讓zoo被推斷為`Animal[]`型別，但是這個陣列裡沒有物件是`Animal`型別的，因此不能推斷出這個結果。
為了更正，當候選型別不能使用的時候我們需要明確的指出型別：

```ts
let zoo: Animal[] = [new Rhino(), new Elephant(), new Snake()];
```

如果沒有找到最佳通用型別的話，型別推斷的結果為聯合陣列型別，`(Rhino | Elephant | Snake)[]`。

# 上下文型別

TypeScript型別推論也可能按照相反的方向進行。
這被叫做「按上下文歸類(contextual typing)」。按上下文歸類會發生在運算式的型別與所處的位置相關時。比如：

```ts
window.onmousedown = function(mouseEvent) {
    console.log(mouseEvent.clickTime);  //<- Error
};
```

這個範例會得到一個型別錯誤，TypeScript型別檢查器使用`Window.onmousedown`函數的型別來推斷右邊函數運算式的型別。
因此，就能推斷出`mouseEvent`參數的型別了。
如果函數運算式不是在上下文型別的位置，`mouseEvent`參數的型別需要指定為`any`，這樣也不會報錯了。

如果上下文型別運算式包含了明確的型別訊息，上下文的型別被忽略。
重寫上面的範例：

```ts
window.onmousedown = function(mouseEvent: any) {
    console.log(mouseEvent.clickTime);  //<- Now, no error is given
};
```

這個函數運算式有明確的參數型別註解，上下文型別被忽略。
這樣的話就不報錯了，因為這裡不會使用到上下文型別。

上下文歸類會在很多情況下使用到。
通常包含函數的參數，賦值運算式的右邊，型別斷言，物件成員和陣列字面量和傳回值敘述。
上下文型別也會做為最佳通用型別的候選型別。比如：

```ts
function createZoo(): Animal[] {
    return [new Rhino(), new Elephant(), new Snake()];
}
```

這個範例裡，最佳通用型別有4個候選者：`Animal`，`Rhino`，`Elephant`和`Snake`。
當然，`Animal`會被做為最佳通用型別。
