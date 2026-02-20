# You Don't Know JS Yet: Get Started - 2nd Edition
# 附錄 A：深入探索

在本附錄中，我們將更詳細地探索主要章節文本中的一些主題。請將此內容視為對本書系列其餘部分中涵蓋的更多細微細節的可選預覽。

## 值 vs. 參考

在第 2 章中，我們介紹了兩種主要的值型別：原始型別（primitives）和物件。但我們尚未討論兩者之間的一個關鍵差異：這些值如何被賦值和傳遞。

在許多語言中，開發者可以選擇以值本身或以值的參考（reference）來賦值/傳遞。然而在 JS 中，這個決定完全取決於值的種類。這讓許多來自其他語言的開發者在開始使用 JS 時感到驚訝。

如果你賦值/傳遞的是值本身，該值會被複製。例如：

```js
var myName = "Kyle";

var yourName = myName;
```

在這裡，`yourName` 變數有一份從 `myName` 中儲存的值複製過來的獨立 `"Kyle"` 字串副本。這是因為該值是原始型別，而原始型別的值總是以**值複製**的方式被賦值/傳遞。

以下是你可以證明涉及兩個獨立值的方式：

```js
var myName = "Kyle";

var yourName = myName;

myName = "Frank";

console.log(myName);
// Frank

console.log(yourName);
// Kyle
```

看到 `yourName` 沒有受到 `myName` 被重新賦值為 `"Frank"` 的影響了嗎？這是因為每個變數都持有自己的值副本。

相比之下，參考是指兩個或多個變數指向同一個值的概念，因此修改這個共享的值會透過任何一個參考的存取而反映出來。在 JS 中，只有物件值（陣列、物件、函式等）會被當作參考來處理。

考慮以下範例：

```js
var myAddress = {
    street: "123 JS Blvd",
    city: "Austin",
    state: "TX"
};

var yourAddress = myAddress;

// I've got to move to a new house!
myAddress.street = "456 TS Ave";

console.log(yourAddress.street);
// 456 TS Ave
```

因為賦值給 `myAddress` 的值是一個物件，它是以參考的方式持有/賦值的，因此賦值給 `yourAddress` 變數的是參考的副本，而不是物件值本身。這就是為什麼更新賦值給 `myAddress.street` 的值會在我們存取 `yourAddress.street` 時反映出來。`myAddress` 和 `yourAddress` 持有的是指向同一個共享物件的參考副本，所以對其中一個的更新就是對兩者的更新。

再次強調，JS 根據值的型別來選擇值複製或參考複製的行為。原始型別以值持有，物件以參考持有。在 JS 中無法覆寫這個行為，無論哪個方向都不行。

## 如此多的函式形式

回想一下第 2 章「函式」一節中的這段程式碼：

```js
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```

這裡的函式表達式被稱為*匿名函式表達式*，因為在 `function` 關鍵字和 `(..)` 參數列表之間沒有名稱識別符。這一點讓許多 JS 開發者感到困惑，因為從 ES6 開始，JS 會對匿名函式執行「名稱推斷」：

```js
awesomeFunction.name;
// "awesomeFunction"
```

函式的 `name` 屬性會顯示其直接給定的名稱（在宣告的情況下）或其在匿名函式表達式情況下的推斷名稱。該值通常被開發者工具在檢查函式值或報告錯誤堆疊追蹤時使用。

所以即使是匿名函式表達式也*可能*獲得一個名稱。然而，名稱推斷只在有限的情況下發生，例如當函式表達式被賦值（使用 `=`）時。如果你將函式表達式作為引數傳遞給函式呼叫，例如，就不會發生名稱推斷；`name` 屬性將是空字串，開發者控制台通常會報告「(anonymous function)」。

即使名稱被推斷了，**它仍然是一個匿名函式。**為什麼？因為推斷的名稱是一個中繼資料字串值，而不是可用來參考該函式的識別符。匿名函式沒有識別符可以從自身內部參考自己——用於遞迴、事件解除綁定等。

將匿名函式表達式形式與以下進行比較：

```js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function someName(coolThings) {
    // ..
    return amazingStuff;
};

awesomeFunction.name;
// "someName"
```

這個函式表達式是一個*具名函式表達式*，因為識別符 `someName` 在編譯時就直接與函式表達式關聯；而與識別符 `awesomeFunction` 的關聯要到執行時該陳述式執行時才會發生。這兩個識別符不必相同；有時候讓它們不同是有意義的，有時候讓它們相同則更好。

還要注意，顯式的函式名稱，即識別符 `someName`，在為 `name` 屬性賦值*名稱*時具有優先權。

函式表達式應該是具名的還是匿名的？對此意見分歧很大。大多數開發者傾向於不在意使用匿名函式。它們更短，而且在廣泛的 JS 程式碼領域中無疑更為常見。

在我看來，如果一個函式存在於你的程式中，它就有其目的；否則，就把它移除！如果它有目的，它就有一個描述該目的的自然名稱。

如果一個函式有名稱，身為程式碼作者的你應該在程式碼中包含該名稱，這樣讀者就不必透過閱讀和在心中執行該函式的原始碼來推斷名稱。即使是像 `x * 2` 這樣瑣碎的函式體，也必須被閱讀才能推斷出像「double」或「multBy2」這樣的名稱；當你只需花一秒鐘將函式命名為「double」或「multBy2」*一次*，就能在未來每次閱讀時省去讀者反覆的心智負擔，這份簡短的額外心智工作是不必要的。

遺憾的是在某些方面，截至 2020 年初，JS 中有許多其他的函式定義形式（未來可能更多！）。

以下是更多的宣告形式：

```js
// generator function declaration
function *two() { .. }

// async function declaration
async function three() { .. }

// async generator function declaration
async function *four() { .. }

// named function export declaration (ES6 modules)
export function five() { .. }
```

以下是更多的（很多！）函式表達式形式：

```js
// IIFE
(function(){ .. })();
(function namedIIFE(){ .. })();

// asynchronous IIFE
(async function(){ .. })();
(async function namedAIIFE(){ .. })();

// arrow function expressions
var f;
f = () => 42;
f = x => x * 2;
f = (x) => x * 2;
f = (x,y) => x * y;
f = x => ({ x: x * 2 });
f = x => { return x * 2; };
f = async x => {
    var y = await doSomethingAsync(x);
    return y * 2;
};
someOperation( x => x * 2 );
// ..
```

請記住，箭頭函式表達式在**語法上是匿名的**，這意味著語法不提供直接為函式提供名稱識別符的方式。函式表達式可能會獲得一個推斷的名稱，但只有在它是賦值形式時，而不是在（更常見的！）作為函式呼叫引數傳遞的形式中（如程式碼片段的最後一行）。

由於我認為在程式中頻繁使用匿名函式不是好主意，我不太喜歡使用 `=>` 箭頭函式形式。這種函式實際上有其特定用途（即以詞法方式處理 `this` 關鍵字），但這並不意味著我們應該將它用於我們編寫的每個函式。為每項工作使用最合適的工具。

函式也可以在類別定義和物件字面值定義中指定。當以這些形式出現時，它們通常被稱為「方法」，儘管在 JS 中這個術語與「函式」之間沒有太多可觀察的差異：

```js
class SomethingKindaGreat {
    // class methods
    coolMethod() { .. }   // no commas!
    boringMethod() { .. }
}

var EntirelyDifferent = {
    // object methods
    coolMethod() { .. },   // commas!
    boringMethod() { .. },

    // (anonymous) function expression property
    oldSchool: function() { .. }
};
```

呼！定義函式的方式真是太多了。

這裡沒有簡單的捷徑；你只需要熟悉所有的函式形式，這樣你才能在現有的程式碼中識別它們，並在你編寫的程式碼中適當地使用它們。仔細研究它們並加以練習！

## 強制轉型條件比較

是的，這個小節名稱確實很拗口。但我們在討論什麼？我們在討論條件表達式需要執行強制轉型導向的比較來做出決策。

`if` 和 `? :`三元陳述式，以及 `while` 和 `for` 迴圈中的測試子句，都會執行隱式的值比較。但是哪種比較？是「嚴格的」還是「強制轉型的」？其實兩者都有。

考慮以下範例：

```js
var x = 1;

if (x) {
    // will run!
}

while (x) {
    // will run, once!
    x = false;
}
```

你可能會這樣思考這些 `(x)` 條件表達式：

```js
var x = 1;

if (x == true) {
    // will run!
}

while (x == true) {
    // will run, once!
    x = false;
}
```

在這個特定的情況下——`x` 的值為 `1`——這個心智模型是有效的，但它在更廣泛的範圍內並不準確。考慮：

```js
var x = "hello";

if (x) {
    // will run!
}

if (x == true) {
    // won't run :(
}
```

糟糕。那麼 `if` 陳述式實際上在做什麼？這是更準確的心智模型：

```js
var x = "hello";

if (Boolean(x) == true) {
    // will run
}

// which is the same as:

if (Boolean(x) === true) {
    // will run
}
```

由於 `Boolean(..)` 函式總是回傳一個布林型別的值，在這段程式碼中 `==` 與 `===` 是無關緊要的；它們都會做同樣的事情。但重要的部分是看到在比較之前，會發生一次強制轉型，將 `x` 當前的型別轉換為布林值。

你在 JS 的比較中就是無法避免強制轉型。下定決心好好學習它們吧。

## 原型「類別」

在第 3 章中，我們介紹了原型，並展示了如何透過原型鏈來連結物件。

另一種建立這種原型連結的方式，是 ES6 `class` 系統（見第 2 章「類別」）的優雅設計的前身（說實話，相當醜陋），被稱為原型類別。

| 提示： |
| :--- |
| 雖然這種風格的程式碼在如今的 JS 中相當少見，但在求職面試中被問到卻仍然出奇地常見！ |

讓我們先回顧 `Object.create(..)` 風格的寫法：

```js
var Classroom = {
    welcome() {
        console.log("Welcome, students!");
    }
};

var mathClass = Object.create(Classroom);

mathClass.welcome();
// Welcome, students!
```

在這裡，`mathClass` 物件透過其原型連結到 `Classroom` 物件。透過這個連結，函式呼叫 `mathClass.welcome()` 被委派到定義在 `Classroom` 上的方法。

原型類別模式會將這種委派行為稱為「繼承」，並以替代方式（具有相同行為）定義如下：

```js
function Classroom() {
    // ..
}

Classroom.prototype.welcome = function hello() {
    console.log("Welcome, students!");
};

var mathClass = new Classroom();

mathClass.welcome();
// Welcome, students!
```

所有函式預設都在名為 `prototype` 的屬性上參考一個空物件。儘管命名令人困惑，這**不是**函式的*原型*（函式的原型連結所指向的），而是當其他物件透過使用 `new` 呼叫該函式而建立時要*連結到*的原型物件。

我們在那個空物件（稱為 `Classroom.prototype`）上新增一個 `welcome` 屬性，指向 `hello()` 函式。

然後 `new Classroom()` 建立一個新物件（賦值給 `mathClass`），並將其原型連結到現有的 `Classroom.prototype` 物件。

雖然 `mathClass` 沒有 `welcome()` 屬性/函式，它成功地委派到了 `Classroom.prototype.welcome()` 函式。

這種「原型類別」模式現在已經被強烈不建議使用，取而代之的是使用 ES6 的 `class` 機制：

```js
class Classroom {
    constructor() {
        // ..
    }

    welcome() {
        console.log("Welcome, students!");
    }
}

var mathClass = new Classroom();

mathClass.welcome();
// Welcome, students!
```

在底層，同樣的原型連結被建立起來，但這個 `class` 語法比「原型類別」更加優雅地符合類別導向的設計模式。
