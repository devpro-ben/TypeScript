# 介紹

隨著TypeScript和ES6里引入了類，在一些場景下我們需要額外的特性來支持標註或修改類及其成員。
裝飾器（Decorators）為我們在類的聲明及成員上通過元編程語法添加標註提供了一種方式。
Javascript裡的裝飾器目前處在[建議徵集的第二階段](https://github.com/tc39/proposal-decorators)，但在TypeScript裡已做為一項實驗性特性予以支持。

> 注意&emsp; 裝飾器是一項實驗性特性，在未來的版本中可能會發生改變。

若要啟用實驗性的裝飾器特性，你必須在命令行或`tsconfig.json`裡啟用`experimentalDecorators`編譯器選項：

**命令行**:

```shell
tsc --target ES5 --experimentalDecorators
```

**tsconfig.json**:

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true
    }
}
```

# 裝飾器

*裝飾器*是一種特殊類型的聲明，它能夠被附加到[類聲明](#class-decorators)，[方法](#method-decorators)，[訪問符](#accessor-decorators)，[屬性](#property-decorators)或[參數](#parameter-decorators)上。
裝飾器使用`@expression`這種形式，`expression`求值後必須為一個函數，它會在運行時被調用，被裝飾的聲明信息做為參數傳入。

例如，有一個`@sealed`裝飾器，我們會這樣定義`sealed`函數：

```ts
function sealed(target) {
    // do something with "target" ...
}
```

> 注意&emsp; 後面[類裝飾器](#class-decorators)小節裡有一個更加詳細的例子。

## <a name="decorator-factories"></a>裝飾器工廠

如果我們要定製一個修飾器如何應用到一個聲明上，我們得寫一個裝飾器工廠函數。
*裝飾器工廠*就是一個簡單的函數，它返回一個表達式，以供裝飾器在運行時調用。

我們可以通過下面的方式來寫一個裝飾器工廠函數：

```ts
function color(value: string) { // 這是一個裝飾器工廠
    return function (target) { //  這是裝飾器
        // do something with "target" and "value"...
    }
}
```

> 注意&emsp; 下面[方法裝飾器](#method-decorators)小節裡有一個更加詳細的例子。

## 裝飾器組合

多個裝飾器可以同時應用到一個聲明上，就像下面的示例：

* 書寫在同一行上：

```ts
@f @g x
```

* 書寫在多行上：

```ts
@f
@g
x
```

當多個裝飾器應用於一個聲明上，它們求值方式與[復合函數](http://en.wikipedia.org/wiki/Function_composition)相似。在這個模型下，當復合*f*和*g*時，復合的結果(*f* ∘ *g*)(*x*)等同於*f*(*g*(*x*))。

同樣的，在TypeScript裡，當多個裝飾器應用在一個聲明上時會進行如下步驟的操作：

1. 由上至下依次對裝飾器表達式求值。
2. 求值的結果會被當作函數，由下至上依次調用。

如果我們使用[裝飾器工廠](#decorator-factories)的話，可以通過下面的例子來觀察它們求值的順序：

```ts
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}
```

在控制台裡會打印出如下結果：

```shell
f(): evaluated
g(): evaluated
g(): called
f(): called
```

## 裝飾器求值

類中不同聲明上的裝飾器將按以下規定的順序應用：

1. *參數裝飾器*，然後依次是*方法裝飾器*，*訪問符裝飾器*，或*屬性裝飾器*應用到每個實例成員。
2. *參數裝飾器*，然後依次是*方法裝飾器*，*訪問符裝飾器*，或*屬性裝飾器*應用到每個靜態成員。
3. *參數裝飾器*應用到構造函數。
4. *類裝飾器*應用到類。

## <a name="class-decorators"></a>類裝飾器

*類裝飾器*在類聲明之前被聲明（緊靠著類聲明）。
類裝飾器應用於類構造函數，可以用來監視，修改或替換類定義。
類裝飾器不能用在聲明文件中(`.d.ts`)，也不能用在任何外部上下文中（比如`declare`的類）。

類裝飾器表達式會在運行時當作函數被調用，類的構造函數作為其唯一的參數。

如果類裝飾器返回一個值，它會使用提供的構造函數來替換類的聲明。

> 注意&nbsp; 如果你要返回一個新的構造函數，你必須注意處理好原來的原型鏈。
在運行時的裝飾器調用邏輯中*不會*為你做這些。

下面是使用類裝飾器(`@sealed`)的例子，應用在`Greeter`類：

```ts
@sealed
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

我們可以這樣定義`@sealed`裝飾器：

```ts
function sealed(constructor: Function) {
    Object.seal(constructor);
    Object.seal(constructor.prototype);
}
```

當`@sealed`被執行的時候，它將密封此類的構造函數和原型。(註：參見[Object.seal](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/seal))

下面是一個重載構造函數的例子。

```ts
function classDecorator<T extends {new(...args:any[]):{}}>(constructor:T) {
    return class extends constructor {
        newProperty = "new property";
        hello = "override";
    }
}

@classDecorator
class Greeter {
    property = "property";
    hello: string;
    constructor(m: string) {
        this.hello = m;
    }
}

console.log(new Greeter("world"));
```

## <a name="method-decorators"></a>方法裝飾器

*方法裝飾器*聲明在一個方法的聲明之前（緊靠著方法聲明）。
它會被應用到方法的*屬性描述符*上，可以用來監視，修改或者替換方法定義。
方法裝飾器不能用在聲明文件(`.d.ts`)，重載或者任何外部上下文（比如`declare`的類）中。

方法裝飾器表達式會在運行時當作函數被調用，傳入下列3個參數：

1. 對於靜態成員來說是類的構造函數，對於實例成員是類的原型對象。
2. 成員的名字。
3. 成員的*屬性描述符*。

> 注意&emsp; 如果代碼輸出目標版本小於`ES5`，*屬性描述符*將會是`undefined`。

如果方法裝飾器返回一個值，它會被用作方法的*屬性描述符*。

> 注意&emsp; 如果代碼輸出目標版本小於`ES5`返回值會被忽略。

下面是一個方法裝飾器（`@enumerable`）的例子，應用於`Greeter`類的方法上：

```ts
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }

    @enumerable(false)
    greet() {
        return "Hello, " + this.greeting;
    }
}
```

我們可以用下面的函數聲明來定義`@enumerable`裝飾器：

```ts
function enumerable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.enumerable = value;
    };
}
```

這裡的`@enumerable(false)`是一個[裝飾器工廠](#decorator-factories)。
當裝飾器`@enumerable(false)`被調用時，它會修改屬性描述符的`enumerable`屬性。

## <a name="accessor-decorators"></a>訪問器裝飾器

*訪問器裝飾器*聲明在一個訪問器的聲明之前（緊靠著訪問器聲明）。
訪問器裝飾器應用於訪問器的*屬性描述符*並且可以用來監視，修改或替換一個訪問器的定義。
訪問器裝飾器不能用在聲明文件中（.d.ts），或者任何外部上下文（比如`declare`的類）裡。

> 注意&emsp; TypeScript不允許同時裝飾一個成員的`get`和`set`訪問器。取而代之的是，一個成員的所有裝飾的必須應用在文檔順序的第一個訪問器上。這是因為，在裝飾器應用於一個*屬性描述符*時，它聯合了`get`和`set`訪問器，而不是分開聲明的。

訪問器裝飾器表達式會在運行時當作函數被調用，傳入下列3個參數：

1. 對於靜態成員來說是類的構造函數，對於實例成員是類的原型對象。
2. 成員的名字。
3. 成員的*屬性描述符*。

> 注意&emsp; 如果代碼輸出目標版本小於`ES5`，*Property Descriptor*將會是`undefined`。

如果訪問器裝飾器返回一個值，它會被用作方法的*屬性描述符*。

> 注意&emsp; 如果代碼輸出目標版本小於`ES5`返回值會被忽略。

下面是使用了訪問器裝飾器（`@configurable`）的例子，應用於`Point`類的成員上：

```ts
class Point {
    private _x: number;
    private _y: number;
    constructor(x: number, y: number) {
        this._x = x;
        this._y = y;
    }

    @configurable(false)
    get x() { return this._x; }

    @configurable(false)
    get y() { return this._y; }
}
```

我們可以通過如下函數聲明來定義`@configurable`裝飾器：

```ts
function configurable(value: boolean) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
        descriptor.configurable = value;
    };
}
```

## <a name="property-decorators"></a>屬性裝飾器

*屬性裝飾器*聲明在一個屬性聲明之前（緊靠著屬性聲明）。
屬性裝飾器不能用在聲明文件中（.d.ts），或者任何外部上下文（比如`declare`的類）裡。

屬性裝飾器表達式會在運行時當作函數被調用，傳入下列2個參數：

1. 對於靜態成員來說是類的構造函數，對於實例成員是類的原型對象。
2. 成員的名字。

> 注意&emsp; *屬性描述符*不會做為參數傳入屬性裝飾器，這與TypeScript是如何初始化屬性裝飾器的有關。
因為目前沒有辦法在定義一個原型對象的成員時描述一個實例屬性，並且沒辦法監視或修改一個屬性的初始化方法。返回值也會被忽略。
因此，屬性描述符只能用來監視類中是否聲明了某個名字的屬性。

如果訪問符裝飾器返回一個值，它會被用作方法的*屬性描述符*。

我們可以用它來記錄這個屬性的元數據，如下例所示：

```ts
class Greeter {
    @format("Hello, %s")
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        let formatString = getFormat(this, "greeting");
        return formatString.replace("%s", this.greeting);
    }
}
```

然後定義`@format`裝飾器和`getFormat`函數：

```ts
import "reflect-metadata";

const formatMetadataKey = Symbol("format");

function format(formatString: string) {
    return Reflect.metadata(formatMetadataKey, formatString);
}

function getFormat(target: any, propertyKey: string) {
    return Reflect.getMetadata(formatMetadataKey, target, propertyKey);
}
```

這個`@format("Hello, %s")`裝飾器是個 [裝飾器工廠](#decorator-factories)。
當`@format("Hello, %s")`被調用時，它添加一條這個屬性的元數據，通過`reflect-metadata`庫裡的`Reflect.metadata`函數。
當`getFormat`被調用時，它讀取格式的元數據。

> 注意&emsp; 這個例子需要使用`reflect-metadata`庫。
查看[元數據](#metadata)瞭解`reflect-metadata`庫更詳細的信息。

## <a name="parameter-decorators"></a>參數裝飾器

*參數裝飾器*聲明在一個參數聲明之前（緊靠著參數聲明）。
參數裝飾器應用於類構造函數或方法聲明。
參數裝飾器不能用在聲明文件（.d.ts），重載或其它外部上下文（比如`declare`的類）裡。

參數裝飾器表達式會在運行時當作函數被調用，傳入下列3個參數：

1. 對於靜態成員來說是類的構造函數，對於實例成員是類的原型對象。
2. 成員的名字。
3. 參數在函數參數列表中的索引。

> 注意&emsp; 參數裝飾器只能用來監視一個方法的參數是否被傳入。

參數裝飾器的返回值會被忽略。

下例定義了參數裝飾器（`@required`）並應用於`Greeter`類方法的一個參數：

```ts
class Greeter {
    greeting: string;

    constructor(message: string) {
        this.greeting = message;
    }

    @validate
    greet(@required name: string) {
        return "Hello " + name + ", " + this.greeting;
    }
}
```

然後我們使用下面的函數定義 `@required` 和 `@validate` 裝飾器：

```ts
import "reflect-metadata";

const requiredMetadataKey = Symbol("required");

function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
    let existingRequiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyKey) || [];
    existingRequiredParameters.push(parameterIndex);
    Reflect.defineMetadata(requiredMetadataKey, existingRequiredParameters, target, propertyKey);
}

function validate(target: any, propertyName: string, descriptor: TypedPropertyDescriptor<Function>) {
    let method = descriptor.value;
    descriptor.value = function () {
        let requiredParameters: number[] = Reflect.getOwnMetadata(requiredMetadataKey, target, propertyName);
        if (requiredParameters) {
            for (let parameterIndex of requiredParameters) {
                if (parameterIndex >= arguments.length || arguments[parameterIndex] === undefined) {
                    throw new Error("Missing required argument.");
                }
            }
        }

        return method.apply(this, arguments);
    }
}
```

`@required`裝飾器添加了元數據實體把參數標記為必需的。
`@validate`裝飾器把`greet`方法包裹在一個函數裡在調用原先的函數前驗證函數參數。

> 注意&emsp; 這個例子使用了`reflect-metadata`庫。
查看[元數據](#metadata)瞭解`reflect-metadata`庫的更多信息。

## 元數據

一些例子使用了`reflect-metadata`庫來支持[實驗性的metadata API](https://github.com/rbuckton/ReflectDecorators)。
這個庫還不是ECMAScript (JavaScript)標準的一部分。
然而，當裝飾器被ECMAScript官方標準採納後，這些擴展也將被推薦給ECMAScript以採納。

你可以通過npm安裝這個庫：

```shell
npm i reflect-metadata --save
```

TypeScript支持為帶有裝飾器的聲明生成元數據。
你需要在命令行或`tsconfig.json`裡啟用`emitDecoratorMetadata`編譯器選項。

**Command Line**:

```shell
tsc --target ES5 --experimentalDecorators --emitDecoratorMetadata
```

**tsconfig.json**:

```json
{
    "compilerOptions": {
        "target": "ES5",
        "experimentalDecorators": true,
        "emitDecoratorMetadata": true
    }
}
```

當啟用後，只要`reflect-metadata`庫被引入了，設計階段添加的類型信息可以在運行時使用。

如下例所示：

```ts
import "reflect-metadata";

class Point {
    x: number;
    y: number;
}

class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
    let set = descriptor.set;
    descriptor.set = function (value: T) {
        let type = Reflect.getMetadata("design:type", target, propertyKey);
        if (!(value instanceof type)) {
            throw new TypeError("Invalid type.");
        }
        set(value);
    }
}
```

TypeScript編譯器可以通過`@Reflect.metadata`裝飾器注入設計階段的類型信息。
你可以認為它相當於下面的TypeScript：

```ts
class Line {
    private _p0: Point;
    private _p1: Point;

    @validate
    @Reflect.metadata("design:type", Point)
    set p0(value: Point) { this._p0 = value; }
    get p0() { return this._p0; }

    @validate
    @Reflect.metadata("design:type", Point)
    set p1(value: Point) { this._p1 = value; }
    get p1() { return this._p1; }
}

```

> 注意&emsp; 裝飾器元數據是個實驗性的特性並且可能在以後的版本中發生破壞性的改變（breaking changes）。
