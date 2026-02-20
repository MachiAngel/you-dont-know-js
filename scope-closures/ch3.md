# 你所不知道的 JS（進階篇）：作用域與閉包 - 第二版
# 第三章：作用域鏈

第一章和第二章奠定了*詞法作用域*（及其組成部分）的具體定義，並以有用的隱喻說明了其概念基礎。在繼續本章之前，找個人用你自己的話解釋（書面或口頭）什麼是詞法作用域以及為什麼理解它很有用。

這看起來像是你可能會跳過的一步，但我發現花時間將這些想法重新組織成向他人解釋的形式真的很有幫助。這能幫助我們的大腦消化我們正在學習的東西！

現在是時候深入細節了，所以預期從這裡開始事情會變得更加詳細。堅持下去，因為這些討論真的會讓你深刻認識到我們對作用域有多少*不了解*的地方。確保你花時間閱讀文字和提供的所有程式碼片段。

為了重新回顧我們一直在用的範例，讓我們回想一下第二章圖 2 中的巢狀作用域泡泡彩色圖解：

<figure>
    <img src="images/fig2.png" width="500" alt="Colored Scope Bubbles" align="center">
    <figcaption><em>圖 2（第二章）：有顏色的作用域泡泡</em></figcaption>
    <br><br>
</figure>

巢狀在其他作用域內部的作用域之間的連接稱為作用域鏈，它決定了變數可以被存取的路徑。這條鏈是有方向的，意味著查找只會向上／向外移動。

## 「查找」（大多數情況下）是概念性的

在圖 2 中，注意 `for` 迴圈中 `students` 變數參考的顏色。我們究竟是如何確定它是一顆 RED(1) 彈珠的？

在第二章中，我們將變數的執行時期存取描述為一種「查找」，其中*引擎*必須從詢問當前作用域的*作用域管理器*是否知道某個識別字／變數開始，並沿著巢狀作用域的鏈向上／向外回溯（朝向全域作用域），直到找到為止。查找在第一個匹配的命名宣告被找到時立即停止。

因此查找過程確定了 `students` 是一顆 RED(1) 彈珠，因為在我們遍歷作用域鏈的過程中，直到到達最終的 RED(1) 全域作用域之前，還沒有找到匹配的變數名稱。

同樣地，`if` 陳述式中的 `studentID` 被確定為一顆 BLUE(2) 彈珠。

這種執行時期查找過程的說法對於概念理解很好用，但它並不是實際上通常運作的方式。

彈珠桶子的顏色（即變數來自哪個作用域的後設資訊）*通常在*初始編譯處理期間就確定了。因為詞法作用域在那個時候基本上已經最終確定了，彈珠的顏色不會因為之後在執行時期發生的任何事情而改變。

由於彈珠的顏色在編譯時就已知，而且是不可變的，這個資訊很可能會與每個變數在 AST 中的條目一起存儲（或至少從中可以存取）；該資訊隨後被構成程式執行時期的可執行指令明確使用。

換句話說，*引擎*（來自第二章）不需要透過一堆作用域進行查找來弄清楚一個變數來自哪個作用域桶子。那個資訊已經知道了！避免執行時期查找的需要是詞法作用域的一個關鍵最佳化優勢。執行時期運作得更有效率，因為不需要花時間在所有這些查找上。

但我剛才說了「⋯⋯通常在⋯⋯確定」，關於在編譯期間確定彈珠的顏色。那麼在什麼情況下它*不會*在編譯期間被知道呢？

考慮一個對在當前檔案中任何詞法上可用的作用域中都未宣告的變數的參考——參見《Get Started》第一章，其中斷言每個檔案從 JS 編譯的角度來看都是獨立的程式。如果找不到宣告，那*不一定*是錯誤。在執行時期中，另一個檔案（程式）可能確實在共享的全域作用域中宣告了該變數。

因此，關於變數是否曾在某個可存取的桶子中被適當宣告的最終確定可能需要推遲到執行時期。

任何最初*未宣告*的變數參考在該檔案的編譯期間會被保留為未著色的彈珠；這個顏色直到其他相關檔案被編譯且應用程式執行時期開始後才能確定。延遲的查找最終會將顏色解析為找到該變數的任何作用域（很可能是全域作用域）。

然而，這種查找最多只需要對每個變數進行一次，因為在執行時期中沒有其他東西可以改變該彈珠的顏色。

第二章的「查找失敗」部分涵蓋了如果彈珠在其參考被執行的那一刻最終仍然未著色時會發生什麼。

## 遮蔽

「遮蔽（Shadowing）」聽起來可能很神秘、有點可疑。但別擔心，它完全合法！

我們在這些章節中一直使用的範例在作用域邊界之間使用了不同的變數名稱。由於它們都有唯一的名稱，在某種程度上，即使它們都只是存儲在一個桶子（如 RED(1)）中也不會有什麼問題。

當你有兩個或更多變數，分別在不同的作用域中但具有相同的詞法名稱時，擁有不同的詞法作用域桶子就開始變得更加重要了。一個作用域不能有兩個或更多同名的變數；這樣的多個參考會被假定為只是一個變數。

所以如果你需要維護兩個或更多同名的變數，你必須使用不同的（通常是巢狀的）作用域。在這種情況下，不同作用域桶子的佈局方式就非常相關了。

考慮：

```js
var studentName = "Suzy";

function printStudent(studentName) {
    studentName = studentName.toUpperCase();
    console.log(studentName);
}

printStudent("Frank");
// FRANK

printStudent(studentName);
// SUZY

console.log(studentName);
// Suzy
```

| 提示： |
| :--- |
| 在你繼續之前，花一些時間使用我們在書中涵蓋的各種技術／隱喻來分析這段程式碼。特別是，確保識別這段程式碼片段中的彈珠／泡泡顏色。這是很好的練習！ |

第 1 行的 `studentName` 變數（`var studentName = ..` 陳述式）建立了一顆 RED(1) 彈珠。第 3 行宣告了一顆同名的 BLUE(2) 彈珠，即 `printStudent(..)` 函式定義中的參數。

在 `studentName = studentName.toUpperCase()` 賦值陳述式和 `console.log(studentName)` 陳述式中，`studentName` 會是什麼顏色的彈珠？所有三個 `studentName` 參考都將是 BLUE(2)。

根據「查找」的概念性說法，我們斷言它從當前作用域開始，向外／向上查找，一旦找到匹配的變數就停止。BLUE(2) `studentName` 會立即被找到。RED(1) `studentName` 甚至不會被考慮。

這是詞法作用域行為的一個關鍵面向，稱為*遮蔽*。BLUE(2) `studentName` 變數（參數）遮蔽了 RED(1) `studentName`。所以，參數遮蔽了（被遮蔽的）全域變數。把這句話對自己重複幾次，確保你理解了這個術語！

這就是為什麼對 `studentName` 的重新賦值只影響內部（參數）變數：BLUE(2) `studentName`，而不是全域的 RED(1) `studentName`。

當你選擇遮蔽一個外部作用域的變數時，一個直接影響是從該作用域向內／向下（透過任何巢狀作用域）現在不可能有任何彈珠被著色為被遮蔽的變數的顏色——（在這種情況下是 RED(1)）。換句話說，任何 `studentName` 識別字參考都會對應到那個參數變數，永遠不會是全域的 `studentName` 變數。在 `printStudent(..)` 函式（或其任何巢狀作用域）內部，在詞法上不可能參考全域的 `studentName`。

### 全域反遮蔽技巧

請注意：利用我即將描述的技術不是很好的做法，因為它的用途有限，會讓程式碼的讀者感到困惑，而且很可能會給你的程式帶來錯誤。我介紹它只是因為你可能在現有的程式中遇到這種行為，而理解正在發生什麼對於不被絆倒是至關重要的。

在一個作用域中，即使該變數已被遮蔽，*也是*有可能存取全域變數的，但不是透過典型的詞法識別字參考。

在全域作用域（RED(1)）中，`var` 宣告和 `function` 宣告也會將自己暴露為*全域物件*上的屬性（與識別字同名）——本質上是全域作用域的物件表示。如果你為瀏覽器環境撰寫過 JS，你可能認得全域物件就是 `window`。這不*完全*準確，但對我們的討論來說足夠了。在下一章，我們將更多地探討全域作用域／物件的主題。

考慮這個程式，特別是作為獨立的 .js 檔案在瀏覽器環境中執行：

```js
var studentName = "Suzy";

function printStudent(studentName) {
    console.log(studentName);
    console.log(window.studentName);
}

printStudent("Frank");
// "Frank"
// "Suzy"
```

注意 `window.studentName` 參考？這個表達式將全域變數 `studentName` 作為 `window` 上的屬性來存取（我們現在暫時假設它與全域物件是同義的）。這是在遮蔽變數存在的作用域內部存取被遮蔽變數的唯一方法。

`window.studentName` 是全域 `studentName` 變數的鏡像，而不是一個單獨的快照副本。對其中一個的更改仍然可以從另一個看到，雙向都是如此。你可以把 `window.studentName` 想像為一個存取實際 `studentName` 變數的 getter/setter。事實上，你甚至可以透過在全域物件上建立／設定屬性來*新增*一個變數到全域作用域。

| 警告： |
| :--- |
| 記住：你*可以*做不代表你*應該*做。不要遮蔽你需要存取的全域變數，反過來，也避免使用這個技巧來存取你已經遮蔽的全域變數。而且絕對不要透過把全域變數建立為 `window` 屬性而不是使用正式宣告來讓程式碼讀者困惑！ |

這個小「技巧」只適用於存取全域作用域變數（不是來自巢狀作用域的被遮蔽變數），而且即便如此，也只限於那些用 `var` 或 `function` 宣告的變數。

其他形式的全域作用域宣告不會建立鏡像的全域物件屬性：

```js
var one = 1;
let notOne = 2;
const notTwo = 3;
class notThree {}

console.log(window.one);       // 1
console.log(window.notOne);    // undefined
console.log(window.notTwo);    // undefined
console.log(window.notThree);  // undefined
```

存在於全域作用域以外任何其他作用域中的變數（無論它們如何宣告！），在它們被遮蔽的作用域中完全不可存取：

```js
var special = 42;

function lookingFor(special) {
    // The identifier `special` (parameter) in this
    // scope is shadowed inside keepLooking(), and
    // is thus inaccessible from that scope.

    function keepLooking() {
        var special = 3.141592;
        console.log(special);
        console.log(window.special);
    }

    keepLooking();
}

lookingFor(112358132134);
// 3.141592
// 42
```

全域 RED(1) `special` 被 BLUE(2) `special`（參數）遮蔽，而 BLUE(2) `special` 本身又被 `keepLooking()` 內部的 GREEN(3) `special` 遮蔽。我們仍然可以使用間接參考 `window.special` 存取 RED(1) `special`。但 `keepLooking()` 沒有任何方法可以存取持有數字 `112358132134` 的 BLUE(2) `special`。

### 複製不是存取

我曾被問過以下「但那⋯⋯呢？」的問題數十次。考慮：

```js
var special = 42;

function lookingFor(special) {
    var another = {
        special: special
    };

    function keepLooking() {
        var special = 3.141592;
        console.log(special);
        console.log(another.special);  // Ooo, tricky!
        console.log(window.special);
    }

    keepLooking();
}

lookingFor(112358132134);
// 3.141592
// 112358132134
// 42
```

噢！那這個 `another` 物件技術是否推翻了我聲稱 `special` 參數在 `keepLooking()` 內部「完全不可存取」的說法？不，這個聲稱仍然是正確的。

`special: special` 是將 `special` 參數變數的值複製到另一個容器（同名的屬性）中。當然，如果你把值放到另一個容器中，遮蔽就不再適用了（除非 `another` 也被遮蔽了！）。但這不代表我們正在存取參數 `special`；它意味著我們正在透過*另一個*容器（物件屬性）存取它在那個時刻的值的副本。我們無法從 `keepLooking()` 內部將 BLUE(2) `special` 參數重新賦值為不同的值。

你可能即將提出另一個「但是⋯⋯！？」：如果我使用物件或陣列作為值而不是數字（`112358132134` 等）呢？我們擁有對物件的參考而不是原始值的副本，是否會「修復」不可存取性？

不會。透過參考副本修改物件值的內容**不是**與詞法上存取變數本身相同的事情。我們仍然無法重新賦值 BLUE(2) `special` 參數。

### 不合法的遮蔽

並非所有宣告遮蔽的組合都是被允許的。`let` 可以遮蔽 `var`，但 `var` 不能遮蔽 `let`：

```js
function something() {
    var special = "JavaScript";

    {
        let special = 42;   // totally fine shadowing

        // ..
    }
}

function another() {
    // ..

    {
        let special = "JavaScript";

        {
            var special = "JavaScript";
            // ^^^ Syntax Error

            // ..
        }
    }
}
```

注意在 `another()` 函式中，內部的 `var special` 宣告試圖宣告一個函式範圍的 `special`，這本身是沒問題的（如 `something()` 函式所示）。

在這種情況下，語法錯誤描述指出 `special` 已經被定義了，但這個錯誤訊息有點誤導——同樣地，在 `something()` 中不會發生這樣的錯誤，因為遮蔽通常是被允許的。

它被作為 `SyntaxError` 引發的真正原因是 `var` 基本上是在試圖「越過邊界」（或跳過）同名的 `let` 宣告，這是不被允許的。

這個邊界越過禁止在每個函式邊界處有效停止，所以這個變體不會引發異常：

```js
function another() {
    // ..

    {
        let special = "JavaScript";

        ajax("https://some.url",function callback(){
            // totally fine shadowing
            var special = "JavaScript";

            // ..
        });
    }
}
```

總結：`let`（在內部作用域中）總是可以遮蔽外部作用域的 `var`。`var`（在內部作用域中）只有在中間有函式邊界的情況下才能遮蔽外部作用域的 `let`。

## 函式名稱作用域

到目前為止你已經看到，`function` 宣告看起來像這樣：

```js
function askQuestion() {
    // ..
}
```

正如第一章和第二章中討論的，這樣的 `function` 宣告會在封閉作用域中建立一個名為 `askQuestion` 的識別字（在這種情況下，是全域作用域）。

那這個程式呢？

```js
var askQuestion = function(){
    // ..
};
```

變數 `askQuestion` 的建立方式是一樣的。但由於這是一個 `function` 表達式——一個函式定義被用作值而不是獨立的宣告——函式本身不會「提升」（參見第五章）。

`function` 宣告和 `function` 表達式之間的一個主要區別是函式的名稱識別字會發生什麼。考慮一個具名 `function` 表達式：

```js
var askQuestion = function ofTheTeacher(){
    // ..
};
```

我們知道 `askQuestion` 最終在外部作用域中。但 `ofTheTeacher` 識別字呢？對於正式的 `function` 宣告，名稱識別字最終在外部／封閉作用域中，所以假設這裡也是這樣可能是合理的。但 `ofTheTeacher` 被宣告為**函式本身內部**的識別字：

```js
var askQuestion = function ofTheTeacher() {
    console.log(ofTheTeacher);
};

askQuestion();
// function ofTheTeacher()...

console.log(ofTheTeacher);
// ReferenceError: ofTheTeacher is not defined
```

| 注意： |
| :--- |
| 實際上，`ofTheTeacher` 並不完全*在函式的作用域中*。附錄 A「隱含的作用域」將進一步解釋。 |

`ofTheTeacher` 不僅是在函式內部而不是外部宣告的，而且它還被定義為唯讀的：

```js
var askQuestion = function ofTheTeacher() {
    "use strict";
    ofTheTeacher = 42;   // TypeError

    //..
};

askQuestion();
// TypeError
```

因為我們使用了嚴格模式，賦值失敗會被報告為 `TypeError`；在非嚴格模式中，這樣的賦值會靜默失敗，不會拋出異常。

當 `function` 表達式沒有名稱識別字時呢？

```js
var askQuestion = function(){
   // ..
};
```

具有名稱識別字的 `function` 表達式被稱為「具名函式表達式」，而沒有名稱識別字的被稱為「匿名函式表達式」。匿名函式表達式顯然沒有影響任何作用域的名稱識別字。

| 注意： |
| :--- |
| 我們將在附錄 A 中更詳細地討論具名 vs. 匿名 `function` 表達式，包括影響使用哪一種決定的因素。 |

## 箭頭函式

ES6 為語言新增了一種額外的 `function` 表達式形式，稱為「箭頭函式」：

```js
var askQuestion = () => {
    // ..
};
```

`=>` 箭頭函式不需要 `function` 關鍵字來定義它。此外，在某些簡單的情況下，參數列表周圍的 `( .. )` 是可選的。同樣地，在某些情況下函式主體周圍的 `{ .. }` 也是可選的。當 `{ .. }` 被省略時，回傳值會在不使用 `return` 關鍵字的情況下發送出去。

| 注意： |
| :--- |
| `=>` 箭頭函式的吸引力通常以「更短的語法」來推銷，並聲稱這等同於客觀上更可讀的程式碼。這個聲稱充其量是值得懷疑的，我認為完全是被誤導的。我們將在附錄 A 中深入探討各種函式形式的「可讀性」。 |

箭頭函式在詞法上是匿名的，意味著它們沒有直接相關的識別字參考指向該函式。對 `askQuestion` 的賦值建立了一個推斷名稱「askQuestion」，但這**與非匿名不是同一回事**：

```js
var askQuestion = () => {
    // ..
};

askQuestion.name;   // askQuestion
```

箭頭函式以語法簡潔為代價，需要在心理上應付一堆針對不同形式／條件的變體。僅舉幾個例子：

```js
() => 42;

id => id.toUpperCase();

(id,name) => ({ id, name });

(...args) => {
    return args[args.length - 1];
};
```

我提起箭頭函式的真正原因是因為一個常見但不正確的說法：箭頭函式在詞法作用域方面的行為與標準 `function` 函式不同。

這是不正確的。

除了匿名（且沒有宣告形式）之外，`=>` 箭頭函式與 `function` 函式具有相同的詞法作用域規則。箭頭函式，無論其主體周圍是否有 `{ .. }`，仍然會建立一個獨立的、內部巢狀的作用域桶子。此巢狀作用域桶子內部的變數宣告行為與 `function` 作用域中的相同。

## 退一步看

當一個函式（宣告或表達式）被定義時，就會建立一個新的作用域。作用域彼此巢狀的位置在整個程式中形成了一個自然的作用域層級結構，稱為作用域鏈。作用域鏈控制變數存取，方向性地朝向上方和外部。

每個新的作用域提供一個乾淨的空間，一個持有自己變數集合的地方。當一個變數名稱在作用域鏈的不同層級重複出現時，遮蔽就會發生，這會阻止從該點向內存取外部變數。

當我們從這些更細緻的細節退一步時，下一章將焦點轉移到所有 JS 程式都包含的主要作用域：全域作用域。
