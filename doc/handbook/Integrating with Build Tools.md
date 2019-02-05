Build tools

* [Browserify](#browserify)
* [Duo](#duo)
* [Grunt](#grunt)
* [Gulp](#gulp)
* [Jspm](#jspm)
* [Webpack](#webpack)
* [MSBuild](#msbuild)
* [NuGet](#nuget)

# Browserify

### 安裝

```sh
npm install tsify
```

### 使用命令行交互

```sh
browserify main.ts -p [ tsify --noImplicitAny ] > bundle.js
```

### 使用API

```js
var browserify = require("browserify");
var tsify = require("tsify");

browserify()
    .add('main.ts')
    .plugin('tsify', { noImplicitAny: true })
    .bundle()
    .pipe(process.stdout);
```

更多詳細信息：[smrq/tsify](https://github.com/smrq/tsify)

# Duo

### 安裝

```sh
npm install duo-typescript
```

### 使用命令行交互

```sh
duo --use duo-typescript entry.ts
```

### 使用API

```js
var Duo = require('duo');
var fs = require('fs')
var path = require('path')
var typescript = require('duo-typescript');

var out = path.join(__dirname, "output.js")

Duo(__dirname)
    .entry('entry.ts')
    .use(typescript())
    .run(function (err, results) {
        if (err) throw err;
        // Write compiled result to output file
        fs.writeFileSync(out, results.code);
    });
```

更多詳細信息：[frankwallis/duo-typescript](https://github.com/frankwallis/duo-typescript)

# Grunt

### 安裝

```sh
npm install grunt-ts
```

### 基本Gruntfile.js

````js
module.exports = function(grunt) {
    grunt.initConfig({
        ts: {
            default : {
                src: ["**/*.ts", "!node_modules/**/*.ts"]
            }
        }
    });
    grunt.loadNpmTasks("grunt-ts");
    grunt.registerTask("default", ["ts"]);
};
````

更多詳細信息：[TypeStrong/grunt-ts](https://github.com/TypeStrong/grunt-ts)

# Gulp

### 安裝

```sh
npm install gulp-typescript
```

### 基本gulpfile.js

```js
var gulp = require("gulp");
var ts = require("gulp-typescript");

gulp.task("default", function () {
    var tsResult = gulp.src("src/*.ts")
        .pipe(ts({
              noImplicitAny: true,
              out: "output.js"
        }));
    return tsResult.js.pipe(gulp.dest('built/local'));
});
```

更多詳細信息：[ivogabe/gulp-typescript](https://github.com/ivogabe/gulp-typescript)

# Jspm

### 安裝

```sh
npm install -g jspm@beta
```

_注意：目前jspm的0.16beta版本支持TypeScript_

更多詳細信息：[TypeScriptSamples/jspm](https://github.com/Microsoft/TypeScriptSamples/tree/master/jspm)

# Webpack

### 安裝

```sh
npm install ts-loader --save-dev
```

### 基本webpack.config.js

```js
module.exports = {
    entry: "./src/index.tsx",
    output: {
        filename: "bundle.js"
    },
    resolve: {
        // Add '.ts' and '.tsx' as a resolvable extension.
        extensions: ["", ".webpack.js", ".web.js", ".ts", ".tsx", ".js"]
    },
    module: {
        loaders: [
            // all files with a '.ts' or '.tsx' extension will be handled by 'ts-loader'
            { test: /\.tsx?$/, loader: "ts-loader" }
        ]
    }
};
```

查看[更多關於ts-loader的詳細信息](https://www.npmjs.com/package/ts-loader)

或者

* [awesome-typescript-loader](https://www.npmjs.com/package/awesome-typescript-loader)

# MSBuild

更新工程文件，包含本地安裝的`Microsoft.TypeScript.Default.props`（在頂端）和`Microsoft.TypeScript.targets`（在底部）文件：

```xml
<?xml version="1.0" encoding="utf-8"?>
<Project ToolsVersion="4.0" DefaultTargets="Build" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
  <!-- Include default props at the top -->
  <Import
      Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.Default.props"
      Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.Default.props')" />

  <!-- TypeScript configurations go here -->
  <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
    <TypeScriptRemoveComments>false</TypeScriptRemoveComments>
    <TypeScriptSourceMap>true</TypeScriptSourceMap>
  </PropertyGroup>
  <PropertyGroup Condition="'$(Configuration)' == 'Release'">
    <TypeScriptRemoveComments>true</TypeScriptRemoveComments>
    <TypeScriptSourceMap>false</TypeScriptSourceMap>
  </PropertyGroup>

  <!-- Include default targets at the bottom -->
  <Import
      Project="$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets"
      Condition="Exists('$(MSBuildExtensionsPath32)\Microsoft\VisualStudio\v$(VisualStudioVersion)\TypeScript\Microsoft.TypeScript.targets')" />
</Project>
```

關於配置MSBuild編譯器選項的更多詳細信息，請參考：[在MSBuild裡使用編譯選項](./Compiler Options in MSBuild.md)

# NuGet

* 右鍵點擊 -> Manage NuGet Packages
* 查找`Microsoft.TypeScript.MSBuild`
* 點擊`Install`
* 安裝完成後，Rebuild。

更多詳細信息請參考[Package Manager Dialog](http://docs.nuget.org/Consume/Package-Manager-Dialog)和[using nightly builds with NuGet](https://github.com/Microsoft/TypeScript/wiki/Nightly-drops#using-nuget-with-msbuild)
