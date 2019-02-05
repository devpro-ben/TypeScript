# 介紹

TypeScript提供一些工具類型來幫助常見的類型轉換。這些類型是全局可見的。

## 目錄

* [`Partial<T>`，TypeScript 2.1](#partialt)
* [`Readonly<T>`，TypeScript 2.1](#readonlyt)
* [`Record<K,T>`，TypeScript 2.1](#recordkt)
* [`Pick<T,K>`，TypeScript 2.1](#picktk)
* [`Exclude<T,U>`，TypeScript 2.8](#excludetu)
* [`Extract<T,U>`，TypeScript 2.8](#extracttu)
* [`NonNullable<T>`，TypeScript 2.8](#nonnullablet)
* [`ReturnType<T>`，TypeScript 2.8](#returntypet)
* [`InstanceType<T>`，TypeScript 2.8](#instancetypet)
* [`Required<T>`，TypeScript 2.8](#requiredt)
* [`ThisType<T>`，TypeScript 2.8](#thistypet)

## `Partial<T>`

構造類型`T`，並將它所有的屬性設置為可選的。它的返回類型表示輸入類型的所有子類型。

### 例子

```ts
interface Todo {
    title: string;
    description: string;
}

function updateTodo(todo: Todo, fieldsToUpdate: Partial<Todo>) {
    return { ...todo, ...fieldsToUpdate };
}

const todo1 = {
    title: 'organize desk',
    description: 'clear clutter',
};

const todo2 = updateTodo(todo1, {
    description: 'throw out trash',
});
```

## `Readonly<T>`

構造類型`T`，並將它所有的屬性設置為`readonly`，也就是說構造出的類型的屬性不能被再次賦值。

### 例子

```ts
interface Todo {
    title: string;
}

const todo: Readonly<Todo> = {
    title: 'Delete inactive users',
};

todo.title = 'Hello'; // Error: cannot reassign a readonly property
```

這個工具可用來表示在運行時會失敗的賦值表達式（比如，當嘗試給[凍結對象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)的屬性再次賦值時）。

### `Object.freeze`

```ts
function freeze<T>(obj: T): Readonly<T>;
```

## `Record<K,T>`

構造一個類型，其屬性名的類型為`K`，屬性值的類型為`T`。這個工具可用來將某個類型的屬性映射到另一個類型上。

### 例子

```ts
interface PageInfo {
    title: string;
}

type Page = 'home' | 'about' | 'contact';

const x: Record<Page, PageInfo> = {
    about: { title: 'about' },
    contact: { title: 'contact' },
    home: { title: 'home' },
};
```

## `Pick<T,K>`

從類型`T`中挑選部分屬性`K`來構造類型。

### 例子

```ts
interface Todo {
    title: string;
    description: string;
    completed: boolean;
}

type TodoPreview = Pick<Todo, 'title' | 'completed'>;

const todo: TodoPreview = {
    title: 'Clean room',
    completed: false,
};
```

## `Exclude<T,U>`

從類型`T`中剔除所有可以賦值給`U`的屬性，然後構造一個類型。

### 例子

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2 = Exclude<string | number | (() => void), Function>;  // string | number
```

## `Extract<T,U>`

從類型`T`中提取所有可以賦值給`U`的類型，然後構造一個類型。

### 例子

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void
```

## `NonNullable<T>`

從類型`T`中剔除`null`和`undefined`，然後構造一個類型。

### 例子

```ts
type T0 = NonNullable<string | number | undefined>;  // string | number
type T1 = NonNullable<string[] | null | undefined>;  // string[]
```

## `ReturnType<T>`

由函數類型`T`的返回值類型構造一個類型。

### 例子

```ts
type T0 = ReturnType<() => string>;  // string
type T1 = ReturnType<(s: string) => void>;  // void
type T2 = ReturnType<(<T>() => T)>;  // {}
type T3 = ReturnType<(<T extends U, U extends number[]>() => T)>;  // number[]
type T4 = ReturnType<typeof f1>;  // { a: number, b: string }
type T5 = ReturnType<any>;  // any
type T6 = ReturnType<never>;  // any
type T7 = ReturnType<string>;  // Error
type T8 = ReturnType<Function>;  // Error
```

## `InstanceType<T>`

由構造函數類型`T`的實例類型構造一個類型。

### 例子

```ts
class C {
    x = 0;
    y = 0;
}

type T0 = InstanceType<typeof C>;  // C
type T1 = InstanceType<any>;  // any
type T2 = InstanceType<never>;  // any
type T3 = InstanceType<string>;  // Error
type T4 = InstanceType<Function>;  // Error
```

## `Required<T>`

構造一個類型，使類型`T`的所有屬性為`required`。

### 例子

```ts
interface Props {
    a?: number;
    b?: string;
};

const obj: Props = { a: 5 }; // OK

const obj2: Required<Props> = { a: 5 }; // Error: property 'b' missing
```

## `ThisType<T>`

這個工具不會返回一個轉換後的類型。它做為上下文的`this`類型的一個標記。注意，若想使用此類型，必須啟用`--noImplicitThis`。

### 例子

```ts
// Compile with --noImplicitThis

type ObjectDescriptor<D, M> = {
    data?: D;
    methods?: M & ThisType<D & M>;  // Type of 'this' in methods is D & M
}

function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
    let data: object = desc.data || {};
    let methods: object = desc.methods || {};
    return { ...data, ...methods } as D & M;
}

let obj = makeObject({
    data: { x: 0, y: 0 },
    methods: {
        moveBy(dx: number, dy: number) {
            this.x += dx;  // Strongly typed this
            this.y += dy;  // Strongly typed this
        }
    }
});

obj.x = 10;
obj.y = 20;
obj.moveBy(5, 5);
```

上面例子中，`makeObject`參數里的`methods`對象具有一個上下文類型`ThisType<D & M>`，因此`methods`對象的方法裡`this`的類型為`{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`。

在`lib.d.ts`裡，`ThisType<T>`標識接口是個簡單的空接口聲明。除了在被識別為對象字面量的上下文類型之外，這個接口與一般的空接口沒有什麼不同。
