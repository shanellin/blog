---
title: "重新認識 Javascript（五）- this"
categories: ["Frontend"]
tags: ["Javascript"]
Keywords: ["this", "js this", "箭頭函式", "arrow function", "new", "Reference", "MemberExpression", "CallExpression", "ImplicitThisValue", "call", "apply", "bind", ""]
images: ["/images/avatars.png"]
date: 2023-09-24T22:17:32+08:00
draft: false
---

## 前言

---

前面說了這麼多，總算輪到我們心心念念的 `this` 了，在正式解釋之前，我們一定要銘記：

- 變量的引用是在變量 `被創建時決定的`。

  ```javascript
  let Name = "Potter";
  function getName() {
    console.log(Name);
  }
  function getPerson() {
    let Name = "Ronald";
    getName();
  }
  getPerson(); // Potter
  ```

* this 值是在 this `被調用時決定`。

  ```javascript
  const person = {
    Name: "Potter",
    getName: function () {
      console.log(this.Name);
    },
  };
  const getName = person.getName;
  person.getName(); // Potter
  getName(); // undefined
  ```

很多時候會誤判 `this` 都是因為錯誤的調用時機所引起的。

## 函式取 this 值規則

---

`this` 可以簡單理解成用來說明是誰在使用某函式的方式。如果函式被物件使用，`this` 就指向這個物件；如果函式被獨立使用，`this` 就指向全域。之所以會有這個情況發生，就要來說說 {{<NewTabLink href="https://262.ecma-international.org/5.1/#sec-11.2.3" title="ECMAScript標準11.2.3">}} 的 `取值規則`：

```md
1. Let ref be the result of evaluating `MemberExpression`.
2. Let func be `GetValue`(ref).
3. Let argList be the result of evaluating Arguments, producing an internal list of argument values.
4. If `Type`(func) is not Object, throw a TypeError exception.
5. If `IsCallable`(func) is false, throw a TypeError exception.
6. If `Type`(ref) is `Reference`, then  
   a. If `IsPropertyReference`(ref) is true, then  
    i. Let thisValue be `GetBase`(ref).  
   b. Else, the base of ref is an `Environment Record`  
    i. Let thisValue be the result of calling the `ImplicitThisValue` concrete method of
   `GetBase`(ref).
7. Else, `Type`(ref) is not Reference.  
   a. Let thisValue be undefined.
8. Return the result of calling the [[Call]] internal method on func, providing thisValue as the
   this value and providing the list argList as the argument values.
```

其中 6.、7. 是拿到 `thisValue` 的關鍵，但為了更好地理解以上內容，會先簡述上面提及的關鍵詞：

1. MemberExpression：描述如何訪問對象的屬性或方法。在 {{<NewTabLink href="https://astexplorer.net/" title="AST Explorer">}} 輸入以下：

   ```javascript
   person.name;
   person["name"];
   person.getName();
   ```

   可以看到三行程式碼對應的 AST Tree 都有 `MemberExpression` 並都包含兩個關鍵屬性：object、property，分別表示 `.` 之前的物件和之後的屬性。

   ```toml
    MemberExpression
    ├─ object: Identifier (name: "person")
    └─ property: Identifier (name: "getName")
   ```

   這裡值得一提的是：

   ```javascript
   getName();
   ```

   雖然在 AST Tree 中僅表示 `CallExpression`，但 getName 也是 `MemberExpression` 的一部份，因為 getName 表示要存取的函式或表達式。

2. ref：是一個抽象概念，是 `MemberExpression` 的代替詞，方便後續說明。
3. func：代表要執行的函式或方法，需透過 GetValue(ref) 取得。
4. GetValue：取得變數或屬性的實際值。
5. Type：用於確定值的類型，如：數字、字串、物件...等。
6. IsCallable：檢查一個值是否可以被呼叫。
7. Reference：這裡用來判斷 `Type(ref)` 是否為引用類型。在 ECMAScript 中為方便說明，由三部分組成：

   ```javascript
   let person = {name: 'Potter'}

   person.name's Reference: {
     baseValue: person, // 指向引用的原對象
     referencedName: name, // 引用的名稱
     strictReferenceFlag: false // 標示是否嚴格模式
   }
   ```

   ```javascript
   let school = 'Hogwarts'

   school's Reference: {
     baseValue: EnvironmentRecord, // 指向引用的原對象
     referencedName: school, // 引用的名稱
     strictReferenceFlag: false // 標示是否嚴格模式
   }
   ```

8. IsPropertyReference：用於檢查 Reference 是否引用了一個物件的屬性。

   ```javascript
   let person = { name: "Potter" };
   person.name; // IsPropertyReference === true
   ```

9. GetBase：取得 Reference 的 `baseValue`，也就是當前 ref 引用的原對象。
10. Environment Record：用於存放 `標識符<->變量` 的映射表。可參考 {{<NewTabLink href="/articles/know-js-again-3-execution-context/#一創建階段" title="創建階段 - 環境紀錄">}}
11. ImplicitThisValue：根據 Reference 的 baseValue 找出 this 值。

配合以上關鍵詞，我們可以將上面 6.、7. 的內容精簡成：

- 若 ref 的類型是 Reference，且 Reference 是引用了一個對象的屬性，那 thisValue 就等於 ref 的 Reference 的 baseValue。
- 若 ref 的類型是 Reference，且 Reference 的 baseValue 等於 Environment Record，那 thisValue 就等於 ImplicitThisValue(ref 的 Reference 的 baseValue)。
- 若 ref 的類型不是 Reference，那 thisValue 等於 undefined。

## 一般函式

---

接著讓我們回頭看看 [前文](/articles/know-js-again-5-this/#前言) 提及的：

```javascript
const person = {
  Name: "Potter",
  getName: function () {
    console.log(this.Name);
  },
};
const getName = person.getName;
person.getName(); // Potter
getName(); // undefined
```

並各別解釋。

### person.getName()

1. 此時 `MemberExpression` 等於 person.getName，而 `ref` 用來代表 person.getName 這個表達式。
2. 透過 `GetValue` 取得 `ref` 要實際呼叫的函式 `getName`，並以 `func` 表示。
3. 沒有傳遞參數，所以 argList 為空。
4. `func` 等於 person.getName 的 getName，是函式物件，所以不報錯。
5. `func` 是函式物件，可以呼叫，所以不報錯。
6. 因為引用了一個函式，所以 `Type(ref)` 是 `Reference`，如下：
   ```javascript
   Reference: {
     baseValue: person, // 指向引用的原對象
     referencedName: getName, // 引用的名稱
     strictReferenceFlag: false // 標示是否嚴格模式
   }
   ```
7. 因為引用的是物件屬性，所以 `IsPropertyReference(ref)` 是 true。
8. 回傳 `GetBase(ref)` 也就是 person。

所以 `person.getName()` 內部的 this 會指向 person，而 person.Name: Potter，最終輸出 Potter。

### getName()

1. 此時 `MemberExpression` 等於 getName，而 `ref` 用來代表 getName 這個表達式。
2. 透過 `GetValue` 取得 ref 要實際呼叫的函式 `getName`，並以 `func` 表示。
3. 沒有傳遞參數，所以 argList 為空。
4. `func` 等於 getName，是函式物件，所以不報錯。
5. `func` 是函式物件，可以呼叫，所以不報錯。
6. 因為引用了一個函式，所以 `Type(ref)` 是 `Reference`，如下：
   ```javascript
   Reference: {
     baseValue: Environment Record, // 指向引用的原對象
     referencedName: getName, // 引用的名稱
     strictReferenceFlag: false // 標示是否嚴格模式
   }
   ```
7. 因為引用的是 Environment Record，所以 `IsPropertyReference(ref)` 是 false。
8. 回傳 `ImplicitThisValue(ref's Reference's baseValue)`。

所以 `getName()` 內部的 this 會指向 Environment Record，而 Environment Record 並不存在 Name，最終輸出 undefined。

### 總結

根據 {{<NewTabLink href="https://262.ecma-international.org/5.1/#sec-10.4.3" title="ECMAScript標準10.4.3">}} 的說明，在非嚴格模式下，當 thisValue 等於 null 或 undefined 時會自動綁定全局物件 window。

因此大部分情況下我們可以通過 `MemberExpression` 判別 this 指向，若 `MemberExpression` 有 object，this 就指向 object，沒有就指向 window。

## 箭頭函式

---

箭頭函式比起一般函式有兩個不同的地方：

1. 自身沒有 this，完全根據父層作用域決定 this，並且無法透過 bind、call 或 apply 等方式改變 this 指向。
2. this 總是指向定義時所在的上下文。

如以下程式碼：

```javascript
const person = {
  Name: "Potter",
  age: 18,
  getName: function () {
    return () => {
      console.log(this.Name);
    };
  },
  getAge: () => {
    console.log(this.age);
  },
};
const getName = person.getName();
const school = { Name: "Hogwarts", getName };
getName(); // Potter
school.getName(); // Potter
person.getAge(); // undefined
```

- `person.getName()` 傳回箭頭函式並賦值給 `getName`。在呼叫時箭頭函式就已經將 this 綁定到 person，所以 `getName` 回傳 Potter。
- `school.getName()` 由於箭頭函式的特性，this 總是指向定義時的上下文，所以也回傳 Potter。
- `person.getAge()` 雖然 `MemberExpression` 有 object，但由於箭頭函式的特性，它會根據父層作用域來確定 this，即 window，而 window 並沒有 age 屬性，所以回傳 undefined。

## new

---

此部份可參考上篇文章 {{<NewTabLink href="/articles/know-js-again-4-prototype-chain/#new" title="原型鏈 - new">}} 中了解到實例化步驟，構造函式內部的 `this` 會綁定到新創建的對象上。因此自然地 `this` 就指向新創建的對象。

如以下程式碼：

```javascript
function Person(name) {
  this.name = name;
}
const person1 = new Person("Potter");
const person2 = new Person("Ronald");
person1.name; // Potter
person2.name; // Ronald
```

## call、apply、bind

---

對於 `this` 而言，這三者均能做到同件事，就是顯性地更改函式內 `this` 的指向。根據 {{<NewTabLink href="https://developer.mozilla.org/zh-TW/docs/Web/JavaScript/Reference/Global_Objects/Function/call" title="MDN">}} 讓我們看看這三者各自的語法：

```javascript
fun.call(thisArg[, arg1[, arg2[, ...]]])

fun.apply(thisArg, [argsArray])

fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

雖然 `call` 和 `bind` 的語法幾乎一樣，但從功能上我們反而是將 `call` 和 `apply` 歸為一組，這是因為：

- `call` 和 `apply` 都是更改 `this` 的指向後，直接調用。
- `bind` 是更改 `this` 的指向後，返回函式供使用者調用。

### 用法

讓我們再次使用前言的範例進行說明：

```javascript
const person = {
  Name: "Potter",
  getName: function (name) {
    console.log(`I'm ${this.Name}. Hi, ${name}`);
  },
};
const getName = person.getName;
getName("Ronale"); // I'm undefined. Hi, Ronale
```

#### 1. call

第一個參數為 `this` 更改後的指向，接下來的參數會被傳入原函式中。

```javascript
getName.call(person, "Ronald"); // I'm Potter. Hi, Ronale
```

#### 2. apply

與 `call` 相似，第一個參數為 `this` 更改後的指向，差別在於剩餘參數需放到陣列中。

```javascript
getName.apply(person, ["Ronald"]); // I'm Potter. Hi, Ronale
```

#### 3. bind

用法與 `call` 相似，差別在於會回傳函式，而非直接執行。這時 `getName` 的 `this` 將會被永久綁定，即使放到其他物件下，依舊會將 `this` 指向原先 `bind` 的物件。

```javascript
const getName = person.getName.bind(person, "Ronald");
getName(); // I'm Potter. Hi, Ronale
const person2 = {
  Name: "Hermione",
  getName,
};
person2.getName(); // I'm Potter. Hi, Ronale
```

### 原理

但 `call`、`apply` 是如何做到更改 `this` 指向的呢？不外乎就是三步驟：

1. 將傳入的第一個參數作為之後要執行的對象。
2. 將當前函式設為對象屬性。
3. 返回執行結果。

而 `bind` 可以看作是返回一個使用 `call` 的函式。

#### 1. call

```javascript
Function.prototype.myCall = function (obj, ...args) {
  // 1. 將傳入的第一個參數作為之後要執行的對象。
  const nextObj = obj !== null && obj !== undefined ? new Object(obj) : window;
  const uniqueID = Symbol(); // 建立一個唯一的識別符
  // 2. 將當前函式設為對象屬性。
  nextObj[uniqueID] = this; // 將函式設定為物件的屬性
  const result = nextObj[uniqueID](...args); // 呼叫函式並傳入參數
  delete nextObj[uniqueID]; // 刪除臨時屬性
  // 3. 返回執行結果。
  return result;
};

getName.myCall(person, "Ronald"); // I'm Potter. Hi, Ronale
```

#### 2. apply

```javascript
Function.prototype.myApply = function (obj, args = []) {
  // 1. 將傳入的第一個參數作為之後要執行的對象。
  const nextObj = obj !== null && obj !== undefined ? new Object(obj) : window;
  const uniqueID = Symbol(); // 建立一個唯一的識別符
  // 2. 將當前函式設為對象屬性。
  nextObj[uniqueID] = this; // 將函式設定為物件的屬性
  const result = nextObj[uniqueID](...args); // 呼叫函式並傳入參數
  delete nextObj[uniqueID]; // 刪除臨時屬性
  // 3. 返回執行結果。
  return result;
};

getName.myApply(person, ["Ronald"]); // I'm Potter. Hi, Ronale
```

#### 3. bind

```javascript
Function.prototype.myBind = function (obj, ...args) {
  const self = this; // 儲存原始函式的引用
  return function (...callArgs) {
    return self.myCall(obj, ...args, ...callArgs); // 使用 myCall 方法呼叫原始函式
  };
};

const getName = person.getName.myBind(person, "Ronald");
getName(); // I'm Potter. Hi, Ronale
const person2 = {
  Name: "Hermione",
  getName,
};
person2.getName(); // I'm Potter. Hi, Ronale
```

## 總結

---

綜合上述所講，我們可以總結出三點：

1. `this` 可以簡單理解成用來說明是誰在使用某函式的方式。如果函式被物件使用，`this` 就指向這個物件；如果函式被獨立使用，`this` 就指向全域。
2. 箭頭函式自身沒有 this，完全根據父層作用域決定 this，且總是指向定義時所在的上下文。
3. `call`、`apply` 和 `bind` 可更改 `this` 的指向，前兩者會直接調用，後者會返回函式供使用者調用。

最後總結前面幾篇文章，可以得到：

![js-this](/images/js/js-this.png)

## 參考文章

---

{{<NewTabLink href="https://zhuanlan.zhihu.com/p/21805749" title="根據JavaScript中的this-ECMAScript規格解讀">}}  
{{<NewTabLink href="https://github.com/mqyqingfeng/Blog/issues/7" title="JavaScript深入之從ECMAScript規範解讀this">}}  
{{<NewTabLink href="https://segmentfault.com/a/1190000011194676#item-15" title="深入理解 js this 綁定">}}  
{{<NewTabLink href="https://262.ecma-international.org/5.1/" title="Standard ECMA-262 5.1">}}  
{{<NewTabLink href="https://juejin.cn/post/6977563249650696206" title="call、apply、bind實作原理">}}  
{{<NewTabLink href="https://developer.aliyun.com/article/624114" title="Javascript AST 編譯器的研究學習">}}
