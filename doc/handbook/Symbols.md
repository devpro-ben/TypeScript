# 介紹

自ECMAScript 2015起，`symbol`成為了一種新的原生類型，就像`number`和`string`一樣。

`symbol`類型的值是通過`Symbol`構造函數創建的。

```ts
let sym1 = Symbol();

let sym2 = Symbol("key"); // 可選的字符串key
```

Symbols是不可改變且唯一的。

```ts
let sym2 = Symbol("key");
let sym3 = Symbol("key");

sym2 === sym3; // false, symbols是唯一的
```

像字符串一樣，symbols也可以被用做對象屬性的鍵。

```ts
let sym = Symbol();

let obj = {
    [sym]: "value"
};

console.log(obj[sym]); // "value"
```

Symbols也可以與計算出的屬性名聲明相結合來聲明對象的屬性和類成員。

```ts
const getClassNameSymbol = Symbol();

class C {
    [getClassNameSymbol](){
       return "C";
    }
}

let c = new C();
let className = c[getClassNameSymbol](); // "C"
```

# 眾所周知的Symbols

除了用戶定義的symbols，還有一些已經眾所周知的內置symbols。
內置symbols用來表示語言內部的行為。

以下為這些symbols的列表：

## `Symbol.hasInstance`

方法，會被`instanceof`運算符調用。構造器對象用來識別一個對象是否是其實例。

## `Symbol.isConcatSpreadable`

布爾值，表示當在一個對象上調用`Array.prototype.concat`時，這個對象的數組元素是否可展開。

## `Symbol.iterator`

方法，被`for-of`語句調用。返回對象的默認迭代器。

## `Symbol.match`

方法，被`String.prototype.match`調用。正則表達式用來匹配字符串。

## `Symbol.replace`

方法，被`String.prototype.replace`調用。正則表達式用來替換字符串中匹配的子串。

## `Symbol.search`

方法，被`String.prototype.search`調用。正則表達式返回被匹配部分在字符串中的索引。

## `Symbol.species`

函數值，為一個構造函數。用來創建派生對象。

## `Symbol.split`

方法，被`String.prototype.split`調用。正則表達式來用分割字符串。

## `Symbol.toPrimitive`

方法，被`ToPrimitive`抽象操作調用。把對象轉換為相應的原始值。

## `Symbol.toStringTag`

方法，被內置方法`Object.prototype.toString`調用。返回創建對象時默認的字符串描述。

## `Symbol.unscopables`

對象，它自己擁有的屬性會被`with`作用域排除在外。
