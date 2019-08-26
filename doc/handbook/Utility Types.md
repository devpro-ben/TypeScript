# 介紹

TypeScript提供一些工具型別來幫助常見的型別轉換。這些型別是全域可見的。

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

構造型別`T`，並將它所有的屬性設置為選擇性的。它的返回型別表示輸入型別的所有子型別。

### 範例

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

構造型別`T`，並將它所有的屬性設置為`readonly`，也就是說構造出的型別的屬性不能被再次賦值。

### 範例

```ts
interface Todo {
    title: string;
}

const todo: Readonly<Todo> = {
    title: 'Delete inactive users',
};

todo.title = 'Hello'; // Error: cannot reassign a readonly property
```

這個工具可用來表示在執行時會失敗的賦值運算式(比如，當嘗試給[凍結物件](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/freeze)的屬性再次賦值時)。

### `Object.freeze`

```ts
function freeze<T>(obj: T): Readonly<T>;
```

## `Record<K,T>`

構造一個型別，其屬性名的型別為`K`，屬性值的型別為`T`。這個工具可用來將某個型別的屬性映射到另一個型別上。

### 範例

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

從型別`T`中挑選部分屬性`K`來構造型別。

### 範例

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

從型別`T`中剔除所有可以賦值給`U`的屬性，然後構造一個型別。

### 範例

```ts
type T0 = Exclude<"a" | "b" | "c", "a">;  // "b" | "c"
type T1 = Exclude<"a" | "b" | "c", "a" | "b">;  // "c"
type T2 = Exclude<string | number | (() => void), Function>;  // string | number
```

## `Extract<T,U>`

從型別`T`中提取所有可以賦值給`U`的型別，然後構造一個型別。

### 範例

```ts
type T0 = Extract<"a" | "b" | "c", "a" | "f">;  // "a"
type T1 = Extract<string | number | (() => void), Function>;  // () => void
```

## `NonNullable<T>`

從型別`T`中剔除`null`和`undefined`，然後構造一個型別。

### 範例

```ts
type T0 = NonNullable<string | number | undefined>;  // string | number
type T1 = NonNullable<string[] | null | undefined>;  // string[]
```

## `ReturnType<T>`

由函數型別`T`的返回值型別構造一個型別。

### 範例

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

由建構函數型別`T`的實例型別構造一個型別。

### 範例

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

構造一個型別，使型別`T`的所有屬性為`required`。

### 範例

```ts
interface Props {
    a?: number;
    b?: string;
};

const obj: Props = { a: 5 }; // OK

const obj2: Required<Props> = { a: 5 }; // Error: property 'b' missing
```

## `ThisType<T>`

這個工具不會返回一個轉換後的型別。它做為上下文的`this`型別的一個標記。注意，若想使用此型別，必須啟用`--noImplicitThis`。

### 範例

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

上面範例中，`makeObject`參數里的`methods`物件具有一個上下文型別`ThisType<D & M>`，因此`methods`物件的方法裡`this`的型別為`{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`。

在`lib.d.ts`裡，`ThisType<T>`標識介面是個簡單的空介面宣告。除了在被識別為物件字面量的上下文型別之外，這個介面與一般的空介面沒有什麼不同。
