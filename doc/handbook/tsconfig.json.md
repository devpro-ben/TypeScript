## 概述

如果一個目錄下存在一個`tsconfig.json`文件，那麼它意味著這個目錄是TypeScript項目的根目錄。
`tsconfig.json`文件中指定了用來編譯這個項目的根文件和編譯選項。
一個項目可以通過以下方式之一來編譯：

## 使用tsconfig.json

* 不帶任何輸入文件的情況下調用`tsc`，編譯器會從當前目錄開始去查找`tsconfig.json`文件，逐級向上搜索父目錄。
* 不帶任何輸入文件的情況下調用`tsc`，且使用命令行參數`--project`（或`-p`）指定一個包含`tsconfig.json`文件的目錄。

當命令行上指定了輸入文件時，`tsconfig.json`文件會被忽略。

## 示例

`tsconfig.json`示例文件:

* 使用`"files"`屬性

```json
{
    "compilerOptions": {
        "module": "commonjs",
        "noImplicitAny": true,
        "removeComments": true,
        "preserveConstEnums": true,
        "sourceMap": true
    },
    "files": [
        "core.ts",
        "sys.ts",
        "types.ts",
        "scanner.ts",
        "parser.ts",
        "utilities.ts",
        "binder.ts",
        "checker.ts",
        "emitter.ts",
        "program.ts",
        "commandLineParser.ts",
        "tsc.ts",
        "diagnosticInformationMap.generated.ts"
    ]
}
```

* 使用`"include"`和`"exclude"`屬性

  ```json
  {
      "compilerOptions": {
          "module": "system",
          "noImplicitAny": true,
          "removeComments": true,
          "preserveConstEnums": true,
          "outFile": "../../built/local/tsc.js",
          "sourceMap": true
      },
      "include": [
          "src/**/*"
      ],
      "exclude": [
          "node_modules",
          "**/*.spec.ts"
      ]
  }
  ```

## 細節

`"compilerOptions"`可以被忽略，這時編譯器會使用默認值。在這裡查看完整的[編譯器選項](./Compiler Options.md)列表。

`"files"`指定一個包含相對或絕對文件路徑的列表。
`"include"`和`"exclude"`屬性指定一個文件glob匹配模式列表。
支持的glob通配符有：

* `*` 匹配0或多個字符（不包括目錄分隔符）
* `?` 匹配一個任意字符（不包括目錄分隔符）
* `**/` 遞歸匹配任意子目錄

如果一個glob模式裡的某部分只包含`*`或`.*`，那麼僅有支持的文件擴展名類型被包含在內（比如默認`.ts`，`.tsx`，和`.d.ts`， 如果`allowJs`設置能`true`還包含`.js`和`.jsx`）。

如果`"files"`和`"include"`都沒有被指定，編譯器默認包含當前目錄和子目錄下所有的TypeScript文件（`.ts`, `.d.ts` 和 `.tsx`），排除在`"exclude"`裡指定的文件。JS文件（`.js`和`.jsx`）也被包含進來如果`allowJs`被設置成`true`。
如果指定了`"files"`或`"include"`，編譯器會將它們結合一併包含進來。
使用`"outDir"`指定的目錄下的文件永遠會被編譯器排除，除非你明確地使用`"files"`將其包含進來（這時就算用`exclude`指定也沒用）。

使用`"include"`引入的文件可以使用`"exclude"`屬性過濾。
然而，通過`"files"`屬性明確指定的文件卻總是會被包含在內，不管`"exclude"`如何設置。
如果沒有特殊指定，`"exclude"`默認情況下會排除`node_modules`，`bower_components`，`jspm_packages`和`<outDir>`目錄。

任何被`"files"`或`"include"`指定的文件所引用的文件也會被包含進來。
`A.ts`引用了`B.ts`，因此`B.ts`不能被排除，除非引用它的`A.ts`在`"exclude"`列表中。

需要注意編譯器不會去引入那些可能做為輸出的文件；比如，假設我們包含了`index.ts`，那麼`index.d.ts`和`index.js`會被排除在外。
通常來講，不推薦只有擴展名的不同來區分同目錄下的文件。

`tsconfig.json`文件可以是個空文件，那麼所有默認的文件（如上面所述）都會以默認配置選項編譯。

在命令行上指定的編譯選項會覆蓋在`tsconfig.json`文件裡的相應選項。

## `@types`，`typeRoots`和`types`

默認所有*可見的*"`@types`"包會在編譯過程中被包含進來。
`node_modules/@types`文件夾下以及它們子文件夾下的所有包都是*可見的*；
也就是說，`./node_modules/@types/`，`../node_modules/@types/`和`../../node_modules/@types/`等等。

如果指定了`typeRoots`，*只有*`typeRoots`下面的包才會被包含進來。
比如：

```json
{
   "compilerOptions": {
       "typeRoots" : ["./typings"]
   }
}
```

這個配置文件會包含*所有*`./typings`下面的包，而不包含`./node_modules/@types`裡面的包。

如果指定了`types`，只有被列出來的包才會被包含進來。
比如：

```json
{
   "compilerOptions": {
        "types" : ["node", "lodash", "express"]
   }
}
```

這個`tsconfig.json`文件將*僅會*包含  `./node_modules/@types/node`，`./node_modules/@types/lodash`和`./node_modules/@types/express`。/@types/。
`node_modules/@types/*`裡面的其它包不會被引入進來。

指定`"types": []`來禁用自動引入`@types`包。

注意，自動引入只在你使用了全局的聲明（相反於模塊）時是重要的。
如果你使用`import "foo"`語句，TypeScript仍然會查找`node_modules`和`node_modules/@types`文件夾來獲取`foo`包。

## 使用`extends`繼承配置

`tsconfig.json`文件可以利用`extends`屬性從另一個配置文件裡繼承配置。

`extends`是`tsconfig.json`文件裡的頂級屬性（與`compilerOptions`，`files`，`include`，和`exclude`一樣）。
`extends`的值是一個字符串，包含指向另一個要繼承文件的路徑。

在原文件裡的配置先被加載，然後被來至繼承文件裡的配置重寫。
如果發現循環引用，則會報錯。

來至所繼承配置文件的`files`，`include`和`exclude`*覆蓋*源配置文件的屬性。

配置文件裡的相對路徑在解析時相對於它所在的文件。

比如：

`configs/base.json`：

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true
  }
}
```

`tsconfig.json`：

```json
{
  "extends": "./configs/base",
  "files": [
    "main.ts",
    "supplemental.ts"
  ]
}
```

`tsconfig.nostrictnull.json`：

```json
{
  "extends": "./tsconfig",
  "compilerOptions": {
    "strictNullChecks": false
  }
}
```

## `compileOnSave`

在最頂層設置`compileOnSave`標記，可以讓IDE在保存文件的時候根據`tsconfig.json`重新生成文件。

```json
{
    "compileOnSave": true,
    "compilerOptions": {
        "noImplicitAny" : true
    }
}
```

要想支持這個特性需要Visual Studio 2015， TypeScript1.8.4以上並且安裝[atom-typescript](https://github.com/TypeStrong/atom-typescript#compile-on-save)插件。

## 模式

到這裡查看模式: [http://json.schemastore.org/tsconfig](http://json.schemastore.org/tsconfig).
