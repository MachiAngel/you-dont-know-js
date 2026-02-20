# 你所不知道的 JS（進階篇）：物件與類別 - 第二版
# 第二章：物件的運作方式

| 備註： |
| :--- |
| 編寫中 |

物件不只是多個值的容器，儘管這顯然是與物件互動的大部分情境。

要完全理解 JS 中的物件機制，並在程式中充分利用物件，我們需要更仔細地研究物件（及其屬性）的一些特性，這些特性會影響與它們互動時的行為。

這些定義物件底層行為的特性在正式術語中被統稱為「元物件協議」（MOP）[^mop]。MOP 不僅對理解物件的行為方式有用，還可以用於覆蓋物件的預設行為，以更充分地彎曲語言來適應我們程式的需求。

## 屬性描述器

物件上的每個屬性都由一個所謂的「屬性描述器」在內部描述。這本身是一個物件（又稱「元物件」），上面有幾個屬性（又稱「特性」），規定目標屬性的行為方式。

我們可以使用 `Object.getOwnPropertyDescriptor(..)` （ES5）來檢索任何現有屬性的屬性描述器：

```js
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle"
};

Object.getOwnPropertyDescriptor(myObj,"favoriteNumber");
// {
//     value: 42,
//     enumerable: true,
//     writable: true,
//     configurable: true
// }
```

我們甚至可以使用這樣的描述器來在物件上定義一個新屬性，使用 `Object.defineProperty(..)` （ES5）：

```js
anotherObj = {};

Object.defineProperty(anotherObj,"fave",{
    value: 42,
    enumerable: true,     // 如果省略則為預設值
    writable: true,       // 如果省略則為預設值
    configurable: true    // 如果省略則為預設值
});

anotherObj.fave;          // 42
```

如果一個現有屬性尚未被標記為不可設定的（在其描述器中 `configurable: false`），它總是可以使用 `Object.defineProperty(..)` 重新定義/覆蓋。

| 警告： |
| :--- |
| 本章前面的章節提到了「複製」或「複製」屬性。人們可能會假設這種複製/複製是在屬性描述器層級進行的。然而，這些操作實際上都不是以那種方式運作的；它們都進行簡單的 `=` 式存取和賦值，這樣做的效果是忽略了屬性描述器定義中的任何細微差別。 |

雖然在實際專案中似乎不太常見，我們甚至可以一次定義多個屬性，每個都有自己的描述器：

```js
anotherObj = {};

Object.defineProperties(anotherObj,{
    "fave": {
        // 一個屬性描述器
    },
    "superFave": {
        // 另一個屬性描述器
    }
});
```

這種用法不是很常見，因為你很少需要具體控制多個屬性的定義。但在某些情況下它可能很有用。

### 存取器屬性

屬性描述器通常定義一個 `value` 屬性，如上所示。然而，有一種特殊的屬性，稱為「存取器屬性」（又稱 getter/setter），可以被定義。對於這類屬性，其描述器不定義一個固定的 `value` 屬性，而是看起來像這樣：

```js
{
    get() { .. },    // 檢索值時調用的函式
    set(v) { .. },   // 賦值時調用的函式
    // .. enumerable 等
}
```

getter 看起來像屬性存取（`obj.prop`），但在底層它調用了定義的 `get()` 方法；這有點像你呼叫了 `obj.prop()`。setter 看起來像屬性賦值（`obj.prop = value`），但它調用了定義的 `set(..)` 方法；這有點像你呼叫了 `obj.prop(value)`。

讓我們來說明一個 getter/setter 存取器屬性：

```js
anotherObj = {};

Object.defineProperty(anotherObj,"fave",{
    get() { console.log("Getting 'fave' value!"); return 123; },
    set(v) { console.log(`Ignoring ${v} assignment.`); }
});

anotherObj.fave;
// Getting 'fave' value!
// 123

anotherObj.fave = 42;
// Ignoring 42 assignment.

anotherObj.fave;
// Getting 'fave' value!
// 123
```

### 可列舉、可寫入、可設定

除了 `value` 或 `get()` / `set(..)`，屬性描述器的其他 3 個特性是（如上所示）：

* `enumerable`
* `writable`
* `configurable`

`enumerable` 特性控制屬性是否會出現在物件屬性的各種列舉中，例如 `Object.keys(..)`、`Object.entries(..)`、`for..in` 迴圈，以及 `...` 物件展開和 `Object.assign(..)` 發生的複製。大多數屬性應該保持可列舉，但你可以將物件上的某些特殊屬性標記為不可列舉，如果它們不應該被迭代/複製的話。

`writable` 特性控制是否允許通過 `=` 進行值賦值。要使屬性「唯讀」，用 `writable: false` 定義它。然而，只要屬性仍然是可設定的，`Object.defineProperty(..)` 仍然可以通過設定不同的 `value` 來更改值。

`configurable` 特性控制屬性的**描述器**是否可以被重新定義/覆蓋。`configurable: false` 的屬性被鎖定在其定義上，任何進一步嘗試使用 `Object.defineProperty(..)` 更改它都會失敗。不可設定的屬性仍然可以被賦予新值（通過 `=`），只要屬性描述器上的 `writable: true` 仍然設定著。

## 物件子型別

JS 中有各種專門的物件子型別。但到目前為止，你最常與之互動的兩個是陣列和 `function`。

| 備註： |
| :--- |
| 所謂「子型別」，我們指的是一種衍生型別的概念，它繼承了父型別的行為，但隨後特化或擴展了這些行為。換句話說，這些子型別的值完全是物件，但也*不僅僅是*物件。 |

### 陣列

陣列是專門設計為**數字索引**的物件，而不是使用字串命名的屬性位置。它們仍然是物件，所以像 `favoriteNumber` 這樣的命名屬性在法律上是允許的。但非常不建議在數字索引的陣列中混入命名屬性。

陣列最好使用字面量語法（類似於物件）來定義，但使用 `[ .. ]` 方括號而不是 `{ .. }` 大括號：

```js
myList = [ 23, 42, 109 ];
```

JS 允許陣列中混合任何類型的值，包括物件、其他陣列、函式等。正如你可能已經知道的，陣列是「零索引」的，意味著陣列中的第一個元素在索引 `0`，而不是 `1`：

```js
myList = [ 23, 42, 109 ];

myList[0];      // 23
myList[1];      // 42
```

回想一下，物件上任何「看起來像」整數的字串屬性名稱——能夠被有效地強制轉型為數字整數——實際上會被當作整數屬性（又稱整數索引）對待。陣列也是如此。你應該始終使用 `42` 作為整數索引（又稱屬性名稱），但如果你使用字串 `"42"`，JS 會假設你指的是整數並為你做轉換。

```js
// "2" 在這裡作為整數索引，但不建議這樣做
myList["2"];    // 109
```

陣列「不要命名屬性」*規則*的一個例外是，所有陣列自動暴露一個 `length` 屬性，它會隨著陣列的「長度」自動保持更新。

```js
myList = [ 23, 42, 109 ];

myList.length;   // 3

// 將另一個值「推入」列表末尾
myList.push("Hello");

myList.length;   // 4
```

| 警告： |
| :--- |
| 許多 JS 開發者錯誤地認為陣列的 `length` 基本上是一個 *getter*（見本章前面的「存取器屬性」），但它不是。其結果是，這些開發者覺得存取這個屬性是「昂貴的」——好像 JS 必須即時重新計算長度——因此會做諸如在對陣列進行非突變迴圈之前捕獲/儲存陣列長度這樣的事情。這曾經是效能方面的「最佳實踐」。但至少在過去 10 年中，這實際上已經是一個反模式了，因為 JS 引擎在管理 `length` 屬性方面比我們的 JS 程式碼試圖「超越」引擎以避免調用我們認為的 *getter* 更有效率。讓 JS 引擎做它的工作更有效率，在需要的時候隨時隨地存取該屬性就好。 |

#### 空插槽

JS 陣列還有一個非常不幸的設計「缺陷」，稱為「空插槽」。如果你在超過陣列當前末尾一個以上位置的索引處賦值，JS 會將中間的插槽保留為「空」，而不是像你可能預期的那樣自動將它們賦值為 `undefined`：

```js
myList = [ 23, 42, 109 ];
myList.length;              // 3

myList[14] = "Hello";
myList.length;              // 15

myList;                     // [ 23, 42, 109, empty x 11, "Hello" ]

// 看起來像一個真正的插槽，
// 裡面有一個真正的 `undefined` 值，
// 但小心，這是一個陷阱！
myList[9];                  // undefined
```

你可能會想為什麼空插槽這麼糟糕？一個原因是：JS 中有些 API，如陣列的 `map(..)`，會令人驚訝地跳過空插槽！永遠不要故意在陣列中建立空插槽。這無可爭議地是 JS 的「壞的部分」之一。

### 函式

關於函式，我在這裡沒有太多特別要說的，除了指出它們也是物件的子型別。這意味著除了可執行之外，它們也可以在上面新增或從中存取命名屬性。

函式有兩個預定義的屬性，你可能會在元程式設計目的中與之互動：

```js
function help(opt1,opt2,...remainingOpts) {
    // ..
}

help.name;          // "help"
help.length;        // 2
```

函式的 `length` 是其顯式定義的參數計數，直到但不包括有預設值定義（例如 `param = 42`）或「其餘參數」（例如 `...remainingOpts`）的參數。

#### 避免在函式物件上設定屬性

你應該避免在函式物件上賦值屬性。如果你想要儲存與函式相關聯的額外資訊，使用單獨的 `Map(..)` （或 `WeakMap(..)`），以函式物件作為鍵，額外資訊作為值。

```js
extraInfo = new Map();

extraInfo.set(help,"this is some important information");

// 稍後：
extraInfo.get(help);   // "this is some important information"
```

## 物件特性

除了為特定屬性定義行為之外，某些行為可以在整個物件上進行設定：

* extensible（可擴展）
* sealed（封閉）
* frozen（凍結）

### 可擴展

可擴展性指的是一個物件是否可以被定義/新增新的屬性。預設情況下，所有物件都是可擴展的，但你可以關閉物件的可擴展性：

```js
myObj = {
    favoriteNumber: 42
};

myObj.firstName = "Kyle";                  // 正常運作

Object.preventExtensions(myObj);

myObj.nicknames = [ "getify", "ydkjs" ];   // 失敗
myObj.favoriteNumber = 123;                // 正常運作
```

在非嚴格模式下，建立新屬性的賦值會靜默失敗，而在嚴格模式下會拋出例外。

### 封閉

// TODO

### 凍結

// TODO

## 擴展 MOP

如同本章開頭所提到的，JS 中的物件根據一組稱為元物件協議（MOP）[^mop] 的規則來運作。現在我們更充分地理解了物件預設的運作方式，我們想要將注意力轉向如何掛接到這些預設行為並覆蓋/自訂它們。

// TODO

## `[[Prototype]]` 鏈

物件最重要但最不明顯的特性之一（MOP 的一部分）被稱為其「原型鏈」；JS 官方規範表示法是 `[[Prototype]]`。確保不要將這個 `[[Prototype]]` 與名為 `prototype` 的公開屬性混淆。儘管命名相同，這些是不同的概念。

`[[Prototype]]` 是物件在建立時預設獲得的內部連結，指向另一個物件。這個連結是物件的一個隱藏的、通常微妙的特性，但它對與物件互動時的行為有深遠的影響。它被稱為「鏈」，因為一個物件連結到另一個，而那個又連結到另一個……以此類推。這條鏈有一個*結尾*或*頂端*，在那裡連結停止，沒有更遠的地方可去。稍後會詳細介紹。

我們已經在第一章中看到了 `[[Prototype]]` 連結的幾個影響。例如，預設情況下，所有物件都通過 `[[Prototype]]` 連結到名為 `Object.prototype` 的內建物件。

| 警告： |
| :--- |
| `Object.prototype` 這個名稱本身可能令人困惑，因為它使用了一個名為 `prototype` 的屬性。`[[Prototype]]` 和 `prototype` 是如何相關的！？先把這些問題/困惑擱置一下，因為我們稍後會在本章中回來解釋 `[[Prototype]]` 和 `prototype` 之間的區別。目前，就假設存在這個重要但名稱奇怪的內建物件 `Object.prototype`。 |

讓我們考慮一些程式碼：

```js
myObj = {
    favoriteNumber: 42
};
```

從第一章來看應該很熟悉。但你在這段程式碼中*看不到的*是，那裡的物件被自動連結（通過其內部的 `[[Prototype]]`）到那個自動內建的、名稱奇怪的 `Object.prototype` 物件。

當我們做這樣的事情時：

```js
myObj.toString();                             // "[object Object]"

myObj.hasOwnProperty("favoriteNumber");   // true
```

我們在利用這個內部的 `[[Prototype]]` 連結，而沒有真正意識到它。由於 `myObj` 上沒有定義 `toString` 或 `hasOwnProperty` 屬性，那些屬性存取實際上最終**委託**存取沿著 `[[Prototype]]` 鏈繼續查找。

由於 `myObj` 通過 `[[Prototype]]` 連結到名為 `Object.prototype` 的物件，`toString` 和 `hasOwnProperty` 屬性的查找繼續在那個物件上進行；而且確實，這些方法在那裡被找到了！

`myObj.toString` 能夠存取 `toString` 屬性即使它實際上沒有這個屬性的能力，通常被稱為「繼承」，或更具體地說，「原型繼承」。`toString` 和 `hasOwnProperty` 屬性，以及許多其他屬性，被稱為 `myObj` 上的「繼承屬性」。

| 備註： |
| :--- |
| 我對這裡使用「繼承」這個詞有很多不滿——它應該叫做「委託」！——但這就是大多數人所說的，所以我們會勉強遵從並暫時使用相同的術語（儘管是在抗議下，加上「」引號）。我會把我的異議留到本書的附錄中。 |

`Object.prototype` 有幾個內建的屬性和方法，所有這些都被任何直接或間接通過另一個物件的連結，連結到 `Object.prototype` 的物件所「繼承」。

一些常見的從 `Object.prototype`「繼承」的屬性包括：

* `constructor`
* `__proto__`
* `toString()`
* `valueOf()`
* `hasOwnProperty(..)`
* `isPrototypeOf(..)`

回想 `hasOwnProperty(..)`，我們前面看到它給我們一個布林值，檢查某個屬性（通過字串名稱）是否由物件擁有：

```js
myObj = {
    favoriteNumber: 42
};

myObj.hasOwnProperty("favoriteNumber");   // true
```

一直被認為有些不幸的是（語義組織、命名衝突等），像 `hasOwnProperty(..)` 這樣重要的工具被包含在 Object `[[Prototype]]` 鏈上作為實例方法，而不是被定義為靜態工具。

截至 ES2022，JS 終於新增了這個工具的靜態版本：`Object.hasOwn(..)`。

```js
myObj = {
    favoriteNumber: 42
};

Object.hasOwn(myObj,"favoriteNumber");   // true
```

這種形式現在被認為是更可取和更穩健的選項，實例方法（`hasOwnProperty(..)`）形式現在通常應該被避免。

有些不幸和不一致的是，目前還沒有（截至撰寫時）對應的靜態工具，如 `Object.isPrototype(..)`（取代實例方法 `isPrototypeOf(..)`）。但至少 `Object.hasOwn(..)` 存在了，這就是進步。

### 建立具有不同 `[[Prototype]]` 的物件

預設情況下，你在程式中建立的任何物件都會通過 `[[Prototype]]` 連結到 `Object.prototype` 物件。然而，你可以建立一個具有不同連結的物件，像這樣：

```js
myObj = Object.create(differentObj);
```

`Object.create(..)` 方法將其第一個引數作為要為新建物件的 `[[Prototype]]` 設定的值。

這種方法的一個缺點是你沒有使用 `{ .. }` 字面量語法，所以你不會最初為 `myObj` 定義任何內容。你通常然後需要逐一定義屬性，使用 `=`。

| 備註： |
| :--- |
| `Object.create(..)` 的第二個可選引數是——像前面討論的 `Object.defineProperties(..)` 的第二個引數——一個物件，其屬性持有描述器來最初定義新物件。在實際專案中，這種形式很少使用，可能是因為指定完整的描述器比僅僅是名稱/值對更笨拙。但在某些有限的情況下它可能會派上用場。 |

或者，雖然不太推薦，你可以使用 `{ .. }` 字面量語法配合一個特殊的（看起來很奇怪的！）屬性：

```js
myObj = {
    __proto__: differentObj,

    // .. 物件定義的其餘部分
};
```

| 警告： |
| :--- |
| 看起來奇怪的 `__proto__` 屬性在某些 JS 引擎中已經存在超過 20 年了，但直到 ES6（2015 年）才在 JS 中標準化。即便如此，它是被加到規範的附錄 B 中[^specApB]，其中列出了 TC39 勉強包含的特性，因為它們在各種基於瀏覽器的 JS 引擎中已經廣泛存在，因此是事實上的現實，即使它們並非源自 TC39。這個特性因此被規範「保證」存在於所有符合標準的基於瀏覽器的 JS 引擎中，但不一定保證在其他獨立的 JS 引擎中運作。Node.js 使用 Chrome 瀏覽器的 JS 引擎（v8），所以 Node.js 預設/偶然地獲得了 `__proto__`。使用 `__proto__` 時要注意你的程式碼將在哪些 JS 引擎環境中運行。 |

無論你使用 `Object.create(..)` 還是 `__proto__`，所建立的物件通常都會通過 `[[Prototype]]` 連結到與預設的 `Object.prototype` 不同的物件。

#### 空的 `[[Prototype]]` 連結

我們上面提到 `[[Prototype]]` 鏈必須在某處停止，以便查找不會永遠繼續。`Object.prototype` 通常是每個 `[[Prototype]]` 鏈的頂端/結尾，因為它自己的 `[[Prototype]]` 是 `null`，因此沒有其他地方可以繼續查找。

然而，你也可以定義具有自己 `null` 值 `[[Prototype]]` 的物件，例如：

```js
emptyObj = Object.create(null);
// 或：emptyObj = { __proto__: null }

emptyObj.toString;   // undefined
```

建立一個沒有到 `Object.prototype` 的 `[[Prototype]]` 連結的物件可能非常有用。例如，如第一章所述，`in` 和 `for..in` 結構會查詢 `[[Prototype]]` 鏈來尋找繼承的屬性。但這可能是不希望的，因為你可能不希望像 `"toString" in myObj` 這樣的查詢成功解析。

此外，具有空 `[[Prototype]]` 的物件不會受到任何意外的「繼承」衝突，即其自身的屬性名稱與它從其他地方「繼承」的屬性名稱之間的衝突。這些類型的（有用的！）物件在流行的說法中有時被稱為「字典物件」。

### `[[Prototype]]` vs `prototype`

注意到 `Object.prototype` 這個特殊物件的名稱/位置中的公開屬性名稱 `prototype` 了嗎？那是怎麼回事？

`Object` 是 `Object(..)` 函式；預設情況下，所有函式（它們本身也是物件！）上都有這樣一個 `prototype` 屬性，指向一個物件。

而這就是 `[[Prototype]]` 和 `prototype` 之間的名稱衝突真正讓我們困擾的地方。函式上的 `prototype` 屬性不定義函式本身所經歷的任何連結。實際上，函式（作為物件）有它們自己的內部 `[[Prototype]]` 連結在別處——稍後會詳細介紹。

相反，函式上的 `prototype` 屬性指向一個物件，當使用 `new` 關鍵字呼叫該函式時，任何其他被建立的物件都應該*連結到*這個物件：

```js
myObj = {};

// 基本上等同於：
myObj = new Object();
```

由於 `{ .. }` 物件字面量語法本質上與 `new Object()` 呼叫相同，位於 `Object.prototype` 的內建物件被用作我們建立並命名為 `myObj` 的新物件的內部 `[[Prototype]]` 值。

呼！一個僅僅因為 `[[Prototype]]` 和 `prototype` 之間的名稱重疊而變得更加令人困惑的主題！

----

但函式本身（作為物件！）在 `[[Prototype]]` 方面連結到哪裡呢？它們連結到 `Function.prototype`，另一個內建物件，位於 `Function(..)` 函式的 `prototype` 屬性上。

換句話說，你可以把函式本身想像成是通過 `new Function(..)` 呼叫「建立」的，然後通過 `[[Prototype]]` 連結到 `Function.prototype` 物件。這個物件包含所有函式預設「繼承」的屬性/方法，例如 `toString()`（用於將函式的原始碼序列化為字串）和 `call(..)` / `apply(..)` / `bind(..)`（我們將在本書後面解釋這些）。

## 物件行為

物件上的屬性在內部由一個「描述器」元物件定義和控制，其中包括像 `value`（屬性的當前值）和 `enumerable`（一個布林值，控制屬性是否包含在僅列舉屬性/屬性名稱的列表中）等特性。

JS 中物件及其屬性的運作方式被稱為「元物件協議」（MOP）[^mop]。我們可以通過 `Object.defineProperty(..)` 控制屬性的精確行為，以及通過 `Object.freeze(..)` 控制物件範圍的行為。但更強大的是，我們可以使用特殊的預定義 Symbol 來掛接和覆蓋物件上的某些預設行為。

原型是物件之間的內部連結，允許對一個物件的屬性或方法存取——如果所請求的屬性/方法不存在——通過將該存取查找「委託」給另一個物件來處理。當委託涉及方法時，方法運行的上下文通過 `this` 關鍵字從初始物件共享到目標物件。

[^mop]: "Metaobject", Wikipedia; https://en.wikipedia.org/wiki/Metaobject ; Accessed July 2022.

[^specApB]: "Appendix B: Additional ECMAScript Features for Web Browsers", ECMAScript 2022 Language Specification; https://262.ecma-international.org/13.0/#sec-additional-ecmascript-features-for-web-browsers ; Accessed July 2022
