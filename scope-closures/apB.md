# You Don't Know JS Yet: Scope & Closures - 2nd Edition
# 附錄 B：練習

本附錄旨在提供一些具有挑戰性且有趣的練習，以測試並鞏固你對本書主要主題的理解。建議你自己動手嘗試這些練習——在實際的程式碼編輯器中！——而不是直接跳到最後的解答。不要作弊！

這些練習沒有你必須完全符合的特定正確答案。你的方法可能與所呈現的解答有些（或很大的！）不同，這沒有關係。

沒有人會評判你的程式碼寫得如何。我的希望是，你讀完這本書後，能夠自信地處理這類建立在堅實知識基礎上的編碼任務。這是這裡唯一的目標。如果你對自己的程式碼感到滿意，我也會感到滿意！

## 彈珠桶

還記得第二章的圖 2 嗎？

<figure>
    <img src="images/fig2.png" width="300" alt="Colored Scope Bubbles" align="center">
    <figcaption><em>圖 2（第二章）：彩色作用域氣泡</em></figcaption>
    <br><br>
</figure>

本練習要求你撰寫一個程式——任何程式！——其中包含巢狀函式和區塊作用域，並滿足以下約束條件：

* 如果你用不同的顏色為所有作用域（包括全域作用域！）著色，你至少需要六種顏色。確保添加程式碼註釋，標記每個作用域的顏色。

    額外挑戰：識別你的程式碼可能具有的任何隱含作用域。

* 每個作用域至少有一個識別符。

* 至少包含兩個函式作用域和至少兩個區塊作用域。

* 至少有一個來自外部作用域的變數必須被巢狀作用域的變數遮蔽（參見第三章）。

* 至少有一個變數參考必須解析到作用域鏈中至少高兩層的變數宣告。

| 提示： |
| :--- |
| 你*可以*只為這個練習撰寫一些無意義的 foo/bar/baz 類型的程式碼，但我建議你嘗試想出一些非瑣碎的、接近真實的程式碼，至少能做一些合理的事情。 |

自己嘗試這個練習，然後查看本附錄末尾的建議解答。

## 閉包（第一部分）

讓我們先用一些常見的計算數學運算來練習閉包：判斷一個值是否為質數（除了 1 和自身之外沒有其他因數），以及生成給定數字的質因數（因數）列表。

例如：

```js
isPrime(11);        // true
isPrime(12);        // false

factorize(11);      // [ 11 ]
factorize(12);      // [ 3, 2, 2 ] --> 3*2*2=12
```

以下是 `isPrime(..)` 的實作，改編自 Math.js 函式庫：[^MathJSisPrime]

```js
function isPrime(v) {
    if (v <= 3) {
        return v > 1;
    }
    if (v % 2 == 0 || v % 3 == 0) {
        return false;
    }
    var vSqrt = Math.sqrt(v);
    for (let i = 5; i <= vSqrt; i += 6) {
        if (v % i == 0 || v % (i + 2) == 0) {
            return false;
        }
    }
    return true;
}
```

以下是 `factorize(..)` 的一個較為基礎的實作（不要與第六章的 `factorial(..)` 混淆）：

```js
function factorize(v) {
    if (!isPrime(v)) {
        let i = Math.floor(Math.sqrt(v));
        while (v % i != 0) {
            i--;
        }
        return [
            ...factorize(i),
            ...factorize(v / i)
        ];
    }
    return [v];
}
```

| 注意： |
| :--- |
| 我稱之為基礎的，是因為它沒有針對效能進行最佳化。它使用二元遞迴（無法進行尾呼叫最佳化），並且會建立大量的中間陣列副本。它也不會以任何方式排序已發現的因數。這個任務有許多許多其他演算法，但我想使用一些簡短且大致可以理解的東西來進行我們的練習。 |

如果你在程式中多次呼叫 `isPrime(4327)`，你可以看到它每次都會經歷所有數十個比較/計算步驟。如果你考慮 `factorize(..)`，它在計算因數列表時會多次呼叫 `isPrime(..)`。而且這些呼叫中很可能大部分都是重複的。這是大量的浪費工作！

這個練習的第一部分是使用閉包來實作一個快取，以記住 `isPrime(..)` 的結果，這樣給定數字的質數性（`true` 或 `false`）只會被計算一次。提示：我們已經在第六章中用 `factorial(..)` 展示了這種快取方式。

如果你看看 `factorize(..)`，它是用遞迴實作的，意味著它會重複呼叫自身。這再次意味著我們很可能會看到大量浪費的呼叫來計算相同數字的質因數。所以練習的第二部分是對 `factorize(..)` 使用相同的閉包快取技術。

對 `isPrime(..)` 和 `factorize(..)` 的快取使用各自獨立的閉包，而不是將它們放在同一個作用域中。

自己嘗試這個練習，然後查看本附錄末尾的建議解答。

### 關於記憶體的一點說明

我想分享一個關於這種閉包快取技術及其對應用程式效能影響的簡短說明。

我們可以看到，透過節省重複呼叫，我們提高了計算速度（在某些情況下，提高幅度相當大）。但這種閉包的用法做出了一個你應該非常清楚的明確取捨。

取捨的是記憶體。我們本質上是在無限制地增長我們的快取（在記憶體中）。如果相關的函式被呼叫了數百萬次且大多數輸入都是唯一的，我們會消耗大量記憶體。這絕對可能值得付出這個代價，但前提是我們認為很可能會看到常見輸入的重複，從而利用到快取的優勢。

如果幾乎每次呼叫都有唯一的輸入，而快取基本上從未被有效*使用*，那麼這就不是一種適合採用的技術。

另外，使用更複雜的快取方法可能是個好主意，例如 LRU（最近最少使用）快取，它限制了其大小；當它達到上限時，LRU 會驅逐那些……嗯，最近最少使用的值！

這裡的缺點是 LRU 本身就相當不簡單。你會想要使用一個高度最佳化的 LRU 實作，並敏銳地意識到所有在起作用的取捨。

## 閉包（第二部分）

在這個練習中，我們將再次透過定義一個 `toggle(..)` 工具函式來練習閉包，它為我們提供一個值的切換器。

你將一個或多個值（作為引數）傳入 `toggle(..)`，並獲得一個函式作為回傳值。該回傳的函式在被重複呼叫時，將按順序在所有傳入的值之間交替/輪換，每次一個。

```js
function toggle(/* .. */) {
    // ..
}

var hello = toggle("hello");
var onOff = toggle("on","off");
var speed = toggle("slow","medium","fast");

hello();      // "hello"
hello();      // "hello"

onOff();      // "on"
onOff();      // "off"
onOff();      // "on"

speed();      // "slow"
speed();      // "medium"
speed();      // "fast"
speed();      // "slow"
```

不傳入任何值給 `toggle(..)` 的邊界情況並不太重要；這樣的切換器實例可以總是回傳 `undefined`。

自己嘗試這個練習，然後查看本附錄末尾的建議解答。

## 閉包（第三部分）

在這第三個也是最後一個關於閉包的練習中，我們將實作一個基本的計算機。`calculator()` 函式將產生一個計算機的實例，它以一個函式的形式（如下方的 `calc(..)`）維護自己的狀態：

```js
function calculator() {
    // ..
}

var calc = calculator();
```

每次呼叫 `calc(..)` 時，你會傳入一個代表計算機按鍵的單一字元。為了讓事情更直接，我們將計算機限制為只支援輸入數字（0-9）、算術運算（+、-、\*、/），以及 "=" 來計算運算結果。運算嚴格按照輸入的順序處理；沒有 "( )" 分組或運算子優先順序。

我們不支援輸入小數，但除法運算可以產生小數。我們不支援輸入負數，但 "-" 運算可以產生負數。因此，你應該能夠透過先輸入一個運算來計算任何負數或小數。然後你可以繼續用那個值進行計算。

`calc(..)` 呼叫的回傳值應該模擬真實計算機上顯示的內容，例如反映剛剛按下的鍵，或在按下 "=" 時計算總數。

例如：

```js
calc("4");     // 4
calc("+");     // +
calc("7");     // 7
calc("3");     // 3
calc("-");     // -
calc("2");     // 2
calc("=");     // 75
calc("*");     // *
calc("4");     // 4
calc("=");     // 300
calc("5");     // 5
calc("-");     // -
calc("5");     // 5
calc("=");     // 0
```

由於這種用法有點笨拙，這裡有一個 `useCalc(..)` 輔助函式，它從一個字串中逐一取出字元來執行計算機，並每次計算顯示結果：

```js
function useCalc(calc,keys) {
    return [...keys].reduce(
        function showDisplay(display,key){
            var ret = String( calc(key) );
            return (
                display +
                (
                  (ret != "" && key == "=") ?
                      "=" :
                      ""
                ) +
                ret
            );
        },
        ""
    );
}

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

這個 `useCalc(..)` 輔助函式最合理的用法是始終讓 "=" 成為最後一個輸入的字元。

計算機顯示的某些總數格式需要特殊處理。我提供了這個 `formatTotal(..)` 函式，每當你的計算機要回傳當前計算的總數時（在輸入 `"="` 之後），都應該使用它：

```js
function formatTotal(display) {
    if (Number.isFinite(display)) {
        // constrain display to max 11 chars
        let maxDigits = 11;
        // reserve space for "e+" notation?
        if (Math.abs(display) > 99999999999) {
            maxDigits -= 6;
        }
        // reserve space for "-"?
        if (display < 0) {
            maxDigits--;
        }

        // whole number?
        if (Number.isInteger(display)) {
            display = display
                .toPrecision(maxDigits)
                .replace(/\.0+$/,"");
        }
        // decimal
        else {
            // reserve space for "."
            maxDigits--;
            // reserve space for leading "0"?
            if (
                Math.abs(display) >= 0 &&
                Math.abs(display) < 1
            ) {
                maxDigits--;
            }
            display = display
                .toPrecision(maxDigits)
                .replace(/0+$/,"");
        }
    }
    else {
        display = "ERR";
    }
    return display;
}
```

不用太擔心 `formatTotal(..)` 的工作原理。它的大部分邏輯是一堆處理，用於將計算機顯示限制在最多 11 個字元，即使需要負號、循環小數，甚至 "e+" 指數表示法。

再次強調，不要太陷入計算機特定行為的泥沼中。專注於閉包的*記憶*能力。

自己嘗試這個練習，然後查看本附錄末尾的建議解答。

## 模組

這個練習是將閉包（第三部分）中的計算機轉換為模組。

我們不會為計算機添加任何額外的功能，只是改變它的介面。我們不再呼叫單一的 `calc(..)` 函式，而是為計算機的每次「按鍵」呼叫公開 API 上的特定方法。輸出保持不變。

這個模組應該表達為一個稱為 `calculator()` 的經典模組工廠函式，而不是一個單例 IIFE，這樣如果需要的話可以建立多個計算機。

公開 API 應該包含以下方法：

* `number(..)`（輸入：「按下」的字元/數字）
* `plus()`
* `minus()`
* `mult()`
* `div()`
* `eq()`

用法如下：

```js
var calc = calculator();

calc.number("4");     // 4
calc.plus();          // +
calc.number("7");     // 7
calc.number("3");     // 3
calc.minus();         // -
calc.number("2");     // 2
calc.eq();            // 75
```

`formatTotal(..)` 與前一個練習保持相同。但 `useCalc(..)` 輔助函式需要調整以配合模組 API：

```js
function useCalc(calc,keys) {
    var keyMappings = {
        "+": "plus",
        "-": "minus",
        "*": "mult",
        "/": "div",
        "=": "eq"
    };

    return [...keys].reduce(
        function showDisplay(display,key){
            var fn = keyMappings[key] || "number";
            var ret = String( calc[fn](key) );
            return (
                display +
                (
                  (ret != "" && key == "=") ?
                      "=" :
                      ""
                ) +
                ret
            );
        },
        ""
    );
}

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

自己嘗試這個練習，然後查看本附錄末尾的建議解答。

在你完成這個練習的同時，也花一些時間思考將計算機表示為模組相較於前一個練習的閉包函式方法的優缺點。

額外挑戰：寫幾句話來解釋你的想法。

額外挑戰 #2：嘗試將你的模組轉換為其他模組格式，包括：UMD、CommonJS 和 ESM（ES Modules）。

## 建議解答

希望你在讀到這裡之前已經嘗試過這些練習了。不要作弊！

記住，每個建議解答只是眾多不同方法中的一種。它們不是「正確答案」，但它們確實展示了處理每個練習的合理方式。

你從閱讀這些建議解答中能獲得的最重要的好處是，將它們與你的程式碼進行比較，並分析我們各自做出相似或不同選擇的原因。不要太糾結於細枝末節；試著專注於主要主題而不是小細節。

### 建議解答：彈珠桶

*彈珠桶練習*可以這樣解決：

```js
// RED(1)
const howMany = 100;

// Sieve of Eratosthenes
function findPrimes(howMany) {
    // BLUE(2)
    var sieve = Array(howMany).fill(true);
    var max = Math.sqrt(howMany);

    for (let i = 2; i < max; i++) {
        // GREEN(3)
        if (sieve[i]) {
            // ORANGE(4)
            let j = Math.pow(i,2);
            for (let k = j; k < howMany; k += i) {
                // PURPLE(5)
                sieve[k] = false;
            }
        }
    }

    return sieve
        .map(function getPrime(flag,prime){
            // PINK(6)
            if (flag) return prime;
            return flag;
        })
        .filter(function onlyPrimes(v){
            // YELLOW(7)
            return !!v;
        })
        .slice(1);
}

findPrimes(howMany);
// [
//    2, 3, 5, 7, 11, 13, 17,
//    19, 23, 29, 31, 37, 41,
//    43, 47, 53, 59, 61, 67,
//    71, 73, 79, 83, 89, 97
// ]
```

### 建議解答：閉包（第一部分）

*閉包練習（第一部分）*中的 `isPrime(..)` 和 `factorize(..)`，可以這樣解決：

```js
var isPrime = (function isPrime(v){
    var primes = {};

    return function isPrime(v) {
        if (v in primes) {
            return primes[v];
        }
        if (v <= 3) {
            return (primes[v] = v > 1);
        }
        if (v % 2 == 0 || v % 3 == 0) {
            return (primes[v] = false);
        }
        let vSqrt = Math.sqrt(v);
        for (let i = 5; i <= vSqrt; i += 6) {
            if (v % i == 0 || v % (i + 2) == 0) {
                return (primes[v] = false);
            }
        }
        return (primes[v] = true);
    };
})();

var factorize = (function factorize(v){
    var factors = {};

    return function findFactors(v) {
        if (v in factors) {
            return factors[v];
        }
        if (!isPrime(v)) {
            let i = Math.floor(Math.sqrt(v));
            while (v % i != 0) {
                i--;
            }
            return (factors[v] = [
                ...findFactors(i),
                ...findFactors(v / i)
            ]);
        }
        return (factors[v] = [v]);
    };
})();
```

我對每個工具函式使用的一般步驟：

1. 包裝一個 IIFE 來定義快取變數所在的作用域。

2. 在底層呼叫中，首先檢查快取，如果結果已經知道，就直接回傳。

3. 在每個原本會 `return` 的地方，賦值給快取並直接回傳該賦值運算的結果——這主要是一個節省空間的技巧，僅為了在書中簡潔起見。

我也將內部函式從 `factorize(..)` 重新命名為 `findFactors(..)`。這在技術上不是必要的，但它有助於更清楚地說明遞迴呼叫調用的是哪個函式。

### 建議解答：閉包（第二部分）

*閉包練習（第二部分）*的 `toggle(..)` 可以這樣解決：

```js
function toggle(...vals) {
    var unset = {};
    var cur = unset;

    return function next(){
        // save previous value back at
        // the end of the list
        if (cur != unset) {
            vals.push(cur);
        }
        cur = vals.shift();
        return cur;
    };
}

var hello = toggle("hello");
var onOff = toggle("on","off");
var speed = toggle("slow","medium","fast");

hello();      // "hello"
hello();      // "hello"

onOff();      // "on"
onOff();      // "off"
onOff();      // "on"

speed();      // "slow"
speed();      // "medium"
speed();      // "fast"
speed();      // "slow"
```

### 建議解答：閉包（第三部分）

*閉包練習（第三部分）*的 `calculator()` 可以這樣解決：

```js
// from earlier:
//
// function useCalc(..) { .. }
// function formatTotal(..) { .. }

function calculator() {
    var currentTotal = 0;
    var currentVal = "";
    var currentOper = "=";

    return pressKey;

    // ********************

    function pressKey(key){
        // number key?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
        // operator key?
        else if (/[+*/-]/.test(key)) {
            // multiple operations in a series?
            if (
                currentOper != "=" &&
                currentVal != ""
            ) {
                // implied '=' keypress
                pressKey("=");
            }
            else if (currentVal != "") {
                currentTotal = Number(currentVal);
            }
            currentOper = key;
            currentVal = "";
            return key;
        }
        // = key?
        else if (
            key == "=" &&
            currentOper != "="
        ) {
            currentTotal = op(
                currentTotal,
                currentOper,
                Number(currentVal)
            );
            currentOper = "=";
            currentVal = "";
            return formatTotal(currentTotal);
        }
        return "";
    };

    function op(val1,oper,val2) {
        var ops = {
            // NOTE: using arrow functions
            // only for brevity in the book
            "+": (v1,v2) => v1 + v2,
            "-": (v1,v2) => v1 - v2,
            "*": (v1,v2) => v1 * v2,
            "/": (v1,v2) => v1 / v2
        };
        return ops[oper](val1,val2);
    }
}

var calc = calculator();

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

| 注意： |
| :--- |
| 記住：這個練習是關於閉包的。不要太關注計算機的實際機制，而是關注你是否正確地在函式呼叫之間*記住*了計算機的狀態。 |

### 建議解答：模組

*模組練習*的 `calculator()` 可以這樣解決：

```js
// from earlier:
//
// function useCalc(..) { .. }
// function formatTotal(..) { .. }

function calculator() {
    var currentTotal = 0;
    var currentVal = "";
    var currentOper = "=";

    var publicAPI = {
        number,
        eq,
        plus() { return operator("+"); },
        minus() { return operator("-"); },
        mult() { return operator("*"); },
        div() { return operator("/"); }
    };

    return publicAPI;

    // ********************

    function number(key) {
        // number key?
        if (/\d/.test(key)) {
            currentVal += key;
            return key;
        }
    }

    function eq() {
        // = key?
        if (currentOper != "=") {
            currentTotal = op(
                currentTotal,
                currentOper,
                Number(currentVal)
            );
            currentOper = "=";
            currentVal = "";
            return formatTotal(currentTotal);
        }
        return "";
    }

    function operator(key) {
        // multiple operations in a series?
        if (
            currentOper != "=" &&
            currentVal != ""
        ) {
            // implied '=' keypress
            eq();
        }
        else if (currentVal != "") {
            currentTotal = Number(currentVal);
        }
        currentOper = key;
        currentVal = "";
        return key;
    }

    function op(val1,oper,val2) {
        var ops = {
            // NOTE: using arrow functions
            // only for brevity in the book
            "+": (v1,v2) => v1 + v2,
            "-": (v1,v2) => v1 - v2,
            "*": (v1,v2) => v1 * v2,
            "/": (v1,v2) => v1 / v2
        };
        return ops[oper](val1,val2);
    }
}

var calc = calculator();

useCalc(calc,"4+3=");           // 4+3=7
useCalc(calc,"+9=");            // +9=16
useCalc(calc,"*8=");            // *5=128
useCalc(calc,"7*2*3=");         // 7*2*3=42
useCalc(calc,"1/0=");           // 1/0=ERR
useCalc(calc,"+3=");            // +3=ERR
useCalc(calc,"51=");            // 51
```

這本書就到這裡了，恭喜你的成就！當你準備好時，繼續前往第三本書，*Objects & Classes*。

[^MathJSisPrime]: *Math.js: isPrime(..)*, https://github.com/josdejong/mathjs/blob/develop/src/function/utils/isPrime.js, 3 March 2020.
