## 概述

編譯選項可以在使用MSBuild的項目裡通過MSBuild屬性指定。

## 例子

```XML
  <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TypeScriptRemoveComments>false</TypeScriptRemoveComments>
    <TypeScriptSourceMap>true</TypeScriptSourceMap>
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TypeScriptRemoveComments>true</TypeScriptRemoveComments>
    <TypeScriptSourceMap>false</TypeScriptSourceMap>
  </PropertyGroup>
  <Import
      Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets"
      Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets')" />
```

## 映射

編譯選項                                      | MSBuild屬性名稱                             | 可用值
---------------------------------------------|--------------------------------------------|-----------------
`--allowJs`                                  | *MSBuild不支持此選項*                        |
`--allowSyntheticDefaultImports`             | TypeScriptAllowSyntheticDefaultImports     | 布爾值
`--allowUnreachableCode`                     | TypeScriptAllowUnreachableCode             | 布爾值
`--allowUnusedLabels`                        | TypeScriptAllowUnusedLabels                | 布爾值
`--alwaysStrict`                             | TypeScriptAlwaysStrict                     | 布爾值
`--baseUrl`                                  | TypeScriptBaseUrl                          | 文件路徑
`--charset`                                  | TypeScriptCharset                          |
`--declaration`                              | TypeScriptGeneratesDeclarations            | 布爾值
`--declarationDir`                           | TypeScriptDeclarationDir                   | 文件路徑
`--diagnostics`                              | *MSBuild不支持此選項*                        |
`--disableSizeLimit`                         | *MSBuild不支持此選項*                        |
`--emitBOM`                                  | TypeScriptEmitBOM                          | 布爾值
`--emitDecoratorMetadata`                    | TypeScriptEmitDecoratorMetadata            | 布爾值
`--experimentalAsyncFunctions`               | TypeScriptExperimentalAsyncFunctions       | 布爾值
`--experimentalDecorators`                   | TypeScriptExperimentalDecorators           | 布爾值
`--forceConsistentCasingInFileNames`         | TypeScriptForceConsistentCasingInFileNames | 布爾值
`--help`                                     | *MSBuild不支持此選項*                        |
`--importHelpers`                            | TypeScriptImportHelpers                    | 布爾值
`--inlineSourceMap`                          | TypeScriptInlineSourceMap                  | 布爾值
`--inlineSources`                            | TypeScriptInlineSources                    | 布爾值
`--init`                                     | *MSBuild不支持此選項*                        |
`--isolatedModules`                          | TypeScriptIsolatedModules                  | 布爾值
`--jsx`                                      | TypeScriptJSXEmit                          | `React`或`Preserve`
`--jsxFactory`                               | TypeScriptJSXFactory                       | 有效的名字
`--lib`                                      | TypeScriptLib                              | 逗號分隔的字符串列表
`--listEmittedFiles`                         | *MSBuild不支持此選項*                        |
`--listFiles`                                | *MSBuild不支持此選項*                        |
`--locale`                                   | *automatic*                                | 自動設置為PreferredUILang值
`--mapRoot`                                  | TypeScriptMapRoot                          | 文件路徑
`--maxNodeModuleJsDepth`                     | *MSBuild不支持此選項*                        |
`--module`                                   | TypeScriptModuleKind                       | `AMD`，`CommonJs`，`UMD`，`System`或`ES6`
`--moduleResolution`                         | TypeScriptModuleResolution                 | `Classic`或`Node`
`--newLine`                                  | TypeScriptNewLine                          | `CRLF`或`LF`
`--noEmit`                                   | *MSBuild不支持此選項*                        |
`--noEmitHelpers`                            | TypeScriptNoEmitHelpers                    | 布爾值
`--noEmitOnError`                            | TypeScriptNoEmitOnError                    | 布爾值
`--noFallthroughCasesInSwitch`               | TypeScriptNoFallthroughCasesInSwitch       | 布爾值
`--noImplicitAny`                            | TypeScriptNoImplicitAny                    | 布爾值
`--noImplicitReturns`                        | TypeScriptNoImplicitReturns                | 布爾值
`--noImplicitThis`                           | TypeScriptNoImplicitThis                   | 布爾值
`--noImplicitUseStrict`                      | TypeScriptNoImplicitUseStrict              | 布爾值
`--noStrictGenericChecks`                    | TypeScriptNoStrictGenericChecks            | 布爾值
`--noUnusedLocals`                           | TypeScriptNoUnusedLocals                   | 布爾值
`--noUnusedParameters`                       | TypeScriptNoUnusedParameters               | 布爾值
`--noLib`                                    | TypeScriptNoLib                            | 布爾值
`--noResolve`                                | TypeScriptNoResolve                        | 布爾值
`--out`                                      | TypeScriptOutFile                          | 文件路徑
`--outDir`                                   | TypeScriptOutDir                           | 文件路徑
`--outFile`                                  | TypeScriptOutFile                          | 文件路徑
`--paths`                                    | *MSBuild不支持此選項*                        |
`--preserveConstEnums`                       | TypeScriptPreserveConstEnums               | 布爾值
`--preserveSymlinks`                         | TypeScriptPreserveSymlinks                 | 布爾值
`--listEmittedFiles`                         | *MSBuild不支持此選項*                        |
`--pretty`                                   | *MSBuild不支持此選項*                        |
`--reactNamespace`                           | TypeScriptReactNamespace                   | 字符串
`--removeComments`                           | TypeScriptRemoveComments                   | 布爾值
`--rootDir`                                  | TypeScriptRootDir                          | 文件路徑
`--rootDirs`                                 | *MSBuild不支持此選項*                        |
`--skipLibCheck`                             | TypeScriptSkipLibCheck                     | 布爾值
`--skipDefaultLibCheck`                      | TypeScriptSkipDefaultLibCheck              | 布爾值
`--sourceMap`                                | TypeScriptSourceMap                        | 文件路徑
`--sourceRoot`                               | TypeScriptSourceRoot                       | 文件路徑
`--strict`                                   | TypeScriptStrict                           | 布爾值
`--strictFunctionTypes`                      | TypeScriptStrictFunctionTypes              | 布爾值
`--strictNullChecks`                         | TypeScriptStrictNullChecks                 | 布爾值
`--stripInternal`                            | TypeScriptStripInternal                    | 布爾值
`--suppressExcessPropertyErrors`             |  TypeScriptSuppressExcessPropertyErrors    | 布爾值
`--suppressImplicitAnyIndexErrors`           | TypeScriptSuppressImplicitAnyIndexErrors   | 布爾值
`--target`                                   | TypeScriptTarget                           | `ES3`，`ES5`，或`ES6`
`--traceResolution`                          | *MSBuild不支持此選項*                        |
`--types`                                    | *MSBuild不支持此選項*                        |
`--typeRoots`                                | *MSBuild不支持此選項*                        |
`--watch`                                    | *MSBuild不支持此選項*                        |
*MSBuild only option*                        | TypeScriptAdditionalFlags                  | *任何編譯選項*

## 我使用的Visual Studio版本裡支持哪些選項?

查找 `C:\Program Files (x86)\MSBuild\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets` 文件。
可用的MSBuild XML標籤與相應的`tsc`編譯選項的映射都在那裡。

## ToolsVersion

工程文件裡的`<TypeScriptToolsVersion>1.7</TypeScriptToolsVersion>`屬性值表明了構建時使用的編譯器的版本號（這個例子裡是1.7）
這樣就允許一個工程在不同的機器上使用相同版本的編譯器進行構建。

如果沒有指定`TypeScriptToolsVersion`，則會使用機器上安裝的最新版本的編譯器去構建。

如果用戶使用的是更新版本的TypeScript，則會在首次加載工程的時候看到一個提示升級工程的對話框。

## TypeScriptCompileBlocked

如果你使用其它的構建工具（比如，gulp， grunt等等）並且使用VS做為開發和調試工具，那麼在工程裡設置`<TypeScriptCompileBlocked>true</TypeScriptCompileBlocked>`。
這樣VS只會提供給你編輯的功能，而不會在你按F5的時候去構建。
