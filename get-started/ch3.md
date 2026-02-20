# You Don't Know JS Yet: Get Started - 2nd Edition
# 第三章：深入 JS 的根基

如果你已經閱讀了第一章和第二章，並花了時間消化和沉澱，那麼你應該開始對 JS 有了更多的*理解*。如果你跳過或略讀了它們（尤其是第二章），我建議你回過頭去花更多時間研讀那些內容。

在第二章中，我們從較高層面概覽了語法、模式和行為。在本章中，我們的注意力將轉向 JS 的一些更底層的根本特性，這些特性幾乎支撐著我們所寫的每一行程式碼。

請注意：本章所深入的程度可能超過你習慣對一門程式語言所思考的範圍。我的目標是幫助你理解 JS 運作的核心，了解它的驅動機制。本章應該開始回答一些在你探索 JS 時可能浮現的「為什麼？」問題。然而，這些內容仍然不是對語言的詳盡闡述；那是本系列其餘書籍的任務！我們在這裡的目標仍然只是*入門*，並更加熟悉 JS 的*感覺*，了解它如何起伏流動。

不要急於瀏覽這些內容以致於迷失在細節中。正如我已經說過無數次的，**慢慢來**。即便如此，你讀完本章後可能仍會有一些疑問。這沒關係，因為前方還有一整個系列的書籍等著你繼續探索！

## 迭代

由於程式本質上是為了處理資料（並根據資料做出決策），用來逐步處理資料的模式對程式的可讀性有很大的影響。

迭代器模式已經存在了數十年，它提出了一種「標準化」的方法，從資料源一次消費一個*區塊*的資料。其思路是，迭代資料源——逐步處理資料集合，先處理第一部分，然後是下一部分，依此類推——比一次處理整個集合更為常見且有用。

想像一個代表關聯式資料庫 `SELECT` 查詢的資料結構，它通常將結果組織為行。如果這個查詢只有一行或幾行結果，你可以一次處理整個結果集，將每行資料賦值給一個區域變數，並對該資料執行任何適當的操作。

但如果查詢有 100 或 1,000（甚至更多！）行，你就需要迭代處理來應對這些資料（通常使用迴圈）。

迭代器模式定義了一種稱為「迭代器」的資料結構，它持有對底層資料源（如查詢結果行）的參考，並暴露一個類似 `next()` 的方法。呼叫 `next()` 會回傳下一筆資料（即資料庫查詢中的一條「記錄」或「行」）。

你並不總是知道需要迭代多少筆資料，因此該模式通常在你迭代完整個集合並*超過末端*時，透過某個特殊值或例外來表示完成。

迭代器模式的重要性在於遵循一種*標準*的迭代處理資料方式，這能產生更乾淨、更容易理解的程式碼，而不是讓每個資料結構/資料源定義自己的自訂資料處理方式。

在 JS 社群多年來圍繞共同認可的迭代技術所做的各種努力之後，ES6 直接在語言中標準化了迭代器模式的特定協定。該協定定義了一個 `next()` 方法，其回傳值是一個稱為*迭代器結果*的物件；該物件具有 `value` 和 `done` 屬性，其中 `done` 是一個布林值，在底層資料源的迭代完成之前為 `false`。

### 消費迭代器

有了 ES6 的迭代協定，就可以一次消費一個值的資料源，在每次 `next()` 呼叫後檢查 `done` 是否為 `true` 來停止迭代。但這種方式相當手動，因此 ES6 也包含了幾種機制（語法和 API）來標準化消費這些迭代器。

其中一種機制是 `for..of` 迴圈：

```js
// given an iterator of some data source:
var it = /* .. */;

// loop over its results one at a time
for (let val of it) {
    console.log(`Iterator value: ${ val }`);
}
// Iterator value: ..
// Iterator value: ..
// ..
```

| 注意： |
| :--- |
| 我們在此省略了等效的手動迴圈，但它的可讀性肯定不如 `for..of` 迴圈！ |

另一個經常用於消費迭代器的機制是 `...` 運算子。這個運算子實際上有兩種對稱的形式：*展開*和 *rest*（或*收集*，這是我比較偏好的稱呼）。*展開*形式是一種迭代器消費者。

要*展開*一個迭代器，你必須有*某個東西*來承接展開的內容。在 JS 中有兩種可能：一個陣列或一個函式呼叫的引數列表。

陣列展開：

```js
// spread an iterator into an array,
// with each iterated value occupying
// an array element position.
var vals = [ ...it ];
```

函式呼叫展開：

```js
// spread an iterator into a function,
// call with each iterated value
// occupying an argument position.
doSomethingUseful( ...it );
```

在這兩種情況下，`...` 的迭代器展開形式都遵循迭代器消費協定（與 `for..of` 迴圈相同）來從迭代器中取出所有可用的值，並將它們放置（即展開）到接收的上下文中（陣列、引數列表）。

### 可迭代物件

迭代器消費協定在技術上是為消費*可迭代物件*而定義的；可迭代物件是一個可以被迭代的值。

該協定會自動從可迭代物件建立一個迭代器實例，並消費*恰好那個迭代器實例*直到完成。這意味著單一可迭代物件可以被消費多次；每次都會建立並使用一個新的迭代器實例。

那麼我們在哪裡可以找到可迭代物件呢？

ES6 將 JS 中的基本資料結構/集合型別定義為可迭代物件。這包括字串、陣列、map、set 和其他。

考慮以下程式碼：

```js
// an array is an iterable
var arr = [ 10, 20, 30 ];

for (let val of arr) {
    console.log(`Array value: ${ val }`);
}
// Array value: 10
// Array value: 20
// Array value: 30
```

由於陣列是可迭代物件，我們可以使用 `...` 展開運算子透過迭代器消費來淺拷貝一個陣列：

```js
var arrCopy = [ ...arr ];
```

我們也可以逐一迭代字串中的字元：

```js
var greeting = "Hello world!";
var chars = [ ...greeting ];

chars;
// [ "H", "e", "l", "l", "o", " ",
//   "w", "o", "r", "l", "d", "!" ]
```

`Map` 資料結構使用物件作為鍵，將一個值（任何型別）與該物件關聯。Map 的預設迭代與這裡看到的不同，其迭代不僅僅是遍歷 map 的值，而是其*條目*。一個*條目*是一個元組（2 個元素的陣列），包含鍵和值。

考慮以下程式碼：

```js
// given two DOM elements, `btn1` and `btn2`

var buttonNames = new Map();
buttonNames.set(btn1,"Button 1");
buttonNames.set(btn2,"Button 2");

for (let [btn,btnName] of buttonNames) {
    btn.addEventListener("click",function onClick(){
        console.log(`Clicked ${ btnName }`);
    });
}
```

在對預設 map 迭代的 `for..of` 迴圈中，我們使用 `[btn,btnName]` 語法（稱為「陣列解構」）將每個消費的元組拆解為對應的鍵/值對（`btn1` / `"Button 1"` 和 `btn2` / `"Button 2"`）。

JS 中每個內建的可迭代物件都暴露了一個預設迭代方式，這通常符合你的直覺。但如有需要，你也可以選擇更具體的迭代方式。例如，如果我們只想消費上述 `buttonNames` map 的值，我們可以呼叫 `values()` 來取得一個僅包含值的迭代器：

```js
for (let btnName of buttonNames.values()) {
    console.log(btnName);
}
// Button 1
// Button 2
```

或者，如果我們想在陣列迭代中同時取得索引*和*值，我們可以使用 `entries()` 方法來建立一個條目迭代器：

```js
var arr = [ 10, 20, 30 ];

for (let [idx,val] of arr.entries()) {
    console.log(`[${ idx }]: ${ val }`);
}
// [0]: 10
// [1]: 20
// [2]: 30
```

大多數情況下，JS 中所有內建的可迭代物件都有三種可用的迭代器形式：僅鍵（`keys()`）、僅值（`values()`）和條目（`entries()`）。

除了使用內建的可迭代物件之外，你也可以確保自己的資料結構遵循迭代協定；這樣做意味著你選擇讓你的資料能夠透過 `for..of` 迴圈和 `...` 運算子來消費。對這個協定的「標準化」意味著整體上更容易辨識和閱讀的程式碼。

| 注意： |
| :--- |
| 你可能注意到了這個討論中發生的一個微妙轉變。我們開始時談論的是消費**迭代器**，但後來轉而談論迭代**可迭代物件**。迭代消費協定期望的是一個*可迭代物件*，但我們之所以能提供一個直接的*迭代器*，是因為迭代器本身就是自己的可迭代物件！當從一個現有迭代器建立迭代器實例時，會回傳迭代器本身。 |

## 閉包

也許你還沒有意識到，幾乎每個 JS 開發者都使用過閉包。事實上，閉包是跨越大多數語言中最普遍的程式設計功能之一。它的重要性甚至可能與變數或迴圈相當；它就是這麼基礎。

然而它感覺有些隱蔽，近乎神奇。而且人們談論它時往往使用非常抽象或非常不正式的術語，這對於幫助我們精確理解它是什麼作用不大。

我們需要能夠辨識程式中使用閉包的地方，因為閉包的存在與否有時是 bug 的原因（甚至是效能問題的原因）。

所以讓我們以務實且具體的方式來定義閉包：

> 閉包是指一個函式記住並持續存取其作用域之外的變數，即使該函式在不同的作用域中執行。

我們在這裡看到兩個定義性的特徵。首先，閉包是函式本質的一部分。物件不會有閉包，函式才有。其次，要觀察到閉包，你必須在與函式最初定義位置不同的作用域中執行該函式。

考慮以下程式碼：

```js
function greeting(msg) {
    return function who(name) {
        console.log(`${ msg }, ${ name }!`);
    };
}

var hello = greeting("Hello");
var howdy = greeting("Howdy");

hello("Kyle");
// Hello, Kyle!

hello("Sarah");
// Hello, Sarah!

howdy("Grant");
// Howdy, Grant!
```

首先，`greeting(..)` 外部函式被執行，建立了內部函式 `who(..)` 的一個實例；該函式閉包了變數 `msg`，這是來自 `greeting(..)` 外部作用域的參數。當內部函式被回傳時，它的參考被賦值給外部作用域中的 `hello` 變數。然後我們第二次呼叫 `greeting(..)`，建立一個新的內部函式實例，帶有對新 `msg` 的新閉包，並將該參考回傳賦值給 `howdy`。

當 `greeting(..)` 函式執行完畢後，通常我們會預期它的所有變數都被垃圾回收（從記憶體中移除）。我們會預期每個 `msg` 都會消失，但它們並沒有。原因就是閉包。由於內部函式實例仍然存活（分別被賦值給 `hello` 和 `howdy`），它們的閉包仍然保存著 `msg` 變數。

這些閉包不是 `msg` 變數值的快照；它們是對變數本身的直接連結和保存。這意味著閉包實際上可以隨時間觀察（或進行！）對這些變數的更新。

```js
function counter(step = 1) {
    var count = 0;
    return function increaseCount(){
        count = count + step;
        return count;
    };
}

var incBy1 = counter(1);
var incBy3 = counter(3);

incBy1();       // 1
incBy1();       // 2

incBy3();       // 3
incBy3();       // 6
incBy3();       // 9
```

內部 `increaseCount()` 函式的每個實例都閉包了其外部 `counter(..)` 函式作用域中的 `count` 和 `step` 變數。`step` 隨時間保持不變，但 `count` 在每次呼叫內部函式時都會被更新。由於閉包是針對變數而非僅僅是值的快照，這些更新會被保留下來。

閉包在處理非同步程式碼時最為常見，例如使用回呼函式。考慮以下程式碼：

```js
function getSomeData(url) {
    ajax(url,function onResponse(resp){
        console.log(
            `Response (from ${ url }): ${ resp }`
        );
    });
}

getSomeData("https://some.url/wherever");
// Response (from https://some.url/wherever): ...
```

內部函式 `onResponse(..)` 閉包了 `url`，因此保存並記住了它，直到 Ajax 呼叫回傳並執行 `onResponse(..)`。即使 `getSomeData(..)` 立即完成，`url` 參數變數仍會因閉包而保持存活，直到不再需要為止。

外部作用域不一定要是一個函式——通常是，但不總是——只需要有至少一個外部作用域中的變數被內部函式存取即可：

```js
for (let [idx,btn] of buttons.entries()) {
    btn.addEventListener("click",function onClick(){
       console.log(`Clicked on button (${ idx })!`);
    });
}
```

因為這個迴圈使用了 `let` 宣告，每次迭代都會獲得新的區塊作用域（即區域性的）`idx` 和 `btn` 變數；迴圈也會每次建立一個新的內部 `onClick(..)` 函式。那個內部函式閉包了 `idx`，只要點擊處理器設定在 `btn` 上就會一直保留它。所以當每個按鈕被點擊時，它的處理器可以印出其關聯的索引值，因為處理器記住了各自的 `idx` 變數。

記住：這個閉包不是針對值（如 `1` 或 `3`），而是針對變數 `idx` 本身。

閉包是任何語言中最普遍和最重要的程式設計模式之一。但在 JS 中尤其如此；很難想像不以某種方式利用閉包就能做出任何有用的事情。

如果你對閉包仍然感到不清楚或不確定，第二冊《*Scope & Closures*》的大部分內容都聚焦在這個主題上。

## `this` 關鍵字

JS 最強大的機制之一，也是最容易被誤解的：`this` 關鍵字。一個常見的誤解是函式的 `this` 指向函式本身。由於 `this` 在其他語言中的運作方式，另一個誤解是 `this` 指向方法所屬的實例。兩者都是錯誤的。

如前所述，當一個函式被定義時，它透過閉包*附著*在其外圍的作用域上。作用域是一組規則，控制如何解析變數的參考。

但函式除了作用域之外還有另一個特性，影響著它們能存取什麼。這個特性最好被描述為一個*執行上下文*，它透過 `this` 關鍵字暴露給函式。

作用域是靜態的，包含你定義函式時那一刻和那個位置可用的一組固定變數，但函式的執行*上下文*是動態的，完全取決於**它是如何被呼叫的**（無論它是在哪裡定義或從哪裡被呼叫的）。

`this` 不是基於函式定義的固定特性，而是在每次函式被呼叫時動態決定的特性。

思考*執行上下文*的一種方式是，它是一個有形的物件，其屬性在函式執行時可供函式使用。與之對比的是作用域，它也可以被視為一個*物件*；不過，*作用域物件*隱藏在 JS 引擎內部，它對於該函式始終是相同的，而其*屬性*以函式內部可用的識別符號變數的形式呈現。

```js
function classroom(teacher) {
    return function study() {
        console.log(
            `${ teacher } says to study ${ this.topic }`
        );
    };
}
var assignment = classroom("Kyle");
```

外部 `classroom(..)` 函式沒有參考 `this` 關鍵字，所以它就像我們目前所見的其他函式一樣。但內部 `study()` 函式確實參考了 `this`，這使它成為一個 `this` 感知函式。換句話說，它是一個依賴其*執行上下文*的函式。

| 注意： |
| :--- |
| `study()` 也閉包了來自其外部作用域的 `teacher` 變數。 |

由 `classroom("Kyle")` 回傳的內部 `study()` 函式被賦值給一個名為 `assignment` 的變數。那麼 `assignment()`（即 `study()`）要如何被呼叫呢？

```js
assignment();
// Kyle says to study undefined  -- Oops :(
```

在這段程式碼中，我們將 `assignment()` 作為一個普通的函式呼叫，沒有為它提供任何*執行上下文*。

由於這個程式不在嚴格模式下（參見第一章「嚴格來說」），被呼叫時**沒有指定任何上下文**的上下文感知函式會將上下文預設為全域物件（在瀏覽器中是 `window`）。由於沒有名為 `topic` 的全域變數（因此全域物件上也沒有這樣的屬性），`this.topic` 解析為 `undefined`。

現在考慮以下程式碼：

```js
var homework = {
    topic: "JS",
    assignment: assignment
};

homework.assignment();
// Kyle says to study JS
```

`assignment` 函式參考的一個副本被設為 `homework` 物件上的一個屬性，然後以 `homework.assignment()` 的形式呼叫。這意味著該函式呼叫的 `this` 將是 `homework` 物件。因此，`this.topic` 解析為 `"JS"`。

最後：

```js
var otherHomework = {
    topic: "Math"
};

assignment.call(otherHomework);
// Kyle says to study Math
```

呼叫函式的第三種方式是使用 `call(..)` 方法，它接受一個物件（這裡是 `otherHomework`）用於設定函式呼叫的 `this` 參考。屬性參考 `this.topic` 解析為 `"Math"`。

同一個上下文感知函式以三種不同方式呼叫，每次對於 `this` 將參考哪個物件都給出不同的答案。

`this` 感知函式——及其動態上下文——的好處在於能夠更靈活地以來自不同物件的資料重複使用單一函式。閉包了某個作用域的函式永遠無法參考不同的作用域或變數集合。但具有動態 `this` 上下文感知的函式對於某些任務可能相當有用。

## 原型

`this` 是函式執行的特性，而原型是物件的特性，特別是屬性存取的解析方面。

將原型視為兩個物件之間的連結；這個連結隱藏在幕後，但有辦法暴露和觀察它。原型連結在物件建立時發生；它連結到另一個已經存在的物件。

透過原型連結在一起的一系列物件稱為「原型鏈」。

這個原型連結的目的（即從物件 B 到另一個物件 A）是為了讓對 B 的屬性/方法存取——如果 B 本身沒有——能被*委託*給 A 來處理。屬性/方法存取的委託允許兩個（或更多！）物件相互合作來完成一個任務。

考慮定義一個物件作為普通的字面值：

```js
var homework = {
    topic: "JS"
};
```

`homework` 物件上只有一個屬性：`topic`。然而，它的預設原型連結連接到 `Object.prototype` 物件，該物件上有常見的內建方法，如 `toString()` 和 `valueOf()` 等。

我們可以觀察到從 `homework` 到 `Object.prototype` 的原型連結*委託*：

```js
homework.toString();    // [object Object]
```

`homework.toString()` 能運作，即使 `homework` 上沒有定義 `toString()` 方法；委託呼叫了 `Object.prototype.toString()` 來代替。

### 物件連結

要定義物件的原型連結，你可以使用 `Object.create(..)` 工具來建立物件：

```js
var homework = {
    topic: "JS"
};

var otherHomework = Object.create(homework);

otherHomework.topic;   // "JS"
```

`Object.create(..)` 的第一個引數指定了要將新建立的物件連結到的物件，然後回傳新建立的（並已連結的！）物件。

圖 4 展示了三個物件（`otherHomework`、`homework` 和 `Object.prototype`）如何在原型鏈中連結：

<figure>
    <img src="images/fig4.png" width="200" alt="Prototype chain with 3 objects" align="center">
    <figcaption><em>圖 4：原型鏈中的物件</em></figcaption>
    <br><br>
</figure>

透過原型鏈的委託僅適用於查詢屬性值的存取。如果你對一個物件的屬性進行賦值，那將直接作用於該物件，無論該物件的原型連結到哪裡。

| 提示： |
| :--- |
| `Object.create(null)` 建立一個不連結到任何原型的物件，所以它純粹就是一個獨立的物件；在某些情況下，這可能是更好的選擇。 |

考慮以下程式碼：

```js
homework.topic;
// "JS"

otherHomework.topic;
// "JS"

otherHomework.topic = "Math";
otherHomework.topic;
// "Math"

homework.topic;
// "JS" -- not "Math"
```

對 `topic` 的賦值直接在 `otherHomework` 上建立了一個同名屬性；對 `homework` 上的 `topic` 屬性沒有影響。接下來的陳述式存取 `otherHomework.topic`，我們看到來自那個新屬性的非委託回答：`"Math"`。

圖 5 展示了在賦值建立 `otherHomework.topic` 屬性之後的物件/屬性：

<figure>
    <img src="images/fig5.png" width="200" alt="3 objects linked, with shadowed property" align="center">
    <figcaption><em>圖 5：被遮蔽的屬性 'topic'</em></figcaption>
    <br><br>
</figure>

`otherHomework` 上的 `topic` 正在「遮蔽」原型鏈中 `homework` 物件上的同名屬性。

| 注意： |
| :--- |
| 另一種坦白說更迂迴但可能仍然更常見的建立具有原型連結的物件方式是使用「原型類別」模式，這是在 ES6 添加 `class`（參見第二章「類別」）之前的做法。我們將在附錄 A「原型『類別』」中更詳細地介紹這個主題。 |

### 重新審視 `this`

我們之前介紹了 `this` 關鍵字，但它的真正重要性在考慮它如何驅動原型委託的函式呼叫時才能展現。事實上，`this` 支援基於函式呼叫方式的動態上下文的主要原因之一，就是為了讓透過原型鏈委託的物件上的方法呼叫仍然維持預期的 `this`。

考慮以下程式碼：

```js
var homework = {
    study() {
        console.log(`Please study ${ this.topic }`);
    }
};

var jsHomework = Object.create(homework);
jsHomework.topic = "JS";
jsHomework.study();
// Please study JS

var mathHomework = Object.create(homework);
mathHomework.topic = "Math";
mathHomework.study();
// Please study Math
```

`jsHomework` 和 `mathHomework` 兩個物件各自透過原型連結到單一的 `homework` 物件，後者擁有 `study()` 函式。`jsHomework` 和 `mathHomework` 各自被賦予了自己的 `topic` 屬性（參見圖 6）。

<figure>
    <img src="images/fig6.png" width="495" alt="4 objects prototype linked" align="center">
    <figcaption><em>圖 6：兩個物件連結到一個共同的父物件</em></figcaption>
    <br><br>
</figure>

`jsHomework.study()` 委託到 `homework.study()`，但該執行中的 `this`（`this.topic`）解析為 `jsHomework`，因為函式是以這種方式被呼叫的，所以 `this.topic` 是 `"JS"`。類似地，`mathHomework.study()` 委託到 `homework.study()`，但仍然將 `this` 解析為 `mathHomework`，因此 `this.topic` 為 `"Math"`。

如果 `this` 被解析為 `homework`，前面的程式碼片段將會遠不那麼有用。然而，在許多其他語言中，`this` 似乎會是 `homework`，因為 `study()` 方法確實定義在 `homework` 上。

與許多其他語言不同，JS 的 `this` 是動態的，這是允許原型委託以及 `class` 按預期運作的關鍵組成部分！

## 問「為什麼？」

本章要帶給你的核心收穫是，JS 的底層比表面上看到的要豐富得多。

當你*開始*更深入地學習和了解 JS 時，你可以練習和加強的最重要技能之一就是好奇心，以及在遇到語言中的某些東西時問「為什麼？」的藝術。

儘管本章已經對一些主題進行了相當深入的探討，但許多細節仍然完全被略過了。還有很多東西需要學習，而通往那裡的路始於你對你的程式碼提出*正確的*問題。提出正確的問題是成為更好開發者的關鍵技能。

在本書的最後一章中，我們將簡要介紹 JS 是如何劃分的，正如本系列其餘的 *You Don't Know JS Yet* 書籍所涵蓋的那樣。另外，不要跳過本書的附錄 B，其中有一些練習程式碼，用於複習本書所涵蓋的一些主要主題。
