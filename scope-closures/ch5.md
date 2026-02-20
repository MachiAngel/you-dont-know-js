# 你所不知道的 JS（進階篇）：作用域與閉包 - 第二版
# 第五章：變數的（不太）秘密生命週期

到目前為止，你應該已經對作用域的巢狀結構有了相當不錯的理解，從全域作用域往下延伸——這稱為程式的作用域鏈。

但僅僅知道一個變數來自哪個作用域只是故事的一部分。如果一個變數宣告出現在作用域的第一個語句之後，那麼在宣告*之前*對該識別字的任何引用會如何表現？如果你嘗試在同一個作用域中宣告兩次相同的變數，會發生什麼事？

JS 特有的詞法作用域風格，在變數何時產生以及何時可供程式使用方面，有著豐富的細微差異。

## 什麼時候可以使用變數？

在什麼時間點，一個變數在其作用域內變為可用？似乎有一個顯而易見的答案：在變數被宣告/建立*之後*。對吧？不完全是。

考慮以下程式碼：

```js
greeting();
// Hello!

function greeting() {
    console.log("Hello!");
}
```

這段程式碼運作正常。你可能之前見過或甚至寫過類似的程式碼。但你有沒有想過它是如何運作的，或者為什麼能運作？具體來說，為什麼你可以在第 1 行存取識別字 `greeting`（用來取得並執行一個函式參考），即使 `greeting()` 函式宣告直到第 4 行才出現？

回想第一章指出，所有識別字在編譯時期都會註冊到各自的作用域中。此外，每個識別字在其所屬作用域的開頭就會被*建立*，**每次進入該作用域時都是如此**。

最常用來描述一個變數從其外圍作用域的開頭就可見（即使其宣告可能出現在作用域更下方）的術語叫做**提升（hoisting）**。

但僅靠提升並不能完全回答這個問題。我們可以從作用域的開頭就看到一個叫做 `greeting` 的識別字，但為什麼我們可以在它被宣告之前就**呼叫** `greeting()` 函式？

換句話說，變數 `greeting` 是如何從作用域開始執行的那一刻起，就已經有值（函式參考）被賦予給它的？答案是正式 `function` 宣告的一個特殊特性，稱為*函式提升*。當一個 `function` 宣告的名稱識別字在其作用域頂部被註冊時，它還會被額外地自動初始化為該函式的參考。這就是為什麼函式可以在整個作用域中被呼叫！

一個關鍵細節是，*函式提升*和 `var` 風格的*變數提升*都會將其名稱識別字附加到最近的外圍**函式作用域**（或者，如果沒有的話，就是全域作用域），而非區塊作用域。

| 注意： |
| :--- |
| 使用 `let` 和 `const` 的宣告仍然會提升（參見本章稍後的 TDZ 討論）。但這兩種宣告形式會附加到其外圍區塊，而非像 `var` 和 `function` 宣告那樣只附加到外圍函式。更多資訊請參見第六章的「使用區塊來限定作用域」。 |

### 提升：宣告 vs. 表達式

*函式提升*只適用於正式的 `function` 宣告（特別是那些出現在區塊外部的——參見第六章的「FiB」），不適用於 `function` 表達式賦值。考慮以下程式碼：

```js
greeting();
// TypeError

var greeting = function greeting() {
    console.log("Hello!");
};
```

第 1 行（`greeting();`）拋出了一個錯誤。但拋出的錯誤*類型*非常重要，需要注意。`TypeError` 表示我們正在嘗試對一個值做不被允許的操作。根據你的 JS 環境，錯誤訊息可能會說類似「'undefined' is not a function」，或者更有幫助地說「'greeting' is not a function」。

注意這個錯誤**不是** `ReferenceError`。JS 並不是告訴我們在作用域中找不到 `greeting` 這個識別字。它是告訴我們 `greeting` 被找到了，但在那個時刻它並不持有一個函式參考。只有函式才能被呼叫，所以嘗試呼叫某個非函式值會導致錯誤。

但如果不是函式參考，`greeting` 持有什麼？

除了被提升之外，用 `var` 宣告的變數在其作用域的開頭也會被自動初始化為 `undefined`——同樣是最近的外圍函式，或者全域作用域。一旦被初始化，它們就可以在整個作用域中被使用（賦值、取值等）。

所以在第一行，`greeting` 存在，但它只持有預設的 `undefined` 值。直到第 4 行，`greeting` 才被賦予函式參考。

請密切注意這裡的區別。`function` 宣告會被提升**並初始化為其函式值**（同樣，這稱為*函式提升*）。`var` 變數也會被提升，然後自動初始化為 `undefined`。任何後續的 `function` 表達式賦值給該變數，要等到在執行時處理到該賦值時才會發生。

在兩種情況下，識別字的名稱都會被提升。但除非識別字是在正式的 `function` 宣告中建立的，否則函式參考的關聯不會在初始化時（作用域開頭）處理。

### 變數提升

讓我們看另一個*變數提升*的例子：

```js
greeting = "Hello!";
console.log(greeting);
// Hello!

var greeting = "Howdy!";
```

雖然 `greeting` 直到第 5 行才被宣告，但早在第 1 行就可以被賦值。為什麼？

解釋需要兩個必要部分：

* 識別字被提升了，
* **並且**它從作用域頂部被自動初始化為 `undefined` 值。

| 注意： |
| :--- |
| 這種*變數提升*的用法可能感覺不太自然，許多讀者可能正確地想要避免在他們的程式中依賴它。但是所有提升（包括*函式提升*）都應該避免嗎？我們將在附錄 A 中更詳細地探討這些關於提升的不同觀點。 |

## 提升：另一個比喻

第二章充滿了比喻（用來說明作用域），但這裡我們面臨另一個比喻：提升本身。提升並不是 JS 引擎執行的具體執行步驟，更有用的是把提升看作是 JS 在**執行之前**設定程式時採取的各種動作的視覺化。

關於提升的典型主張是：*提起*——像提起重物一樣——將任何識別字一路提升到作用域的頂部。通常的解釋是 JS 引擎會在執行前*重寫*程式，使其看起來更像這樣：

```js
var greeting;           // hoisted declaration
greeting = "Hello!";    // the original line 1
console.log(greeting);  // Hello!
greeting = "Howdy!";    // `var` is gone!
```

提升（比喻）提議 JS 預處理原始程式並重新排列一下，使所有宣告在執行之前都被移到各自作用域的頂部。此外，提升比喻主張 `function` 宣告會完整地被提升到每個作用域的頂部。考慮以下程式碼：

```js
studentName = "Suzy";
greeting();
// Hello Suzy!

function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;
```

提升比喻的「規則」是函式宣告先被提升，然後變數在所有函式之後立即被提升。因此，提升的故事暗示程式被 JS 引擎*重新排列*成這樣：

```js
function greeting() {
    console.log(`Hello ${ studentName }!`);
}
var studentName;

studentName = "Suzy";
greeting();
// Hello Suzy!
```

這個提升比喻很方便。它的好處是讓我們可以忽略那些必要的神奇的預先處理，不用去找出埋藏在作用域深處的所有宣告並以某種方式將它們移動（提升）到頂部；我們可以只把程式想成是由 JS 引擎以**單次遍歷**的方式從上到下執行的。

單次遍歷確實比第一章所主張的兩階段處理看起來更直接。

提升作為重新排列程式碼的機制可能是一個吸引人的簡化，但它並不準確。JS 引擎實際上並不重新排列程式碼。它無法神奇地預先查看並找到宣告；準確找到它們以及程式中所有作用域邊界的唯一方法是完整解析程式碼。

猜猜什麼是解析？就是兩階段處理的第一階段！沒有什麼神奇的心理體操可以繞過這個事實。

所以如果提升比喻（充其量）是不準確的，我們應該如何處理這個術語？我認為它仍然有用——事實上，就連 TC39 的成員也經常使用它！——但我不認為我們應該聲稱它是源碼的實際重新排列。

| 警告： |
| :--- |
| 不正確或不完整的心智模型通常看起來仍然足夠，因為它們偶爾可以導出意外正確的答案。但從長遠來看，如果你的思維與 JS 引擎的實際運作方式不太一致，那麼準確分析和預測結果會更加困難。 |

我主張提升*應該*用來指代在作用域開頭自動註冊變數的**編譯時期操作**，每次進入該作用域時都會生成執行時期指令。

這是一個細微但重要的轉變，從提升作為執行時期行為到它在編譯時期任務中的正確位置。

## 重複宣告？

你認為當一個變數在同一個作用域中被宣告超過一次時會發生什麼？考慮以下程式碼：

```js
var studentName = "Frank";
console.log(studentName);
// Frank

var studentName;
console.log(studentName);   // ???
```

你預期第二條訊息會印出什麼？許多人認為第二個 `var studentName` 重新宣告了變數（因此「重設」了它），所以他們預期會印出 `undefined`。

但變數真的存在在同一個作用域中被「重複宣告」這回事嗎？不存在。

如果你從提升比喻的角度來考慮這個程式，程式碼會為了執行目的被重新排列成這樣：

```js
var studentName;
var studentName;    // clearly a pointless no-op!

studentName = "Frank";
console.log(studentName);
// Frank

console.log(studentName);
// Frank
```

由於提升實際上是在作用域開頭註冊變數，在作用域中間原始程式實際上有第二個 `var studentName` 語句的位置，沒有什麼需要做的。它只是一個空操作（no-op），一個無意義的語句。

| 提示： |
| :--- |
| 按照第二章的對話敘事風格，*編譯器*會找到第二個 `var` 宣告語句，並詢問*作用域管理者*是否已經見過 `studentName` 識別字；既然已經見過了，就不需要做其他任何事情。 |

同樣重要的是要指出 `var studentName;` 不等於 `var studentName = undefined;`，如同大多數人所假設的。讓我們證明它們是不同的，考慮這個程式的變體：

```js
var studentName = "Frank";
console.log(studentName);   // Frank

var studentName;
console.log(studentName);   // Frank <--- still!

// let's add the initialization explicitly
var studentName = undefined;
console.log(studentName);   // undefined <--- see!?
```

看到明確的 `= undefined` 初始化產生了與假設它在省略時隱含發生的不同結果嗎？在下一節中，我們將重新討論變數從其宣告進行初始化的主題。

在同一個作用域中重複 `var` 宣告相同的識別字名稱，實質上是一個空操作。這裡是另一個說明，這次跨越一個同名的函式：

```js
var greeting;

function greeting() {
    console.log("Hello!");
}

// basically, a no-op
var greeting;

typeof greeting;        // "function"

var greeting = "Hello!";

typeof greeting;        // "string"
```

第一個 `greeting` 宣告將識別字註冊到作用域，因為它是 `var`，自動初始化將是 `undefined`。`function` 宣告不需要重新註冊識別字，但由於*函式提升*，它會覆蓋自動初始化以使用函式參考。第二個 `var greeting` 本身不做任何事情，因為 `greeting` 已經是一個識別字，而且*函式提升*已經優先處理了自動初始化。

實際上將 `"Hello!"` 賦值給 `greeting` 會將其值從初始的函式 `greeting()` 改變為字串；`var` 本身沒有任何效果。

那麼在同一個作用域中使用 `let` 或 `const` 重複宣告呢？

```js
let studentName = "Frank";

console.log(studentName);

let studentName = "Suzy";
```

這個程式不會執行，而是立即拋出 `SyntaxError`。根據你的 JS 環境，錯誤訊息會指出類似：「studentName has already been declared」。換句話說，這是一個明確不允許嘗試「重複宣告」的情況！

不僅是兩個涉及 `let` 的宣告會拋出此錯誤。如果任一宣告使用 `let`，另一個可以是 `let` 或 `var`，錯誤仍然會發生，如以下兩個變體所示：

```js
var studentName = "Frank";

let studentName = "Suzy";
```

以及：

```js
let studentName = "Frank";

var studentName = "Suzy";
```

在兩種情況下，`SyntaxError` 都會在*第二個*宣告處拋出。換句話說，「重複宣告」變數的唯一方法是所有（兩個或更多）宣告都使用 `var`。

但為什麼要禁止它？禁止的原因本質上並非技術性的，因為 `var` 的「重複宣告」一直都是被允許的；顯然，同樣的允許可以適用於 `let`。

這更多是一個「社會工程」問題。「重複宣告」變數被一些人（包括 TC39 的許多成員）視為一種可能導致程式錯誤的壞習慣。所以當 ES6 引入 `let` 時，他們決定透過錯誤來防止「重複宣告」。

| 注意： |
| :--- |
| 這當然是一個風格上的觀點，並不真的是技術論點。許多開發者同意這個立場，這可能部分是 TC39 包含此錯誤的原因（以及 `let` 與 `const` 保持一致）。但也有合理的理由認為與 `var` 的先例保持一致更為審慎，這類觀點的強制最好留給可選擇的工具如 linter。在附錄 A 中，我們將探討 `var`（及其相關行為，如「重複宣告」）在現代 JS 中是否仍然有用。 |

當*編譯器*詢問*作用域管理者*關於一個宣告時，如果該識別字已經被宣告過，且任一/兩個宣告是用 `let` 做的，就會拋出錯誤。給開發者的預期信號是「停止依賴草率的重複宣告！」

### 常數？

`const` 關鍵字比 `let` 有更多限制。像 `let` 一樣，`const` 不能在同一個作用域中用相同的識別字重複。但實際上有一個壓倒性的技術原因使這種「重複宣告」被禁止，不像 `let` 主要是出於風格原因禁止「重複宣告」。

`const` 關鍵字要求變數必須被初始化，所以在宣告中省略賦值會導致 `SyntaxError`：

```js
const empty;   // SyntaxError
```

`const` 宣告建立的變數不能被重新賦值：

```js
const studentName = "Frank";
console.log(studentName);
// Frank

studentName = "Suzy";   // TypeError
```

`studentName` 變數不能被重新賦值，因為它是用 `const` 宣告的。

| 警告： |
| :--- |
| 重新賦值 `studentName` 時拋出的錯誤是 `TypeError`，不是 `SyntaxError`。這裡的細微區別實際上非常重要，但不幸的是太容易被忽略。語法錯誤代表程式中的缺陷，會阻止程式開始執行。型別錯誤代表在程式執行期間出現的錯誤。在前面的程式碼片段中，`"Frank"` 在我們處理 `studentName` 的重新賦值之前就已經被印出，然後才拋出錯誤。 |

所以如果 `const` 宣告不能被重新賦值，且 `const` 宣告始終需要賦值，那麼我們就有了一個明確的技術原因說明為什麼 `const` 必須禁止任何「重複宣告」：任何 `const` 的「重複宣告」也必然是 `const` 的重新賦值，而這是不被允許的！

```js
const studentName = "Frank";

// obviously this must be an error
const studentName = "Suzy";
```

由於 `const` 的「重複宣告」必須被禁止（基於這些技術理由），TC39 基本上認為 `let` 的「重複宣告」也應該被禁止，以保持一致性。這是否是最佳選擇值得商榷，但至少我們了解了這個決定背後的推理。

### 迴圈

從我們之前的討論中可以清楚看出，JS 確實不希望我們在同一個作用域中「重複宣告」變數。這可能看起來是一個直接的告誡，直到你考慮到它對迴圈中重複執行宣告語句的含義。考慮以下程式碼：

```js
var keepGoing = true;
while (keepGoing) {
    let value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

`value` 在這個程式中被「重複宣告」了嗎？我們會收到錯誤嗎？不會。

所有作用域的規則（包括 `let` 建立的變數的「重複宣告」）都是*按作用域實例*應用的。換句話說，每次在執行期間進入一個作用域，一切都會重設。

每次迴圈迭代都是其自己的新作用域實例，在每個作用域實例中，`value` 只被宣告了一次。所以沒有嘗試「重複宣告」，因此沒有錯誤。在我們考慮其他迴圈形式之前，如果前面程式碼片段中的 `value` 宣告改為 `var` 會怎樣？

```js
var keepGoing = true;
while (keepGoing) {
    var value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

`value` 在這裡被「重複宣告」了嗎，特別是我們知道 `var` 允許這樣做？不是。因為 `var` 不被視為區塊作用域宣告（參見第六章），它將自己附加到全域作用域。所以只有一個 `value` 變數，與 `keepGoing` 在同一個作用域中（在這個情況下是全域作用域）。這裡也沒有「重複宣告」！

保持清楚的一種方法是記住 `var`、`let` 和 `const` 關鍵字在程式碼開始執行時實際上已經被*移除*了。它們完全由編譯器處理。

如果你在心裡擦除宣告關鍵字然後嘗試處理程式碼，應該能幫助你判斷是否以及何時可能發生（重複）宣告。

那麼其他迴圈形式（如 `for` 迴圈）的「重複宣告」呢？

```js
for (let i = 0; i < 3; i++) {
    let value = i * 10;
    console.log(`${ i }: ${ value }`);
}
// 0: 0
// 1: 10
// 2: 20
```

應該很清楚每個作用域實例只有一個 `value` 被宣告。但 `i` 呢？它被「重複宣告」了嗎？

要回答這個問題，考慮 `i` 在什麼作用域中。看起來它可能在外部（在這個情況下是全域）作用域中，但實際上不是。它在 `for` 迴圈主體的作用域中，就像 `value` 一樣。事實上，你可以把那個迴圈大致想像成這種更詳細的等效形式：

```js
{
    // a fictional variable for illustration
    let $$i = 0;

    for ( /* nothing */; $$i < 3; $$i++) {
        // here's our actual loop `i`!
        let i = $$i;

        let value = i * 10;
        console.log(`${ i }: ${ value }`);
    }
    // 0: 0
    // 1: 10
    // 2: 20
}
```

現在應該很清楚了：`i` 和 `value` 變數都恰好**每個作用域實例**宣告一次。沒有「重複宣告」。

那麼其他的 `for` 迴圈形式呢？

```js
for (let index in students) {
    // this is fine
}

for (let student of students) {
    // so is this
}
```

`for..in` 和 `for..of` 迴圈也是一樣的：宣告的變數被視為在迴圈主體*內部*，因此是按迭代處理的（也就是按作用域實例）。沒有「重複宣告」。

好的，我知道此時你覺得我像是一張壞掉的唱片。但讓我們探討 `const` 如何影響這些迴圈結構。考慮以下程式碼：

```js
var keepGoing = true;
while (keepGoing) {
    // ooo, a shiny constant!
    const value = Math.random();
    if (value > 0.5) {
        keepGoing = false;
    }
}
```

就像我們之前看到的這個程式的 `let` 變體一樣，`const` 在每次迴圈迭代中恰好執行一次，所以不會有「重複宣告」的問題。但當我們談到 `for` 迴圈時，事情變得更複雜了。

`for..in` 和 `for..of` 可以與 `const` 一起使用：

```js
for (const index in students) {
    // this is fine
}

for (const student of students) {
    // this is also fine
}
```

但一般的 `for` 迴圈不行：

```js
for (const i = 0; i < 3; i++) {
    // oops, this is going to fail with
    // a Type Error after the first iteration
}
```

這裡有什麼問題？我們可以在這個結構中使用 `let`，而且我們斷言它為每個迴圈迭代作用域建立一個新的 `i`，所以它甚至看起來不像是「重複宣告」。

讓我們像之前那樣在心裡「展開」那個迴圈：

```js
{
    // a fictional variable for illustration
    const $$i = 0;

    for ( ; $$i < 3; $$i++) {
        // here's our actual loop `i`!
        const i = $$i;
        // ..
    }
}
```

你發現問題了嗎？我們的 `i` 確實只在迴圈內建立了一次。那不是問題。問題在於概念上的 `$$i` 每次都必須用 `$$i++` 表達式遞增。那是**重新賦值**（不是「重複宣告」），這對常數來說是不被允許的。

記住，這個「展開」形式只是一個幫助你直覺理解問題來源的概念模型。你可能會想 JS 是否可以有效地將 `const $$i = 0` 改為 `let $ii = 0`，這樣就可以讓 `const` 與我們經典的 `for` 迴圈一起使用？這是可能的，但這可能會引入 `for` 迴圈語意上的潛在令人驚訝的例外。

例如，允許 `for` 迴圈標頭中的 `i++` 規避 `const` 賦值的嚴格性，但不允許迴圈迭代內部其他 `i` 的重新賦值（這有時是有用的），會是一個相當武斷的（且可能令人困惑的）細微例外。

直接的答案是：`const` 不能與經典的 `for` 迴圈形式一起使用，因為需要重新賦值。

有趣的是，如果你不做重新賦值，那麼它是有效的：

```js
var keepGoing = true;

for (const i = 0; keepGoing; /* nothing here */ ) {
    keepGoing = (Math.random() > 0.5);
    // ..
}
```

這可以運作，但毫無意義。沒有理由在那個位置用 `const` 宣告 `i`，因為這樣一個變數在那個位置的全部意義就是**用於計算迭代次數**。只需使用不同的迴圈形式，如 `while` 迴圈，或使用 `let`！

## 未初始化的變數（又稱 TDZ）

對於 `var` 宣告，變數被「提升」到其作用域的頂部。但它也會被自動初始化為 `undefined` 值，這樣變數就可以在整個作用域中使用。

然而，`let` 和 `const` 宣告在這方面並不完全相同。

考慮以下程式碼：

```js
console.log(studentName);
// ReferenceError

let studentName = "Suzy";
```

這個程式的結果是在第一行拋出 `ReferenceError`。根據你的 JS 環境，錯誤訊息可能會說類似：「Cannot access studentName before initialization」。

| 注意： |
| :--- |
| 這裡看到的錯誤訊息過去曾經含糊得多或更具誤導性。值得慶幸的是，我們社群中的幾個人成功地遊說 JS 引擎改善了這個錯誤訊息，使其更準確地告訴你出了什麼問題！ |

那個錯誤訊息非常明確地指出了問題所在：`studentName` 在第 1 行存在，但它還沒有被初始化，所以還不能被使用。讓我們試試這個：

```js
studentName = "Suzy";   // let's try to initialize it!
// ReferenceError

console.log(studentName);

let studentName;
```

糟糕。我們仍然得到 `ReferenceError`，但現在是在第一行，我們嘗試賦值（即初始化！）這個所謂「未初始化」的變數 `studentName`。這是怎麼回事！？

真正的問題是，我們如何初始化一個未初始化的變數？對於 `let`/`const`，**唯一的方法**是透過附加到宣告語句的賦值。僅僅賦值是不夠的！考慮以下程式碼：

```js
let studentName = "Suzy";
console.log(studentName);   // Suzy
```

在這裡，我們透過 `let` 宣告語句形式並搭配賦值來初始化 `studentName`（在這個情況下，初始化為 `"Suzy"` 而不是 `undefined`）。

或者：

```js
// ..

let studentName;
// or:
// let studentName = undefined;

// ..

studentName = "Suzy";

console.log(studentName);
// Suzy
```

| 注意： |
| :--- |
| 這很有趣！回想之前，我們說 `var studentName;` 與 `var studentName = undefined;` *不*相同，但這裡的 `let` 它們行為相同。差異在於 `var studentName` 在作用域頂部自動初始化，而 `let studentName` 不會。 |

記住我們已經多次斷言*編譯器*最終會移除任何 `var`/`let`/`const` 宣告子，用在每個作用域頂部註冊適當識別字的指令來替換它們。

所以如果我們分析這裡發生的事情，我們會看到一個額外的細微差異是*編譯器*也在程式中間、變數 `studentName` 被宣告的地方添加了一條指令，來處理該宣告的自動初始化。在自動初始化發生之前的任何時間點，我們都不能使用該變數。`const` 與 `let` 的情況相同。

TC39 創造的術語，用來指代從進入作用域到變數自動初始化發生的這段*時間*是：暫時性死區（Temporal Dead Zone，TDZ）。

TDZ 是一個時間窗口，在此期間變數存在但仍未初始化，因此無法以任何方式存取。只有*編譯器*在原始宣告位置留下的指令的執行才能進行該初始化。在那個時刻之後，TDZ 結束，變數可以在其餘作用域中自由使用。

`var` 在技術上也有 TDZ，但它的長度為零，因此對我們的程式不可觀察！只有 `let` 和 `const` 有可觀察的 TDZ。

順便說一下，TDZ 中的「暫時」確實指的是*時間*而非*程式碼中的位置*。考慮以下程式碼：

```js
askQuestion();
// ReferenceError

let studentName = "Suzy";

function askQuestion() {
    console.log(`${ studentName }, do you know?`);
}
```

即使從位置上看，引用 `studentName` 的 `console.log(..)` 出現在 `let studentName` 宣告*之後*，但從時間上看，`askQuestion()` 函式在 `let` 語句被處理*之前*就被呼叫了，此時 `studentName` 仍在其 TDZ 中！因此產生了錯誤。

有一個常見的誤解認為 TDZ 意味著 `let` 和 `const` 不會提升。這是一個不準確的，或至少是稍微有誤導性的說法。它們確實會提升。

實際的差異是 `let`/`const` 宣告不會像 `var` 那樣在作用域開頭自動初始化。那麼*爭論*就是自動初始化是否是提升的*一部分*？我認為在作用域頂部自動註冊變數（即我所說的「提升」）和在作用域頂部自動初始化（為 `undefined`）是不同的操作，不應該被歸為「提升」這個單一術語。

我們已經看到 `let` 和 `const` 不會在作用域頂部自動初始化。但讓我們證明 `let` 和 `const` *確實*會提升（在作用域頂部自動註冊），這要感謝我們的朋友遮蔽（參見第三章的「遮蔽」）：

```js
var studentName = "Kyle";

{
    console.log(studentName);
    // ???

    // ..

    let studentName = "Suzy";

    console.log(studentName);
    // Suzy
}
```

第一個 `console.log(..)` 語句會發生什麼？如果 `let studentName` 沒有提升到作用域頂部，那麼第一個 `console.log(..)` *應該*印出 `"Kyle"`，對吧？在那個時刻，看起來只有外部的 `studentName` 存在，所以那應該是 `console.log(..)` 存取並印出的變數。

但實際上，第一個 `console.log(..)` 拋出了 TDZ 錯誤，因為事實上，內部作用域的 `studentName` **確實**被提升了（在作用域頂部自動註冊）。**沒有**發生的（還沒有！）是那個內部 `studentName` 的自動初始化；在那個時刻它仍然是未初始化的，因此產生了 TDZ 違規！

所以總結一下，TDZ 錯誤的發生是因為 `let`/`const` 宣告*確實*將它們的宣告提升到作用域頂部，但與 `var` 不同，它們將變數的自動初始化延遲到程式碼序列中原始宣告出現的時刻。這個時間窗口（提示：暫時的），無論其長度如何，就是 TDZ。

如何避免 TDZ 錯誤？

我的建議是：始終將你的 `let` 和 `const` 宣告放在任何作用域的頂部。將 TDZ 窗口縮小到零（或接近零）長度，然後它就無關緊要了。

但為什麼 TDZ 會存在？為什麼 TC39 沒有規定 `let`/`const` 像 `var` 那樣自動初始化？請耐心等待，我們將在附錄 A 中回來探討 TDZ 的*原因*。

## 終於初始化了

處理變數比初看之下有更多的細微差異。*提升*、*（重複）宣告*和 *TDZ* 是開發者常見的困惑來源，尤其是那些在接觸 JS 之前曾使用其他語言的人。在繼續之前，確保你的心智模型完全建立在 JS 作用域和變數的這些面向上。

提升通常被引用為 JS 引擎的一個明確機制，但它實際上更像是一個比喻，用來描述 JS 在編譯期間處理變數宣告的各種方式。但即使作為比喻，提升也為思考變數的生命週期提供了有用的結構——它何時被建立、何時可以使用、何時消失。

變數的宣告和重複宣告在被視為執行時期操作時往往會引起困惑。但如果你轉向對這些操作的編譯時期思維，那些怪異和*遮蔽*就會減少。

TDZ（暫時性死區）錯誤在遇到時很奇怪且令人沮喪。幸運的是，如果你始終小心地將 `let`/`const` 宣告放在任何作用域的頂部，TDZ 就相對容易避免。

當你成功地駕馭了變數作用域的這些曲折之後，下一章將闡述引導我們決定在各種作用域（尤其是巢狀區塊）中放置宣告的因素。
