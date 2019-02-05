TypeScript不是憑空存在的。
它從JavaScript生態系統和大量現存的JavaScript而來。
將JavaScript程式碼轉換成TypeScript雖乏味卻不是難事。
接下來這篇教學將教您怎麼做。
在開始轉換TypeScript之前，我們假設您已經理解了足夠多本手冊裡的內容。

如果您打算要轉換一個React工程，推薦您先閱讀[React轉換指南](https://github.com/Microsoft/TypeScript-React-Conversion-Guide#typescript-react-conversion-guide)。

# 設置目錄

如果您在寫純JavaScript，您大概是想直接執行這些JavaScript檔案，
這些檔案存在於`src`，`lib`或`dist`目錄裡，它們可以按照預想執行。

若如此，那麼您寫的純JavaScript檔案將做為TypeScript的輸入，您將要執行的是TypeScript的輸出。
在從JS到TS的轉換程序中，我們會分離輸入檔案以防TypeScript覆蓋它們。
您也可以指定輸出目錄。

您可能還需要對JavaScript做一些中間處理，比如合併或經過Babel再次編譯。
在這種情況下，您應該已經有了如下的目錄結構。

那麼現在，我們假設您已經設置了這樣的目錄結構：

```text
projectRoot
├── src
│   ├── file1.js
│   └── file2.js
├── built
└── tsconfig.json
```

如果您在`src`目錄外還有`tests`資料夾，那麼在`src`裡可以有一個`tsconfig.json`檔案，在`tests`裡還可以有一個。

# 書寫組態檔案

TypeScript使用`tsconfig.json`檔案管理工程組態，例如您想包含哪些檔案和進行哪些檢查。
讓我們先建立一個簡單的工程組態檔案：

```json
{
    "compilerOptions": {
        "outDir": "./built",
        "allowJs": true,
        "target": "es5"
    },
    "include": [
        "./src/**/*"
    ]
}
```

這裡我們為TypeScript設置了一些東西:

1. 讀取所有可識別的`src`目錄下的檔案(透過`include`)。
2. 接受JavaScript做為輸入(透過`allowJs`)。
3. 產生的所有檔案放在`built`目錄下(透過`outDir`)。
4. 將JavaScript程式碼降級到低版本比如ECMAScript 5(透過`target`)。

現在，如果您在專案根目錄下執行`tsc`，就可以在`built`目錄下看到產生的檔案。
`built`下的檔案應該與`src`下的檔案相同。
現在您的專案裡的TypeScript已經可以工作了。

## 早期收益

現在您已經可以看到TypeScript帶來的好處，它能幫助我們理解當前工程。
如果您打開像[VS Code](https://code.visualstudio.com)或[Visual Studio](https://visualstudio.com)這樣的編譯器，您就能使用像自動補全這樣的工具。
您還可以組態如下的選項來幫助尋找BUG：

* `noImplicitReturns` 會防止您忘記在函數末尾返回值。
* `noFallthroughCasesInSwitch` 會防止在`switch`程式碼塊裡的兩個`case`之間忘記加入`break`敘述。

TypeScript還能發現那些執行不到的程式碼和標籤，您可以透過設置`allowUnreachableCode`和`allowUnusedLabels`選項來禁用。

# 與建構工具進行整合

在您的建構管道中可能包含多個步驟。
比如為每個檔案加入一些內容。
每種工具的使用方法都是不同的，我們會儘可能的包涵主流的工具。

## Gulp

如果您在使用時髦的Gulp，我們已經有一篇關於[使用Gulp](./Gulp.md)結合TypeScript並與常見建構工具Browserify，Babelify和Uglify進行整合的教學。
請閱讀這篇教學。

## Webpack

Webpack整合非常簡單。
您可以使用`awesome-typescript-loader`，它是一個TypeScript的載入器，結合`source-map-loader`方便偵錯。
執行：

```shell
npm install awesome-typescript-loader source-map-loader
```

並將下面的選項合併到您的`webpack.config.js`檔案裡：

```js
module.exports = {
    entry: "./src/index.ts",
    output: {
        filename: "./dist/bundle.js",
    },

    // Enable sourcemaps for debugging webpack's output.
    devtool: "source-map",

    resolve: {
        // Add '.ts' and '.tsx' as resolvable extensions.
        extensions: ["", ".webpack.js", ".web.js", ".ts", ".tsx", ".js"]
    },

    module: {
        loaders: [
            // All files with a '.ts' or '.tsx' extension will be handled by 'awesome-typescript-loader'.
            { test: /\.tsx?$/, loader: "awesome-typescript-loader" }
        ],

        preLoaders: [
            // All output '.js' files will have any sourcemaps re-processed by 'source-map-loader'.
            { test: /\.js$/, loader: "source-map-loader" }
        ]
    },

    // Other options...
};
```

要注意的是，`awesome-typescript-loader`必須在其它處理`.js`檔案的載入器之前執行。

這與另一個TypeScript的Webpack載入器[ts-loader](https://github.com/TypeStrong/ts-loader)是一樣的。
您可以到[這裡](https://github.com/s-panferov/awesome-typescript-loader#differences-between-ts-loader)瞭解兩者之間的差別。

您可以在[React和Webpack教學](./React & Webpack.md)裡找到使用Webpack的範例。

# 轉換到TypeScript檔案

到目前為止，您已經做好了使用TypeScript檔案的準備。
第一步，將`.js`檔案重命名為`.ts`檔案。
如果您使用了JSX，則重命名為`.tsx`檔案。

第一步達成？
太棒了!
您已經成功地將一個檔案從JavaScript轉換成了TypeScript!

當然了，您可能感覺哪裡不對勁兒。
如果您在支援TypeScript的編輯器(或執行`tsc --pretty`)裡打開了那個檔案，您可能會看到有些行上有紅色的波浪線。
您可以把它們當做在Microsoft Word裡看到的紅色波浪線一樣。
但是TypeScript仍然會編譯您的程式碼，就好比Word還是允許您列印您的文件一樣。

如果對您來說這種行為太隨便了，您可以讓它變得嚴格些。
如果，您*不想*在發生錯誤的時候，TypeScript還會被編譯成JavaScript，您可以使用`noEmitOnError`選項。
從某種意義上來講，TypeScript具有一個調整它的嚴格性的刻度盤，您可以將指標拔動到您想要的位置。

如果您計畫使用可用的高度嚴格的設置，最好現在就啟用它們(查看[啟用嚴格檢查](#getting-stricter-checks))。
比如，如果您不想讓TypeScript將沒有明確指定的型別默默地推斷為`any`型別，可以在修改檔案之前啟用`noImplicitAny`。
您可能會覺得這有些過度嚴格，但是長期收益很快就能顯現出來。

## 去除錯誤

我們提到過，若不出所料，在轉換後將會看到錯誤訊息。
重要的是我們要逐一的查看它們並決定如何處理。
通常這些都是真正的BUG，但有時必須要告訴TypeScript您要做的是什麼。

### 由模組導入

首先您可能會看到一些類似`Cannot find name 'require'.`和`Cannot find name 'define'.`的錯誤。
遇到這種情況說明您在使用模組。
您僅需要告訴TypeScript它們是存在的：

```ts
// For Node/CommonJS
declare function require(path: string): any;
```

或

```ts
// For RequireJS/AMD
declare function define(...args: any[]): any;
```

最好是避免使用這些呼叫而改用TypeScript的導入語法。

首先，您要使用TypeScript的`module`標記來啟用一些模組系統。
可用的選項有`commonjs`，`amd`，`system`，and `umd`。

如果程式碼裡存在下面的Node/CommonJS程式碼：

```js
var foo = require("foo");

foo.doStuff();
```

或者下面的RequireJS/AMD程式碼：

```js
define(["foo"], function(foo) {
    foo.doStuff();
})
```

那麼可以寫做下面的TypeScript程式碼：

```ts
import foo = require("foo");

foo.doStuff();
```

### 獲取宣告檔案

如果您開始做轉換到TypeScript導入，您可能會遇到`Cannot find module 'foo'.`這樣的錯誤。
問題出在沒有*宣告檔案*來描述您的程式碼庫。
幸運的是這非常簡單。
如果TypeScript報怨像是沒有`lodash`包，那您只需這樣做

```shell
npm install -S @types/lodash
```

如果您沒有使用`commonjs`模組模組選項，那麼就需要將`moduleResolution`選項設置為`node`。

之後，您應該就可以導入`lodash`了，並且會獲得精確的自動補全功能。

### 由模組導出

通常來講，由模組導出涉及加入屬性到`exports`或`module.exports`。
TypeScript允許您使用頂級的導出敘述。
比如，您要導出下面的函數：

```js
module.exports.feedPets = function(pets) {
    // ...
}
```

那麼您可以這樣寫：

```ts
export function feedPets(pets) {
    // ...
}
```

有時您會完全重寫導出物件。
這是一個常見模式，這會將模組變為可立即呼叫的模組：

```js
var express = require("express");
var app = express();
```

之前您可以是這樣寫的：

```js
function foo() {
    // ...
}
module.exports = foo;
```

在TypeScript裡，您可以使用`export =`來代替。

```ts
function foo() {
    // ...
}
export = foo;
```

### 過多或過少的參數

有時您會發現您在呼叫一個具有過多或過少參數的函數。
通常，這是一個BUG，但在某些情況下，您可以宣告一個使用`arguments`物件的函數而不需要寫出所有參數:

```js
function myCoolFunction() {
    if (arguments.length == 2 && !Array.isArray(arguments[1])) {
        var f = arguments[0];
        var arr = arguments[1];
        // ...
    }
    // ...
}

myCoolFunction(function(x) { console.log(x) }, [1, 2, 3, 4]);
myCoolFunction(function(x) { console.log(x) }, 1, 2, 3, 4);
```

這種情況下，我們需要利用TypeScript的函數重載來告訴呼叫者`myCoolFunction`函數的呼叫方式。

```ts
function myCoolFunction(f: (x: number) => void, nums: number[]): void;
function myCoolFunction(f: (x: number) => void, ...nums: number[]): void;
function myCoolFunction() {
    if (arguments.length == 2 && !Array.isArray(arguments[1])) {
        var f = arguments[0];
        var arr = arguments[1];
        // ...
    }
    // ...
}
```

我們為`myCoolFunction`函數加入了兩個重載簽名。
第一個檢查`myCoolFunction`函數是否接收一個函數(它又接收一個`number`參數)和一個`number`陣列。
第二個同樣是接收了一個函數，並且使用剩餘參數(`...nums`)來表示之後的其它所有參數必須是`number`型別。

### 連續加入屬性

有些人可能會因為程式碼美觀性而喜歡先建立一個物件然後立即加入屬性：

```js
var options = {};
options.color = "red";
options.volume = 11;
```

TypeScript會提示您不能給`color`和`volumn`賦值，因為先前指定`options`的型別為`{}`並不帶有任何屬性。
如果您將宣告變成物件字面量的形式將不會產生錯誤：

```ts
let options = {
    color: "red",
    volume: 11
};
```

您還可以定義`options`的型別並且加入型別斷言到物件字面量上。

```ts
interface Options { color: string; volume: number }

let options = {} as Options;
options.color = "red";
options.volume = 11;
```

或者，您可以將`options`指定成`any`型別，這是最簡單的，但也是獲益最少的。

### `any`，`Object`，和`{}`

您可能會試圖使用`Object`或`{}`來表示一個值可以具有任意屬性，因為`Object`是最通用的型別。
然而在這種情況下**`any`是真正想要使用的型別**，因為它是最*靈活*的型別。

比如，有一個`Object`型別的東西，您將不能夠在其上呼叫`toLowerCase()`。

越普通意味著更少的利用型別，但是`any`比較特殊，它是最普通的型別但是允許您在上面做任何事情。
也就是說您可以在上面呼叫，構造它，存取它的屬性等等。
記住，當您使用`any`時，您會失去大多數TypeScript提供的錯誤檢查和編譯器支援。

如果您還是決定使用`Object`和`{}`，您應該選擇`{}`。
雖說它們基本一樣，但是從技術角度上來講`{}`在一些深奧的情況裡比`Object`更普通。

## <a name="getting-stricter-checks"></a>啟用嚴格檢查

TypeScript提供了一些檢查來保證安全以及幫助分析您的程式。
當您將程式碼轉換為了TypeScript後，您可以啟用這些檢查來幫助您獲得高度安全性。

### 沒有隱式的`any`

在某些情況下TypeScript沒法確定某些值的型別。
那麼TypeScript會使用`any`型別代替。
這對程式碼轉換來講是不錯，但是使用`any`意味著失去了型別安全保障，並且您得不到工具的支援。
您可以使用`noImplicitAny`選項，讓TypeScript標記出發生這種情況的地方，並給出一個錯誤。

### 嚴格的`null`與`undefined`檢查

預設地，TypeScript把`null`和`undefined`當做屬於任何型別。
這就是說，宣告為`number`型別的值可以為`null`和`undefined`。
因為在JavaScript和TypeScript裡，`null`和`undefined`經常會導致BUG的產生，所以TypeScript包含了`strictNullChecks`選項來幫助我們減少對這種情況的擔憂。

當啟用了`strictNullChecks`，`null`和`undefined`獲得了它們自己各自的型別`null`和`undefined`。
當任何值*可能*為`null`，您可以使用聯合型別。
比如，某值可能為`number`或`null`，您可以宣告它的型別為`number | null`。

假設有一個值TypeScript認為可以為`null`或`undefined`，但是您更清楚它的型別，您可以使用`!`後綴。

```ts
declare var foo: string[] | null;

foo.length;  // error - 'foo' is possibly 'null'

foo!.length; // okay - 'foo!' just has type 'string[]'
```

要當心，當您使用`strictNullChecks`，您的依賴也需要對應地啟用`strictNullChecks`。

### `this`沒有隱式的`any`

當您在類的外部使用`this`關鍵字時，它會預設獲得`any`型別。
比如，假設有一個`Point`類，並且我們要加入一個函數做為它的方法：

```ts
class Point {
    constructor(public x, public y) {}
    getDistance(p: Point) {
        let dx = p.x - this.x;
        let dy = p.y - this.y;
        return Math.sqrt(dx ** 2 + dy ** 2);
    }
}
// ...

// Reopen the interface.
interface Point {
    distanceFromOrigin(point: Point): number;
}
Point.prototype.distanceFromOrigin = function(point: Point) {
    return this.getDistance({ x: 0, y: 0});
}
```

這就產生了我們上面提到的錯誤 - 如果我們錯誤地拼寫了`getDistance`並不會得到一個錯誤。
正因此，TypeScript有`noImplicitThis`選項。
當設置了它，TypeScript會產生一個錯誤當沒有明確指定型別(或透過型別推斷)的`this`被使用時。
解決的方法是在介面或函數上使用指定了型別的`this`參數：

```ts
Point.prototype.distanceFromOrigin = function(this: Point, point: Point) {
    return this.getDistance({ x: 0, y: 0});
}
```
