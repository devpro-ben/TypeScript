讓我們使用TypeScript來創建一個簡單的Web應用程式。

## 安裝TypeScript

有兩種主要的方式來獲取TypeScript工具：

* 通過npm（Node.js包管理器）
* 安裝Visual Studio的TypeScript插件

Visual Studio 2017和Visual Studio 2015 Update 3預設包含了TypeScript。
如果你的Visual Studio還沒有安裝TypeScript，你可以[下載](/#download-links)它。

針對使用npm的用戶：

```shell
> npm install -g typescript
```

## 構建你的第一個TypeScript檔案

在編輯器，將下面的代碼輸入到`greeter.ts`文件裡：

```ts
function greeter(person) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.innerHTML = greeter(user);
```

## 編譯代碼

我們使用了`.ts`副檔名，但是這段代碼僅僅是JavaScript而已。
你可以直接從現有的JavaScript應用裡複製/貼上這段代碼。

在命令行上，運行TypeScript編譯器：

```shell
tsc greeter.ts
```

輸出結果為一個`greeter.js`文件，它包含了和輸入檔案中相同的JavsScript代碼。
一切準備就緒，我們可以運行這個使用TypeScript寫的JavaScript應用程式了！

接下來讓我們看看TypeScript工具帶來的進階功能。
給`person`函數的參數添加`: string`型別註解，如下：

```ts
function greeter(person: string) {
    return "Hello, " + person;
}

let user = "Jane User";

document.body.innerHTML = greeter(user);
```

## 型別註解

TypeScript裡的型別註解是一種輕量級的為函數或變數添加約束的方式。
在這個例子裡，我們希望`greeter`函數接收一個字串參數。
然後嘗試把`greeter`的呼叫改成傳入一個陣列：

```ts
function greeter(person: string) {
    return "Hello, " + person;
}

let user = [0, 1, 2];

document.body.innerHTML = greeter(user);
```

重新編譯，你會看到產生了一個錯誤。

```shell
error TS2345: Argument of type 'number[]' is not assignable to parameter of type 'string'.
```

類似地，嘗試刪除`greeter`呼叫的所有參數。
TypeScript會告訴你使用了非期望個數的參數呼叫了這個函數。
在這兩種情況中，TypeScript提供了靜態的代碼分析，它可以分析代碼結構和提供的型別註解。

要注意的是儘管有錯誤，`greeter.js`文件還是被創建了。
就算你的代碼裡有錯誤，你仍然可以使用TypeScript。但在這種情況下，TypeScript會警告你代碼可能不會按預期執行。

## 介面

讓我們開發這個範例應用程式。這裡我們使用介面來描述一個擁有`firstName`和`lastName`欄位的物件。
在TypeScript裡，只在兩個型別內部的結構相容那麼這兩個型別就是相容的。
這就允許我們在實現介面時候只要保證包含了介面要求的結構就可以，而不必明確地使用`implements`敘述。

```ts
interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person: Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = { firstName: "Jane", lastName: "User" };

document.body.innerHTML = greeter(user);
```

## 類別

最後，讓我們使用類別來改寫這個例子。
TypeScript支持JavaScript的新特性，比如支持基於類別的物件導向編程。

讓我們創建一個`Student`類別，它帶有一個建構函數和一些公開欄位。
注意類別和介面可以一起工作，程序員可以自行決定抽象的層級。

還要注意的是，在建構函數的參數上使用`public`等同於創建了同名的成員變數。

```ts
class Student {
    fullName: string;
    constructor(public firstName: string, public middleInitial: string, public lastName: string) {
        this.fullName = firstName + " " + middleInitial + " " + lastName;
    }
}

interface Person {
    firstName: string;
    lastName: string;
}

function greeter(person : Person) {
    return "Hello, " + person.firstName + " " + person.lastName;
}

let user = new Student("Jane", "M.", "User");

document.body.innerHTML = greeter(user);
```

重新運行`tsc greeter.ts`，你會看到生成的JavaScript代碼和原先的一樣。
TypeScript裡的類別只是JavaScript裡常用的基於原型物件導向編程的簡寫。

## 運行TypeScript Web應用程式

在`greeter.html`裡輸入如下內容：

```html
<!DOCTYPE html>
<html>
    <head><title>TypeScript Greeter</title></head>
    <body>
        <script src="greeter.js"></script>
    </body>
</html>
```

在瀏覽器裡打開`greeter.html`運行這個應用程式！

可選地：在Visual Studio裡打開`greeter.ts`或者把代碼複製到TypeScript playground。
將滑鼠指標懸停在識別字上查看它們的型別。
注意在某些情況下它們的型別可以被自動地推斷出來。
重新輸入一下最後一行代碼，看一下自動補全列表和參數列表，它們會根據DOM元素型別而變化。
將光標放在`greeter`函數上，點擊F12可以跟蹤到它的定義。
還有一點，你可以右鍵點擊識別字，使用重構功能來重命名。

這些型別資訊以及工具可以很好的和JavaScript一起工作。
更多的TypeScript功能演示，請查看本網站的範例部分。

![Visual Studio picture](https://www.typescriptlang.org/assets/images/docs/greet_person.png)
