# 可迭代性

當一個對象實現了[`Symbol.iterator`](Symbols.md#symboliterator)屬性時，我們認為它是可迭代的。
一些內置的類型如`Array`，`Map`，`Set`，`String`，`Int32Array`，`Uint32Array`等都已經實現了各自的`Symbol.iterator`。
對象上的`Symbol.iterator`函數負責返回供迭代的值。

## `for..of` 語句

`for..of`會遍歷可迭代的對象，調用對象上的`Symbol.iterator`方法。
下面是在數組上使用`for..of`的簡單例子：

```ts
let someArray = [1, "string", false];

for (let entry of someArray) {
    console.log(entry); // 1, "string", false
}
```

### `for..of` vs. `for..in` 語句

`for..of`和`for..in`均可迭代一個列表；但是用於迭代的值卻不同，`for..in`迭代的是對象的 *鍵* 的列表，而`for..of`則迭代對象的鍵對應的值。

下面的例子展示了兩者之間的區別：

```ts
let list = [4, 5, 6];

for (let i in list) {
    console.log(i); // "0", "1", "2",
}

for (let i of list) {
    console.log(i); // "4", "5", "6"
}
```

另一個區別是`for..in`可以操作任何對象；它提供了查看對象屬性的一種方法。
但是`for..of`關注於迭代對象的值。內置對象`Map`和`Set`已經實現了`Symbol.iterator`方法，讓我們可以訪問它們保存的值。

```ts
let pets = new Set(["Cat", "Dog", "Hamster"]);
pets["species"] = "mammals";

for (let pet in pets) {
    console.log(pet); // "species"
}

for (let pet of pets) {
    console.log(pet); // "Cat", "Dog", "Hamster"
}
```

### 代碼生成

#### 目標為 ES5 和 ES3

當生成目標為ES5或ES3，迭代器只允許在`Array`類型上使用。
在非數組值上使用`for..of`語句會得到一個錯誤，就算這些非數組值已經實現了`Symbol.iterator`屬性。

編譯器會生成一個簡單的`for`循環做為`for..of`循環，比如：

```ts
let numbers = [1, 2, 3];
for (let num of numbers) {
    console.log(num);
}
```

生成的代碼為：

```js
var numbers = [1, 2, 3];
for (var _i = 0; _i < numbers.length; _i++) {
    var num = numbers[_i];
    console.log(num);
}
```

#### 目標為 ECMAScript 2015 或更高

當目標為兼容ECMAScipt 2015的引擎時，編譯器會生成相應引擎的`for..of`內置迭代器實現方式。
