# TypeScript

[![Build Status](https://travis-ci.org/zhongsp/TypeScript.svg?branch=master)](https://travis-ci.org/zhongsp/TypeScript) [![PRs Welcome](https://img.shields.io/badge/PRs-welcome-brightgreen.svg?style=flat-square)](http://makeapullrequest.com)

<img src="./misc/ts_logo.jpg" alt="TypeScript" width="24px" height="24px" style="vertical-align: bottom;">  [TypeScript 3.1 (September 27, 2018)](https://blogs.msdn.microsoft.com/typescript/2018/09/27/announcing-typescript-3-1/)
|
[版本發佈說明](./doc/release-notes/TypeScript%203.1.md)

:heavy_check_mark: TypeScript語言用於大規模應用的JavaScript開發。  :heavy_check_mark: TypeScript支持類型，是JavaScript的超集且可以編譯成純JavaScript代碼。  :heavy_check_mark: TypeScript兼容所有瀏覽器，所有宿主環境，所有操作系統。  :heavy_check_mark: TypeScript是開源的。

:book: [在GitBook網站上閱讀本手冊](http://zhongsp.gitbooks.io/typescript-handbook/content/)  :arrow_down: [下載本手冊 PDF 版](https://legacy.gitbook.com/download/pdf/book/zhongsp/typescript-handbook)  :arrow_down: [下載本手冊 Mobi 版](https://legacy.gitbook.com/download/mobi/book/zhongsp/typescript-handbook)  :arrow_down: [下載本手冊 ePub 版](https://legacy.gitbook.com/download/epub/book/zhongsp/typescript-handbook)

:link: [一大波新的快速開始指南：React，Angular，Nodejs，ASP.NET Core，React Native，Vue，Glimmer，WeChat，Dojo2，Knockout等](./doc/quick-start/README.md)

<img src="./misc/reward.jpg" alt="Reward the Author" width="300px" height="300px" style="vertical-align: bottom;">  如果覺得不錯可以微信打賞喲 <3

## 目錄

* [快速上手](./doc/handbook/tutorials/README.md)
  * [5分鐘瞭解TypeScript](./doc/handbook/tutorials/TypeScript%20in%205%20minutes.md)
  * [ASP.NET Core](./doc/handbook/tutorials/ASP.NET%20Core.md)
  * [ASP.NET 4](./doc/handbook/tutorials/ASP.NET%204.md)
  * [Gulp](./doc/handbook/tutorials/Gulp.md)
  * [Knockout.js](./doc/handbook/tutorials/Knockout.md)
  * [React與webpack](./doc/handbook/tutorials/React%20&%20Webpack.md)
  * [React](./doc/handbook/tutorials/React.md)
  * [Angular 2](./doc/handbook/tutorials/Angular%202.md)
  * [從JavaScript遷移到TypeScript](./doc/handbook/tutorials/Migrating%20from%20JavaScript.md)
* [手冊](./doc/handbook/README.md)
  * [基礎型別](./doc/handbook/Basic%20Types.md)
  * [變數宣告](./doc/handbook/Variable%20Declarations.md)
  * [介面](./doc/handbook/Interfaces.md)
  * [類別](./doc/handbook/Classes.md)
  * [函數](./doc/handbook/Functions.md)
  * [泛型](./doc/handbook/Generics.md)
  * [列舉](./doc/handbook/Enums.md)
  * [型別推論](./doc/handbook/Type%20Inference.md)
  * [型別相容性](./doc/handbook/Type%20Compatibility.md)
  * [高級型別](./doc/handbook/Advanced%20Types.md)
  * [實用工具型別](./doc/handbook/Utility%20Types.md)
  * [Symbols](./doc/handbook/Symbols.md)
  * [Iterators 和 Generators](./doc/handbook/Iterators%20and%20Generators.md)
  * [模組](./doc/handbook/Modules.md)
  * [命名空間](./doc/handbook/Namespaces.md)
  * [命名空間和模塊](./doc/handbook/Namespaces%20and%20Modules.md)
  * [模塊解析](./doc/handbook/Module%20Resolution.md)
  * [聲明合併](./doc/handbook/Declaration%20Merging.md)
  * [書寫.d.ts文件](./doc/handbook/Writing%20Definition%20Files.md)
  * [JSX](./doc/handbook/JSX.md)
  * [Decorators](./doc/handbook/Decorators.md)
  * [混入](./doc/handbook/Mixins.md)
  * [三斜線指令](./doc/handbook/Triple-Slash%20Directives.md)
  * [JavaScript文件裡的類型檢查](./doc/handbook/Type%20Checking%20JavaScript%20Files.md)
* [如何書寫聲明文件](./doc/handbook/declaration%20files/Introduction.md)
  * [結構](./doc/handbook/declaration%20files/Library%20Structures.md)
  * [規範](./doc/handbook/declaration%20files/Do's%20and%20Don'ts.md)
  * [舉例](./doc/handbook/declaration%20files/By%20Example.md)
  * [深入](./doc/handbook/declaration%20files/Deep%20Dive.md)
  * [發佈](./doc/handbook/declaration%20files/Publishing.md)
  * [使用](./doc/handbook/declaration%20files/Consumption.md)
* [工程配置](./doc/handbook/tsconfig.json.md)
  * [tsconfig.json](./doc/handbook/tsconfig.json.md)
  * [工程引用](./doc/handbook/Project%20References.md)
  * [NPM包的類型](./doc/handbook/Typings%20for%20NPM%20Packages.md)
  * [編譯選項](./doc/handbook/Compiler%20Options.md)
  * [配置 Watch](./doc/handbook/Configuring%20Watch.md)
  * [在MSBuild裡使用編譯選項](./doc/handbook/Compiler%20Options%20in%20MSBuild.md)
  * [與其它構建工具整合](./doc/handbook/Integrating%20with%20Build%20Tools.md)
  * [使用TypeScript的每日構建版本](./doc/handbook/Nightly%20Builds.md)
* [Wiki](./doc/wiki/README.md)
  * [TypeScript裡的this](./doc/wiki/this-in-TypeScript.md)
  * [編碼規範](./doc/wiki/coding_guidelines.md)
  * [常見編譯錯誤](./doc/wiki/Common%20Errors.md)
  * [支持TypeScript的編輯器](./doc/wiki/TypeScript-Editor-Support.md)
  * [結合ASP.NET v5使用TypeScript](./doc/wiki/Using-TypeScript-With-ASP.NET-5.md)
  * [架構概述](./doc/wiki/Architectural-Overview.md)
  * [發展路線圖](./doc/wiki/Roadmap.md)
* [新增功能](./doc/release-notes/README.md)
  * [TypeScript 3.1](./doc/release-notes/TypeScript%203.1.md)
  * [TypeScript 3.0](./doc/release-notes/TypeScript%203.0.md)
  * [TypeScript 2.9](./doc/release-notes/TypeScript%202.9.md)
  * [TypeScript 2.8](./doc/release-notes/TypeScript%202.8.md)
  * [TypeScript 2.7](./doc/release-notes/TypeScript%202.7.md)
  * [TypeScript 2.6](./doc/release-notes/TypeScript%202.6.md)
  * [TypeScript 2.5](./doc/release-notes/TypeScript%202.5.md)
  * [TypeScript 2.4](./doc/release-notes/TypeScript%202.4.md)
  * [TypeScript 2.3](./doc/release-notes/TypeScript%202.3.md)
  * [TypeScript 2.2](./doc/release-notes/TypeScript%202.2.md)
  * [TypeScript 2.1](./doc/release-notes/TypeScript%202.1.md)
  * [TypeScript 2.0](./doc/release-notes/TypeScript%202.0.md)
  * [TypeScript 1.8](./doc/release-notes/TypeScript%201.8.md)
  * [TypeScript 1.7](./doc/release-notes/TypeScript%201.7.md)
  * [TypeScript 1.6](./doc/release-notes/TypeScript%201.6.md)
  * [TypeScript 1.5](./doc/release-notes/TypeScript%201.5.md)
  * [TypeScript 1.4](./doc/release-notes/TypeScript%201.4.md)
  * [TypeScript 1.3](./doc/release-notes/TypeScript%201.3.md)
  * [TypeScript 1.1](./doc/release-notes/TypeScript%201.1.md)
* [Breaking Changes](./doc/breaking-changes/breaking-changes.md)
  * [TypeScript 3.1](./doc/breaking-changes/TypeScript%203.1.md)
  * [TypeScript 2.8](./doc/breaking-changes/TypeScript%202.8.md)
  * [TypeScript 2.7](./doc/breaking-changes/TypeScript%202.7.md)
  * [TypeScript 2.6](./doc/breaking-changes/TypeScript%202.6.md)
  * [TypeScript 2.4](./doc/breaking-changes/TypeScript%202.4.md)
  * [TypeScript 2.3](./doc/breaking-changes/TypeScript%202.3.md)
  * [TypeScript 2.2](./doc/breaking-changes/TypeScript%202.2.md)
  * [TypeScript 2.1](./doc/breaking-changes/TypeScript%202.1.md)
  * [TypeScript 2.0](./doc/breaking-changes/TypeScript%202.0.md)
  * [TypeScript 1.8](./doc/breaking-changes/TypeScript%201.8.md)
  * [TypeScript 1.7](./doc/breaking-changes/TypeScript%201.7.md)
  * [TypeScript 1.6](./doc/breaking-changes/TypeScript%201.6.md)
  * [TypeScript 1.5](./doc/breaking-changes/TypeScript%201.5.md)
  * [TypeScript 1.4](./doc/breaking-changes/TypeScript%201.4.md)

**TypeScript Handbook**

* Read [TypeScript Handbook (Recommended, BUT not up to date officially)](http://www.typescriptlang.org/Handbook)
* Read [TypeScript手冊中文版 - Published with GitBook（持續更新中，最新版）](http://zhongsp.gitbooks.io/typescript-handbook/content/):book:

**TypeScript Language Specification**

* Read [TypeScript Language Specification](https://github.com/Microsoft/TypeScript/blob/master/doc/spec.md)

I'd love for you to contribute to the translation:)