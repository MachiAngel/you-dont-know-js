# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 第八章：模組模式

在本章中，我們將透過探討所有程式設計中最重要的程式碼組織模式之一——模組，來為本書的正文劃下句點。正如我們即將看到的，模組本質上是建構在我們已經學過的內容之上的：這是你在學習詞法作用域與閉包方面所付出努力的回報。

我們已經從各個角度檢視了詞法作用域，從全域作用域的廣度到巢狀的區塊作用域，深入到變數生命週期的複雜細節。然後我們利用詞法作用域來理解閉包的全部威力。

花一點時間回顧一下你在這段旅程中已經走了多遠；你在深入了解 JS 方面已經邁出了重要的一步！

本書的核心主題一直是：理解並掌握作用域和閉包是正確地組織我們程式碼的關鍵，尤其是在決定將資訊儲存在變數中的位置方面。

我們在最後一章的目標是體會模組如何體現這些主題的重要性，將它們從抽象概念提升為建構程式時具體、實用的改進。

## 封裝與最小暴露原則（POLE）

封裝經常被引述為物件導向（OO）程式設計的一項原則，但它比那更為基礎且適用範圍更廣。封裝的目標是將共同服務於某一目的的資訊（資料）和行為（函式）捆綁或共同放置在一起。

不依賴於任何語法或程式碼機制，封裝的精神可以簡單地透過使用獨立的檔案來存放程式中具有共同目的的部分來實現。如果我們將驅動搜尋結果列表的所有內容打包到一個名為「search-list.js」的檔案中，我們就封裝了程式的那個部分。

近年來現代前端程式設計中圍繞元件架構來組織應用程式的趨勢，將封裝推進得更遠。對許多人來說，將構成搜尋結果列表的所有內容——甚至超越程式碼，包括展示性的標記語言和樣式——整合到程式邏輯的單一單元中，感覺是很自然的，一個我們可以與之互動的具體事物。然後我們將該集合標記為「SearchList」元件。

另一個關鍵目標是控制封裝資料和功能中某些面向的可見性。回想第六章中的*最小暴露*原則（POLE），它力圖防禦性地防範作用域過度暴露的各種*危險*；這些影響變數和函式。在 JS 中，我們最常透過詞法作用域的機制來實現可見性控制。

其理念是將相似的程式片段分組在一起，並選擇性地限制對我們認為是*私有*細節部分的程式化存取。不被認為是*私有*的部分則被標記為*公開*的，可供整個程式存取。

這些努力的自然效果就是更好的程式碼組織。當我們知道事物在哪裡，有清晰明確的邊界和連接點時，建構和維護軟體就更容易了。如果我們避免資料和功能過度暴露的陷阱，維護品質也會更容易。

這些就是將 JS 程式組織成模組的一些主要好處。

## 什麼是模組？

模組是一組相關的資料和函式（在此語境中常被稱為方法）的集合，其特徵是在隱藏的*私有*細節和*公開*可存取的細節之間有所區分，後者通常稱為「公開 API」。

模組也是有狀態的：它隨著時間維護一些資訊，以及存取和更新該資訊的功能。

| 注意： |
| :--- |
| 模組模式更廣泛的關注點是透過鬆耦合和其他程式架構技術來完全擁抱系統層級的模組化。這是一個複雜的主題，遠超我們討論的範圍，但值得在本書之外進一步研究。 |

為了更好地理解什麼是模組，讓我們將一些模組特徵與一些有用但不完全是模組的程式碼模式進行比較。

### 命名空間（無狀態分組）

如果你將一組相關的函式分組在一起，但沒有資料，那麼你實際上並沒有模組所隱含的預期封裝。對這種*無狀態*函式分組更好的術語是命名空間：

```js
// namespace, not module
var Utils = {
    cancelEvt(evt) {
        evt.preventDefault();
        evt.stopPropagation();
        evt.stopImmediatePropagation();
    },
    wait(ms) {
        return new Promise(function c(res){
            setTimeout(res,ms);
        });
    },
    isValidEmail(email) {
        return /[^@]+@[^@.]+\.[^@.]+/.test(email);
    }
};
```

這裡的 `Utils` 是一組有用的工具函式集合，但它們都是與狀態無關的函式。將功能聚集在一起通常是良好的做法，但這並不能使其成為模組。更確切地說，我們定義了一個 `Utils` 命名空間，並將函式組織在其下。

### 資料結構（有狀態分組）

即使你將資料和有狀態的函式捆綁在一起，如果你沒有限制其中任何部分的可見性，那麼你就沒有達到封裝的 POLE 層面；將其標記為模組並不是特別有益。

考慮以下程式碼：

```js
// data structure, not module
var Student = {
    records: [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ],
    getName(studentID) {
        var student = this.records.find(
            student => student.id == studentID
        );
        return student.name;
    }
};

Student.getName(73);
// Suzy
```

由於 `records` 是可公開存取的資料，而非隱藏在公開 API 之後，這裡的 `Student` 並不真正算是一個模組。

`Student` 確實具有封裝的資料與功能面向，但缺乏可見性控制面向。最好將其標記為資料結構的一個實例。

### 模組（有狀態的存取控制）

要完整體現模組模式的精神，我們不僅需要分組和狀態，還需要透過可見性（私有 vs. 公開）進行存取控制。

讓我們把前面章節中的 `Student` 轉變為一個模組。我們將從一種我稱之為「經典模組」的形式開始，這種形式最初在 2000 年代初期出現時被稱為「揭示模組」。考慮以下程式碼：

```js
var Student = (function defineStudent(){
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
})();

Student.getName(73);   // Suzy
```

`Student` 現在是一個模組的實例。它具有一個公開 API，其中包含一個方法：`getName(..)`。這個方法能夠存取私有隱藏的 `records` 資料。

| 警告： |
| :--- |
| 我應該指出，明確的學生資料被硬編碼到這個模組定義中只是為了我們的說明目的。你程式中的典型模組會從外部來源接收這些資料，通常是從資料庫、JSON 資料檔案、Ajax 呼叫等載入。然後資料通常透過模組公開 API 上的方法注入到模組實例中。 |

經典模組格式是如何運作的？

注意模組的實例是透過執行 `defineStudent()` IIFE（立即呼叫函式表達式）來建立的。這個 IIFE（立即呼叫函式表達式）回傳一個物件（名為 `publicAPI`），該物件上有一個屬性引用了內部的 `getName(..)` 函式。

將物件命名為 `publicAPI` 是我個人的風格偏好。這個物件可以用你喜歡的任何名稱來命名（JS 不在意），或者你可以直接回傳一個物件而不將其賦值給任何內部命名的變數。更多關於這個選擇的內容在附錄 A 中。

從外部來看，`Student.getName(..)` 呼叫了這個暴露的內部函式，該函式透過閉包維持對內部 `records` 變數的存取。

你不*一定*要回傳一個以函式作為其屬性之一的物件。你可以直接回傳一個函式來代替物件。這仍然滿足經典模組的所有核心要素。

藉由詞法作用域的運作方式，在你的外部模組定義函式內部定義的變數和函式*預設*都是私有的。只有添加到從函式回傳的公開 API 物件上的屬性才會被匯出供外部公開使用。

使用 IIFE（立即呼叫函式表達式）意味著我們的程式只需要該模組的一個中央實例，通常被稱為「單例」。確實，這個特定的範例足夠簡單，沒有明顯的理由需要 `Student` 模組的多個實例。

#### 模組工廠（多個實例）

但如果我們確實想要定義一個在程式中支援多個實例的模組，我們可以稍微調整程式碼：

```js
// factory function, not singleton IIFE
function defineStudent() {
    var records = [
        { id: 14, name: "Kyle", grade: 86 },
        { id: 73, name: "Suzy", grade: 87 },
        { id: 112, name: "Frank", grade: 75 },
        { id: 6, name: "Sarah", grade: 91 }
    ];

    var publicAPI = {
        getName
    };

    return publicAPI;

    // ************************

    function getName(studentID) {
        var student = records.find(
            student => student.id == studentID
        );
        return student.name;
    }
}

var fullTime = defineStudent();
fullTime.getName(73);            // Suzy
```

我們不再將 `defineStudent()` 指定為 IIFE（立即呼叫函式表達式），而是將其定義為一個普通的獨立函式，在此語境中通常被稱為「模組工廠」函式。

然後我們呼叫模組工廠，產生一個我們標記為 `fullTime` 的模組實例。這個模組實例意味著一個新的內部作用域實例，因此也是一個新的閉包，`getName(..)` 透過閉包持有對 `records` 的參考。`fullTime.getName(..)` 現在呼叫的是該特定實例上的方法。

#### 經典模組定義

所以，為了釐清什麼構成經典模組：

* 必須有一個外部作用域，通常來自至少執行過一次的模組工廠函式。

* 模組的內部作用域必須至少有一筆代表模組狀態的隱藏資訊。

* 模組必須在其公開 API 上回傳至少一個對隱藏模組狀態持有閉包的函式的參考（這樣該狀態才能實際被保留）。

你可能會遇到這種經典模組方法的其他變體，我們將在附錄 A 中更詳細地探討。

## Node CommonJS 模組

在第四章中，我們介紹了 Node 使用的 CommonJS 模組格式。與前面描述的經典模組格式不同——你可以將模組工廠或 IIFE（立即呼叫函式表達式）與任何其他程式碼（包括其他模組）捆綁在一起——CommonJS 模組是基於檔案的；一個檔案對應一個模組。

讓我們調整我們的模組範例以符合該格式：

```js
module.exports.getName = getName;

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```

`records` 和 `getName` 識別字位於此模組的頂層作用域中，但那不是全域作用域（如第四章所解釋的）。因此，這裡的一切*預設*都是模組私有的。

要在 CommonJS 模組的公開 API 上暴露某些東西，你需要向作為 `module.exports` 提供的空物件添加屬性。在一些較舊的遺留程式碼中，你可能會遇到僅使用裸 `exports` 的參考，但為了程式碼清晰度，你應該始終使用 `module.` 前綴來完整限定該參考。

就風格而言，我喜歡把我的「匯出」放在頂部，模組實作放在底部。但這些匯出可以放在任何地方。我強烈建議將它們全部集中在一起，放在檔案的頂部或底部。

有些開發者習慣替換預設的匯出物件，像這樣：

```js
// defining a new object for the API
module.exports = {
    // ..exports..
};
```

這種方法有一些怪異之處，包括當多個這樣的模組循環依賴彼此時的意外行為。因此，我建議不要替換該物件。如果你想一次賦值多個匯出，使用物件字面量風格的定義，你可以改用這種方式：

```js
Object.assign(module.exports,{
   // .. exports ..
});
```

這裡發生的事情是用你模組的公開 API 定義 `{ .. }` 物件字面量，然後 `Object.assign(..)` 對現有的 `module.exports` 物件執行所有這些屬性的淺拷貝，而不是替換它。這在便利性和更安全的模組行為之間取得了良好的平衡。

要在你的模組/程式中引入另一個模組實例，使用 Node 的 `require(..)` 方法。假設這個模組位於「/path/to/student.js」，我們可以這樣存取它：

```js
var Student = require("/path/to/student.js");

Student.getName(73);
// Suzy
```

`Student` 現在參考了我們範例模組的公開 API。

CommonJS 模組的行為類似單例實例，與前面介紹的 IIFE（立即呼叫函式表達式）模組定義風格相似。無論你 `require(..)` 同一個模組多少次，你得到的只是對單一共享模組實例的額外參考。

`require(..)` 是一種全有或全無的機制；它包含了模組整個暴露的公開 API 的參考。要有效地只存取 API 的一部分，典型的方法如下：

```js
var getName = require("/path/to/student.js").getName;

// or alternately:

var { getName } = require("/path/to/student.js");
```

與經典模組格式類似，CommonJS 模組 API 中公開匯出的方法持有對內部模組細節的閉包。這就是模組單例狀態在程式的整個生命週期中得以維護的方式。

| 注意： |
| :--- |
| 在 Node 的 `require("student")` 語句中，非絕對路徑（`"student"`）會假設有「.js」副檔名，並搜尋「node_modules」。 |

## 現代 ES 模組（ESM）

ESM 格式與 CommonJS 格式有幾個相似之處。ESM 是基於檔案的，模組實例是單例的，所有內容*預設*都是私有的。一個值得注意的區別是 ESM 檔案被假設為嚴格模式，不需要在頂部加上 `"use strict"` 指令。沒有辦法將 ESM 定義為非嚴格模式。

ESM 使用 `export` 關鍵字來在模組的公開 API 上暴露某些東西，而不是 CommonJS 中的 `module.exports`。`import` 關鍵字取代了 `require(..)` 語句。讓我們調整「students.js」以使用 ESM 格式：

```js
export { getName };

// ************************

var records = [
    { id: 14, name: "Kyle", grade: 86 },
    { id: 73, name: "Suzy", grade: 87 },
    { id: 112, name: "Frank", grade: 75 },
    { id: 6, name: "Sarah", grade: 91 }
];

function getName(studentID) {
    var student = records.find(
        student => student.id == studentID
    );
    return student.name;
}
```

這裡唯一的變化是 `export { getName }` 語句。如前所述，`export` 語句可以出現在檔案中的任何位置，但 `export` 必須位於頂層作用域；它不能在任何其他區塊或函式內部。

ESM 在如何指定 `export` 語句方面提供了相當多的變體。例如：

```js
export function getName(studentID) {
    // ..
}
```

即使 `export` 出現在 `function` 關鍵字之前，這種形式仍然是一個 `function` 宣告，只是同時也被匯出了。也就是說，`getName` 識別字是*函式提升*的（見第五章），所以它在模組的整個作用域中都可用。

另一種允許的變體：

```js
export default function getName(studentID) {
    // ..
}
```

這是所謂的「預設匯出」，它與其他匯出有不同的語義。本質上，「預設匯出」是模組消費者在 `import` 時的一種簡寫，當他們只需要這個單一的預設 API 成員時，為他們提供更簡潔的語法。

非 `default` 的匯出被稱為「具名匯出」。

`import` 關鍵字——像 `export` 一樣，它必須只在 ESM 的頂層、任何區塊或函式之外使用——在語法上也有多種變體。第一種被稱為「具名匯入」：

```js
import { getName } from "/path/to/students.js";

getName(73);   // Suzy
```

如你所見，這種形式只從模組中匯入特定命名的公開 API 成員（跳過任何未明確命名的），並將這些識別字添加到當前模組的頂層作用域中。這種匯入風格對於習慣 Java 等語言中套件匯入的人來說很熟悉。

多個 API 成員可以列在 `{ .. }` 集合中，用逗號分隔。具名匯入也可以用 `as` 關鍵字*重新命名*：

```js
import { getName as getStudentName }
   from "/path/to/students.js";

getStudentName(73);
// Suzy
```

如果 `getName` 是模組的「預設匯出」，我們可以這樣匯入它：

```js
import getName from "/path/to/students.js";

getName(73);   // Suzy
```

這裡唯一的區別是去掉了匯入繫結周圍的 `{ }`。如果你想混合使用預設匯入和其他具名匯入：

```js
import { default as getName, /* .. others .. */ }
   from "/path/to/students.js";

getName(73);   // Suzy
```

相對地，`import` 的另一個主要變體被稱為「命名空間匯入」：

```js
import * as Student from "/path/to/students.js";

Student.getName(73);   // Suzy
```

顯而易見的是，`*` 匯入了所有匯出到 API 的內容，包括預設和具名的，並將它們全部儲存在指定的單一命名空間識別字下。這種方法最接近 JS 歷史上大部分時間的經典模組形式。

| 注意： |
| :--- |
| 截至本文撰寫時，現代瀏覽器已經支援 ESM 好幾年了，但 Node 對 ESM 的穩定支援是相當近期的事情，而且已經持續演進了相當長的時間。這種演進可能還會持續一年或更久；ESM 在 ES6 中被引入 JS 時，為 Node 與 CommonJS 模組的互通性帶來了許多具有挑戰性的相容性問題。請參閱 Node 的 ESM 文件以獲取所有最新細節：https://nodejs.org/api/esm.html |

## 離開作用域

無論你使用經典模組格式（瀏覽器或 Node）、CommonJS 格式（在 Node 中），還是 ESM 格式（瀏覽器或 Node），模組都是組織你程式的功能和資料最有效的方式之一。

模組模式是我們在這本學習書中旅程的結論，我們學習了如何使用詞法作用域的規則來將變數和函式放置在適當的位置。POLE 是我們始終採取的防禦性*預設私有*姿態，確保我們避免過度暴露，並只與必要的最小公開 API 表面積互動。

而在模組之下，我們所有模組狀態得以維護的*魔法*就是閉包利用了詞法作用域系統。

正文到此為止。恭喜你走過了這段相當精彩的旅程！正如我在全書中多次說過的，暫停一下、反思一下、並練習我們剛剛討論的內容，是一個非常好的主意。

當你感到舒適並準備好時，請查看附錄，它們會更深入地探討這些主題的一些角落，並且用一些練習題來挑戰你，以鞏固你所學到的知識。
