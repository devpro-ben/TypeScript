# 介紹

除了傳統的面向對象繼承方式，還流行一種通過可重用組件創建類的方式，就是聯合另一個簡單類的代碼。
你可能在Scala等語言裡對mixins及其特性已經很熟悉了，但它在JavaScript中也是很流行的。

# 混入示例

下面的代碼演示了如何在TypeScript裡使用混入。
後面我們還會解釋這段代碼是怎麼工作的。

```ts
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}

class SmartObject implements Disposable, Activatable {
    constructor() {
        setInterval(() => console.log(this.isActive + " : " + this.isDisposed), 500);
    }

    interact() {
        this.activate();
    }

    // Disposable
    isDisposed: boolean = false;
    dispose: () => void;
    // Activatable
    isActive: boolean = false;
    activate: () => void;
    deactivate: () => void;
}
applyMixins(SmartObject, [Disposable, Activatable]);

let smartObj = new SmartObject();
setTimeout(() => smartObj.interact(), 1000);

////////////////////////////////////////
// In your runtime library somewhere
////////////////////////////////////////

function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        });
    });
}
```

# 理解這個例子

代碼裡首先定義了兩個類，它們將做為mixins。
可以看到每個類都只定義了一個特定的行為或功能。
稍後我們使用它們來創建一個新類，同時具有這兩種功能。

```ts
// Disposable Mixin
class Disposable {
    isDisposed: boolean;
    dispose() {
        this.isDisposed = true;
    }

}

// Activatable Mixin
class Activatable {
    isActive: boolean;
    activate() {
        this.isActive = true;
    }
    deactivate() {
        this.isActive = false;
    }
}
```

下面創建一個類，結合了這兩個mixins。
下面來看一下具體是怎麼操作的：

```ts
class SmartObject implements Disposable, Activatable {
```

首先應該注意到的是，沒使用`extends`而是使用`implements`。
把類當成了接口，僅使用Disposable和Activatable的類型而非其實現。
這意味著我們需要在類裡面實現接口。
但是這是我們在用mixin時想避免的。

我們可以這麼做來達到目的，為將要mixin進來的屬性方法創建出佔位屬性。
這告訴編譯器這些成員在運行時是可用的。
這樣就能使用mixin帶來的便利，雖說需要提前定義一些佔位屬性。

```ts
// Disposable
isDisposed: boolean = false;
dispose: () => void;
// Activatable
isActive: boolean = false;
activate: () => void;
deactivate: () => void;
```

最後，把mixins混入定義的類，完成全部實現部分。

```ts
applyMixins(SmartObject, [Disposable, Activatable]);
```

最後，創建這個幫助函數，幫我們做混入操作。
它會遍歷mixins上的所有屬性，並複製到目標上去，把之前的佔位屬性替換成真正的實現代碼。

```ts
function applyMixins(derivedCtor: any, baseCtors: any[]) {
    baseCtors.forEach(baseCtor => {
        Object.getOwnPropertyNames(baseCtor.prototype).forEach(name => {
            derivedCtor.prototype[name] = baseCtor.prototype[name];
        })
    });
}

```
