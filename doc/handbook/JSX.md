# 介紹

[JSX](https://facebook.github.io/jsx/)是一種嵌入式的類似XML的語法。
它可以被轉換成合法的JavaScript，儘管轉換的語義是依據不同的實現而定的。
JSX因[React](https://reactjs.org/)框架而流行，但也存在其它的實現。
TypeScript支持內嵌，類型檢查以及將JSX直接編譯為JavaScript。

# 基本用法

想要使用JSX必須做兩件事：

1. 給文件一個`.tsx`擴展名
2. 啟用`jsx`選項

TypeScript具有三種JSX模式：`preserve`，`react`和`react-native`。
這些模式只在代碼生成階段起作用 - 類型檢查並不受影響。
在`preserve`模式下生成代碼中會保留JSX以供後續的轉換操作使用（比如：[Babel](https://babeljs.io/)）。
另外，輸出文件會帶有`.jsx`擴展名。
`react`模式會生成`React.createElement`，在使用前不需要再進行轉換操作了，輸出文件的擴展名為`.js`。
`react-native`相當於`preserve`，它也保留了所有的JSX，但是輸出文件的擴展名是`.js`。

模式            | 輸入      | 輸出                          | 輸出文件擴展名
---------------|-----------|------------------------------|----------------------
`preserve`     | `<div />` | `<div />`                    | `.jsx`
`react`        | `<div />` | `React.createElement("div")` | `.js`
`react-native` | `<div />` | `<div />`                    | `.js`

你可以通過在命令行裡使用`--jsx`標記或[tsconfig.json](./tsconfig.json.md)裡的選項來指定模式。

> *注意：`React`標識符是寫死的硬編碼，所以你必須保證React（大寫的R）是可用的。*

# `as`操作符

回想一下怎麼寫類型斷言：

```ts
var foo = <foo>bar;
```

這裡斷言`bar`變量是`foo`類型的。
因為TypeScript也使用尖括號來表示類型斷言，在結合JSX的語法後將帶來解析上的困難。因此，TypeScript在`.tsx`文件裡禁用了使用尖括號的類型斷言。

由於不能夠在`.tsx`文件裡使用上述語法，因此我們應該使用另一個類型斷言操作符：`as`。
上面的例子可以很容易地使用`as`操作符改寫：

```ts
var foo = bar as foo;
```

`as`操作符在`.ts`和`.tsx`裡都可用，並且與尖括號類型斷言行為是等價的。

# 類型檢查

為了理解JSX的類型檢查，你必須首先理解固有元素與基於值的元素之間的區別。
假設有這樣一個JSX表達式`<expr />`，`expr`可能引用環境自帶的某些東西（比如，在DOM環境裡的`div`或`span`）或者是你自定義的組件。
這是非常重要的，原因有如下兩點：

1. 對於React，固有元素會生成字符串（`React.createElement("div")`），然而由你自定義的組件卻不會生成（`React.createElement(MyComponent)`）。
2. 傳入JSX元素裡的屬性類型的查找方式不同。
  固有元素屬性*本身*就支持，然而自定義的組件會自己去指定它們具有哪個屬性。

TypeScript使用[與React相同的規範](http://facebook.github.io/react/docs/jsx-in-depth.html#html-tags-vs.-react-components) 來區別它們。
固有元素總是以一個小寫字母開頭，基於值的元素總是以一個大寫字母開頭。

## 固有元素

固有元素使用特殊的接口`JSX.IntrinsicElements`來查找。
默認地，如果這個接口沒有指定，會全部通過，不對固有元素進行類型檢查。
然而，如果這個接口存在，那麼固有元素的名字需要在`JSX.IntrinsicElements`接口的屬性裡查找。
例如：

```ts
declare namespace JSX {
    interface IntrinsicElements {
        foo: any
    }
}

<foo />; // 正確
<bar />; // 錯誤
```

在上例中，`<foo />`沒有問題，但是`<bar />`會報錯，因為它沒在`JSX.IntrinsicElements`裡指定。

> 注意：你也可以在`JSX.IntrinsicElements`上指定一個用來捕獲所有字符串索引：
>```ts
>declare namespace JSX {
>    interface IntrinsicElements {
>        [elemName: string]: any;
>    }
>}
>```

## 基於值的元素

基於值的元素會簡單的在它所在的作用域裡按標識符查找。

```ts
import MyComponent from "./myComponent";

<MyComponent />; // 正確
<SomeOtherComponent />; // 錯誤
```

有兩種方式可以定義基於值的元素：

1. 無狀態函數組件 (SFC)
2. 類組件

由於這兩種基於值的元素在JSX表達式裡無法區分，因此TypeScript首先會嘗試將表達式做為無狀態函數組件進行解析。如果解析成功，那麼TypeScript就完成了表達式到其聲明的解析操作。如果按照無狀態函數組件解析失敗，那麼TypeScript會繼續嘗試以類組件的形式進行解析。如果依舊失敗，那麼將輸出一個錯誤。

### 無狀態函數組件

正如其名，組件被定義成JavaScript函數，它的第一個參數是`props`對象。
TypeScript會強制它的返回值可以賦值給`JSX.Element`。

```ts
interface FooProp {
  name: string;
  X: number;
  Y: number;
}

declare function AnotherComponent(prop: {name: string});
function ComponentFoo(prop: FooProp) {
  return <AnotherComponent name={prop.name} />;
}

const Button = (prop: {value: string}, context: { color: string }) => <button>
```

由於無狀態函數組件是簡單的JavaScript函數，所以我們還可以利用函數重載。

```ts
interface ClickableProps {
  children: JSX.Element[] | JSX.Element
}

interface HomeProps extends ClickableProps {
  home: JSX.Element;
}

interface SideProps extends ClickableProps {
  side: JSX.Element | string;
}

function MainButton(prop: HomeProps): JSX.Element;
function MainButton(prop: SideProps): JSX.Element {
  ...
}
```

### 類組件

我們可以定義類組件的類型。
然而，我們首先最好弄懂兩個新的術語：*元素類的類型*和*元素實例的類型*。

現在有`<Expr />`，*元素類的類型*為`Expr`的類型。
所以在上面的例子裡，如果`MyComponent`是ES6的類，那麼類類型就是類的構造函數和靜態部分。
如果`MyComponent`是個工廠函數，類類型為這個函數。

一旦建立起了類類型，實例類型由類構造器或調用簽名（如果存在的話）的返回值的聯合構成。
再次說明，在ES6類的情況下，實例類型為這個類的實例的類型，並且如果是工廠函數，實例類型為這個函數返回值類型。

```ts
class MyComponent {
  render() {}
}

// 使用構造簽名
var myComponent = new MyComponent();

// 元素類的類型 => MyComponent
// 元素實例的類型 => { render: () => void }

function MyFactoryFunction() {
  return {
    render: () => {
    }
  }
}

// 使用調用簽名
var myComponent = MyFactoryFunction();

// 元素類的類型 => FactoryFunction
// 元素實例的類型 => { render: () => void }
```

元素的實例類型很有趣，因為它必須賦值給`JSX.ElementClass`或拋出一個錯誤。
默認的`JSX.ElementClass`為`{}`，但是它可以被擴展用來限制JSX的類型以符合相應的接口。

```ts
declare namespace JSX {
  interface ElementClass {
    render: any;
  }
}

class MyComponent {
  render() {}
}
function MyFactoryFunction() {
  return { render: () => {} }
}

<MyComponent />; // 正確
<MyFactoryFunction />; // 正確

class NotAValidComponent {}
function NotAValidFactoryFunction() {
  return {};
}

<NotAValidComponent />; // 錯誤
<NotAValidFactoryFunction />; // 錯誤
```

## 屬性類型檢查

屬性類型檢查的第一步是確定*元素屬性類型*。
這在固有元素和基於值的元素之間稍有不同。

對於固有元素，這是`JSX.IntrinsicElements`屬性的類型。

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { bar?: boolean }
  }
}

// `foo`的元素屬性類型為`{bar?: boolean}`
<foo bar />;
```

對於基於值的元素，就稍微複雜些。
它取決於先前確定的在元素實例類型上的某個屬性的類型。
至於該使用哪個屬性來確定類型取決於`JSX.ElementAttributesProperty`。
它應該使用單一的屬性來定義。
這個屬性名之後會被使用。
TypeScript 2.8，如果未指定`JSX.ElementAttributesProperty`，那麼將使用類元素構造函數或SFC調用的第一個參數的類型。

```ts
declare namespace JSX {
  interface ElementAttributesProperty {
    props; // 指定用來使用的屬性名
  }
}

class MyComponent {
  // 在元素實例類型上指定屬性
  props: {
    foo?: string;
  }
}

// `MyComponent`的元素屬性類型為`{foo?: string}`
<MyComponent foo="bar" />
```

元素屬性類型用於的JSX裡進行屬性的類型檢查。
支持可選屬性和必須屬性。

```ts
declare namespace JSX {
  interface IntrinsicElements {
    foo: { requiredProp: string; optionalProp?: number }
  }
}

<foo requiredProp="bar" />; // 正確
<foo requiredProp="bar" optionalProp={0} />; // 正確
<foo />; // 錯誤, 缺少 requiredProp
<foo requiredProp={0} />; // 錯誤, requiredProp 應該是字符串
<foo requiredProp="bar" unknownProp />; // 錯誤, unknownProp 不存在
<foo requiredProp="bar" some-unknown-prop />; // 正確, `some-unknown-prop`不是個合法的標識符
```

> 注意：如果一個屬性名不是個合法的JS標識符（像`data-*`屬性），並且它沒出現在元素屬性類型裡時不會當做一個錯誤。

另外，JSX還會使用`JSX.IntrinsicAttributes`接口來指定額外的屬性，這些額外的屬性通常不會被組件的props或arguments使用 - 比如React裡的`key`。還有，`JSX.IntrinsicClassAttributes<T>`泛型類型也可以用來做同樣的事情。這裡的泛型參數表示類實例類型。在React裡，它用來允許`Ref<T>`類型上的`ref`屬性。通常來講，這些接口上的所有屬性都是可選的，除非你想要用戶在每個JSX標籤上都提供一些屬性。

延展操作符也可以使用：

```JSX
var props = { requiredProp: 'bar' };
<foo {...props} />; // 正確

var badProps = {};
<foo {...badProps} />; // 錯誤
```

## 子孫類型檢查

從TypeScript 2.3開始，我們引入了*children*類型檢查。*children*是*元素屬性(attribute)類型*的一個特殊屬性(property)，子*JSXExpression*將會被插入到屬性裡。
與使用`JSX.ElementAttributesProperty`來決定*props*名類似，我們可以利用`JSX.ElementChildrenAttribute`來決定*children*名。
`JSX.ElementChildrenAttribute`應該被聲明在單一的屬性(property)裡。

```ts
declare namespace JSX {
  interface ElementChildrenAttribute {
    children: {};  // specify children name to use
  }
}
```

如不特殊指定子孫的類型，我們將使用[React typings](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)裡的默認類型。

```ts
<div>
  <h1>Hello</h1>
</div>;

<div>
  <h1>Hello</h1>
  World
</div>;

const CustomComp = (props) => <div>props.children</div>
<CustomComp>
  <div>Hello World</div>
  {"This is just a JS expression..." + 1000}
</CustomComp>
```

```ts
interface PropsType {
  children: JSX.Element
  name: string
}

class Component extends React.Component<PropsType, {}> {
  render() {
    return (
      <h2>
        {this.props.children}
      </h2>
    )
  }
}

// OK
<Component>
  <h1>Hello World</h1>
</Component>

// Error: children is of type JSX.Element not array of JSX.Element
<Component>
  <h1>Hello World</h1>
  <h2>Hello World</h2>
</Component>

// Error: children is of type JSX.Element not array of JSX.Element or string.
<Component>
  <h1>Hello</h1>
  World
</Component>
```

# JSX結果類型

默認地JSX表達式結果的類型為`any`。
你可以自定義這個類型，通過指定`JSX.Element`接口。
然而，不能夠從接口裡檢索元素，屬性或JSX的子元素的類型信息。
它是一個黑盒。

# 嵌入的表達式

JSX允許你使用`{ }`標籤來內嵌表達式。

```JSX
var a = <div>
  {['foo', 'bar'].map(i => <span>{i / 2}</span>)}
</div>
```

上面的代碼產生一個錯誤，因為你不能用數字來除以一個字符串。
輸出如下，若你使用了`preserve`選項：

```JSX
var a = <div>
  {['foo', 'bar'].map(function (i) { return <span>{i / 2}</span>; })}
</div>
```

# React整合

要想一起使用JSX和React，你應該使用[React類型定義](https://github.com/DefinitelyTyped/DefinitelyTyped/tree/master/types/react)。
這些類型聲明定義了`JSX`合適命名空間來使用React。

```ts
/// <reference path="react.d.ts" />

interface Props {
  foo: string;
}

class MyComponent extends React.Component<Props, {}> {
  render() {
    return <span>{this.props.foo}</span>
  }
}

<MyComponent foo="bar" />; // 正確
<MyComponent foo={0} />; // 錯誤
```

# 工廠函數

`jsx: react`編譯選項使用的工廠函數是可以配置的。可以使用`jsxFactory`命令行選項，或內聯的`@jsx`註釋指令在每個文件上設置。比如，給`createElement`設置`jsxFactory`，`<div />`會使用`createElement("div")`來生成，而不是`React.createElement("div")`。

註釋指令可以像下面這樣使用（在TypeScript 2.8里）：

```ts
import preact = require("preact");
/* @jsx preact.h */
const x = <div />;
```

生成：

```ts
const preact = require("preact");
const x = preact.h("div", null);
```

工廠函數的選擇同樣會影響`JSX`命名空間的查找（類型檢查）。如果工廠函數使用`React.createElement`定義（默認），編譯器會先檢查`React.JSX`，之後才檢查全局的`JSX`。如果工廠函數定義為`h`，那麼在檢查全局的`JSX`之前先檢查`h.JSX`。
