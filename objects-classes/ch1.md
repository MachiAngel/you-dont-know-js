# 你所不知道的 JS（進階篇）：物件與類別 - 第二版
# 第一章：物件基礎

| 備註： |
| :--- |
| 編寫中 |

> JS 中的一切都是物件。

這是關於 JS 最普遍流傳，但也最不正確的「事實」之一。讓我們開始破除這個迷思吧。

JS 確實有物件，但這並不意味著所有的值都是物件。儘管如此，物件可以說是這門語言中最重要（也是最多樣化的！）的值型別，所以掌握它們對你的 JS 學習之旅至關重要。

物件機制無疑是最靈活且強大的容器型別——你可以將其他值放入其中；你編寫的每個 JS 程式都會以某種方式使用它們。但這並不是物件在本書中佔據重要地位的原因。物件是 JS 三大支柱中第二個支柱的基礎：原型。

為什麼原型（以及本書後面會介紹的 `this` 關鍵字）對 JS 如此核心，以至於成為其三大支柱之一？除了其他原因之外，原型是 JS 的物件系統用來表達類別設計模式的方式，而類別設計模式是所有程式設計中最廣泛使用的設計模式之一。

因此，我們的旅程將從物件開始，建立對原型的完整理解，揭開 `this` 關鍵字的神秘面紗，並探索 `class` 系統。

## 關於本書

歡迎來到《你所不知道的 JS》系列的第三本書！如果你已經讀完了《入門篇》（第一本書）和《作用域與閉包》（第二本書），那你來對地方了！如果還沒有，在你繼續之前，我建議你先閱讀那兩本書作為基礎，然後再深入本書。

本書第一版的書名是「this 與物件原型」。在那本書中，我們的焦點始於 `this` 關鍵字，因為它可以說是整個 JS 中最令人困惑的主題之一。然後那本書花了大部分時間闡述原型系統，並倡導擁抱較少為人知的「委託」模式，而非類別設計。在那本書撰寫時（2014 年），ES6 距離完成還有將近兩年的時間，所以我覺得 `class` 關鍵字的早期草案只值得在附錄中簡要介紹。

說自那本書出版以來，JS 的生態系統發生了巨大變化，這絕對是輕描淡寫的說法。ES6 現在已經是舊聞了；在撰寫*這本*書的時候，JS 在 **ES6 之後**已經經歷了 7 次年度更新（ES2016 到 ES2022）。

現在，我們仍然需要討論 `this` 的工作原理，以及它與針對各種物件調用方法之間的關係。而 `class` 實際上（大部分！）是透過底層的原型鏈運作的。但 2022 年的 JS 開發者幾乎不再編寫程式碼來顯式地設定原型繼承了。儘管我個人希望情況有所不同，但類別設計模式——而非「行為委託」——才是 JS 中表達大多數資料和行為組織（資料結構）的方式。

本書反映了 JS 的當前現實：因此有了新的副標題、新的主題組織與焦點，以及對前一版文本的完全重寫。

## 物件作為容器

將多個值聚集到一個容器中的常見方式之一就是使用物件。物件是鍵/值對的集合。JS 中還有一些具有特殊行為的物件子型別，例如陣列（以數字索引）甚至函式（可呼叫的）；稍後會介紹更多關於這些子型別的內容。

| 備註： |
| :--- |
| 鍵通常被稱為「屬性名稱」，而屬性名稱和值的配對通常被稱為「屬性」。本書將以這種方式明確使用這些術語。 |

常規的 JS 物件通常使用字面量語法來宣告，像這樣：

```js
myObj = {
    // ..
};
```

**注意：** 還有一種替代的方式來建立物件（使用 `myObj = new Object()`），但這並不常見也不推薦，而且幾乎從來不是適當的做法。請堅持使用物件字面量語法。

很容易搞混 `{ .. }` 大括號的含義，因為 JS 在不同的上下文中賦予了大括號多種含義：

* 界定值，如物件字面量
* 定義物件解構模式（稍後會詳細介紹）
* 界定插值字串表達式，如 `` `some ${ getNumber() } thing` ``
* 定義區塊，如 `if` 和 `for` 迴圈
* 定義函式主體

雖然在閱讀程式碼時有時可能具有挑戰性，但要注意 `{ .. }` 大括號對是否用在程式中值/表達式有效出現的位置；如果是，它就是物件字面量，否則就是其他多載用法之一。

## 定義屬性

在物件字面量的大括號內，你使用 `propertyName: propertyValue` 對來定義屬性（名稱和值），像這樣：

```js
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle"
};
```

你賦予屬性的值可以是字面量，如上所示，也可以是通過表達式計算的：

```js
function twenty() { return 20; }

myObj = {
    favoriteNumber: (twenty() + 1) * 2,
};
```

表達式 `(twenty() + 1) * 2` 會立即求值，結果（`42`）被賦值為屬性值。

開發者有時會想知道是否有辦法為屬性值定義一個「惰性」的表達式，意思是它不在賦值時計算，而是稍後才定義。JS 沒有惰性表達式，所以唯一的方法是將表達式包裝在一個函式中：

```js
function twenty() { return 20; }
function myNumber() { return (twenty() + 1) * 2; }

myObj = {
    favoriteNumber: myNumber   // 注意，不是 `myNumber()` 函式呼叫
};
```

在這種情況下，`favoriteNumber` 不是持有一個數值，而是持有一個函式參照。要計算結果，必須顯式地執行該函式參照。

### 看起來像 JSON？

你可能注意到我們目前看到的物件字面量語法類似於一個相關的語法，「JSON」（JavaScript 物件表示法）：

```json
{
    "favoriteNumber": 42,
    "isDeveloper": true,
    "firstName": "Kyle"
}
```

JS 的物件字面量和 JSON 之間最大的區別是，對於以 JSON 定義的物件：

1. 屬性名稱必須用 `"` 雙引號括起來

2. 屬性值必須是字面量（原始值、物件或陣列），不能是任意的 JS 表達式

在 JS 程式中，物件字面量不要求屬性名稱加引號——你*可以*加引號（允許 `'` 或 `"`），但通常是可選的。然而，有些字元在屬性名稱中是有效的，但必須用引號括起來才能包含；例如，前導數字或空白：

```js
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle",
    "2 nicknames": [ "getify", "ydkjs" ]
};
```

另一個小差異是，JSON 語法——也就是將被*解析*為 JSON 的文字，例如來自 `.json` 檔案——比一般的 JS 更嚴格。例如，JS 允許註解（`// ..` 和 `/* .. */`），以及在物件和陣列表達式中使用尾隨 `,` 逗號；JSON 不允許這些。不過，JSON 仍然允許任意的空白。

### 屬性名稱

物件字面量中的屬性名稱幾乎總是被視為/強制轉型為字串值。一個例外是整數（或「看起來像整數」的）屬性「名稱」：

```js
anotherObj = {
    42:       "<-- this property name will be treated as an integer",
    "41":     "<-- ...and so will this one",

    true:     "<-- this property name will be treated as a string",
    [myObj]:  "<-- ...and so will this one"
};
```

`42` 屬性名稱將被視為整數屬性名稱（又稱索引）；`"41"` 字串值也會被如此處理，因為它*看起來像*整數。相比之下，`true` 值將變成字串屬性名稱 `"true"`，而 `myObj` 識別字參照，透過周圍的 `[ .. ]` *計算*，會將物件的值強制轉型為字串（通常是預設的 `"[object Object]"`）。

| 警告： |
| :--- |
| 如果你需要實際使用物件作為鍵/屬性名稱，永遠不要依賴這種計算字串強制轉型；其行為令人驚訝，幾乎肯定不是預期的結果，因此很可能會出現程式錯誤。相反，請使用更專門的資料結構，稱為 `Map`（在 ES6 中新增），其中用作屬性「名稱」的物件會保持原樣，而不是被強制轉型為字串值。 |

如同上面的 `[myObj]`，你可以在物件字面量定義時*計算*任何**屬性名稱**（與計算屬性值不同）：

```js
anotherObj = {
    ["x" + (21 * 2)]: true
};
```

表達式 `"x" + (21 * 2)`，必須出現在 `[ .. ]` 括號內，會立即計算，結果（`"x42"`）被用作屬性名稱。

### Symbol 作為屬性名稱

ES6 新增了一個新的原始值型別 `Symbol`，它常被用作儲存和檢索屬性值的特殊屬性名稱。它們透過 `Symbol(..)` 函式呼叫建立（**不**使用 `new` 關鍵字），可以接受一個可選的描述字串，僅用於更友好的除錯目的；如果指定了，該描述對 JS 程式是不可存取的，因此除了除錯輸出外不會用於其他目的。

```js
myPropSymbol = Symbol("optional, developer-friendly description");
```

| 備註： |
| :--- |
| Symbol 有點像數字或字串，不同之處在於它們的值對 JS 程式來說是*不透明的*，並且在 JS 程式中是全域唯一的。換句話說，你可以建立和使用 Symbol，但 JS 不會讓你知道或對底層值做任何事情；這是由 JS 引擎保持的隱藏實作細節。 |

如前所述，計算屬性名稱是在物件字面量上定義 Symbol 屬性名稱的方式：

```js
myPropSymbol = Symbol("optional, developer-friendly description");

anotherObj = {
    [myPropSymbol]: "Hello, symbol!"
};
```

用於在 `anotherObj` 上定義屬性的計算屬性名稱將是實際的原始 Symbol 值（不管它是什麼），而不是可選的描述字串（`"optional, developer-friendly description"`）。

因為 Symbol 在你的程式中是全域唯一的，**不會**有意外衝突的機會，即程式的一部分可能意外地定義了與另一部分程式嘗試定義/賦值的相同屬性名稱。

Symbol 對於掛接到物件的特殊預設行為也很有用，我們將在下一章的「擴展 MOP」中更詳細地介紹。

### 簡寫屬性

在定義物件字面量時，常常使用與已在作用域中持有你想要賦值的值的現有識別字相同的屬性名稱。

```js
coolFact = "the first person convicted of speeding was going 8 mph";

anotherObj = {
    coolFact: coolFact
};
```

| 備註： |
| :--- |
| 這與用引號括起的屬性名稱定義 `"coolFact": coolFact` 是一樣的，但 JS 開發者很少在非必要的情況下使用引號括屬性名稱。事實上，慣例是避免不必要地加上引號，所以不建議不必要地包含它們。 |

在這種情況下，當屬性名稱和值表達式的識別字相同時，你可以省略屬性定義中的屬性名稱部分，這就是所謂的「簡寫屬性」定義：

```js
coolFact = "the first person convicted of speeding was going 8 mph";

anotherObj = {
    coolFact   // <-- 簡寫屬性
};
```

屬性名稱是 `"coolFact"`（字串），賦予屬性的值是此時 `coolFact` 變數中的內容：`"the first person convicted of speeding was going 8 mph"`。

起初，這種簡寫便利可能看起來令人困惑。但隨著你越來越熟悉這個非常常見和流行的特性的使用，你可能會偏好它，因為可以少打（和少讀！）一些字。

### 簡寫方法

另一種類似的簡寫是在物件字面量中使用更簡潔的形式定義函式/方法：

```js
anotherObj = {
    // 標準函式屬性
    greet: function() { console.log("Hello!"); },

    // 簡寫函式/方法屬性
    greet2() { console.log("Hello, friend!"); }
};
```

在討論簡寫方法屬性的同時，我們也可以定義產生器函式（另一個 ES6 特性）：

```js
anotherObj = {
    // 取代：
    //   greet3: function*() { yield "Hello, everyone!"; }

    // 簡寫產生器方法
    *greet3() { yield "Hello, everyone!"; }
};
```

雖然不是特別常見，簡寫方法/產生器甚至可以有帶引號或計算的名稱：

```js
anotherObj = {
    "greet-4"() { console.log("Hello, audience!"); },

    // 簡寫計算名稱
    [ "gr" + "eet 5" ]() { console.log("Hello, audience!"); },

    // 簡寫計算產生器名稱
    *[ "ok, greet 6".toUpperCase() ]() { yield "Hello, audience!"; }
};
```

### 物件展開

在物件字面量建立時定義屬性的另一種方式是使用 `...` 語法的一種形式——它在技術上不是運算子，但看起來確實像一個——通常被稱為「物件展開」。

當 `...` 在物件字面量內使用時，會將另一個物件值的內容（屬性，即鍵/值對）「展開」到正在定義的物件中：

```js
anotherObj = {
    favoriteNumber: 12,

    ...myObj,   // 物件展開，淺複製 `myObj`

    greeting: "Hello!"
}
```

`myObj` 屬性的展開是淺層的，意味著它只複製 `myObj` 的頂層屬性；這些屬性持有的任何值都只是簡單地賦值過去。如果這些值中有任何是對其他物件的參照，那麼參照本身會被賦值（通過複製），但底層的物件值*不會*被複製——因此你最終會得到多個共享參照指向相同的物件。

你可以把物件展開想像成一個 `for` 迴圈，逐一遍歷屬性，並從來源物件（`myObj`）到目標物件（`anotherObj`）進行 `=` 式的賦值。

另外，請將這些屬性定義操作視為從物件字面量的頂部到底部「按順序」發生。在上面的程式碼片段中，由於 `myObj` 有一個 `favoriteNumber` 屬性，物件展開最終會覆蓋前一行的 `favoriteNumber: 12` 屬性賦值。此外，如果 `myObj` 包含一個被複製過來的 `greeting` 屬性，下一行（`greeting: "Hello!"`）會覆蓋該屬性定義。

| 備註： |
| :--- |
| 物件展開也只複製*自有的*屬性（直接在物件上的）且是*可列舉的*（允許被列舉/列出的）。它不會複製屬性——也就是說，不會真正模仿屬性的確切特性——而是進行簡單的賦值式複製。我們將在下一章的「屬性描述器」章節中介紹更多此類細節。 |

`...` 物件展開的一個常見用法是執行*淺*物件複製：

```js
myObjShallowCopy = { ...myObj };
```

請記住，你不能將 `...` 展開到一個已存在的物件值中；`...` 物件展開語法只能出現在 `{ .. }` 物件字面量內，而物件字面量是在建立一個新物件值。要執行類似的淺物件複製但使用 API 而非語法，請參閱本章後面的「物件條目」章節（涵蓋 `Object.entries(..)` 和 `Object.fromEntries(..)`）。

但如果你想將物件屬性（淺層地）複製到一個*已存在的*物件中，請參閱本章後面的「賦值屬性」章節（涵蓋 `Object.assign(..)`）。

### 深層物件複製

另外，由於 `...` 不進行完整的深層物件複製，物件展開通常只適合複製只包含簡單原始值的物件，而不是對其他物件的參照。

深層物件複製是一個極其複雜和微妙的操作。複製像 `42` 這樣的值是明顯且直截了當的，但複製一個函式（它是一種特殊的物件，也是通過參照持有的）意味著什麼呢？或者複製一個外部（不完全在 JS 中的）物件參照，例如 DOM 元素？如果物件有循環參照（例如巢狀的後代物件持有一個指回外部祖先物件的參照）會怎樣？關於如何處理所有這些邊界情況，有各種各樣的意見，因此不存在單一的深層物件複製標準。

對於深層物件複製，標準方法一直是：

1. 使用一個程式庫工具，它對複製行為/細微差別應如何處理有明確的主張。

2. 使用 `JSON.parse(JSON.stringify(..))` 往返技巧——這只在沒有循環參照，且物件中沒有無法用 JSON 正確序列化的值（如函式）時才能「正確」運作。

不過最近，第三個選項出現了。這不是一個 JS 特性，而是由 Web 平台等環境提供給 JS 的配套 API。現在可以使用 `structuredClone(..)`[^structuredClone] 來深層複製物件。

```js
myObjCopy = structuredClone(myObj);
```

這個內建工具背後的底層演算法支援複製循環參照，以及比 `JSON` 往返技巧**多得多**的值型別。然而，這個演算法仍然有其限制，包括不支援複製函式或 DOM 元素。

## 存取屬性

對現有物件的屬性存取最好使用 `.` 運算子：

```js
myObj.favoriteNumber;    // 42
myObj.isDeveloper;       // true
```

如果可以用這種方式存取屬性，強烈建議這樣做。

如果屬性名稱包含不能出現在識別字中的字元，例如前導數字或空白，可以使用 `[ .. ]` 括號代替 `.`：

```js
myObj["2 nicknames"];    // [ "getify", "ydkjs" ]
```

```js
anotherObj[42];          // "<-- this property name will..."
anotherObj["41"];        // "<-- this property name will..."
```

即使數字屬性「名稱」保持為數字，透過 `[ .. ]` 括號的屬性存取也會將字串表示強制轉型為數字（例如，`"42"` 作為 `42` 的數字等價物），然後相應地存取關聯的數字屬性。

與物件字面量類似，要存取的屬性名稱可以透過 `[ .. ]` 括號計算。表達式可以是一個簡單的識別字：

```js
propName = "41";
anotherObj[propName];
```

實際上，你放在 `[ .. ]` 括號之間的可以是任何任意的 JS 表達式，不僅僅是識別字或像 `42` 或 `"isDeveloper"` 這樣的字面量值。JS 會先求值表達式，然後將結果值用作在物件上查找的屬性名稱：

```js
function howMany(x) {
    return x + 1;
}

myObj[`${ howMany(1) } nicknames`];   // [ "getify", "ydkjs" ]
```

在這個程式碼片段中，表達式是一個反引號界定的 `` `樣板字串字面量` ``，其中有一個內插表達式為函式呼叫 `howMany(1)`。該表達式的整體結果是字串值 `"2 nicknames"`，然後用作存取的屬性名稱。

### 物件條目

你可以獲取物件中屬性的列表，作為元組（兩個元素的子陣列）的陣列，其中包含屬性名稱和值：

```js
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle"
};

Object.entries(myObj);
// [ ["favoriteNumber",42], ["isDeveloper",true], ["firstName","Kyle"] ]
```

在 ES6 中新增的 `Object.entries(..)` 從來源物件檢索這個條目列表——僅包含自有且可列舉的屬性；參見下一章的「屬性描述器」章節。

這樣的列表可以被迴圈/迭代，潛在地將屬性賦值到另一個已存在的物件。然而，也可以使用 `Object.fromEntries(..)` （在 ES2019 中新增）從條目列表建立一個新物件：

```js
myObjShallowCopy = Object.fromEntries( Object.entries(myObj) );

// 替代前面討論的方法：
// myObjShallowCopy = { ...myObj };
```

### 解構

存取屬性的另一種方式是透過物件解構（在 ES6 中新增）。將解構想像成定義一個「模式」，描述物件值應該「看起來像」什麼（結構上），然後要求 JS 遵循該「模式」來系統地存取物件值的內容。

物件解構的最終結果不是另一個物件，而是將來源物件中的值賦值到其他目標（變數等）的一個或多個賦值操作。

想像這種 ES6 之前的程式碼：

```js
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle"
};

const favoriteNumber = (
    myObj.favoriteNumber !== undefined ? myObj.favoriteNumber : 42
);
const isDev = myObj.isDeveloper;
const firstName = myObj.firstName;
const lname = (
    myObj.lastName !== undefined ? myObj.lastName : "--missing--"
);
```

這些對屬性值的存取和賦值到其他識別字，通常被稱為「手動解構」。要使用宣告式的物件解構語法，它可能看起來像這樣：

```js
myObj = {
    favoriteNumber: 42,
    isDeveloper: true,
    firstName: "Kyle"
};

const { favoriteNumber = 12 } = myObj;
const {
    isDeveloper: isDev,
    firstName: firstName,
    lastName: lname = "--missing--"
} = myObj;

favoriteNumber;   // 42
isDev;            // true
firstName;        // "Kyle"
lname;            // "--missing--"
```

如上所示，`{ .. }` 物件解構類似於物件字面量值的定義，但它出現在 `=` 運算子的左側，而不是右側（物件值表達式出現的位置）。這使得左側的 `{ .. }` 成為解構模式而非另一個物件定義。

`{ favoriteNumber } = myObj` 解構告訴 JS 在物件上找到名為 `favoriteNumber` 的屬性，並將其值賦給同名的識別字。模式中 `favoriteNumber` 識別字的單一實例類似於本章前面討論的「簡寫屬性」：如果來源（屬性名稱）和目標（識別字）相同，你可以省略其中一個，只列出一次。

`= 12` 部分告訴 JS，如果來源物件沒有 `favoriteNumber` 屬性，或者該屬性持有 `undefined` 值，則為 `favoriteNumber` 的賦值提供 `12` 作為預設值。

在第二個解構模式中，`isDeveloper: isDev` 模式指示 JS 在來源物件上找到名為 `isDeveloper` 的屬性，並將其值賦給名為 `isDev` 的識別字。這是一種從來源到目標的「重新命名」。相比之下，`firstName: firstName` 提供了賦值的來源和目標，但是多餘的，因為它們是相同的；這裡只用一個 `firstName` 就足夠了，通常更受偏好。

`lastName: lname = "--missing--"` 結合了來源-目標重新命名和預設值（如果 `lastName` 來源屬性不存在或為 `undefined`）。

上面的程式碼片段將物件解構與變數宣告結合——在這個例子中使用了 `const`，但 `var` 和 `let` 也可以——但它本質上不是一個宣告機制。解構是關於存取和賦值（從來源到目標），所以它也可以對已存在的目標操作，而不是宣告新的：

```js
let fave;

// 這裡需要外圍的 ( )，
// 當不使用宣告子時
({ favoriteNumber: fave } = myObj);

fave;  // 42
```

物件解構語法通常因其宣告式和更可讀的風格而受到偏好，優於 ES6 之前大量命令式的等價寫法。但不要過度使用解構。有時候只寫 `x = someObj.x` 就完全可以了！

### 條件屬性存取

最近（在 ES2020 中），一個名為「可選鏈」的特性被加入到 JS 中，它增強了屬性存取能力（特別是巢狀屬性存取）。主要形式是兩個字元的複合運算子 `?.`，如 `A?.B`。

這個運算子會檢查左側的參照（`A`）是否為空值（`null` 或 `undefined`）。如果是，則屬性存取表達式的其餘部分會被短路（跳過），並且回傳 `undefined` 作為結果（即使實際遇到的是 `null`！）。否則，`?.` 會像正常的 `.` 運算子一樣存取屬性。

例如：

```js
myObj?.favoriteNumber
```

這裡，空值檢查是對 `myObj` 執行的，意味著 `favoriteNumber` 屬性存取只在 `myObj` 中的值為非空值時才會執行。注意它不會驗證 `myObj` 是否真的持有一個真正的物件，只是它是非空值的。然而，所有非空值的值都可以透過 `.` 運算子「安全地」（不會拋出 JS 例外）被「存取」，即使沒有匹配的屬性可以檢索。

很容易搞混，以為空值檢查是對 `favoriteNumber` 屬性進行的。但記住這一點的一種方式是：`?` 在進行安全檢查的那一側，而 `.` 在只有當非空值檢查通過時才會條件性求值的那一側。

通常，`?.` 運算子用於可能有 3 層或更深的巢狀屬性存取中，例如：

```js
myObj?.address?.city
```

使用 `?.` 運算子的等價操作看起來像這樣：

```js
(myObj != null && myObj.address != null) ? myObj.address.city : undefined
```

再次記住，這裡沒有對最右側的屬性（`city`）進行檢查。

另外，`?.` 不應該被普遍地用來取代程式中每一個 `.` 運算子。你應該盡可能在進行存取之前就知道 `.` 屬性存取是否會成功。只在被存取的值的性質受到無法預測/控制的條件影響時才使用 `?.`。

例如，在前面的程式碼片段中，`myObj?.` 的用法可能是不恰當的，因為情況確實不應該是你在一個可能連頂層物件都沒有的變數上開始一連串屬性存取（撇開其內容在某些條件下可能缺少某些屬性不談）。

相反，我會建議更像這樣的用法：

```js
myObj.address?.city
```

而且這個表達式只應該在你確定 `myObj` 至少持有一個有效物件（不管它是否有一個帶有子物件的 `address` 屬性）的程式部分中使用。

「可選鏈」運算子的另一種形式是 `?.[`，當你想要進行條件/安全的屬性存取需要使用 `[ .. ]` 括號時使用。

```js
myObj["2 nicknames"]?.[0];   // "getify"
```

關於 `?.` 行為的所有說明同樣適用於 `?.[`。

| 警告： |
| :--- |
| 這個特性有第三種形式，稱為「可選呼叫」，使用 `?.(` 作為運算子。它用於在執行屬性中的函式值之前，對屬性進行非空值檢查。例如，不使用 `myObj.someFunc(42)`，你可以使用 `myObj.someFunc?.(42)`。`?.(` 會在呼叫 `myObj.someFunc` 之前（用 `(42)` 部分）檢查它是否為非空值。雖然這聽起來可能是一個有用的特性，但我認為它夠危險，值得完全避免使用這種形式/結構。<br><br>我的擔憂是，`?.(` 讓人覺得我們在確保函式在呼叫之前是「可呼叫的」，但實際上我們只是在檢查它是否為非空值。不像 `?.` 可以允許對一個既不是空值也不是物件的值進行「安全的」`.` 存取，`?.(` 的非空值檢查並不同樣「安全」。如果該屬性中有任何非空值、非函式的值，如 `true` 或 `"Hello"`，`(42)` 呼叫部分將被調用但會拋出 JS 例外。所以換句話說，這種形式不幸地偽裝成比它實際上更「安全」，因此在基本上所有情況下都應該避免使用。如果一個屬性值可能*不是*函式，在嘗試調用之前進行更充分的函式性檢查。不要假裝 `?.(` 在為你做這件事，否則你程式碼的未來讀者/維護者（包括你未來的自己！）可能會後悔。 |

### 在非物件上存取屬性

這聽起來可能違反直覺，但你通常可以從不是物件的值上存取屬性/方法：

```js
fave = 42;

fave;              // 42
fave.toString();   // "42"
```

這裡，`fave` 持有一個原始的 `42` 數值。那麼我們如何能對它進行 `.toString` 來存取屬性，然後用 `()` 來調用持有在該屬性中的函式呢？

這是一個比我們在本書中將要深入討論的更加深入的主題；參見本系列的第四本書《型別與文法》以了解更多。然而，作為一個快速瞥見：如果你對一個非物件、非空值的值執行屬性存取（`.` 或 `[ .. ]`），JS 會預設地（暫時地！）將該值強制轉型為一個物件包裝的表示，允許對那個隱式實例化的物件進行屬性存取。

這個過程通常被稱為「裝箱」，就像將一個值放進一個「箱子」（物件容器）中一樣。

所以在上面的程式碼片段中，只在 `.toString` 被存取 `42` 值的那一刻，JS 會將這個值裝箱為一個 `Number` 物件，然後執行屬性存取。

注意 `null` 和 `undefined` 可以被物件化，通過呼叫 `Object(null)` / `Object(undefined)`。然而，JS 不會自動裝箱這些空值，所以對它們的屬性存取會失敗（如前面在「條件屬性存取」章節中討論的）。

| 備註： |
| :--- |
| 裝箱有一個對應物：拆箱。例如，JS 引擎會取一個物件包裝器——像是用 `Number(42)` 或 `Object(42)` 建立的包裝 `42` 的 `Number` 物件——並解開它以取回底層的原始值 `42`，每當遇到數學運算（如 `*` 或 `-`）時會這樣做。拆箱行為超出了我們討論的範圍，但在前述的《型別與文法》書中有完整介紹。 |

## 賦值屬性

無論屬性是在物件字面量定義時定義的，還是稍後新增的，屬性值的賦值都是使用 `=` 運算子完成的，就像任何其他正常的賦值一樣：

```js
myObj.favoriteNumber = 123;
```

如果 `favoriteNumber` 屬性尚不存在，該語句會建立一個同名的新屬性並賦值。但如果它已經存在，該語句會重新賦值。

| 警告： |
| :--- |
| 對屬性的 `=` 賦值可能會失敗（靜默地或拋出例外），或者它可能不會直接賦值而是調用一個執行某些操作的 *setter* 函式。更多關於這些行為的細節在下一章。 |

也可以一次賦值一個或多個屬性——假設來源屬性（名稱和值對）在另一個物件中——使用 `Object.assign(..)` （在 ES6 中新增）方法：

```js
// 從 `myObj` 淺複製所有（自有且可列舉的）屬性到 `anotherObj`
Object.assign(anotherObj,myObj);

Object.assign(
    /*target=*/anotherObj,
    /*source1=*/{
        someProp: "some value",
        anotherProp: 1001,
    },
    /*source2=*/{
        yetAnotherProp: false
    }
);
```

`Object.assign(..)` 將第一個物件作為目標，第二個（以及可選的後續）物件作為來源。複製方式與前面在「物件展開」章節中描述的相同。

## 刪除屬性

一旦在物件上定義了屬性，移除它的唯一方法是使用 `delete` 運算子：

```js
anotherObj = {
    counter: 123
};

anotherObj.counter;   // 123

delete anotherObj.counter;

anotherObj.counter;   // undefined
```

與常見的誤解相反，JS 的 `delete` 運算子**不會**直接通過垃圾回收（GC）進行任何記憶體的釋放/回收。它唯一做的事情就是從物件中移除屬性。如果屬性中的值是一個參照（指向另一個物件等），並且在屬性被移除後沒有其他存活的參照指向該值，那麼該值在未來的 GC 清掃中可能會有資格被移除。

在物件屬性以外的任何東西上呼叫 `delete` 是對 `delete` 運算子的誤用，會在非嚴格模式下靜默失敗，或在嚴格模式下拋出例外。

從物件中刪除屬性與賦值 `undefined` 或 `null` 是不同的。被賦值為 `undefined` 的屬性，不管是初始的還是稍後的，仍然存在於物件上，並且在列舉內容時可能仍然會被揭示。

## 判斷容器內容

你可以用多種方式判斷物件的內容。要詢問物件是否有特定的屬性：

```js
myObj = {
    favoriteNumber: 42,
    coolFact: "the first person convicted of speeding was going 8 mph",
    beardLength: undefined,
    nicknames: [ "getify", "ydkjs" ]
};

"favoriteNumber" in myObj;            // true

myObj.hasOwnProperty("coolFact");     // true
myObj.hasOwnProperty("beardLength");  // true

myObj.nicknames = undefined;
myObj.hasOwnProperty("nicknames");    // true

delete myObj.nicknames;
myObj.hasOwnProperty("nicknames");    // false
```

`in` 運算子和 `hasOwnProperty(..)` 方法之間*有*一個重要的區別。`in` 運算子不僅會檢查指定的目標物件，如果在那裡找不到，還會查詢物件的 `[[Prototype]]` 鏈（在下一章介紹）。相比之下，`hasOwnProperty(..)` 只查詢目標物件。

如果你仔細觀察，你可能注意到 `myObj` 看起來有一個名為 `hasOwnProperty(..)` 的方法屬性，即使我們沒有定義它。那是因為 `hasOwnProperty(..)` 被定義為 `Object.prototype` 上的內建方法，預設情況下被所有正常物件「繼承」。然而，存取這樣一個「繼承」的方法有固有的風險。再次，更多關於原型的內容在下一章。

### 更好的存在性檢查

ES2022（在撰寫時幾乎已正式通過）已經確定了一個新特性，`Object.hasOwn(..)`。它基本上做與 `hasOwnProperty(..)` 相同的事情，但它是作為一個外部於物件值的靜態輔助方法調用的，而不是透過物件的 `[[Prototype]]`，使其更安全且使用更一致：

```js
// 取代：
myObj.hasOwnProperty("favoriteNumber")

// 我們現在應該偏好：
Object.hasOwn(myObj,"favoriteNumber")
```

即使（在撰寫時）這個特性剛剛在 JS 中出現，也有 polyfill 可以讓你在運行於尚未具有該特性的先前 JS 環境中使用此 API。例如，一個快速的替代 polyfill 草稿：

```js
// Object.hasOwn(..) 的簡單 polyfill 草稿
if (!Object.hasOwn) {
    Object.hasOwn = function hasOwn(obj,propName) {
        return Object.prototype.hasOwnProperty.call(obj,propName);
    };
}
```

在你的程式中包含這樣的 polyfill 補丁意味著你可以安全地開始使用 `Object.hasOwn(..)` 進行屬性存在性檢查，無論 JS 環境是否已經內建了 `Object.hasOwn(..)`。

### 列出所有容器內容

我們已經在前面討論了 `Object.entries(..)` API，它告訴我們物件有哪些屬性（只要它們是可列舉的——更多內容在下一章）。

還有各種其他可用的機制。`Object.keys(..)` 給我們物件中可列舉屬性名稱（又稱鍵）的列表——只有名稱，沒有值；`Object.values(..)` 則給我們可列舉屬性中持有的所有值的列表。

但如果我們想獲取物件中*所有*的鍵（不管是否可列舉）呢？`Object.getOwnPropertyNames(..)` 似乎做了我們想要的，它像 `Object.keys(..)` 但也回傳不可列舉的屬性名稱。然而，這個列表**不會**包含任何 Symbol 屬性名稱，因為它們被視為物件上的特殊位置。`Object.getOwnPropertySymbols(..)` 回傳物件的所有 Symbol 屬性。所以如果你把這兩個列表串接在一起，你就會得到物件的所有直接（*自有的*）內容。

然而，正如我們已經暗示了好幾次，並將在下一章中完整介紹的，物件也可以從其 `[[Prototype]]` 鏈「繼承」內容。這些不被認為是*自有的*內容，所以它們不會出現在任何這些列表中。

回想一下，`in` 運算子會潛在地遍歷整個鏈來尋找屬性的存在。類似地，`for..in` 迴圈會遍歷鏈並列出任何可列舉的（自有的或繼承的）屬性。但沒有內建的 API 可以遍歷整個鏈並回傳*自有的*和*繼承的*內容的組合集合列表。

## 暫時性容器

使用容器來持有多個值有時只是一種暫時的傳輸機制，例如當你想透過單一引數將多個值傳遞給函式時，或者當你想讓函式回傳多個值時：

```js
function formatValues({ one, two, three }) {
    // 作為引數傳入的實際物件是
    // 不可存取的，因為我們將它
    // 解構為三個獨立的變數

    one = one.toUpperCase();
    two = `--${two}--`;
    three = three.substring(0,5);

    // 這個物件只是為了在
    // 單一 return 語句中傳輸
    // 所有三個值
    return { one, two, three };
}

// 解構函式的回傳值，因為
// 那個回傳的物件只是一個暫時的
// 容器，用來傳輸多個值給我們
const { one, two, three } =

    // 這個物件引數是一個暫時的
    // 多個輸入值的傳輸工具
    formatValues({
       one: "Kyle",
       two: "Simpson",
       three: "getify"
    });

one;     // "KYLE"
two;     // "--Simpson--"
three;   // "getif"
```

傳入 `formatValues(..)` 的物件字面量立即被參數解構，所以在函式內部我們只處理三個獨立的變數（`one`、`two` 和 `three`）。從函式 `return` 的物件字面量也立即被解構，所以我們同樣只處理三個獨立的變數（`one`、`two`、`three`）。

這個程式碼片段說明了一個慣用法/模式：物件有時只是一個暫時的傳輸容器，而不是一個有意義的值本身。

## 容器是屬性的集合

物件最常見的用法是作為多個值的容器。我們通過以下方式建立和管理屬性容器物件：

* 定義屬性（命名位置），可以在物件建立時或稍後
* 賦值，可以在物件建立時或稍後
* 稍後存取值，使用位置名稱（屬性名稱）
* 通過 `delete` 刪除屬性
* 使用 `in`、`hasOwnProperty(..)` / `hasOwn(..)`、`Object.entries(..)` / `Object.keys(..)` 等來判斷容器內容

但物件遠不只是靜態的屬性名稱和值的集合。在下一章中，我們將深入了解它們實際上是如何運作的。

[^structuredClone]: "Structured Clone Algorithm", HTML Specification; https://html.spec.whatwg.org/multipage/structured-data.html#structured-cloning ; Accessed July 2022
