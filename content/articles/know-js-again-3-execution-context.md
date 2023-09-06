---
title: "重新認識 Javascript（三）- 執行上下文"
categories: ["Frontend"]
tags: ["Javascript"]
Keywords: ["執行上下文", "執行上下文棧", "eval", "生命週期", "詞法環境", "變量環境", "LexicalEnvironment", "VariableEnvironment", "VO", "AO", "全局變量對象", "活動變量對象", "環境紀錄", "外層引用", "全局環境", "函式環境", "模塊環境"]
images: ["/images/avatars.png"]
date: 2023-09-06T15:42:29+08:00
draft: false
---

## 前言

---

上篇中我們提到了兩個問題：

1. 如何確保程式碼按照正確的順序執行
2. 什麼是 `VO`、`AO`

接下來就讓我們一一解答。

本篇涉及了兩個知識點：`執行上下文`、`生命週期`、`執行上下文棧`，以下將依序說明。

> 因 `this 指向` 方法較為特殊，會額外寫一篇文章做探討，此篇就不多做說明。

## 執行上下文

---

`執行上下文` 和 `作用域` 都是 JS 中的抽象概念，它們描述了程式執行過程中的不同方面。在前文中我們提到 `作用域` 定義了在一個區域內可訪問的變量和函式的範圍。而 `執行上下文` 可理解成是程式執行環境，描述了執行過程中的所有細節，如：變量對象賦值、堆棧內存引用、確定作用域鏈、this 指向...等。

根據執行方式 `執行上下文` 可分成三種：

### 一、全局執行上下文

執行階段最一開始便會創建，是整個 JS 環境的最頂層上下文，裡面聲明的變量和函式皆可在任何地方訪問到。

主要會做兩件事：

1. 創建全局對象：全局對象是一個特殊的對象，除了開發者透過 `var`、`let`、`const`、`function` 所宣告的變量外，還包含了許多內置對象，如：`Math`、`String`、`Array`、`Promise`...等，在瀏覽器環境中，這個全局對象也被稱為 window 對象。

2. 將 this 指向這個全局對象：在全局執行上下文中，this 通常指向全局對象，因此在瀏覽器環境中，this 指向 window。這也是為何在全局作用域中，可以通過 this 來訪問全局對象的屬性和方法。

### 二、函式執行上下文

在 `執行階段` 中每次調用函式，環境都會為它創造一個 `函式執行上下文`，並為其分配內存空間，用於存儲函式的局部變量、參數、函式引用等信息，最後在函式執行時動態地確定該函式內部的 `this` 指向。

如以下程式碼：

```javascript
function fn1() {
  function fn2() {
    console.log(`fn2 called`);
  }
  fn2();
  console.log(`fn1 called`);
}
fn1();
// fn2 called
// fn1 called
```

![js-context-example](/images/js/js-context-example.png)

### 三、eval 執行上下文

eval 是一個特殊的函式，根據 {{<NewTabLink href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/eval" title="MDN">}} 的解釋，它可以運行由執行階段動態生成的字符串，但不推薦使用，甚至被瀏覽器 dev tool 禁止。

如以下程式碼：

```javascript
let x = 10;
let code = "x = x * 2;";
eval(code);
console.log(x); // 20
```

想必大部分開發者應該能知道不推薦的原因吧，除了執行階段需要額外進行 `解析` 導致性能低落，程式碼維護及安全問題更嚴重，光能直接運行 JS 這一點，可謂是擁抱 {{<NewTabLink href="https://zh.wikipedia.org/zh-tw/%E8%B7%A8%E7%B6%B2%E7%AB%99%E6%8C%87%E4%BB%A4%E7%A2%BC" title="XSS">}}，為人生增添一份精彩。

eval 執行上下文與普通函式執行上下文相似，但有幾點不同：

1. 作用域鏈：普通函式的作用域鏈會包含自身作用域及外部函式或全局作用域的作用域鏈。但 eval 會包含當前執行 eval 的函式作用域，當 eval 內需要訪問變量時會優先在該函式作用域中尋找。

2. 變量提升：普通函式的變量和函式聲明會被提升到函式作用域的頂部，供函式內部的任何位置訪問。但 eval 是提升到內部函式頂部，而不是包含它的那個函式頂部。

3. this：普通函式的 this 是根據被調用的方式決定。但 eval 是取決於包含它的那個函式 this。

## 生命週期

---

執行上下文的生命週期包含三個階段：創建階段 -> 執行階段 -> 回收階段。

由於目前 `執行上下文` 的說明根據 ES6 可分成新舊版本，以下會各別說明新舊的差異：

### 一、創建階段

#### ES6 前：

此階段主要用於創建 `全局執行上下文` 和 `全局變量對象（Variable object，VO）`。

在全局執行上下文創建時會對變量和函式聲明做兩件事：

1. 若聲明對象是引用類型（array、object、function）會將該參數、值存入堆內存，並為其標記唯一引用地址，反之若是基本類型（string、number、boolean）則存入棧內存。

2. 將此引用地址指向對象本身。

而以上對象與引用地址的關聯就會存在 `全局變量對象 VO` 中。如以下程式碼：

```javascript
let x = ["hello", "Peter"];
function sayHello(person) {
  console.log(`hello, ${person[1]}`);
}
```

{{<ImageSize src="/images/js/js-global-vo.png" alt="js-global-vo" width="600">}}

#### ES6 後：

> 注意：以下 `外層引用`、`this 指向` 均在 `執行階段` 才會被確定，但為了說明方便，才提前到 `創建階段`。

此階段主要用於創建 `詞法環境（LexicalEnvironment）` 和 `變量環境（VariableEnvironment）`。

`詞法環境（LexicalEnvironment）`可以簡單想成是執行上下文的概念，分成三種類型：

1. 全局環境：儲存開發者宣告的變量及瀏覽器內置對象。對應上面的 [全局執行上下文](/articles/know-js-again-3-execution-context/#一全局執行上下文)。

2. 函式環境：儲存執行階段，函式的局部變量、參數、函式引用...等。對應上面的 [函式執行上下文](/articles/know-js-again-3-execution-context/#二函式執行上下文)。

3. 模塊環境：用於記錄 `import`、`export`、`module` 等變量。

每種 `詞法環境` 由三部分組成：

1. `[創建階段]` 環境紀錄（EnvironmentRecord）：用於存放 `標識符<->變量` 的映射表。標識符指的是變量名或函式名，而變量則是對象、函式的引用地址。基本對應 `ES6 前` 的 VO。

2. `[執行階段]` 外層引用（Outer）：用於存放自身環境與外層環境的引用關係，執行階段若當前環境紀錄中找不到變量就會向外層的環境紀錄中尋找。對應上個章節講的 {{<NewTabLink href="/articles/know-js-again-2-scope-chain/" title="作用域鏈">}}。

3. `[執行階段]` this 指向。

{{<ImageSize src="/images/js/js-context-lexical-compare.png" alt="js-context-lexical-compare" width="600">}}

`環境紀錄` 又可再細分成兩種：

1. 聲明式環境紀錄（declarative）：記錄如非全局函式聲明（function）、變量聲明（let、const）、模塊引入（import、export）、class 的語法。

2. 對象式環境紀錄（object）：紀錄與全局函式聲明（function）、全局變量有關的聲明（var）、與全局對象屬性有關的語法結構（如 with 語句）相關聯。根據 {{<NewTabLink href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/with" title="MDN">}} 目前已不建議使用 `with`。

簡單來說若聲明後可透過 `window` 調用即為 `對象式`，反之為 `聲明式`。

最後 `變量環境（VariableEnvironment）` 其實跟 `詞法環境` 大致同個概念，只是 `ES6` 區隔了 `let、const` 和 `var`，由 `let、const` 聲明的變量會存到 `詞法環境`，而 `var` 聲明的變量會存到 `變量環境`。

> 額外提下創建階段若碰到 `let、const` 環境紀錄並不會初始化該變量，呼叫會回傳 `xxx is not defined`，也就不存在變量提升，但 `var` 則會在初始化變量，生成 `undefined`，所以在聲明前呼叫會回傳 `undefined`，這也呼應了上章節的 {{<NewTabLink href="articles/know-js-again-2-scope-chain/#三變量提升hoisting" title="變量提升">}}。

### 二、執行階段

#### ES6 前：

此階段遇到函式被調用時會根據 `VO` 生成對應函式執行上下文的 `活動變量對象（Active Object，AO）`，因此 `VO`、`AO` 大致是相同概念，只是不同階段的稱呼以及內部屬性會有些微不同。

活動變量對象（Active Object，AO）會根據函式做以下事情：

1. 根據自身及外層作用域生成 {{<NewTabLink href="/articles/know-js-again-2-scope-chain/" title="作用域鏈">}}。

2. 根據調用對象確定 `this` 指向。

3. 創建實參集合 `arguments`。

4. 參數賦值。

根據 {{<NewTabLink href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Functions/arguments" title="MDN">}} 的解釋，`arguments` 是一個對應傳遞給函式的參數的類數組對象，裡面包含索引、參數數量（length）、當前函式引用（callee）、迭代器（Symbol(Symbol.iterator)）。類數組沒有真正數組的方法，只能通過索引訪問參數，除非轉成真正的數組，如：

```javascript
Array.from(arguments);
Array.prototype.slice.call(arguments);
```

接下來也讓我們示例 `活動變量對象 AO`，如以下程式碼：

```javascript
let x = ["hello", "Peter"];
function sayHello(person) {
  console.log(`hello, ${person[1]}`);
}
sayHello(x);
```

{{<ImageSize src="/images/js/js-function-ao.png" alt="js-function-ao" width="600">}}

#### ES6 後：

對應 [創建階段](/articles/know-js-again-3-execution-context/#一創建階段) 的提前說明，此階段會創建 `外層引用（Outer）` 及確定 `this 指向`，至此完成 `詞法環境` 的三部分。

{{<ImageSize src="/images/js/js-env-record-outer.png" alt="js-env-record-outer" width="600">}}

### 三、回收階段

此階段負責清理執行完畢的 `執行上下文` 相關的資源，以釋放內存，如以下：

1. 資源釋放：銷毀上下文不再需要的變量、對象和函式，以釋放它們佔用的內存。

   ```javascript
   function checkMainCharacter() {
     let person = "Peter";
     let friendList = ["Ronald", "Hermione"];
     let book = { title: "Harry Potter" };
     console.log({ person, friendList, book });
   }
   checkMainCharacter(); // 執行完畢後釋放 person、friendList、book
   ```

2. 函式調用棧的彈出：{{<NewTabLink href="articles/know-js-again-1-js-engine-runtime/#一調用堆棧call-stack" title="調用堆棧">}} 採取 `先進後出` 的策略，釋放已彈出的函式執行上下文所佔的內存。

   ```javascript
   function fn1() {
     console.log("fn1");
   }
   function fn2() {
     console.log("fn2");
   }
   function callFn() {
     fn1();
     fn2();
   }
   callFn(); // 執行完畢後釋放 fn1、fn2、callFn 之執行上下文
   ```

3. 垃圾回收：標記全域不再被引用的對象，如：閉包、循環引用和全局對象等，在回收內存新舊生代接替過程中進行銷毀。

> 其實 `垃圾回收（Garbage Collection）` 也是一個值得探討的主題，包括新舊生代、WeakSet、WeakMap，之後若有機會會再額外探討。

## 執行上下文棧

---

經過上面 `生命週期` 的解釋後應該大致能猜到 `執行上下文棧（Execution context stack）` 其實也就是 `調用堆棧（Call Stack）` 有著 `先進後出（Last-In-First-Out，LIFO）` 的特性，接下來就來示例 `執行上下文棧` 的運作過程吧。

以下方程式碼為例：

```javascript
let x = ["hello", "Peter"];
function sayHello(person) {
  console.log(`hello, ${person[1]}`);
  function sayBye(person) {
    console.log(`bye, ${person[1]}`);
  }
  sayBye(person);
}
sayHello(x);
```

![js-ecs](/images/js/js-ecs.png)

而上方的運作過程便是 `如何確保程式碼按照正確的順序執行` 的答案。

## 總結

---

結合 {{<NewTabLink href="/articles/know-js-again-2-scope-chain/" title="作用域鏈">}} 及 `執行上下文棧` 的概念，大概可以知道程式在 `解析階段` 會先確定 `作用域`、`作用域規則`，到了 `執行階段` 在 `執行上下文` 中確定 `作用域鏈`，而 `執行上下文生命週期` 會有三階段：

1. 創建階段：生成 `全局執行上下文` 和 `Variable Object（VO）`，然後將它們壓入 `執行上下文棧` 中。全局執行上下文代表了整個程序的上下文，而 VO 包含了全局作用域中的變量和函式聲明。

2. 執行階段：生成 `函式執行上下文` 和 `Activation Object（AO）`。過程中會逐步確定 `Variable Object（作用域）`、`作用域鏈`、`this 指向`。然後將函式執行上下文壓入 `執行上下文棧`。一旦函式開始執行，它將根據 `作用域鏈` 查找變量。如果在當前上下文的變量對象中找不到變量，將按照作用域鏈向上查找，直到全局執行上下文。

3. 回收階段：在這個階段，JS 引擎會釋放不再需要的內存資源。這包括銷毀不再使用的局部變量、函式上下文以及其他不再被引用的對象。垃圾回收確保內存資源的有效使用，防止內存洩漏。

且 `執行上下文棧` 具有 `先進後出（Last-In-First-Out，LIFO）` 的特性。

最後總結前面幾篇文章，我們可以得到較完整的流程圖：

![js-full-flow](/images/js/js-full-flow.png)

了解了執行上下文棧和作用域鏈後，會發現還有幾個問題存在：

1. 開發時，定義的對象明明沒有為其添加方法，但為何可以使用？

   ```javascript
   let person = "Peter";
   console.log(person.length); // 5
   console.log(person.toUpperCase()); // PETER
   ```

2. 該如何讓對象間共享屬性與方法？

接下來就讓我們來介紹下 `原型鏈`。

## 參考文章

---

{{<NewTabLink href="https://zhuanlan.zhihu.com/p/311196297" title="通過動圖了解JS中的ECStack、EC、VO 和AO">}}  
{{<NewTabLink href="https://juejin.cn/post/6896668614272024589" title="深入淺出執行上下文、詞法環境、變量環境">}}  
{{<NewTabLink href="https://juejin.cn/post/6844904145372053511#heading-4" title="JS夯實之執行上下文與詞法環境">}}  
{{<NewTabLink href="https://vue3js.cn/interview/JavaScript/context_stack.html#%E4%B8%80%E3%80%81%E6%89%A7%E8%A1%8C%E4%B8%8A%E4%B8%8B%E6%96%87" title="面試官：JavaScript中執行上下文和執行棧是什麼？">}}  
{{<NewTabLink href="https://github.com/mqyqingfeng/Blog/issues/4" title="JavaScript深入之執行上下文棧">}}  
{{<NewTabLink href="https://github.com/mqyqingfeng/Blog/issues/5" title="JavaScript深入之變量對象">}}