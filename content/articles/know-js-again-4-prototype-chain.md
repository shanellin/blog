---
title: "重新認識 Javascript（四）- 原型鏈"
categories: ["Frontend"]
tags: ["Javascript"]
Keywords: ["構造函式", "constructor", "原型對象", "Prototype", "原型對象引用", "＿proto＿", "原型鏈", "Prototype Chain", "new", "class"]
images: ["/images/avatars.png"]
date: 2023-09-09T09:43:29+08:00
draft: false
---

## 前言

---

上篇我們提到了兩個問題：

1. 開發時，定義的對象明明沒有為其添加方法，但為何可以使用？
2. 該如何讓對象間共享屬性與方法？

接下來就讓我們一一解答。

本篇涉及了六個知識點：`構造函式`、`原型對象`、`原型對象引用`、`原型鏈`、`new`、`class`，以下將依序說明。

## 構造函式（constructor）

---

稍微了解過 {{<NewTabLink href="https://zh.wikipedia.org/zh-tw/JavaScript" title="JS 歷史">}} 的開發者應該知道在當時 JS 可說是一個奇蹟，{{<NewTabLink href="https://zh.wikipedia.org/zh-tw/%E5%B8%83%E8%98%AD%E7%99%BB%C2%B7%E8%89%BE%E5%85%8B" title="Brendan Eich">}} 僅花了十天便將 JS 的前身 LiveScript 給設計出來，並在短短四個月就佔據了四分之三的瀏覽器市場。除此之外，JS 有個特殊的地方，雖然當時盛行基於類的 {{<NewTabLink href="https://zh.wikipedia.org/zh-tw/%E9%9D%A2%E5%90%91%E5%AF%B9%E8%B1%A1%E7%A8%8B%E5%BA%8F%E8%AE%BE%E8%AE%A1" title="OOP">}} 語言，但為了實現網頁上的實時交互和靈活性，`Brendan Eich` 最終選擇了 `構造函式（constructor）` 和 `原型繼承` 作為 JS 創建對象的核心機制，而非 `類（class）`。

在 ES6 之前，JS 創建對象的方式有以下三種：

1. 對象字面量

   ```javascript
   let person = { name: "Potter" };
   ```

2. new Object()

   ```javascript
   let person = new Object();
   person.name = "Potter";
   ```

3. 自定義構造函式

   ```javascript
   function Person(name) {
     this.name = name;
   }
   let user = new Person("Potter");
   ```

但需要注意的是，這三者中只有 `自定義構造函式` 允許開發者自定義對象的 `原型鏈`，從而具備 `繼承`、`共享屬性與方法` 的能力。

在 JS 中，使用構造函式要注意以下三點：

1. 構造函式名稱首字母要大寫。

2. 若要創建對象必須使用 new。

3. 構造函式內 this 引用的是即將生成的對象，而不是構造函式本身或其他對象。

構造函式可根據用途分成兩種成員：

1. 靜態成員（Static Members）：直接添加到構造函式本身的屬性或方法。不能被實例化的對象訪問，只能透過構造函式本身訪問。通常用於存儲全局屬性和方法，以此做到 {{<NewTabLink href="https://zh.wikipedia.org/zh-tw/%E5%8D%95%E4%BE%8B%E6%A8%A1%E5%BC%8F" title="單例模式">}}，並能節省記憶體。

2. 實例成員（Instance Members）：在內部通過 this 添加的屬性或方法。只能由構造函式創建的對象實例訪問，並且實例間相互獨立，不共享內部屬性或方法。

如以下程式碼：

```javascript
function Person(name) {
  this.name = name; // 實例成員
}
let user = new Person("Potter");
console.log(user.name);

Person.type = "human"; // 靜態成員
console.log(Person.type); // human
console.log(user.type); // undefined
```

但這樣遠遠不夠，有時出於管理方便並不想要將所有共享屬性和方法都加到 `靜態成員` 內，但依然想讓多個對象共享屬性和方法，這該如何做到呢？

## 原型對象（Prototype）

---

又稱 `顯式原型`。

在 JS 中萬物皆對象，而每個對象都會關聯一個名為 `prototype` 的子對象。這個 `prototype` 表示該對象的原型，其中包含了與其相關的 `構造函式` 的成員集合，這些成員可以被創建的實例對象所共享。

也就是說，`構造函式` 會通過 `prototype` 存儲屬性和方法，後續被創建的實例皆可從 `prototype` 上繼承對應的屬性與方法。

### 未用 prototype

```javascript
function Person(name) {
  this.name = name;
  this.run = function () {
    console.log("run");
  };
}
let user1 = new Person("Potter");
let user2 = new Person("Ronald");
console.log(user1.run === user2.run); // false
```

### 使用 prototype

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.run = function () {
  console.log("run");
};
let user1 = new Person("Potter");
let user2 = new Person("Ronald");
console.log(user1.run === user2.run); // true
```

如此便可做到不將共享方法加到 `構造函式` 的 `靜態成員` 內，但依然能讓多個對象擁有相同方法，做到內存優化。

但這又引起了另一個疑問，user1 和 user2 是如何獲取到 `構造函式 Person` 的 `prototype`？

## 原型對象引用（＿proto＿）

---

又稱 `隱式原型`。

如果說 `prototype` 是用來儲存共享屬性與方法，供實例對象繼承，那 `＿proto＿` 就是這之間的橋樑，也就是 `原型對象的引用`。

讓我們看看 Person 的 `prototype` 和 user1 的 `＿proto＿`：

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.run = function () {
  console.log("run");
};
let user1 = new Person("Potter");
console.log(user1.__proto__);
console.log(Person.prototype);
```

會發現兩者居然一模一樣，除了熟悉的 `run` 和 `constructor: Person` 還有一個陌生的 `[[Prototype]]`：

```javascript
{
  run: ƒ ()
  constructor: ƒ Person(name)
  [[Prototype]]: Object
}
```

這正是 `原型對象的引用` 的意思，當對象實例時會自動將 `prototype` 的共享屬性、方法引用到其中，之後若需要調用，則會透過 `__proto__` 向上層尋找，以此做到繼承。

讓我們通過以下驗證：

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.run = function () {
  console.log("run");
};
let user1 = new Person("Potter");
console.log(user1.__proto__ === Person.prototype); // true
```

實例對象的 `＿proto＿` 確實等價於 `原型對象 prototype`。

但又有一個疑問，`[[Prototype]]` 是什麼？

## 原型鏈（Prototype Chain）

---

`[[Prototype]]` 是 JS 的內部隱藏屬性，用於建立對象間的 `原型鏈` 關係。但當我們點開 `[[Prototype]]` 會發現所有實例對象的 `[[Prototype]]` 都長一樣，這是因為 JS 內部為避免訊息冗餘而刻意設計的，僅會輸出默認對象，而非實際內容，但實際上在內部 `[[Prototype]]` 確實保存了上層 `prototype` 的引用。

另外當我們仔細看 `[[Prototype]]` 會發現其中包含 `＿proto＿` 的 getter 和 setter，這正是 JS 的設計，`[[Prototype]]` 用來儲存 `prototype` 的引用，而 `＿proto＿` 用來查找。也就是說當訪問對象不存在的屬性時，對象會由 `＿proto＿` 透過 `[[Prototype]]` 向上查找，即父層的 `prototype`，若父層的 `prototype` 也沒有，那又會根據父層的 `＿proto＿` 繼續往上找，最終找到 JS 頂層對象的原型 Object.prototype，再往上找就沒有了，也就是 null。

而以上的查找過程，正是我們所謂的 `原型鏈`。如以下程式碼：

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.run = function () {
  console.log("run");
};
let user1 = new Person("Potter");
console.log(user1.__proto__ === Person.prototype); // true
console.log(user1.__proto__.__proto__ === Object.prototype); // true
console.log(user1.__proto__.__proto__.__proto__ === null); // true
```

這也解釋了 `開發時，定義的對象明明沒有為其添加方法，但為何可以使用？`，如以下程式碼：

```javascript
let name = "Potter";
console.log(name.__proto__); // String
let age = 18;
console.log(age.__proto__); // Number
let friendList = ["Ronald", "Hermione"];
console.log(friendList.__proto__); // Array
let book = { title: "Harry Potter" };
console.log(book.__proto__); // Object
```

name、age、friendList、book 均會透過 `__proto__` 向上找到對應的 `prototype` 即 `String、Number、Array、Object`，之後便可使用各自的成員集合。

## new

---

根據前文可以知道若要實例對象就必不可少 `new`，讓我們一步一步說明 `new` 到底做了什麼：

1. 創建一個空對象，該對象將成為新創建的實例。

2. 將構造函式的 `prototype（原型對象）` 透過新創建對象的 `__proto__（原型對象引用）` 將引用連接到 `[[Prototype]]`，這樣新對象就可以訪問構造函式原型中定義的屬性和方法。

3. 將構造函式內部的 `this` 綁定到新創建的對象上。

4. 執行構造函式內部的程式碼，初始化新對象的屬性並對其賦值。

5. 判斷返回結果，若是引用類型（Object）則會直接返回，若是基本數據類型（String、Number、Boolean、Null、Undefined、Symbol），實例仍會創建，但返回的基本數據類型值會被忽略。

如以下程式碼：

```javascript
function Person(value) {
  if (value.constructor === String) {
    this.name = value;
  } else {
    return value;
  }
}
const person1 = new Person({ name: "Potter" });
console.log(person1);
const person2 = new Person(42);
console.log(person2);
const person3 = new Person("Potter");
console.log(person3);
```

會發現三者均會成功創建實例對象，person2 因為回傳基本數據類型，因此 42 被忽略了：

```javascript
// console.log(person1);
{
  name: "Potter"
  [[Prototype]]: Object
}
// console.log(person2);
Person {
  [[Prototype]]: Object
}
// console.log(person3);
Person {
  name: "Potter"
  [[Prototype]]: Object
}
```

## class

---

在 ES6 之後，JS 引入了 class 的概念，使得開發者可以更方便地創建實例對象，不再需要直接操作 `prototype（原型對象）` 來實現繼承的過程。但需要注意的是，此 class 非 OOP 之 class，而是 `構造函式（constructor）` 的語法糖，不過它卻能讓開發者更方便地管理程式碼。

接下來就讓我們將以下程式碼

```javascript
function Person(name) {
  this.name = name;
}
Person.eat = function () {
  console.log("eat");
};
Person.prototype.run = function () {
  console.log("run");
};
let user1 = new Person("Potter");
```

轉成 class：

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  static eat() {
    console.log("eat");
  }
  run() {
    console.log("run");
  }
}
let user1 = new Person("Potter");
```

### 靜態方法

根據上方程式碼可以發現 class 多了 `static` 這個關鍵字，但想必立馬就可以聯想到其實 `static` 就等價於 `構造函式（constructor）` 的靜態成員，也就是說實例對象無法直接使用。

### Getter、Setter

Getter、Setter 允許你隱藏對象屬性的內部實現細節，並提供一種更受控制的方式來訪問和修改屬性，如：添加驗證邏輯。這有助於確保數據的完整性和一致性。

```javascript
class Person {
  constructor(name) {
    this._name = name;
  }
  get name() {
    return `Hello, ${this._name}`;
  }
  set name(value) {
    if (this._name.includes(value)) {
      this._name += "+";
    }
  }
}
let user = new Person("Potter");
console.log(user.name); // Hello, Potter
user.name = "Potter";
console.log(user.name); // Hello, Potter+
```

### 私有屬性、方法

此為 ES12 引入的新特性，根據 {{<NewTabLink href="https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Classes/Private_class_fields" title="MDN">}} 可通過在屬性或方法名稱前添加 `井號（#）` 來標識為私有。這意味著只有 `class` 內部的程式碼可以訪問這些私有成員，外部程式碼無法直接訪問它們。

這個特性增強了 JavaScript 類的封裝性和安全性。如以下程式碼：

```javascript
class Person {
  #name; // 聲明私有屬性
  constructor(name) {
    this.#name = name;
  }
  #sayHello() {
    // 聲明私有方法
    console.log(`Hello, ${this.#name}`);
  }
  introduce() {
    this.#sayHello(); // 在類內部可以訪問私有方法
  }
}

let user1 = new Person("Potter");
user1.introduce(); // Hello, Potter
console.log(user1.#name); // error
user1.#sayHello(); // error
```

> 由於 ES12 會受瀏覽器版本限制，可以到 {{<NewTabLink href="https://jsfiddle.net/" title="Jsfiddle">}} 並將 JS 版本更改成 JavaScript1.7，就可以測試 `私有屬性、方法`。

### 繼承 extends/super

在 JS 中，class 可以使用 `extends` 使一個類（子類）繼承另一個類（父類）的屬性和方法，並搭配 `super` 調用父類的構造函式和方法。如以下程式碼：

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHello() {
    console.log(`Hello, ${this.name}`);
  }
}
class Student extends Person {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
  sayAge() {
    console.log(`I'm ${this.age} years old!`);
  }
}
const student = new Student("Potter", 20);
console.log(student.name); // Potter
console.log(student.age); // 20
student.sayHello(); // Hello, Potter
student.sayAge(); // I'm 20 years old!
```

可以發現即使子類 `Student` 並沒有定義 `sayHello` 用法，但通過 `Student` 實例出的對象依舊可以使用父類的 `sayHello`。除此之外，若子類的方法名稱與父類重複，則會覆蓋方法，如以下程式碼：

```javascript
class Person {
  constructor(name) {
    this.name = name;
  }
  sayHello() {
    console.log(`Hello, ${this.name}`);
  }
}
class Student extends Person {
  constructor(name, age) {
    super(name);
    this.age = age;
  }
  sayHello() {
    console.log(`Bye, ${this.name}`);
  }
}
const student = new Student("Potter", 20);
student.sayHello(); // Bye, Potter
```

## 總結

---

結合上述內容，我們可以把以下程式碼

```javascript
function Person(name) {
  this.name = name;
}
Person.prototype.run = function () {
  console.log("run");
};
let user = new Person("Potter");
```

轉成流程圖：

![js-prototype-chain](/images/js/js-prototype-chain.png)

接下來就讓我們來介紹下這幾篇文章經常提及的 `this`。

## 參考文章

---

{{<NewTabLink href="https://juejin.cn/post/7132476402674171940" title="從prototype的設計初衷剖析JS原型和原型鏈">}}  
{{<NewTabLink href="https://github.com/mqyqingfeng/Blog/issues/2" title="JavaScript深入之從原型到原型鏈">}}  
{{<NewTabLink href="https://github.com/mqyqingfeng/Blog/issues/13" title="JavaScript深入之new的模擬實現">}}  
{{<NewTabLink href="https://segmentfault.com/a/1190000022776150" title="JS中的構造函式、原型、原型鏈">}}  
{{<NewTabLink href="https://ithelp.ithome.com.tw/articles/10194356" title="重新認識 JavaScript: Day 25 原型與繼承">}}

{{<NextArticle href="/articles/know-js-again-5-this" article="重新認識 Javascript（五）- this">}}