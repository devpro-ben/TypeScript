# 介紹

TypeScript中有些獨特的概念可以在類型層面上描述JavaScript對象的模型。
這其中尤其獨特的一個例子是“聲明合併”的概念。
理解了這個概念，將有助於操作現有的JavaScript代碼。
同時，也會有助於理解更多高級抽象的概念。

對本文件來講，“聲明合併”是指編譯器將針對同一個名字的兩個獨立聲明合併為單一聲明。
合併後的聲明同時擁有原先兩個聲明的特性。
任何數量的聲明都可被合併；不侷限於兩個聲明。

# 基礎概念

TypeScript中的聲明會創建以下三種實體之一：命名空間，類型或值。
創建命名空間的聲明會新建一個命名空間，它包含了用（.）符號來訪問時使用的名字。
創建類型的聲明是：用聲明的模型創建一個類型並綁定到給定的名字上。
最後，創建值的聲明會創建在JavaScript輸出中看到的值。

| Declaration Type | Namespace | Type | Value |
|------------------|:---------:|:----:|:-----:|
| Namespace        |     X     |      |   X   |
| Class            |           |   X  |   X   |
| Enum             |           |   X  |   X   |
| Interface        |           |   X  |       |
| Type Alias       |           |   X  |       |
| Function         |           |      |   X   |
| Variable         |           |      |   X   |

理解每個聲明創建了什麼，有助於理解當聲明合併時有哪些東西被合併了。

# 合併接口

最簡單也最常見的聲明合併類型是接口合併。
從根本上說，合併的機制是把雙方的成員放到一個同名的接口裡。

```ts
interface Box {
    height: number;
    width: number;
}

interface Box {
    scale: number;
}

let box: Box = {height: 5, width: 6, scale: 10};
```

接口的非函數的成員應該是唯一的。
如果它們不是唯一的，那麼它們必須是相同的類型。
如果兩個接口中同時聲明了同名的非函數成員且它們的類型不同，則編譯器會報錯。

對於函數成員，每個同名函數聲明都會被當成這個函數的一個重載。
同時需要注意，當接口`A`與後來的接口`A`合併時，後面的接口具有更高的優先級。

如下例所示：

```ts
interface Cloner {
    clone(animal: Animal): Animal;
}

interface Cloner {
    clone(animal: Sheep): Sheep;
}

interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
}
```

這三個接口合併成一個聲明：

```ts
interface Cloner {
    clone(animal: Dog): Dog;
    clone(animal: Cat): Cat;
    clone(animal: Sheep): Sheep;
    clone(animal: Animal): Animal;
}
```

注意每組接口裡的聲明順序保持不變，但各組接口之間的順序是後來的接口重載出現在靠前位置。

這個規則有一個例外是當出現特殊的函數簽名時。
如果簽名裡有一個參數的類型是*單一*的字符串字面量（比如，不是字符串字面量的聯合類型），那麼它將會被提升到重載列表的最頂端。

比如，下面的接口會合併到一起：

```ts
interface Document {
    createElement(tagName: any): Element;
}
interface Document {
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
}
interface Document {
    createElement(tagName: string): HTMLElement;
    createElement(tagName: "canvas"): HTMLCanvasElement;
}
```

合併後的`Document`將會像下面這樣：

```ts
interface Document {
    createElement(tagName: "canvas"): HTMLCanvasElement;
    createElement(tagName: "div"): HTMLDivElement;
    createElement(tagName: "span"): HTMLSpanElement;
    createElement(tagName: string): HTMLElement;
    createElement(tagName: any): Element;
}
```

# 合併命名空間

與接口相似，同名的命名空間也會合併其成員。
命名空間會創建出命名空間和值，我們需要知道這兩者都是怎麼合併的。

對於命名空間的合併，模塊導出的同名接口進行合併，構成單一命名空間內含合併後的接口。

對於命名空間裡值的合併，如果當前已經存在給定名字的命名空間，那麼後來的命名空間的導出成員會被加到已經存在的那個模塊裡。

`Animals`聲明合併示例：

```ts
namespace Animals {
    export class Zebra { }
}

namespace Animals {
    export interface Legged { numberOfLegs: number; }
    export class Dog { }
}
```

等同於：

```ts
namespace Animals {
    export interface Legged { numberOfLegs: number; }

    export class Zebra { }
    export class Dog { }
}
```

除了這些合併外，你還需要瞭解非導出成員是如何處理的。
非導出成員僅在其原有的（合併前的）命名空間內可見。這就是說合併之後，從其它命名空間合併進來的成員無法訪問非導出成員。

下例提供了更清晰的說明：

```ts
namespace Animal {
    let haveMuscles = true;

    export function animalsHaveMuscles() {
        return haveMuscles;
    }
}

namespace Animal {
    export function doAnimalsHaveMuscles() {
        return haveMuscles;  // Error, because haveMuscles is not accessible here
    }
}
```

因為`haveMuscles`並沒有導出，只有`animalsHaveMuscles`函數共享了原始未合併的命名空間可以訪問這個變量。
`doAnimalsHaveMuscles`函數雖是合併命名空間的一部分，但是訪問不了未導出的成員。

# 命名空間與類和函數和枚舉類型合併

命名空間可以與其它類型的聲明進行合併。
只要命名空間的定義符合將要合併類型的定義。合併結果包含兩者的聲明類型。
TypeScript使用這個功能去實現一些JavaScript裡的設計模式。

## 合併命名空間和類

這讓我們可以表示內部類。

```ts
class Album {
    label: Album.AlbumLabel;
}
namespace Album {
    export class AlbumLabel { }
}
```

合併規則與上面`合併命名空間`小節裡講的規則一致，我們必須導出`AlbumLabel`類，好讓合併的類能訪問。
合併結果是一個類並帶有一個內部類。
你也可以使用命名空間為類增加一些靜態屬性。

除了內部類的模式，你在JavaScript裡，創建一個函數稍後擴展它增加一些屬性也是很常見的。
TypeScript使用聲明合併來達到這個目的並保證類型安全。

```ts
function buildLabel(name: string): string {
    return buildLabel.prefix + name + buildLabel.suffix;
}

namespace buildLabel {
    export let suffix = "";
    export let prefix = "Hello, ";
}

console.log(buildLabel("Sam Smith"));
```

相似的，命名空間可以用來擴展枚舉型：

```ts
enum Color {
    red = 1,
    green = 2,
    blue = 4
}

namespace Color {
    export function mixColor(colorName: string) {
        if (colorName == "yellow") {
            return Color.red + Color.green;
        }
        else if (colorName == "white") {
            return Color.red + Color.green + Color.blue;
        }
        else if (colorName == "magenta") {
            return Color.red + Color.blue;
        }
        else if (colorName == "cyan") {
            return Color.green + Color.blue;
        }
    }
}
```

# 非法的合併

TypeScript並非允許所有的合併。
目前，類不能與其它類或變量合併。
想要瞭解如何模仿類的合併，請參考[TypeScript的混入](./Mixins.md)。

# 模塊擴展

雖然JavaScript不支持合併，但你可以為導入的對象打補丁以更新它們。讓我們考察一下這個玩具性的示例：

```js
// observable.js
export class Observable<T> {
    // ... implementation left as an exercise for the reader ...
}

// map.js
import { Observable } from "./observable";
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}
```

它也可以很好地工作在TypeScript中， 但編譯器對 `Observable.prototype.map`一無所知。
你可以使用擴展模塊來將它告訴編譯器：

```ts
// observable.ts stays the same
// map.ts
import { Observable } from "./observable";
declare module "./observable" {
    interface Observable<T> {
        map<U>(f: (x: T) => U): Observable<U>;
    }
}
Observable.prototype.map = function (f) {
    // ... another exercise for the reader
}


// consumer.ts
import { Observable } from "./observable";
import "./map";
let o: Observable<number>;
o.map(x => x.toFixed());
```

模塊名的解析和用`import`/`export`解析模塊標識符的方式是一致的。
更多信息請參考 [Modules](./Modules.md)。
當這些聲明在擴展中合併時，就好像在原始位置被聲明了一樣。
但是，你不能在擴展中聲明新的頂級聲明－僅可以擴展模塊中已經存在的聲明。

## 全局擴展

你也以在模塊內部添加聲明到全局作用域中。

```ts
// observable.ts
export class Observable<T> {
    // ... still no implementation ...
}

declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
}
```

全局擴展與模塊擴展的行為和限制是相同的。
