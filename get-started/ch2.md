# You Don't Know JS Yet: Get Started - 2nd Edition
# 第二章：概覽 JS

學習 JS 最好的方式就是開始撰寫 JS。

要做到這一點，你需要知道這門語言是如何運作的，而這就是我們在這裡要專注的內容。即使你之前已經用其他語言寫過程式，也請花時間熟悉 JS，並確保實際練習每個部分。

這一章不是 JS 語言每一個語法的詳盡參考。它也不打算成為一份完整的「JS 入門」指南。

相反地，我們只是要概覽這門語言的一些主要主題領域。我們的目標是對它有更好的*感覺*，這樣我們就能更有信心地繼續撰寫自己的程式。隨著你閱讀本書的其餘部分以及整個系列，我們會逐步更深入地重新探討這些主題。

請不要期望這一章能快速讀完。它很長，而且有很多細節需要消化。請慢慢來。

| 提示： |
| :--- |
| 如果你還在熟悉 JS，我建議你預留充足的額外時間來研讀這一章。針對每個小節，花些時間思考和探索該主題。瀏覽現有的 JS 程式，並將你在其中看到的內容與這裡呈現的程式碼和解釋（以及觀點！）進行比較。有了 JS *本質* 的紮實基礎，你將從本書和系列的其餘部分中獲得更多收穫。 |

## 每個檔案都是一個程式

幾乎你使用的每個網站（網頁應用程式）都由許多不同的 JS 檔案組成（通常使用 .js 副檔名）。很容易把整個東西（應用程式）想成一個程式。但 JS 有不同的看法。

在 JS 中，每個獨立的檔案都是自己獨立的程式。

這之所以重要，主要是因為錯誤處理。由於 JS 將檔案視為程式，一個檔案可能會失敗（在解析／編譯或執行期間），而這不一定會阻止下一個檔案的處理。顯然，如果你的應用程式依賴五個 .js 檔案，而其中一個失敗了，整體應用程式充其量可能只能部分運作。重要的是確保每個檔案都能正確運作，並且盡可能地優雅處理其他檔案的失敗。

你可能會驚訝地認為分開的 .js 檔案是分開的 JS 程式。從你使用應用程式的角度來看，它確實看起來像是一個大程式。那是因為應用程式的執行允許這些個別的*程式*合作並作為一個程式運行。

| 注意： |
| :--- |
| 許多專案使用建置工具，最終將專案中的分開檔案合併成單一檔案以傳遞到網頁。當這種情況發生時，JS 將這個合併後的單一檔案視為整個程式。 |

多個獨立的 .js 檔案作為單一程式運作的唯一方式是透過「全域作用域」共享它們的狀態（以及對其公開功能的存取）。它們在這個全域作用域命名空間中混合在一起，因此在執行時期它們作為一個整體運作。

自 ES6 以來，JS 除了典型的獨立 JS 程式格式之外，還支援了模組格式。模組也是基於檔案的。如果一個檔案透過模組載入機制（如 `import` 陳述式或 `<script type=module>` 標籤）載入，其所有程式碼都被視為單一模組。

雖然你通常不會把一個模組——一組狀態和公開暴露的方法來操作該狀態——想成一個獨立程式，但 JS 實際上仍然分開處理每個模組。類似於「全域作用域」允許獨立檔案在執行時期混合在一起，將一個模組匯入另一個模組允許它們之間的執行時期互操作。

無論檔案使用哪種程式碼組織模式（以及載入機制）（獨立或模組），你仍然應該把每個檔案想成自己的（迷你）程式，它可能與其他（迷你）程式合作來執行你整體應用程式的功能。

## 值

程式中最基本的資訊單位是值。值就是資料。它們是程式維護狀態的方式。值在 JS 中有兩種形式：**原始值（primitive）**和**物件**。

值透過*字面值（literals）*嵌入在程式中：

```js
greeting("My name is Kyle.");
```

在這個程式中，值 `"My name is Kyle."` 是一個原始字串字面值；字串是字元的有序集合，通常用來表示單詞和句子。

我使用了雙引號 `"` 字元來*定界*（圍繞、分隔、定義）字串值。但我也可以使用單引號 `'` 字元。選擇哪個引號字元完全是風格問題。重要的是，為了程式碼的可讀性和可維護性，選一個並在整個程式中一致地使用它。

定界字串字面值的另一個選擇是使用反引號 `` ` `` 字元。然而，這個選擇不僅僅是風格問題；還有行為上的差異。考慮以下範例：

```js
console.log("My name is ${ firstName }.");
// My name is ${ firstName }.

console.log('My name is ${ firstName }.');
// My name is ${ firstName }.

console.log(`My name is ${ firstName }.`);
// My name is Kyle.
```

假設這個程式已經定義了一個變數 `firstName`，其字串值為 `"Kyle"`，那麼 `` ` `` 定界的字串就會將變數表達式（以 `${ .. }` 表示）解析為其當前值。這稱為**插值（interpolation）**。

反引號 `` ` `` 定界的字串可以在不包含插值表達式的情況下使用，但這就失去了那個替代字串字面值語法的全部意義：

```js
console.log(
    `Am I confusing you by omitting interpolation?`
);
// Am I confusing you by omitting interpolation?
```

較好的做法是對字串使用 `"` 或 `'`（同樣，選一個並堅持使用！），*除非你需要*插值；僅在字串將包含插值表達式時才使用 `` ` ``。

除了字串之外，JS 程式通常還包含其他原始字面值，例如布林值和數字：

```js
while (false) {
    console.log(3.141592);
}
```

`while` 代表一種迴圈型別，一種在其條件為真時重複操作的方式。

在這個例子中，迴圈永遠不會執行（也不會印出任何東西），因為我們使用了 `false` 布林值作為迴圈條件。`true` 則會導致迴圈永遠持續下去，所以要小心！

數字 `3.141592` 是，如你可能知道的，數學 PI 的前六位近似值。然而，與其嵌入這樣的值，你通常會使用預定義的 `Math.PI` 值來達到該目的。數字的另一個變體是 `bigint`（大整數）原始型別，用於儲存任意大的數字。

數字在程式中最常用於計算步驟，例如迴圈疊代，以及存取數字位置中的資訊（即陣列索引）。我們稍後會介紹陣列／物件，但作為一個例子，如果有一個名為 `names` 的陣列，我們可以像這樣存取其第二個位置的元素：

```js
console.log(`My name is ${ names[1] }.`);
// My name is Kyle.
```

我們使用 `1` 來存取第二個位置的元素，而不是 `2`，因為像大多數程式語言一樣，JS 陣列索引是從 0 開始的（`0` 是第一個位置）。

除了字串、數字和布林值之外，JS 程式中還有兩個其他*原始*值：`null` 和 `undefined`。雖然它們之間存在差異（一些是歷史性的，一些是當代的），但在大多數情況下，這兩個值都用於表示值的*空*（或缺失）。

許多開發者偏好以一致的方式處理它們，也就是說，這些值被假設為不可區分的。如果小心處理，這通常是可能的。然而，最安全和最好的做法是只使用 `undefined` 作為單一的空值，即使 `null` 看起來更吸引人，因為它打起來更短！

```js
while (value != undefined) {
    console.log("Still got something!");
}
```

最後一個需要知道的原始值是 symbol，它是一種特殊用途的值，行為像一個隱藏的不可猜測的值。Symbol 幾乎專門用作物件上的特殊鍵：

```js
hitchhikersGuide[ Symbol("meaning of life") ];
// 42
```

你在典型的 JS 程式中不會經常遇到 symbol 的直接使用。它們主要用於底層程式碼，例如函式庫和框架中。

### 陣列與物件

除了原始值之外，JS 中的另一種值型別是物件值。

如前所述，陣列是一種特殊的物件型別，由有序的、數值索引的資料列表組成：

```js
var names = [ "Frank", "Kyle", "Peter", "Susan" ];

names.length;
// 4

names[0];
// Frank

names[1];
// Kyle
```

JS 陣列可以容納任何值型別，無論是原始值或物件（包括其他陣列）。正如我們將在第三章末尾看到的，甚至函式也是可以存放在陣列或物件中的值。

| 注意： |
| :--- |
| 函式，就像陣列一樣，是物件的一種特殊類型（亦即子型別）。我們稍後會更詳細地介紹函式。 |

物件更為通用：一個無序的、以鍵為索引的各種值集合。換句話說，你透過字串位置名稱（亦稱「鍵」或「屬性」）來存取元素，而不是透過其數字位置（如陣列那樣）。例如：

```js
var me = {
    first: "Kyle",
    last: "Simpson",
    age: 39,
    specialties: [ "JS", "Table Tennis" ]
};

console.log(`My name is ${ me.first }.`);
```

這裡，`me` 代表一個物件，而 `first` 代表該物件（值集合）中一個資訊位置的名稱。另一種存取物件中資訊的語法選擇是使用方括號 `[ ]`，例如 `me["first"]`。

### 值的型別判定

為了區分值，`typeof` 運算子會告訴你其內建型別（如果是原始值），否則返回 `"object"`：

```js
typeof 42;                  // "number"
typeof "abc";               // "string"
typeof true;                // "boolean"
typeof undefined;           // "undefined"
typeof null;                // "object" -- oops, bug!
typeof { "a": 1 };          // "object"
typeof [1,2,3];             // "object"
typeof function hello(){};  // "function"
```

| 警告： |
| :--- |
| `typeof null` 不幸地返回 `"object"` 而不是預期的 `"null"`。此外，`typeof` 對函式返回特定的 `"function"`，但對陣列卻不是返回預期的 `"array"`。 |

從一種值型別轉換到另一種，例如從字串轉換到數字，在 JS 中稱為「強制轉型」。我們將在本章稍後更詳細地介紹這個主題。

原始值和物件值在被賦值或傳遞時表現不同。我們將在附錄 A 的「值與參考」中介紹這些細節。

## 宣告和使用變數

為了明確說明前一節中可能不太明顯的事情：在 JS 程式中，值可以作為字面值出現（如前面許多範例所示），或者它們可以存放在變數中；將變數想成只是值的容器。

變數必須被宣告（建立）才能使用。有各種語法形式可以宣告變數（亦即「識別碼」），每種形式都有不同的隱含行為。

例如，考慮 `var` 陳述式：

```js
var myName = "Kyle";
var age;
```

`var` 關鍵字宣告一個變數，以在程式的該部分中使用，並可選擇性地允許初始賦值。

另一個類似的關鍵字是 `let`：

```js
let myName = "Kyle";
let age;
```

`let` 關鍵字與 `var` 有一些差異，最明顯的是 `let` 允許對變數的存取比 `var` 更為有限。這稱為「區塊作用域」，相對於一般的或函式作用域。

考慮以下範例：

```js
var adult = true;

if (adult) {
    var myName = "Kyle";
    let age = 39;
    console.log("Shhh, this is a secret!");
}

console.log(myName);
// Kyle

console.log(age);
// Error!
```

嘗試在 `if` 陳述式外部存取 `age` 會導致錯誤，因為 `age` 被區塊作用域限制在 `if` 中，而 `myName` 則沒有。

區塊作用域對於限制變數宣告在程式中的傳播範圍非常有用，這有助於防止變數名稱的意外重疊。

但 `var` 仍然是有用的，因為它傳達了「這個變數將被更廣的作用域（整個函式）看到」的訊息。兩種宣告形式都可以在程式的任何部分中適當使用，取決於具體情況。

| 注意： |
| :--- |
| 很常見的建議是應該避免使用 `var` 而改用 `let`（或 `const`！），通常是因為認為 `var` 從 JS 開始以來的作用域行為令人困惑。我認為這是過度限制性的建議，最終是無益的。它假設你無法學習和正確使用一個功能與其他功能的組合。我相信你*可以*也*應該*學習所有可用的功能，並在適當的地方使用它們！ |

第三種宣告形式是 `const`。它像 `let`，但有一個額外的限制，即它必須在宣告時就賦予一個值，且之後不能被重新賦值為不同的值。

考慮以下範例：

```js
const myBirthday = true;
let age = 39;

if (myBirthday) {
    age = age + 1;    // OK!
    myBirthday = false;  // Error!
}
```

`myBirthday` 常數不允許被重新賦值。

用 `const` 宣告的變數不是「不可改變的」，它們只是不能被重新賦值。使用 `const` 來宣告物件值是不明智的，因為那些值仍然可以被改變，即使變數不能被重新賦值。這會導致日後的潛在混淆，所以我認為明智的做法是避免像這樣的情況：

```js
const actors = [
    "Morgan Freeman", "Jennifer Aniston"
];

actors[2] = "Tom Cruise";   // OK :(
actors = [];                // Error!
```

`const` 最佳的語義用途是當你有一個簡單的原始值，而你想給它一個有用的名稱時，例如使用 `myBirthday` 而不是 `true`。這使程式更容易閱讀。

| 提示： |
| :--- |
| 如果你堅持只對原始值使用 `const`，你就能避免重新賦值（不允許）與突變（允許）之間的任何混淆！這是使用 `const` 最安全和最好的方式。 |

除了 `var` / `let` / `const` 之外，還有其他語法形式可以在各種作用域中宣告識別碼（變數）。例如：

```js
function hello(myName) {
    console.log(`Hello, ${ myName }.`);
}

hello("Kyle");
// Hello, Kyle.
```

識別碼 `hello` 在外部作用域中被建立，它也會自動關聯以參考該函式。但具名參數 `myName` 只在函式內部被建立，因此只能在該函式的作用域內存取。`hello` 和 `myName` 的行為通常類似於用 `var` 宣告的變數。

另一個宣告變數的語法是 `catch` 子句：

```js
try {
    someError();
}
catch (err) {
    console.log(err);
}
```

`err` 是一個區塊作用域變數，只存在於 `catch` 子句內部，就好像它是用 `let` 宣告的一樣。

## 函式

「函式」這個詞在程式設計中有多種含義。例如，在函式程式設計（Functional Programming）的世界中，「函式」有精確的數學定義，並暗示一套嚴格的規則需要遵守。

在 JS 中，我們應該將「函式」理解為另一個相關術語的更廣泛含義：「程序（procedure）」。程序是一組可以被呼叫一次或多次的陳述式，可能被提供一些輸入，也可能回傳一個或多個輸出。

從 JS 的早期開始，函式定義看起來像這樣：

```js
function awesomeFunction(coolThings) {
    // ..
    return amazingStuff;
}
```

這稱為函式宣告，因為它作為一個獨立的陳述式出現，而不是作為另一個陳述式中的表達式。識別碼 `awesomeFunction` 與函式值之間的關聯發生在程式碼的編譯階段，在該程式碼執行之前。

與函式宣告陳述式相對的是，函式表達式可以像這樣定義和賦值：

```js
// let awesomeFunction = ..
// const awesomeFunction = ..
var awesomeFunction = function(coolThings) {
    // ..
    return amazingStuff;
};
```

這個函式是一個被賦值給變數 `awesomeFunction` 的表達式。與函式宣告形式不同，函式表達式在執行時期的該陳述式之前不會與其識別碼關聯。

非常重要的一點是，在 JS 中，函式是可以被賦值（如此程式碼片段所示）和傳遞的值。事實上，JS 函式是物件值型別的一種特殊型別。並非所有語言都將函式視為值，但這對於支援函式程式設計模式的語言來說是必不可少的，而 JS 就是這樣做的。

JS 函式可以接收參數輸入：

```js
function greeting(myName) {
    console.log(`Hello, ${ myName }!`);
}

greeting("Kyle");   // Hello, Kyle!
```

在這個程式碼片段中，`myName` 被稱為參數，它在函式內部充當區域變數。函式可以被定義為接收任意數量的參數，從零到任意多個，依你所需。每個參數會被賦予你在呼叫的該位置（這裡是 `"Kyle"`）傳入的引數值。

函式也可以使用 `return` 關鍵字來回傳值：

```js
function greeting(myName) {
    return `Hello, ${ myName }!`;
}

var msg = greeting("Kyle");

console.log(msg);   // Hello, Kyle!
```

你只能 `return` 一個單一的值，但如果你有更多值要回傳，你可以將它們包裝成一個物件／陣列。

由於函式是值，它們可以被賦值為物件上的屬性：

```js
var whatToSay = {
    greeting() {
        console.log("Hello!");
    },
    question() {
        console.log("What's your name?");
    },
    answer() {
        console.log("My name is Kyle.");
    }
};

whatToSay.greeting();
// Hello!
```

在這個程式碼片段中，三個函式的參考（`greeting()`、`question()` 和 `answer()`）被包含在由 `whatToSay` 持有的物件中。每個函式可以透過存取屬性來取得函式參考值而被呼叫。將這種在物件上定義函式的直接風格與本章稍後討論的更複雜的 `class` 語法進行比較。

`function` 在 JS 中有許多不同的形式。我們在附錄 A 的「如此多的函式形式」中深入探討這些變體。

## 比較

在程式中做決定需要比較值以確定它們的身份和彼此之間的關係。JS 有幾種機制來實現值的比較，所以讓我們更仔細地看看它們。

### 相等......大概吧

JS 程式中最常見的比較問的問題是：「這個 X 值與那個 Y 值*相同*嗎？」但對 JS 來說，「相同」到底真正意味著什麼？

由於人體工學和歷史原因，其含義比明顯的*精確相同*匹配更為複雜。有時候相等性比較意圖是*精確*匹配，但其他時候所需的比較更為寬泛，允許*非常相似*或*可互換*的匹配。換句話說，我們必須注意**相等性**比較和**等價性**比較之間的細微差異。

如果你花了一些時間使用和閱讀 JS，你一定見過所謂的「三等號」`===` 運算子，也被描述為「嚴格相等」運算子。這似乎相當直接，對吧？當然，「嚴格」意味著嚴格，即狹窄和*精確*。

不*完全*是。

是的，大多數參與 `===` 相等性比較的值會符合那種*完全相同*的直覺。考慮一些例子：

```js
3 === 3.0;              // true
"yes" === "yes";        // true
null === null;          // true
false === false;        // true

42 === "42";            // false
"hello" === "Hello";    // false
true === 1;             // false
0 === null;             // false
"" === null;            // false
null === undefined;     // false
```

| 注意： |
| :--- |
| `===` 的相等性比較經常被描述為「同時檢查值和型別」。在我們目前看到的幾個例子中，像 `42 === "42"`，兩個值的*型別*（數字、字串等）似乎是區分因素。但不僅僅如此。JS 中的**所有**值比較都會考慮被比較值的型別，不*僅僅*是 `===` 運算子。具體來說，`===` 不允許在比較中進行任何型別轉換（亦即「強制轉型」），而其他 JS 比較*確實*允許強制轉型。 |

但 `===` 運算子確實有一些細微之處，這是許多 JS 開發者忽略的事實，而這對他們不利。`===` 運算子被設計為在兩種特殊值的情況下*說謊*：`NaN` 和 `-0`。考慮以下範例：

```js
NaN === NaN;            // false
0 === -0;               // true
```

在 `NaN` 的情況下，`===` 運算子*說謊*了，說一個 `NaN` 不等於另一個 `NaN`。在 `-0` 的情況下（是的，這是一個真實的、獨特的值，你可以在程式中有意地使用！），`===` 運算子*說謊*了，說它等於普通的 `0` 值。

由於這些比較的*說謊*可能令人煩惱，最好避免對它們使用 `===`。對於 `NaN` 比較，使用 `Number.isNaN(..)` 工具函式，它不會*說謊*。對於 `-0` 比較，使用 `Object.is(..)` 工具函式，它也不會*說謊*。`Object.is(..)` 也可以用於不*說謊*的 `NaN` 檢查，如果你偏好的話。幽默地說，你可以把 `Object.is(..)` 想成「四等號」`====`，真正真正嚴格的比較！

這些*說謊*背後有更深層的歷史和技術原因，但這並不改變 `===` 在*最嚴格*的意義上實際上並非*嚴格精確相等*比較的事實。

當我們考慮物件值（非原始值）的比較時，故事變得更加複雜。考慮以下範例：

```js
[ 1, 2, 3 ] === [ 1, 2, 3 ];    // false
{ a: 42 } === { a: 42 }         // false
(x => x * 2) === (x => x * 2)   // false
```

這是怎麼回事？

假設相等性檢查考慮值的*性質*或*內容*似乎是合理的；畢竟，`42 === 42` 考慮的是實際的 `42` 值並進行比較。但當涉及到物件時，內容感知的比較通常被稱為「結構相等性」。

JS 並沒有將 `===` 定義為物件值的*結構相等性*。相反地，`===` 對物件值使用*身份相等性*。

在 JS 中，所有物件值都透過參考持有（見附錄 A 的「值與參考」），透過參考複製來賦值和傳遞，**而且**就我們目前的討論而言，透過參考（身份）相等性來比較。考慮以下範例：

```js
var x = [ 1, 2, 3 ];

// assignment is by reference-copy, so
// y references the *same* array as x,
// not another copy of it.
var y = x;

y === x;              // true
y === [ 1, 2, 3 ];    // false
x === [ 1, 2, 3 ];    // false
```

在這個程式碼片段中，`y === x` 為真，因為兩個變數都持有對同一個初始陣列的參考。但 `=== [1,2,3]` 的比較都失敗了，因為 `y` 和 `x` 分別被與新的*不同*陣列 `[1,2,3]` 比較。陣列的結構和內容在這個比較中不重要，只有**參考身份**才重要。

JS 沒有提供物件值的結構相等性比較機制，只有參考身份比較。要進行結構相等性比較，你需要自己實作檢查。

但要注意，這比你想像的更複雜。例如，你要如何確定兩個函式參考是否「結構上等價」？即使將其字串化來比較原始碼文字，也不會考慮到閉包之類的東西。JS 不提供結構相等性比較，因為要處理所有的邊角情況幾乎是不可行的！

### 強制轉型比較

強制轉型意味著一種型別的值被轉換為其在另一種型別中的相應表示（盡可能地）。正如我們將在第四章中討論的，強制轉型是 JS 語言的核心支柱，而不是一些可以合理避免的可選功能。

但當強制轉型遇到比較運算子（如相等性）時，不幸的是，混淆和挫折往往比不會更常出現。

很少有 JS 功能比 `==` 運算子在更廣泛的 JS 社群中引起更多憤怒，它通常被稱為「寬鬆相等」運算子。大多數關於 JS 的寫作和公開討論都譴責這個運算子設計得很差，在 JS 程式中使用時很危險／容易產生錯誤。甚至這門語言的創造者本人 Brendan Eich 也哀嘆它的設計是一個大錯誤。

據我所知，大部分的挫折來自一個相當短的令人困惑的邊角情況列表，但更深層的問題是一個極其普遍的誤解，認為它在比較時不考慮被比較值的型別。

`==` 運算子執行相等性比較的方式與 `===` 執行的方式類似。事實上，兩個運算子都考慮被比較值的型別。如果比較是在相同值型別之間，`==` 和 `===` **做的完全相同的事情，毫無差異。**

如果被比較的值型別不同，`==` 與 `===` 的不同之處在於它允許在比較之前進行強制轉型。換句話說，它們都想比較相同型別的值，但 `==` 允許*先*進行型別轉換，一旦型別在兩邊都被轉換為相同的，那麼 `==` 就做與 `===` 相同的事情。與其說「寬鬆相等」，`==` 運算子應該被描述為「強制轉型相等」。

考慮以下範例：

```js
42 == "42";             // true
1 == true;              // true
```

在兩個比較中，值型別不同，所以 `==` 使非數字值（`"42"` 和 `true`）在比較之前被轉換為數字（分別是 `42` 和 `1`）。

只要意識到 `==` 的這個特性——它偏好原始數值比較——就能幫助你避免大多數麻煩的邊角情況，例如遠離像 `"" == 0` 或 `0 == false` 這樣的陷阱。

你可能在想，「噢，好吧，我就永遠避免任何強制轉型相等性比較（改用 `===`）以避免那些邊角情況」！呃，抱歉，這不像你希望的那樣可行。

你很有可能會使用關係比較運算子，如 `<`、`>`（甚至 `<=` 和 `>=`）。

就像 `==` 一樣，如果被關係比較的型別已經匹配，這些運算子會表現得像是「嚴格」的，但如果型別不同，它們會先允許強制轉型（通常轉換為數字）。

考慮以下範例：

```js
var arr = [ "1", "10", "100", "1000" ];
for (let i = 0; i < arr.length && arr[i] < 500; i++) {
    // will run 3 times
}
```

`i < arr.length` 比較對強制轉型是「安全的」，因為 `i` 和 `arr.length` 始終是數字。`arr[i] < 500` 則會觸發強制轉型，因為 `arr[i]` 的值都是字串。因此這些比較變成了 `1 < 500`、`10 < 500`、`100 < 500` 和 `1000 < 500`。由於第四個是假的，迴圈在第三次疊代後停止。

這些關係運算子通常使用數值比較，除了**兩個**被比較的值都已經是字串的情況；在這種情況下，它們使用字串的字母順序（類似字典的）比較：

```js
var x = "10";
var y = "9";

x < y;      // true, watch out!
```

沒有辦法讓這些關係運算子避免強制轉型，除了永遠不在比較中使用不匹配的型別。這作為目標也許值得欽佩，但你仍然很有可能遇到型別*可能*不同的情況。

更明智的做法不是避免強制轉型比較，而是擁抱並學習它們的來龍去脈。

強制轉型比較在 JS 的其他地方也會出現，例如條件式（`if` 等），我們將在附錄 A 的「強制轉型條件比較」中重新探討。

## 我們如何在 JS 中組織程式碼

在 JS 生態系統中廣泛使用兩種主要的程式碼（資料和行為）組織模式：類別和模組。這些模式並不互斥；許多程式可以也確實同時使用兩者。其他程式只會堅持使用一種模式，甚至兩者都不使用！

在某些方面，這些模式非常不同。但有趣的是，在其他方面，它們只是同一枚硬幣的不同面。精通 JS 需要理解這兩種模式以及它們在哪裡適用（以及不適用！）。

### 類別

「物件導向」、「類別導向」和「類別」這些術語都充滿了大量的細節和細微差別；它們在定義上並不是通用的。

我們在這裡將使用一個常見且較傳統的定義，最可能為那些有「物件導向」語言（如 C++ 和 Java）背景的人所熟悉的定義。

程式中的類別是一種「型別」的自訂資料結構的定義，它包含資料和操作該資料的行為。類別定義了這種資料結構的運作方式，但類別本身不是具體的值。要獲得你可以在程式中使用的具體值，類別必須被*實例化*（使用 `new` 關鍵字）一次或多次。

考慮以下範例：

```js
class Page {
    constructor(text) {
        this.text = text;
    }

    print() {
        console.log(this.text);
    }
}

class Notebook {
    constructor() {
        this.pages = [];
    }

    addPage(text) {
        var page = new Page(text);
        this.pages.push(page);
    }

    print() {
        for (let page of this.pages) {
            page.print();
        }
    }
}

var mathNotes = new Notebook();
mathNotes.addPage("Arithmetic: + - * / ...");
mathNotes.addPage("Trigonometry: sin cos tan ...");

mathNotes.print();
// ..
```

在 `Page` 類別中，資料是存儲在 `this.text` 成員屬性中的文字字串。行為是 `print()`，一個將文字輸出到主控台的方法。

對於 `Notebook` 類別，資料是一個 `Page` 實例的陣列。行為是 `addPage(..)`，一個實例化新的 `Page` 頁面並將它們加入列表的方法，以及 `print()`（印出筆記本中所有頁面）。

陳述式 `mathNotes = new Notebook()` 建立了 `Notebook` 類別的一個實例，而 `page = new Page(text)` 是建立 `Page` 類別實例的地方。

行為（方法）只能在實例上呼叫（而不是類別本身），例如 `mathNotes.addPage(..)` 和 `page.print()`。

`class` 機制允許將資料（`text` 和 `pages`）與其行為（例如 `addPage(..)` 和 `print()`）組織在一起。同樣的程式可以在沒有任何 `class` 定義的情況下建構，但它可能會更缺乏組織性、更難以閱讀和理解，並且更容易有錯誤和欠佳的維護性。

#### 類別繼承

傳統「類別導向」設計固有的另一個面向，雖然在 JS 中較少使用，是「繼承」（和「多型」）。考慮以下範例：

```js
class Publication {
    constructor(title,author,pubDate) {
        this.title = title;
        this.author = author;
        this.pubDate = pubDate;
    }

    print() {
        console.log(`
            Title: ${ this.title }
            By: ${ this.author }
            ${ this.pubDate }
        `);
    }
}
```

這個 `Publication` 類別定義了任何出版物可能需要的一組共同行為。

現在讓我們考慮更具體的出版物型別，如 `Book` 和 `BlogPost`：

```js
class Book extends Publication {
    constructor(bookDetails) {
        super(
            bookDetails.title,
            bookDetails.author,
            bookDetails.pubDate
        );
        this.publisher = bookDetails.publisher;
        this.ISBN = bookDetails.ISBN;
    }

    print() {
        super.print();
        console.log(`
            Publisher: ${ this.publisher }
            ISBN: ${ this.ISBN }
        `);
    }
}

class BlogPost extends Publication {
    constructor(title,author,pubDate,URL) {
        super(title,author,pubDate);
        this.URL = URL;
    }

    print() {
        super.print();
        console.log(this.URL);
    }
}
```

`Book` 和 `BlogPost` 都使用 `extends` 子句來*擴展* `Publication` 的一般定義，以包含額外的行為。每個建構子中的 `super(..)` 呼叫委託給父類別 `Publication` 的建構子進行初始化工作，然後它們根據各自的出版物型別（亦即「子類別」或「子類」）做更具體的事情。

現在考慮使用這些子類別：

```js
var YDKJS = new Book({
    title: "You Don't Know JS",
    author: "Kyle Simpson",
    pubDate: "June 2014",
    publisher: "O'Reilly",
    ISBN: "123456-789"
});

YDKJS.print();
// Title: You Don't Know JS
// By: Kyle Simpson
// June 2014
// Publisher: O'Reilly
// ISBN: 123456-789

var forAgainstLet = new BlogPost(
    "For and against let",
    "Kyle Simpson",
    "October 27, 2014",
    "https://davidwalsh.name/for-and-against-let"
);

forAgainstLet.print();
// Title: For and against let
// By: Kyle Simpson
// October 27, 2014
// https://davidwalsh.name/for-and-against-let
```

注意兩個子類別實例都有一個 `print()` 方法，它是從父類別 `Publication` *繼承*的 `print()` 方法的覆寫。每個覆寫的子類別 `print()` 方法都呼叫 `super.print()` 來調用繼承版本的 `print()` 方法。

繼承的和覆寫的方法可以擁有相同的名稱並共存的事實稱為*多型*。

繼承是一個強大的工具，用於將資料／行為組織在不同的邏輯單元（類別）中，同時允許子類別透過存取／使用父類別的行為和資料來與之合作。

### 模組

模組模式本質上與類別模式有相同的目標，即將資料和行為組合成邏輯單元。同樣像類別一樣，模組可以「包含」或「存取」其他模組的資料和行為，以進行合作。

但模組與類別有一些重要的差異。最值得注意的是，語法完全不同。

#### 經典模組

ES6 為原生 JS 語法新增了一種模組語法形式，我們稍後會看到。但從 JS 的早期開始，模組就是一種重要且常見的模式，在無數 JS 程式中被使用，即使沒有專門的語法。

*經典模組*的關鍵特徵是一個外部函式（至少執行一次），它回傳模組的一個「實例」，其中有一個或多個暴露的函式可以操作模組實例的內部（隱藏）資料。

因為這種形式的模組*只是一個函式*，而呼叫它會產生模組的一個「實例」，這些函式的另一種描述是「模組工廠」。

考慮前面的 `Publication`、`Book` 和 `BlogPost` 類別的經典模組形式：

```js
function Publication(title,author,pubDate) {
    var publicAPI = {
        print() {
            console.log(`
                Title: ${ title }
                By: ${ author }
                ${ pubDate }
            `);
        }
    };

    return publicAPI;
}

function Book(bookDetails) {
    var pub = Publication(
        bookDetails.title,
        bookDetails.author,
        bookDetails.publishedOn
    );

    var publicAPI = {
        print() {
            pub.print();
            console.log(`
                Publisher: ${ bookDetails.publisher }
                ISBN: ${ bookDetails.ISBN }
            `);
        }
    };

    return publicAPI;
}

function BlogPost(title,author,pubDate,URL) {
    var pub = Publication(title,author,pubDate);

    var publicAPI = {
        print() {
            pub.print();
            console.log(URL);
        }
    };

    return publicAPI;
}
```

將這些形式與 `class` 形式進行比較，相似之處多於差異。

`class` 形式將方法和資料存儲在物件實例上，必須使用 `this.` 前綴來存取。使用模組，方法和資料作為作用域中的識別碼變數來存取，不需要任何 `this.` 前綴。

使用 `class` 時，實例的「API」隱含在類別定義中——同時，所有資料和方法都是公開的。使用模組工廠函式時，你明確地建立和回傳一個具有任何公開暴露方法的物件，而任何資料或其他未被參考的方法在工廠函式內部保持私有。

這個工廠函式形式還有其他在 JS 中相當常見的變體，即使在 2020 年也是如此；你可能會在不同的 JS 程式中遇到這些形式：AMD（非同步模組定義）、UMD（通用模組定義）和 CommonJS（經典 Node.js 風格模組）。這些變體的差異很小（不完全相容）。然而，所有這些形式都依賴相同的基本原理。

也考慮這些模組工廠函式的用法（亦即「實例化」）：

```js
var YDKJS = Book({
    title: "You Don't Know JS",
    author: "Kyle Simpson",
    publishedOn: "June 2014",
    publisher: "O'Reilly",
    ISBN: "123456-789"
});

YDKJS.print();
// Title: You Don't Know JS
// By: Kyle Simpson
// June 2014
// Publisher: O'Reilly
// ISBN: 123456-789

var forAgainstLet = BlogPost(
    "For and against let",
    "Kyle Simpson",
    "October 27, 2014",
    "https://davidwalsh.name/for-and-against-let"
);

forAgainstLet.print();
// Title: For and against let
// By: Kyle Simpson
// October 27, 2014
// https://davidwalsh.name/for-and-against-let
```

這裡唯一可觀察到的差異是不使用 `new`，而是將模組工廠作為普通函式呼叫。

#### ES 模組

ES 模組（ESM），在 ES6 中引入 JS 語言，旨在服務於與剛才描述的現有*經典模組*基本相同的精神和目的，特別是考慮到 AMD、UMD 和 CommonJS 的重要變體和使用案例。

然而，實作方式確實有顯著的不同。

首先，沒有包裝函式來*定義*模組。包裝的上下文是一個檔案。ESM 總是基於檔案的；一個檔案，一個模組。

其次，你不會明確地與模組的「API」互動，而是使用 `export` 關鍵字將變數或方法加入其公開 API 定義。如果某個東西在模組中被定義但沒有被 `export`，那它就保持隱藏（就像*經典模組*一樣）。

第三，也許與之前討論的模式最明顯不同的是，你不需要「實例化」一個 ES 模組，你只需 `import` 它來使用其單一實例。ESM 實際上是「單例」，因為只有一個實例會被建立，在你程式中第一次 `import` 時建立，而所有其他 `import` 只是接收對那個同一單一實例的參考。如果你的模組需要支援多次實例化，你必須在你的 ESM 定義上提供一個*經典模組風格*的工廠函式來達到該目的。

在我們正在進行的範例中，我們確實假設了多次實例化，所以以下的程式碼片段會混合使用 ESM 和*經典模組*。

考慮檔案 `publication.js`：

```js
function printDetails(title,author,pubDate) {
    console.log(`
        Title: ${ title }
        By: ${ author }
        ${ pubDate }
    `);
}

export function create(title,author,pubDate) {
    var publicAPI = {
        print() {
            printDetails(title,author,pubDate);
        }
    };

    return publicAPI;
}
```

要從另一個 ES 模組（如 `blogpost.js`）匯入和使用這個模組：

```js
import { create as createPub } from "publication.js";

function printDetails(pub,URL) {
    pub.print();
    console.log(URL);
}

export function create(title,author,pubDate,URL) {
    var pub = createPub(title,author,pubDate);

    var publicAPI = {
        print() {
            printDetails(pub,URL);
        }
    };

    return publicAPI;
}
```

最後，要使用這個模組，我們匯入到另一個 ES 模組（如 `main.js`）：

```js
import { create as newBlogPost } from "blogpost.js";

var forAgainstLet = newBlogPost(
    "For and against let",
    "Kyle Simpson",
    "October 27, 2014",
    "https://davidwalsh.name/for-and-against-let"
);

forAgainstLet.print();
// Title: For and against let
// By: Kyle Simpson
// October 27, 2014
// https://davidwalsh.name/for-and-against-let
```

| 注意： |
| :--- |
| `import` 陳述式中的 `as newBlogPost` 子句是可選的；如果省略，將會匯入一個名為 `create(..)` 的頂層函式。在這個例子中，我為了可讀性而重新命名它；它較通用的工廠名稱 `create(..)` 變成了語義上更具描述性的 `newBlogPost(..)`，更能說明其用途。 |

如所示，如果 ES 模組需要支援多次實例化，它們可以在內部使用*經典模組*。或者，我們也可以從模組中暴露一個 `class` 而不是 `create(..)` 工廠函式，結果大致相同。然而，既然你已經在使用 ESM 了，我建議堅持使用*經典模組*而不是 `class`。

如果你的模組只需要一個單一實例，你可以跳過額外的複雜性層級：直接 `export` 其公開方法。

## 兔子洞越來越深

正如本章開頭所承諾的，我們只是瀏覽了 JS 語言主要部分的廣泛表面。你的頭可能還在暈，但在這樣一個資訊轟炸之後，這完全是自然的！

即使只是對 JS 的「簡短」概覽，我們涵蓋或暗示了大量你應該仔細考慮並確保你感到舒適的細節。當我建議：重新閱讀這一章，也許好幾遍，我是認真的。

在下一章中，我們將更深入地挖掘 JS 核心運作方式的一些重要面向。但在你跟隨那個兔子洞更深入之前，請確保你已經花了足夠的時間來充分消化我們剛才在這裡所涵蓋的內容。
