在太平洋標準時間每天午夜會自動構建[TypeScript的`master`](https://github.com/Microsoft/TypeScript/tree/master)分支代碼並發布到NPM和NuGet上。
下面將介紹如何獲得並在工具裡使用它們。

## 使用 npm

```shell
npm install -g typescript@next
```

## 使用 NuGet 和 MSBuild

> 注意：你需要配置工程來使用NuGet包。
詳細信息參考[配置MSBuild工程來使用NuGet](https://github.com/Microsoft/TypeScript/wiki/Configuring-MSBuild-projects-to-use-NuGet)。

[www.myget.org](https://www.myget.org/gallery/typescript-preview)。

有兩個包：

* `Microsoft.TypeScript.Compiler`: 僅包含工具 (`tsc.exe`，`lib.d.ts`，等。) 。
* `Microsoft.TypeScript.MSBuild`: 和上面一樣的工具，還有MSBuild的任務和目標(`Microsoft.TypeScript.targets`，`Microsoft.TypeScript.Default.props`，等。)

## 更新IDE來使用每日構建

你還可以配置IDE來使用每日構建。
首先你要通過npm安裝包。
你可以進行全局安裝或者安裝到本地的`node_modules`目錄下。

下面的步驟裡我們假設你已經安裝好了`typescript@next`。

### Visual Studio Code

更新`.vscode/settings.json`如下：

```json
"typescript.tsdk": "<path to your folder>/node_modules/typescript/lib"
```

詳細信息參見[VSCode文檔](https://code.visualstudio.com/Docs/languages/typescript#_using-newer-typescript-versions)。

### Sublime Text

更新`Settings - User`如下：

```json
"typescript_tsdk": "<path to your folder>/node_modules/typescript/lib"
```

詳細信息參見[如何在Sublime Text裡安裝TypeScript插件](https://github.com/Microsoft/TypeScript-Sublime-Plugin#installation)。

### Visual Studio 2013 and 2015

> 注意：大多數的改變不需要你安裝新版本的VS TypeScript插件。

當前的每日構建不包含完整的插件安裝包，但是我們正在試著提供每日構建的安裝包。

1. 下載[VSDevMode.ps1](https://github.com/Microsoft/TypeScript/blob/master/scripts/VSDevMode.ps1)腳本。

   > 參考wiki文檔：[使用自定義語言服務文件](https://github.com/Microsoft/TypeScript/wiki/Dev-Mode-in-Visual-Studio#using-a-custom-language-service-file)。

2. 在PowerShell命令行窗口裡執行：

  VS 2015：

  ```posh
  VSDevMode.ps1 14 -tsScript <path to your folder>/node_modules/typescript/lib
  ```

  VS 2013：

  ```posh
  VSDevMode.ps1 12 -tsScript <path to your folder>/node_modules/typescript/lib
  ```

### IntelliJ IDEA (Mac)

前往`Preferences` > `Languages & Frameworks` > `TypeScript`：
 > TypeScript Version: 如果通過NPM安裝：`/usr/local/lib/node_modules/typescript/lib`
 