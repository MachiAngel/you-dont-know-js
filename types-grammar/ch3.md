# You Don't Know JS Yet: Types & Grammar - 2nd Edition
# 第三章：物件值

| 注意： |
| :--- |
| 撰寫中 |

現在我們已經熟悉了內建的原始型別，接下來讓我們把注意力轉向 JS 中的 `object` 型別。

我可以寫一整本書來深入探討物件；事實上，我已經寫了！本系列的「Objects & Classes」篇已經深入涵蓋了物件，所以在繼續本章之前，請確保你已經讀過那本書。

我們不會重複那本書的內容，而是將注意力集中在 `object` 值型別如何運作，以及它如何與 JS 中其他值互動。

## 物件的型別

`object` 值型別包含了幾種子型別，每種都有其特殊的行為，包括：

* 普通物件（plain objects）
* 基本物件（boxed primitives）
* 內建物件（built-in objects）
* 陣列（arrays）
* 正規表達式（regular expressions）
* 函式（又稱「可呼叫物件」）

除了這些特殊行為之外，所有物件共有的一個特徵是它們都可以作為集合（屬性的集合），持有值（包括函式/方法）。

## 普通物件

一般的物件值型別有時被稱為*普通的 JavaScript 物件*（POJOs）。

普通物件有字面量形式：

```js
address = {
    street: "12345 Market St",
    city: "San Francisco",
    state: "CA",
    zip: "94114"
};
```

這個普通物件（POJO），以 `{ .. }` 大括號定義，是一個具名屬性的集合（`street`、`city`、`state` 和 `zip`）。屬性可以持有任何值，原始值或其他物件（包括陣列、函式等）。

同樣的物件也可以使用 `new Object()` 建構式以命令式方式定義：

```js
address = new Object();
address.street = "12345 Market St";
address.city = "San Francisco";
address.state = "CA";
address.zip = "94114";
```

普通物件預設透過 `[[Prototype]]` 連結到 `Object.prototype`，使它們可以委派存取數個通用的物件方法，例如：

* `toString()` / `toLocaleString()`
* `valueOf()`
* `isPrototypeOf(..)`
* `hasOwnProperty(..)` （最近已被棄用——替代方案：靜態方法 `Object.hasOwn(..)` 工具函式）
* `propertyIsEnumerable(..)`
* `__proto__`（getter 函式）

```js
address.isPrototypeOf(Object.prototype);    // true
address.isPrototypeOf({});                  // false
```

## 基本物件

JS 定義了數個*基本*物件型別，它們是各種內建建構式的實例，包括：

* `new String()`
* `new Number()`
* `new Boolean()`

請注意，這些建構式必須搭配 `new` 關鍵字來建構基本物件的實例。否則，這些函式實際上會執行型別強制轉型（見第四章）。

這些基本物件建構式建立的是物件值型別，而不是原始值：

```js
myName = "Kyle";
typeof myName;                      // "string"

myNickname = new String("getify");
typeof myNickname;                  // "object"
```

換句話說，基本物件建構式的實例實際上可以被視為對應底層原始值的包裝器。

| 警告： |
| :--- |
| 幾乎被普遍認為是*不好的做法*——直接實例化這些基本物件。原始值的對應形式通常更可預測、效能更好，而且在需要底層物件包裝形式來存取屬性/方法時，提供了*自動裝箱*（見下方「自動物件」章節）。 |

`Symbol(..)` 和 `BigInt(..)` 函式在規範中被稱為「建構式」，儘管它們不搭配 `new` 關鍵字使用，而且它們在 JS 程式中產生的值確實是原始值。

然而，這兩種型別存在內部的*基本物件*，用於原型委派和*自動裝箱*。

相比之下，對於 `null` 和 `undefined` 原始值，不存在 `Null()` 或 `Undefined()` 「建構式」，也沒有對應的基本物件或原型。

### 原型

基本物件建構式的實例透過 `[[Prototype]]` 連結到其建構式的 `prototype` 物件：

* `String.prototype`：定義了 `length` 屬性，以及字串特有的方法，如 `toUpperCase()` 等。

* `Number.prototype`：定義了數字特有的方法，如 `toPrecision(..)`、`toFixed(..)` 等。

* `Boolean.prototype`：定義了預設的 `toString()` 和 `valueOf()` 方法。

* `Symbol.prototype`：定義了 `description`（getter），以及預設的 `toString()` 和 `valueOf()` 方法。

* `BigInt.prototype`：定義了預設的 `toString()`、`toLocaleString()` 和 `valueOf()` 方法。

任何內建建構式的直接實例都可以透過 `[[Prototype]]` 委派存取其各自的 `prototype` 屬性/方法。此外，對應的原始值也可以透過*自動裝箱*來取得這樣的委派存取。

### 自動物件

我已經提過*自動裝箱*好幾次了（包括第一章和第二章，以及本章到目前為止也提過幾次）。現在終於是我們解釋這個概念的時候了。

存取值的屬性或方法需要該值是一個物件。正如我們在第一章中已經看到的，原始值*不是*物件，所以 JS 需要暫時將這樣的原始值轉換/包裝為其對應的基本物件[^AutoBoxing]來執行該存取。

例如：

```js
myName = "Kyle";

myName.length;              // 4

myName.toUpperCase();       // "KYLE"
```

存取 `length` 屬性或 `toUpperCase()` 方法，之所以能在原始字串值上進行，是因為 JS 將原始 `string` *自動裝箱*為一個包裝用的基本物件，也就是 `new String(..)` 的實例。否則，所有此類存取都會失敗，因為原始值沒有任何屬性。

更重要的是，當原始值被*自動裝箱*為其對應的基本物件時，這些內部建立的物件可以透過 `[[Prototype]]` 連結來存取預定義的屬性/方法（如 `length` 和 `toUpperCase()`），連結到它們各自基本物件的原型。

因此，一個*自動裝箱*的 `string` 是 `new String()` 的實例，從而連結到 `String.prototype`。同樣地，`number`（包裝為 `new Number()` 的實例）和 `boolean`（包裝為 `new Boolean()` 的實例）也是如此。

即使 `Symbol(..)` 和 `BigInt(..)` 「建構式」（不搭配 `new` 使用）產生原始值，這些原始值也可以被*自動裝箱*為其內部的基本物件包裝形式，以便委派存取屬性/方法。

| 注意： |
| :--- |
| 關於 `[[Prototype]]` 連結和對基本物件建構式原型物件的委派/繼承存取，請參閱本系列的「Objects & Classes」書。 |

由於 `null` 和 `undefined` 沒有對應的基本物件，這些值不會進行*自動裝箱*。

一個值得考慮的主觀問題：*自動裝箱*是一種強制轉型嗎？我認為是的，儘管有些人不同意。在內部，原始值被轉換為一個物件，意味著值型別發生了變化。是的，這是暫時的，但許多強制轉型也是暫時的。此外，這種轉換相當*隱式*的（由屬性/方法存取所暗示，但僅在內部發生）。我們將在第四章中重新討論強制轉型的本質。

## 其他內建物件

除了基本物件建構式之外，JS 還定義了許多其他內建建構式，用以建立更多特殊的物件子型別：

* `new Date(..)`
* `new Error(..)`
* `new Map(..)`、`new Set(..)`、`new WeakMap(..)`、`new WeakSet(..)` —— 鍵值集合
* `new Int8Array(..)`、`new Uint32Array(..)` 等 —— 索引式、型別化陣列集合
* `new ArrayBuffer(..)`、`new SharedArrayBuffer(..)` 等 —— 結構化資料集合

## 陣列

陣列是特殊化的物件，其行為是數值索引的值集合，而不是像普通物件那樣在具名屬性中持有值。

陣列有字面量形式：

```js
favoriteNumbers = [ 3, 12, 42 ];

favoriteNumbers[2];                 // 42
```

同樣的陣列也可以使用 `new Array()` 建構式以命令式方式定義：

```js
favoriteNumbers = new Array();
favoriteNumbers[0] = 3;
favoriteNumbers[1] = 12;
favoriteNumbers[2] = 42;
```

陣列透過 `[[Prototype]]` 連結到 `Array.prototype`，使它們可以委派存取各種面向陣列的方法，例如 `map(..)`、`includes(..)` 等：

```js
favoriteNumbers.map(v => v * 2);
// [ 6, 24, 84 ]

favoriteNumbers.includes(42);       // true
```

定義在 `Array.prototype` 上的某些方法——例如 `push(..)`、`pop(..)`、`sort(..)` 等——會就地修改陣列值。其他方法——例如 `concat(..)`、`map(..)`、`slice(..)` ——則會建立一個新陣列來回傳，保持原始陣列不變。第三類陣列函式——例如 `indexOf(..)`、`includes(..)` 等——僅計算並回傳一個（非陣列的）結果。

## 正規表達式

// TODO

## 函式

// TODO

## 提案中：Records/Tuples

在撰寫本文時，一個（第二階段）提案[^RecordsTuplesProposal]已經存在，計畫為 JS 新增一組功能，這些功能與普通物件和陣列密切對應，但有一些值得注意的差異。

Records 類似於普通物件，但它們是不可變的（密封的、唯讀的），而且（不像物件）在值賦值和相等比較的目的上被視為原始值。語法上的差異是在 `{ }` 分隔符前加上 `#`。Records 只能包含原始值（包括 records 和 tuples）。

Tuples 與陣列有完全相同的關係，包括在 `[ ]` 分隔符前加上 `#`。

重要的是要注意，雖然它們看起來像物件/陣列，但它們確實是原始（非物件）值。

[^FundamentalObjects]: "20 Fundamental Objects", EcamScript 2022 Language Specification; https://262.ecma-international.org/13.0/#sec-fundamental-objects ; Accessed August 2022

[^AutoBoxing]: "6.2.4.6 PutValue(V,W)", Step 5.a, ECMAScript 2022 Language Specification; https://262.ecma-international.org/13.0/#sec-putvalue ; Accessed August 2022

[^RecordsTuplesProposal]: "JavaScript Records & Tuples Proposal"; Robin Ricard, Rick Button, Nicolò Ribaudo;
https://github.com/tc39/proposal-record-tuple ; Accessed August 2022
