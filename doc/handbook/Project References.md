工程引用是TypeScript 3.0的新特性，它支持將TypeScript程序的結構分割成更小的組成部分。

這樣可以改善構建時間，強制在邏輯上對組件進行分離，更好地組織你的代碼。

TypeScript 3.0還引入了`tsc`的一種新模式，即`--build`標記，它與工程引用協同工作可以加速TypeScript的構建。

# 一個工程示例

讓我們來看一個非常普通的工程，並瞧瞧工程引用特性是如何幫助我們更好地組織代碼的。
假設這個工程具有兩個模塊：`converter`和`unites`，以及相應的測試代碼：

```shell
/src/converter.ts
/src/units.ts
/test/converter-tests.ts
/test/units-tests.ts
/tsconfig.json
```

測試文件導入相應的實現文件並進行測試：

```ts
// converter-tests.ts
import * as converter from "../converter";

assert.areEqual(converter.celsiusToFahrenheit(0), 32);
```

在之前，這種使用單一`tsconfig`文件的結構會稍顯笨拙：

* 實現文件也可以導入測試文件
* 無法同時構建`test`和`src`，除非把`src`也放在輸出文件夾中，但通常並不想這樣做
* 僅對實現文件的*內部*細節進行改動，必需再次對測試進行*類型檢查*，儘管這是根本不必要的
* 僅對測試文件進行改動，必需再次對實現文件進行*類型檢查*，儘管其實什麼都沒有變

你可以使用多個`tsconfig`文件來解決*部分*問題，但是又會出現新問題：

* 缺少內置的實時檢查，因此你得多次運行`tsc`
* 多次調用`tsc`會增加我們等待的時間
* `tsc -w`不能一次在多個配置文件上運行

工程引用可以解決全部這些問題，而且還不止。

# 何為工程引用？

`tsconfig.json`增加了一個新的頂層屬性`references`。它是一個對象的數組，指明要引用的工程：

```js
{
    "compilerOptions": {
        // The usual
    },
    "references": [
        { "path": "../src" }
    ]
}
```

每個引用的`path`屬性都可以指向到包含`tsconfig.json`文件的目錄，或者直接指向到配置文件本身（名字是任意的）。

當你引用一個工程時，會發生下面的事：

* 導入引用工程中的模塊實際加載的是它*輸出*的聲明文件（`.d.ts`）。
* 如果引用的工程生成一個`outFile`，那麼這個輸出文件的`.d.ts`文件裡的聲明對於當前工程是可見的。
* 構建模式（後文）會根據需要自動地構建引用的工程。

當你拆分成多個工程後，會顯著地加速類型檢查和編譯，減少編輯器的內存佔用，還會改善程序在邏輯上進行分組。

# `composite`

引用的工程必須啟用新的`composite`設置。
這個選項用於幫助TypeScript快速確定引用工程的輸出文件位置。
若啟用`composite`標記則會發生如下變動：

* 對於`rootDir`設置，如果沒有被顯式指定，默認為包含`tsconfig`文件的目錄
* 所有的實現文件必須匹配到某個`include`模式或在`files`數組裡列出。如果違反了這個限制，`tsc`會提示你哪些文件未指定。
* 必須開啟`declaration`選項。

# `declarationMaps`

我們增加了對[declaration source maps](https://github.com/Microsoft/TypeScript/issues/14479)的支持。
如果啟用`--declarationMap`，在某些編輯器上，你可以使用諸如“Go to Definition”，重命名以及跨工程編輯文件等編輯器特性。

# 帶`prepend`的`outFile`

你可以在引用中使用`prepend`選項來啟用前置某個依賴的輸出：

```js
   "references": [
       { "path": "../utils", "prepend": true }
   ]
```

前置工程會將工程的輸出添加到當前工程的輸出之前。
它對`.js`文件和`.d.ts`文件都有效，`source map`文件也同樣會正確地生成。

`tsc`永遠只會使用磁盤上已經存在的文件來進行這個操作，因此你可能會創建出一個無法生成正確輸出文件的工程，因為有些工程的輸出可能會在結果文件中重覆了多次。
例如：

```txt
   A
  ^ ^
 /   \
B     C
 ^   ^
  \ /
   D
```

這種情況下，不能前置引用，因為在`D`的最終輸出裡會有兩份`A`存在 - 這可能會發生未知錯誤。

# 關於工程引用的說明

工程引用在某些方面需要你進行權衡.

因為有依賴的工程要使用它的依賴生成的`.d.ts`，因此你必須要檢查相應構建後的輸出*或*在下載源碼後進行構建，然後才能在編輯器裡自由地導航。
我們是在操控幕後的`.d.ts`生成過程，我們應該減少這種情況，但是目前還們建議提示開發者在下載源碼後進行構建。

此外，為了兼容已有的構建流程，`tsc`*不會*自動地構建依賴項，除非啟用了`--build`選項。
下面讓我們看看`--build`。

# TypeScript構建模式

在TypeScript工程裡支持增量構建是個期待已久的功能。
在TypeScrpt 3.0里，你可以在`tsc`上使用`--build`標記。
它實際上是個新的`tsc`入口點，它更像是一個構建的協調員而不是簡簡單單的編譯器。

運行`tsc --build`（簡寫`tsc -b`）會執行如下操作：

* 找到所有引用的工程
* 檢查它們是否為最新版本
* 按順序構建非最新版本的工程

可以給`tsc -b`指定多個配置文件地址（例如：`tsc -b src test`）。
如同`tsc -p`，如果配置文件名為`tsconfig.json`，那麼文件名則可省略。

## `tsc -b`命令行

你可以指令任意數量的配置文件：

```shell
 > tsc -b                                # Build the tsconfig.json in the current directory
 > tsc -b src                            # Build src/tsconfig.json
 > tsc -b foo/release.tsconfig.json bar  # Build foo/release.tsconfig.json and bar/tsconfig.json
```

不需要擔心命令行上指定的文件順序 - `tsc`會根據需要重新進行排序，被依賴的項會優先構建。

`tsc -b`還支持其它一些選項：

* `--verbose`：打印詳細的日誌（可以與其它標記一起使用）
* `--dry`: 顯示將要執行的操作但是並不真正進行這些操作
* `--clean`: 刪除指定工程的輸出（可以與`--dry`一起使用）
* `--force`: 把所有工程當作非最新版本對待
* `--watch`: 觀察模式（可以與`--verbose`一起使用）

# 說明

一般情況下，就算代碼裡有語法或類型錯誤，`tsc`也會生成輸出（`.js`和`.d.ts`），除非你啟用了`noEmitOnError`選項。
這在增量構建系統裡就不好了 - 如果某個過期的依賴裡有一個新的錯誤，那麼你只能看到它*一次*，因為後續的構建會跳過這個最新的工程。
正是這個原因，`tsc -b`的作用就好比在所有工程上啟用了`noEmitOnError`。

如果你想要提交所有的構建輸出（`.js`, `.d.ts`, `.d.ts.map`等），你可能需要運行`--force`來構建，因為一些源碼版本管理操作依賴於源碼版本管理工具保存的本地拷貝和遠程拷貝的時間戳。

# MSBuild

如果你的工程使用msbuild，你可以用下面的方式開啟構建模式。

```xml
    <TypeScriptBuildMode>true</TypeScriptBuildMode>
```

將這段代碼添加到`proj`文件。它會自動地啟用增量構建模式和清理工作。

注意，在使用`tsconfig.json` / `-p`時，已存在的TypeScript工程屬性會被忽略 - 因此所有的設置需要在`tsconfig`文件裡進行。

一些團隊已經設置好了基於msbuild的構建流程，並且`tsconfig`文件具有和它們匹配的工程一致的*隱式*圖序。
若你的項目如此，那麼可以繼續使用`msbuild`和`tsc -p`以及工程引用；它們是完全互通的。

# 指導

## 整體結構

當`tsconfig.json`多了以後，通常會使用[配置文件繼承](https://www.typescriptlang.org/docs/handbook/tsconfig-json.html)來集中管理公共的編譯選項。
這樣你就可以在一個文件裡更改配置而不必在多個文件中進行修改。

另一個最佳實踐是有一個`solution`級別的`tsconfig.json`文件，它僅僅用於引用所有的子工程。
它用於提供一個簡單的入口；比如，在TypeScript源碼裡，我們可以簡單地運行`tsc -b src`來構建所有的節點，因為我們在`src/tsconfig.json`文件裡列出了所有的子工程。
注意從3.0開始，如果`tsconfig.json`文件裡有至少一個工程引用`reference`，那麼`files`數組為空的話也不會報錯。

你可以在TypeScript源碼倉庫裡看到這些模式 - 閱讀`src/tsconfig_base.json`，`src/tsconfig.json`和`src/tsc/tsconfig.json`。

## 相對模塊的結構

通常地，將代碼轉成使用相對模塊並不需要改動太多。
只需在某個給定父目錄的每個子目錄裡放一個`tsconfig.json`文件，並相應添加`reference`。
然後將`outDir`指定為輸出目錄的子目錄或將`rootDir`指定為所有工程的某個公共根目錄。

## `outFile`的結構

使用了`outFile`的編譯輸出結構十分靈活，因為相對路徑是無關緊要的。
要注意的是，你通常不需要使用`prepend` - 因為這會改善構建時間並結省I/O。
TypeScript項目本身是一個好的參照 - 我們有一些“library”的工程和一些“endpoint”工程，“endpoint”工程會確保足夠小並僅僅導入它們需要的“library”。

<!--
## Structuring for monorepos

TODO: Experiment more and figure this out. Rush and Lerna seem to have different models that imply different things on our end
-->
