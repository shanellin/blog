---
title: "重新認識 Javascript（二）- 作用域鏈"
categories: ["Frontend"]
tags: ["Javascript"]
Keywords: ["作用域", "Scope", "作用域規則", "Scope Rules", "作用域鏈", "Scope Chain", "變數提升", "Hoisting", "靜態作用域", "詞法作用域"]
images: ["/images/avatars.png"]
date: 2023-09-02T15:09:33+08:00
draft: false
---

## 前言

---

在前文介紹完 `Js Engine`、`Js Runtime` 後，我們對程式碼從 `解析` 到 `執行` 的過程有了初步的了解。然而，在執行階段中，仍然存在一些問題：何時定義的變量和函式、函式間獲取變量的規則、如何確保程式碼按照正確的順序執行...等。

為了解答上述問題，我們需要深入了解 `作用域鏈`、`執行上下文棧`、`原型鏈` 及 `this` 的概念。這些概念彼此關聯，共同構成了程式碼在運行過程中的基礎機制。本篇將從 `作用域鏈` 開始，逐步揭示它們之間的聯繫與作用。

本篇涉及了三個知識點：`作用域`、`作用域規則`、`作用域鏈`，以下將依序說明。

## 作用域（Scope）

---

首先讓我們看看 {{<NewTabLink href="https://developer.mozilla.org/zh-CN/docs/Glossary/Scope" title="MDN Web">}} 對於 `作用域（Scope）` 的解釋：

> 作用域是當前的執行上下文，值和表達式在其中“可見”或可被訪問。如果一個變量或表達式不在當前的作用域中，那麼它是不可用的。作用域也可以堆疊成層次結構，子作用域可以訪問父作用域，反過來則不行。

好像不是很好懂。

> 其實作用域是一個概念，可以簡單想成是在闡述一個家的內部狀況，`變量、函式、類、對象` 都可能是這個家的成員，只有這個家知道裡面會有哪些成員可以訪問，反之外人若無透過其他方式跟這個家建立聯繫，是無從得知的。

以上就是所謂的 `作用域決定一個變量的可見範圍`。接下來讓我們根據 `確定時機`、`作用範圍` 對作用域這個概念進行說明：

### 一、確定時機

目前的程式語言可以簡單分成兩種作用域：

1. 靜態作用域：又稱詞法作用域，指的是作用域在 `定義的時候` 就確定好了，也就是 {{<NewTabLink href="/articles/know-js-again-1-js-engine-runtime/#一解析器parser" title="JS Engine 的解析階段">}}。

2. 動態作用域：指的是作用域在 `調用的時候` 才會確定。

而 JS 採用的是 `靜態作用域`，這使得 JS 容易分析、調試及維護，除了未來會講到的 `this`。

### 二、作用範圍

可分成三種：

1. 全局作用域（Global Scope）：整個程式的最外層作用域，它包含了所有非嵌套的程式碼。在全局作用域中定義的變量都可以在任何位置被訪問。

2. 函式作用域（Function Scope）：各函式本身的內部範圍。根據 [作用域規則](/articles/know-js-again-2-scope-chain/#作用域規則scope-rules)，在函式內部定義的變量只能在函式內部訪問，函式外部無法直接訪問。若訪問函式內部不存在的變量則會向上層尋找。

3. 區塊作用域（Block Scope）：ES6 提供的 `let` 、`const` 宣告的變數，或一對 `{}`（一個程式碼塊）創建出來的作用域，如：if、for、while 語句的花括號內。

以下方程式碼為例：

```javascript
// 全局作用域
var person = "Potter";

function hello() {
  // 函式作用域
  console.log(`Hello, ${person}`);
}

if (person === "Potter") {
  // 區塊作用域
  let girl = "Hermione";
  hello();
}
console.log(`Hello, ${girl}`); // girl is not defined
```

### 三、變量提升（Hoisting）

說到區塊作用域就不得不提容易讓開發者掉入陷阱的 `變數提升（Hoisting）`。

變數提升是 JS 特有的行為，指的是在執行程式碼前，`Js Engine` 會將變數和函式的聲明提升到它們所在作用域的頂部。這就意味著即使程式碼的實際位置是在聲明之後，在執行之前，也可以訪問這些變數和函式。這也是 `var` 為何會被人所詬病的原因之一。

以下方程式碼為例：

```javascript
console.log(x); // undefined
var x = 10;
console.log(x); // 10

foo(); // "Hello, world!"
function foo() {
  console.log("Hello, world!");
}
```

但在 ES6 提供 `let` 、`const` 後這個狀況有了好轉，搭配區塊作用域，開發者能主動避免 `變量提升` 的狀況發生。

以下方程式碼為例：

```javascript
console.log(x); // x is not defined
let x = 10;
console.log(x); // 10

foo(); // foo is not defined
let foo = () => {
  console.log("Hello, world!");
};
```

## 作用域規則（Scope Rules）

---

> 如果將 `作用域` 比喻成一個家，那 `作用域規則` 就好比是一個地區的地址編排規則，想找到特定地址就必須跟著這個規則走。

也就是 {{<NewTabLink href="/articles/know-js-again-1-js-engine-runtime/#一解析器parser" title="JS Engine 的解析階段">}} 中確定 `變量、函式、類、對象` 在程式碼中的查找及訪問方式。

### ㄧ、常見的作用域規則

1. 變量查找：當在作用域中調用一個變量時，會先在當前作用域中查找該變量。如果在當前作用域中找不到變量，那麼會逐級向上層作用域查找，直到找到該變量或達到全局作用域。

2. 作用域嵌套：內部作用域可以訪問外部作用域中的變量，但外部作用域無法訪問內部作用域的變量。

   ```javascript
   function outerFn() {
     function innerFn() {
       var innerText = "outerText";
     }
     console.log(innerText); // innerText is not defined
   }
   outerFn();
   ```

3. 內部變量優先：如果在內部作用域聲明了一個與外部作用域同名的變量，那麼內部作用域的變量會 `優先於` 外部作用域的同名變量。在內部作用域中訪問該變量時，將優先使用內部作用域的變量。

4. 閉包：函式內部的作用域可以訪問函式外部的作用域，形成了閉包。這使得函式可以 `記住` 在其外部作用域中聲明的變量，即使外部作用域被回收了，也不會忘記。但對應的 [作用域鏈](/articles/know-js-again-2-scope-chain/#作用域鏈scope-chain) 也無法被回收，這也是為何閉包容易造成記憶體洩漏的原因。

   ```javascript
   function outerFn() {
     var outerText = "outerText";
     function innerFn() {
       console.log(outerText); // 在內部函式中訪問外部函式的變量
     }
     return innerFn; // 返回內部函式
   }
   var closureFn = outerFn(); // 調用外部函式並將返回的內部函式存儲在變量中
   closureFn(); // 調用內部函式，輸出 "outerText"
   ```

### 二、示例說明

```javascript
var person = "Potter";
function hello() {
  console.log(`Hello, ${person}`);
}
function sayHello() {
  var person = "Ronald";
  hello();
  console.log(`Bye, ${person}`);
}
sayHello();
// Hello, Potter
// Bye, Ronald
```

為何輸出結果會是 `Hello, Potter` 然後接 `Bye, Ronald` 呢？讓我們按照步驟說明：

1. 首先在全局作用域中，定義了一個值為 "Potter" 的變量 person。

2. 定義函式 hello，根據 `變量查找規則`，分析時發現函式作用域中並無變量 person，因此向上層作用域（即全局作用域）查找，最終找到 "Potter"。（注意：是定義時的上層，而不是被呼叫時的上層）。

3. 定義函式 sayHello，在函式作用域中定義了一個同名變量 person 並賦值為 "Ronald"，接著調用函式 hello。

4. 雖然 sayHello 重新定義了變量 person，但根據靜態作用域，函式在定義時就已經確定，不受調用位置的影響，因此 hello 最終輸出 `Hello, Potter`。

5. 執行 console.log，根據 `內部變量優先規則` 輸出 `Bye, Ronald`。

若 JS 變成動態作用域，那結果就會是 `Hello, Ronald` 和 `Bye, Ronald`。

## 作用域鏈（Scope Chain）

---

> 作用域鏈就好比是一台 GPS，它記錄了所有地區的地址及規則，幫助你找到不同地區的訊息。

當函式作用域內找不到變數時，會不停地向上層作用域尋找變數，最終若在全局作用域中也沒找到就會拋出錯誤。這種向上層作用域尋找的行為就是靠 `作用域鏈（Scope Chain）` 做到的。

作用域鏈是 `作用域規則` 在程式執行階段時的實現，雖然 JS 是靜態作用域，大部分的情況可以透過 `詞法分析`、`語法分析` 去判斷 `作用域規則`，但對於那些非固定調用和執行順序的函式而言，解釋階段並不能夠確定所有情境，比如：

1. 條件式調用：JS 允許在運行時根據條件決定調用哪個函式，甚至可以在函式內部動態地創建函式。因此，作用域鏈需要根據函式的調用位置和調用順序來動態地創建，以便在函式內部能夠正確地訪問變量。

2. 閉包：閉包之所以可以 `記住` 其外部作用域中聲明的變量，是因為閉包的作用域鏈會包含自身及包裹它的外部函式的變量。因此，作用域鏈需要等到閉包被創建後才能被確定。

3. 動態作用域：也就是上面提及的 `this`，JS 可以通過 .call()、.apply() 或 .bind() 方法改變 `this` 指向，從而影響函式內部的作用域鏈。

> 當初在比較 `作用域規則` 和 `作用域鏈` 時自己也很困惑，兩者不都做一樣的事情嗎？
>
> 其實 `作用域規則` 和 `作用域鏈` 確實是同一個東西來著的，只是解釋階段的 `作用域規則` 是根據詞法所生成，並無法完全確定執行階段的邏輯，所以在執行階段時會先複製解釋階段的 `作用域規則`，接著根據環境生成變量對象，最後完善自身的 `作用域鏈`。

所以函式在解釋、執行階段中都會用 `作用域鏈 [[scopes]]` 紀錄與自身相關的父級函式的作用域，如以下程式碼：

```javascript
// console.dir() 可用來顯示指定對象的屬性列表。

function fn1() {
  let a = 1;
  function fn2() {
    let b = 2;
    if (showfn3) {
      function fn3() {
        let c = 3;
        console.log(a, b, c);
      }
      fn3();
      console.dir(fn3);
    }
  }
  fn2();
  console.dir(fn2);
}
var showfn3 = true;
fn1();
console.dir(fn1);
```

讓我們按照步驟說明：

解釋階段：

1. 創建全局作用域：初始化全局作用域的 `[[Scopes]]`。

2. 創建 fn1 函式：初始化 fn1 函式作用域的 `[[Scopes]]`，並引入全局作用域的 `[[Scopes]]`。

3. 創建 fn2 函式：初始化 fn2 函式作用域的 `[[Scopes]]`，並引入 fn1 函式作用域的 `[[Scopes]]`。

執行階段（Execution Phase）：

1. 執行 fn1 函式：進入 fn1 函式，呼叫 fn2() 函式。

2. 執行 fn2 函式：進入 fn2 函式，根據 `變量查找規則` 從全局作用域得知 `showfn3 === true`，調用 fn3 函式 。

3. 創建 fn3 函式：初始化 fn3 函式作用域的 `[[Scopes]]`，並引入 fn2 函式作用域的 `[[Scopes]]`。

4. 執行 fn3 函式。

因此輸出結果如下：

```javascript
// console.dir(fn3);
ƒ fn3()
├─ ...
├─ arguments: null
├─ name: "fn3"
└─ [[Scopes]]:
   ├─ Closure (fn2) {b: 2}
   ├─ Closure (fn1) {a: 1}
   └─ Global

// console.dir(fn2);
ƒ fn2()
├─ ...
├─ arguments: null
├─ name: "fn2"
└─ [[Scopes]]:
   ├─ Closure (fn1) {a: 1}
   └─ Global

// console.dir(fn1);
ƒ fn1()
├─ ...
├─ arguments: null
├─ name: "fn1"
└─ [[Scopes]]:
   └─ Global
```

最後附上 fn3 透過 `作用域鏈` 取得 `a, b , c` 的簡略流程圖：

![js-scope-chain](/images/js/js-scope-chain.png)

## 閉包（Closure）

---

說完 `作用域鏈` 讓我們來講講 `閉包`，雖然它聽起來不是很重要，但日常開發時卻經常會用到。

### 一、什麼是閉包？

`閉包` 是指在 JS 中調用某函式時，即使其父作用域已被銷毀，依舊可以訪問到父作用域的變量。簡單來說 `閉包` 就是開闢了一個空間，用來存放私有變量及函式，並且不會被自動回收記憶體，而此空間正是上面 `作用域鏈` 提到的 `[[Scopes]]`。

如以下程式碼：

```javascript
function sayHello() {
  let hello = "Hello, Potter";
  function executeFn() {
    console.log(hello);
  }
  return executeFn;
}
let sayHelloFn = sayHello();
sayHelloFn(); 
console.dir(sayHelloFn);
```

輸出結果如下：

```javascript
// console.dir(sayHelloFn);
ƒ executeFn()
├─ ...
├─ arguments: null
├─ name: "executeFn"
└─ [[Scopes]]:
   ├─ Closure (sayHello) {hello: 'Hello, Potter'}
   ├─ Script {sayHelloFn: ƒ}
   └─ Global
```

### 二、閉包的用途？

1. 封裝數據：防止外部直接訪問或修改內部私有變量。

2. 模塊化編程：強制其他開發者按照規則訪問或修改私有變量。

如以下程式碼：

```javascript
function createPerson(name) {
  return {
    getName: function () {
      return name;
    },
    sayHello: function () {
      console.log(`Hello, ${name}`);
    },
  };
}
const person = createPerson("Potter");
console.log(person.getName()); // Potter
person.sayHello(); // Hello, Potter
console.log(person.name); // undefined
```

而日常最常見的用途就是 `Redux 的 createStore`，如以下程式碼：

```javascript
export default function createStore(reducer, preloadedState, enhancer) {
  // ...
  let currentReducer = reducer;
  let currentState = preloadedState;
  function getState() {
    return currentState;
  }
  function dispatch(action) {
    // ....
    return action;
  }
  return {
    dispatch,
    subscribe,
    getState,
    replaceReducer,
    [$$observable]: observable,
  };
}
```

僅能透過 `getState` 讀取 `currentState`，若要修改則要透過 `dispatch`。

### 三、閉包的缺點？

所謂 `成也蕭何，敗也蕭何`，`閉包` 的缺點也出在 `[[Scopes]]`，既然它不會自動被回收，那也代表若是使用不當，將造成 `記憶體洩漏`。如以下程式碼：

```javascript
function sayHello() {
  let hello = "Hello, Potter";
  function executeFn() {
    console.log(hello);
  }
  return executeFn;
}
let sayHelloFn = sayHello();
```

`sayHelloFn` 使用了閉包，卻沒有使用。要解決其實也很簡單：

```javascript
sayHelloFn = null;
```

## 總結

---

在此篇我們了解了 `變量和函式` 會在程式解析階段就被定義，並且會創建對應的 `作用域` 及其 `作用域規則`，而 `函式間獲取變量的規則` 就是依靠 `作用域鏈` 記錄函式間的關係，若當前作用域未找到變量就會不停地向上層作用域尋找，直到全局作用域。

但還是有些問題沒有解答：

1. 如何確保程式碼按照正確的順序執行？
2. 上面作用域鏈圖中的 `VO`、`AO` 是什麼？

接下來就讓我們來介紹下 `執行上下文棧`。

## 參考文章

---

{{<NewTabLink href="https://www.cnblogs.com/samve/p/14165711.html" title="專題3：javascript作用域、作用域鏈與閉包">}}  
{{<NewTabLink href="https://www.163.com/dy/article/H1P3CNFN0516W3V7.html" title="手把手教會你JavaScript引擎如何執行JavaScript代碼">}}  
{{<NewTabLink href="https://hackmd.io/@Heidi-Liu/note-js201-closure#week-16-JavaScript-%E9%80%B2%E9%9A%8E---%E4%BB%80%E9%BA%BC%E6%98%AF%E9%96%89%E5%8C%85%EF%BC%9F%E6%8E%A2%E8%A8%8E-Closure-amp-Scope-Chain" title="[week 16] JavaScript 進階 - 什麼是閉包？探討 Closure & Scope Chain">}}  
{{<NewTabLink href="https://juejin.cn/post/6974017383425900558#heading-1" title="你應該知道的執行上下文、調用棧、閉包、this、作用域等之間的關係">}}  
{{<NewTabLink href="https://github.com/mqyqingfeng/Blog/issues/6" title="JavaScript深入之作用域鏈">}}  
{{<NewTabLink href="https://nissentech.org/why-do-we-need-closure/" title="為什麼我們需要閉包(Closure)？它是冷知識還是真有用途？">}}

{{<NextArticle href="/articles/know-js-again-3-execution-context" article="重新認識 Javascript（三）- 執行上下文">}}
