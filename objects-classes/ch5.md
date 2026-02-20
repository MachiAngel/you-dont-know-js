# 你所不知道的 JS：物件與類別 - 第二版
# 第五章：委託

| 備註： |
| :--- |
| 撰寫中 |

我們已經徹底探索了物件、原型、類別，以及現在的 `this` 關鍵字。但我們現在要從一個稍微不同的角度，重新審視到目前為止所學的內容。

如果你能同時運用物件、原型和動態 `this` 機制的所有威力，卻完全不使用 `class` 或其任何衍生物呢？

事實上，我認為 JS 本質上並不像 `class` 關鍵字所表現的那樣以類別為導向。因為 JS 是一門動態的、基於原型的語言，它真正的強項其實是……*委託*。

## 前言

在我們開始探討委託之前，我想先提出一個警告。這種關於 JS 物件 `[[Prototype]]` 和 `this` 函式上下文機制的觀點並*非*主流。它*不是*框架作者和函式庫使用 JS 的方式。據我所知，你不會找到任何大型應用程式使用這種模式。

那麼，如果這種模式如此不受歡迎，我為什麼要用一整章來介紹它呢？

好問題。俏皮的回答是：因為這是我的書，我想做什麼就做什麼！

但更深層的回答是，因為我認為培養對語言核心支柱之一的*這種*理解，即使你只會使用 `class` 風格的 JS 模式，也能幫助你。

需要澄清的是，委託不是我發明的。它作為一種設計模式已經存在了幾十年。長久以來，開發者們爭論原型委託*只是*繼承的動態形式。[^TreatyOfOrlando] 但我認為將兩者混為一談是個錯誤。[^ClassVsPrototype]

在本章中，我將透過 JS 機制來實現委託，將其作為一種替代設計模式來呈現，定位在類別導向和物件閉包/模組模式之間。

第一步是將 `class` 機制*解構*為其各個組成部分。然後我們會挑選這些部分，並以稍微不同的方式重新組合。

## 建構子到底是什麼？

在第三章中，我們看到 `constructor(..)` 是建構 `class` 實例的主要入口點。但 `constructor(..)` 實際上並不做任何*建立*工作，它只是做*初始化*工作。換句話說，實例在 `constructor(..)` 執行並初始化它之前就已經被建立了——例如，`this.whatever` 類型的賦值。

那麼*建立*工作實際上發生在哪裡？在 `new` 運算子中。正如第四章「新上下文調用」一節所解釋的，`new` 關鍵字執行四個步驟；其中第一步就是建立一個新的空物件（實例）。`constructor(..)` 直到 `new` 工作的第 3 步才被調用。

但 `new` 不是*建立*物件「實例」的唯一方式——甚至可能不是最好的方式。考慮以下程式碼：

```js
// a non-class "constructor"
function Point2d(x,y) {
    // create an object (1)
    var instance = {};

    // initialize the instance (3)
    instance.x = x;
    instance.y = y;

    // return the instance (4)
    return instance;
}

var point = Point2d(3,4);

point.x;                    // 3
point.y;                    // 4
```

這裡沒有 `class`，只有一個普通的函式定義（`Point2d(..)`）。沒有 `new` 調用，只有一個普通的函式呼叫（`Point2d(3,4)`）。也沒有 `this` 參照，只有普通的物件屬性賦值（`instance.x = ..`）。

最常用來指稱這種程式碼模式的術語是，這裡的 `Point2d(..)` 是一個*工廠函式*。調用它會導致物件的建構（建立和初始化），並將其回傳給我們。這是一種極其常見的模式，至少和類別導向的程式碼一樣常見。

我在上面的程式碼片段中用註解標註了 `(1)`、`(3)` 和 `(4)`，大致對應 `new` 操作的步驟 1、3 和 4。但步驟 2 在哪裡？

如果你還記得，`new` 的步驟 2 是透過 `[[Prototype]]` 槽將物件（在步驟 1 中建立的）連結到另一個物件（見第二章）。那麼我們可能想要將 `instance` 物件連結到什麼物件呢？我們可以將它連結到一個持有函式的物件，這些函式是我們想要與實例關聯/使用的。

讓我們修改之前的程式碼片段：

```js
var prototypeObj = {
    toString() {
        return `(${this.x},${this.y})`;
    },
}

// a non-class "constructor"
function Point2d(x,y) {
    // create an object (1)
    var instance = {
        // link the instance's [[Prototype]] (2)
        __proto__: prototypeObj,
    };

    // initialize the instance (3)
    instance.x = x;
    instance.y = y;

    // return the instance (4)
    return instance;
}

var point = Point2d(3,4);

point.toString();           // (3,4)
```

現在你可以看到 `__proto__` 賦值正在設定內部 `[[Prototype]]` 連結，這就是之前缺少的步驟 2。我在這裡使用 `__proto__` 只是為了說明目的；使用第四章中展示的 `setPrototypeOf(..)` 也可以完成同樣的任務。

### *新的*工廠實例

如果我們使用 `new` 來調用這裡展示的 `Point2d(..)` 函式，你認為會發生什麼？

```js
var anotherPoint = new Point2d(5,6);

anotherPoint.toString(5,6);         // (5,6)
```

等等！這是怎麼回事？一個普通的、非 `class` 工廠函式被用 `new` 關鍵字調用，就好像它是一個 `class` 一樣。這會改變程式碼的結果嗎？

不會……但也會。這裡的 `anotherPoint` 和不使用 `new` 時的物件完全一樣。但是！`new` 建立、連結並指定為 `this` 上下文的那個物件呢？*那個*物件被完全忽略並丟棄了，最終會被 JS 的垃圾回收機制回收。不幸的是，JS 引擎無法預測你不會使用 `new` 建立的物件，所以即使它不會被使用，它仍然會被建立。

沒錯！對工廠函式使用 `new` 關鍵字可能*感覺*更符合人體工學或更熟悉，但它相當浪費，因為它建立了**兩個**物件，並且浪費地丟棄了其中一個。

### 工廠初始化

在目前的程式碼範例中，`Point2d(..)` 函式看起來仍然很像 `class` 定義的普通 `constructor(..)`。但如果我們把初始化程式碼移到一個單獨的函式中，比如命名為 `init(..)` 呢：

```js
var prototypeObj = {
    init(x,y) {
        // initialize the instance (3)
        this.x = x;
        this.y = y;
    },
    toString() {
        return `(${this.x},${this.y})`;
    },
}

// a non-class "constructor"
function Point2d(x,y) {
    // create an object (1)
    var instance = {
        // link the instance's [[Prototype]] (2)
        __proto__: prototypeObj,
    };

    // initialize the instance (3)
    instance.init(x,y);

    // return the instance (4)
    return instance;
}

var point = Point2d(3,4);

point.toString();           // (3,4)
```

`instance.init(..)` 呼叫利用了透過 `__proto__` 賦值建立的 `[[Prototype]]` 連結。因此，它沿著原型鏈向上*委託*到 `prototypeObj.init(..)`，並以 `instance` 的 `this` 上下文調用它——透過*隱式上下文*指定（見第四章）。

讓我們繼續解構。準備好迎接一個大轉換！

```js
var Point2d = {
    init(x,y) {
        // initialize the instance (3)
        this.x = x;
        this.y = y;
    },
    toString() {
        return `(${this.x},${this.y})`;
    },
};
```

什麼！？我丟棄了 `Point2d(..)` 函式，轉而將 `prototypeObj` 重新命名為 `Point2d`。很奇怪。

但讓我們來看看其餘的程式碼：

```js
// steps 1, 2, and 4
var point = { __proto__: Point2d, };

// step 3
point.init(3,4);

point.toString();           // (3,4)
```

最後一個改進：讓我們使用 JS 提供給我們的內建工具，叫做 `Object.create(..)`：

```js
// steps 1, 2, and 4
var point = Object.create(Point2d);

// step 3
point.init(3,4);

point.toString();           // (3,4)
```

`Object.create(..)` 執行了什麼操作？

1. 憑空建立一個全新的空物件。

2. 將該新空物件的 `[[Prototype]]` 連結到函式的 `.prototype` 物件。

如果這些看起來很熟悉，那是因為它們與 `new` 關鍵字的前兩個步驟完全相同（見第四章）。

讓我們現在把這些重新組合起來：

```js
var Point2d = {
    init(x,y) {
        this.x = x;
        this.y = y;
    },
    toString() {
        return `(${this.x},${this.y})`;
    },
};

var point = Object.create(Point2d);

point.init(3,4);

point.toString();           // (3,4)
```

嗯。花幾分鐘時間思考一下這裡推導出了什麼。它與 `class` 方法相比如何？

這種模式拋棄了 `class` 和 `new` 關鍵字，但達成了完全相同的結果。*代價*是什麼？單一的 `new` 操作被分解為兩個語句：`Object.create(Point2d)` 和 `point.init(3,4)`。

#### 幫我重新建構！

如果將這兩個操作分開讓你感到困擾——這是不是*解構得太過頭*了！？——它們總是可以在一個小型工廠輔助函式中重新組合：

```js
function make(objType,...args) {
    var instance = Object.create(objType);
    instance.init(...args);
    return instance;
}

var point = make(Point2d,3,4);

point.toString();           // (3,4)
```

| 提示： |
| :--- |
| 這樣的 `make(..)` 工廠函式輔助器可以通用於任何物件類型，只要你遵循隱含的慣例，即你連結到的每個 `objType` 上都有一個名為 `init(..)` 的函式。 |

當然，你仍然可以建立任意數量的實例：

```js
var point = make(Point2d,3,4);

var anotherPoint = make(Point2d,5,6);
```

## 拋棄類別思維

坦白說，我們剛才進行的*解構*，與 `class` 風格相比，最終只是得到了略有不同、也許稍好或稍差的程式碼。如果委託僅僅是這樣，它可能連一個註腳都不值得，更不用說一整章了。

但這正是我們要真正開始推開類別導向思維本身的地方，不只是語法。

類別導向設計本質上建立了一個*分類*層次結構，意即我們如何劃分和分組特徵，然後將它們垂直堆疊在繼承鏈中。此外，定義子類別是對通用基礎類別的特化。實例化是對通用類別的特化。

在傳統的類別層次結構中，行為是透過繼承鏈各層的垂直組合。幾十年來，人們一直嘗試——有時甚至變得相當流行——扁平化深層的繼承層次結構，並偏好透過 *mixin* 和相關概念進行更水平的組合。

我並不是在斷言這些方法有什麼問題。但我想說的是，它們並*不是* JS 天生運作的方式，所以在 JS 中採用它們一直是一條漫長、曲折、複雜的道路，並且逐漸累積了大量細微的語法來改裝在 JS 的核心 `[[Prototype]]` 和 `this` 支柱之上。

在本章的剩餘部分，我打算同時拋棄 `class` 的語法*和*類別的*思維*。

## 委託圖解

那麼委託到底是什麼？在其核心，它是關於兩個或更多*事物*共同分擔完成一項任務的努力。

委託不是定義一個代表共享行為的 `Point2d` 通用父*物件*，讓一組一個或多個子 `point` / `anotherPoint` *物件*從中繼承，而是引導我們使用離散的對等*物件*來建構程式，這些物件彼此合作。

我將用一些程式碼來描繪：

```js
var Coordinates = {
    setX(x) {
        this.x = x;
    },
    setY(y) {
        this.y = y;
    },
    setXY(x,y) {
        this.setX(x);
        this.setY(y);
    },
};

var Inspect = {
    toString() {
        return `(${this.x},${this.y})`;
    },
};

var point = {};

Coordinates.setXY.call(point,3,4);
Inspect.toString.call(point);         // (3,4)

var anotherPoint = Object.create(Coordinates);

anotherPoint.setXY(5,6);
Inspect.toString.call(anotherPoint);  // (5,6)
```

讓我們分解一下這裡發生了什麼。

我定義了 `Coordinates` 作為一個具體物件，持有一些我與設定點座標（`x` 和 `y`）相關聯的行為。我還定義了 `Inspect` 作為一個具體物件，持有一些除錯檢查邏輯，例如 `toString()`。

然後我建立了兩個更多的具體物件，`point` 和 `anotherPoint`。

`point` 沒有特定的 `[[Prototype]]`（預設值：`Object.prototype`）。使用*顯式上下文*指定（見第四章），我在 `point` 的上下文中調用了 `Coordinates.setXY(..)` 和 `Inspect.toString()` 工具函式。這就是我所說的*顯式委託*。

`anotherPoint` 透過 `[[Prototype]]` 連結到 `Coordinates`，主要是為了方便。這讓我可以使用*隱式上下文*指定來呼叫 `anotherPoint.setXY(..)`。但我仍然可以*顯式地*將 `anotherPoint` 作為 `Inspect.toString()` 呼叫的上下文共享。這就是我所說的*隱式委託*。

**不要錯過*這*點：** 我們仍然完成了組合：我們在執行時的函式調用中透過 `this` 上下文共享，組合了來自 `Coordinates` 和 `Inspect` 的行為。我們不必將這些行為在撰寫時組合到單一的 `class`（或基礎-子類別 `class` 層次結構）中供 `point` / `anotherPoint` 繼承。我喜歡稱這種執行時組合為**虛擬組合**。

這裡的*重點*是：這四個物件中沒有一個是父或子。它們都是彼此的對等體，而且它們都有不同的用途。我們可以將行為組織在邏輯區塊中（在各自的物件上），並透過 `this`（以及可選的 `[[Prototype]]` 連結）共享上下文，這最終與我們在本書中至今檢視的其他模式產生相同的組合結果。

*這*就是**委託**模式的核心，正如 JS 所體現的。

| 提示： |
| :--- |
| 在本書系列的第一版中，這本書（「this 與物件原型」）創造了一個術語「OLOO」，代表「Objects Linked to Other Objects」（物件連結到其他物件）——以對比「OO」（「Object Oriented」，物件導向）。在前面的範例中，你可以看到 OLOO 的精髓：我們擁有的只是物件，連結到其他物件並與之合作。我覺得這種簡潔性很美。 |

## 組合對等物件

讓我們將*這種委託*更進一步。

在前面的程式碼片段中，`point` 和 `anotherPoint` 只是持有資料，它們委託的行為在其他物件上（`Coordinates` 和 `Inspect`）。但我們可以直接在委託鏈中的任何物件上添加行為，而且這些行為甚至可以透過*虛擬組合*（`this` 上下文共享）的魔力相互作用。

為了說明，我們將把目前的*點*範例進行相當多的演進。作為額外好處，我們實際上會將點繪製在 DOM 中的 `<canvas>` 元素上。讓我們來看看：

```js
var Canvas = {
    setOrigin(x,y) {
        this.ctx.translate(x,y);

        // flip the canvas context vertically,
        // so coordinates work like on a normal
        // 2d (x,y) graph
        this.ctx.scale(1,-1);
    },
    pixel(x,y) {
        this.ctx.fillRect(x,y,1,1);
    },
    renderScene() {
        // clear the canvas
        var matrix = this.ctx.getTransform();
        this.ctx.resetTransform();
        this.ctx.clearRect(
            0, 0,
            this.ctx.canvas.width,
            this.ctx.canvas.height
        );
        this.ctx.setTransform(matrix);

        this.draw();  // <-- where is draw()?
    },
};

var Coordinates = {
    setX(x) {
        this.x = Math.round(x);
    },
    setY(y) {
        this.y = Math.round(y);
    },
    setXY(x,y) {
        this.setX(x);
        this.setY(y);
        this.render();   // <-- where is render()?
    },
};

var ControlPoint = {
    // delegate to Coordinates
    __proto__: Coordinates,

    // NOTE: must have a <canvas id="my-canvas">
    // element in the DOM
    ctx: document.getElementById("my-canvas")
        .getContext("2d"),

    rotate(angleRadians) {
        var rotatedX = this.x * Math.cos(angleRadians) -
            this.y * Math.sin(angleRadians);
        var rotatedY = this.x * Math.sin(angleRadians) +
            this.y * Math.cos(angleRadians);
        this.setXY(rotatedX,rotatedY);
    },
    draw() {
        // plot the point
        Canvas.pixel.call(this,this.x,this.y);
    },
    render() {
        // clear the canvas, and re-render
        // our control-point
        Canvas.renderScene.call(this);
    },
};

// set the logical (0,0) origin at this
// physical location on the canvas
Canvas.setOrigin.call(ControlPoint,100,100);

ControlPoint.setXY(30,40);
// [renders point (30,40) on the canvas]

// ..
// later:

// rotate the point about the (0,0) origin
// 90 degrees counter-clockwise
ControlPoint.rotate(Math.PI / 2);
// [renders point (-40,30) on the canvas]
```

好的，這是大量需要消化的程式碼。慢慢來，多讀幾遍這段程式碼。我在之前的 `Coordinates` 物件旁邊添加了幾個新的具體物件（`Canvas` 和 `ControlPoint`）。

確保你看到並理解這三個具體物件之間的互動。

`ControlPoint` 透過 `__proto__` 連結，以*隱式委託*（`[[Prototype]]` 鏈）到 `Coordinates`。

這是一個*顯式委託*：`Canvas.setOrigin.call(ControlPoint,100,100);`；我在 `ControlPoint` 的上下文中調用 `Canvas.setOrigin(..)` 呼叫。這具有透過 `this` 將 `ctx` 與 `setOrigin(..)` 共享的效果。

`ControlPoint.setXY(..)` *隱式*委託到 `Coordinates.setXY(..)`，但仍然在 `ControlPoint` 的上下文中。這裡有一個容易忽略的關鍵細節：看到 `Coordinates.setXY(..)` 裡面的 `this.render()` 了嗎？它從哪裡來？由於 `this` 上下文是 `ControlPoint`（不是 `Coordinates`），它調用的是 `ControlPoint.render()`。

`ControlPoint.render()` *顯式委託*到 `Canvas.renderScene()`，同樣仍然在 `ControlPoint` 的上下文中。`renderScene()` 呼叫 `this.draw()`，但它從哪裡來？沒錯，仍然來自 `ControlPoint`（透過 `this` 上下文）。

而 `ControlPoint.draw()` 呢？它*顯式委託*到 `Canvas.pixel(..)`，再次仍然在 `ControlPoint` 的上下文中。

所有三個物件都有最終會相互調用的方法。但這些呼叫並不是特別硬編碼的。`Canvas.renderScene()` 不是呼叫 `ControlPoint.draw()`，它呼叫的是 `this.draw()`。這很重要，因為它意味著 `Canvas.renderScene()` 在不同的 `this` 上下文中使用時更加靈活——例如，對著 `ControlPoint` 以外的另一種*點*物件使用。

正是透過 `this` 上下文和 `[[Prototype]]` 鏈，這三個物件基本上在每一步按需被虛擬地混合（組合）在一起，使它們**就像是一個物件而非三個獨立的物件**一樣協同工作。

這就是 JS 中委託模式實現的虛擬組合之*美*。

### 靈活的上下文

我在上面提到，我們可以很容易地將其他具體物件加入其中。這裡有一個範例：

```js
var Coordinates = { /* .. */ };

var Canvas = {
    /* .. */
    line(start,end) {
        this.ctx.beginPath();
        this.ctx.moveTo(start.x,start.y);
        this.ctx.lineTo(end.x,end.y);
        this.ctx.stroke();
    },
};

function lineAnchor(x,y) {
    var anchor = {
        __proto__: Coordinates,
        render() {},
    };
    anchor.setXY(x,y);
    return anchor;
}

var GuideLine = {
    // NOTE: must have a <canvas id="my-canvas">
    // element in the DOM
    ctx: document.getElementById("my-canvas")
        .getContext("2d"),

    setAnchors(sx,sy,ex,ey) {
        this.start = lineAnchor(sx,sy);
        this.end = lineAnchor(ex,ey);
        this.render();
    },
    draw() {
        // plot the point
        Canvas.line.call(this,this.start,this.end);
    },
    render() {
        // clear the canvas, and re-render
        // our line
        Canvas.renderScene.call(this);
    },
};

// set the logical (0,0) origin at this
// physical location on the canvas
Canvas.setOrigin.call(GuideLine,100,100);

GuideLine.setAnchors(-30,65,45,-17);
// [renders line from (-30,65) to (45,-17)
//   on the canvas]
```

我覺得這相當不錯！

但我認為另一個不太明顯的好處是，透過 `this` 上下文動態連結物件，往往使得獨立測試程式的不同部分變得更加容易。

例如，`Object.setPrototypeOf(..)` 可以用來動態改變物件的 `[[Prototype]]` 連結，將其委託到一個不同的物件，例如一個模擬物件。或者你可以動態重新定義 `GuideLine.draw()` 和 `GuideLine.render()`，使其*顯式委託*到 `MockCanvas` 而不是 `Canvas`。

當你充分理解並運用 `this` 關鍵字和 `[[Prototype]]` 連結時，它們是一個極其靈活的機制。

## 為什麼用 *This*？

好的，希望已經清楚，委託模式嚴重依賴隱式輸入，透過 `this` 共享上下文，而不是透過顯式參數。

你可能有道理地問，為什麼不總是顯式地傳遞上下文？我們當然可以這樣做，但是……要手動傳遞必要的上下文，我們將不得不改變幾乎每一個函式簽名，以及任何相應的呼叫處。

讓我們重新審視之前的 `ControlPoint` 委託範例，並在沒有任何委託導向的 `this` 上下文共享的情況下實現它。請仔細注意差異：

```js
var Canvas = {
    setOrigin(ctx,x,y) {
        ctx.translate(x,y);
        ctx.scale(1,-1);
    },
    pixel(ctx,x,y) {
        ctx.fillRect(x,y,1,1);
    },
    renderScene(ctx,entity) {
        // clear the canvas
        var matrix = ctx.getTransform();
        ctx.resetTransform();
        ctx.clearRect(
            0, 0,
            ctx.canvas.width,
            ctx.canvas.height
        );
        ctx.setTransform(matrix);

        entity.draw();
    },
};

var Coordinates = {
    setX(entity,x) {
        entity.x = Math.round(x);
    },
    setY(entity,y) {
        entity.y = Math.round(y);
    },
    setXY(entity,x,y) {
        this.setX(entity,x);
        this.setY(entity,y);
        entity.render();
    },
};

var ControlPoint = {
    // NOTE: must have a <canvas id="my-canvas">
    // element in the DOM
    ctx: document.getElementById("my-canvas")
        .getContext("2d"),

    setXY(x,y) {
        Coordinates.setXY(this,x,y);
    },
    rotate(angleRadians) {
        var rotatedX = this.x * Math.cos(angleRadians) -
            this.y * Math.sin(angleRadians);
        var rotatedY = this.x * Math.sin(angleRadians) +
            this.y * Math.cos(angleRadians);
        this.setXY(rotatedX,rotatedY);
    },
    draw() {
        // plot the point
        Canvas.pixel(this.ctx,this.x,this.y);
    },
    render() {
        // clear the canvas, and re-render
        // our control-point
        Canvas.renderScene(this.ctx,this);
    },
};

// set the logical (0,0) origin at this
// physical location on the canvas
Canvas.setOrigin(ControlPoint.ctx,100,100);

// ..
```

說實話，你們中有些人可能更喜歡這種風格的程式碼。如果你屬於那個陣營，那也沒關係。這個程式碼片段完全避免了 `[[Prototype]]`，只依賴遠少於此的基本 `this.` 風格的屬性和方法參照。

相比之下，我在本章中倡導的委託風格是不熟悉的，並且以你可能不熟悉的方式使用 `[[Prototype]]` 和 `this` 共享。要有效地使用這種風格，你必須投入時間和練習來建立更深的熟悉度。

但在我看來，避免透過委託進行虛擬組合的「成本」可以在所有函式簽名和呼叫處感受到；我覺得它們更加雜亂。那種顯式上下文傳遞是相當大的負擔。

事實上，我根本不會提倡那種風格的程式碼。如果你想避免委託，最好還是堅持使用 `class` 風格的程式碼，如第三章所示。作為留給讀者的練習，試著將之前的 `ControlPoint` / `GuideLine` 程式碼片段轉換為使用 `class`。

[^TreatyOfOrlando]: "Treaty of Orlando"; Henry Lieberman, Lynn Andrea Stein, David Ungar; Oct 6, 1987; https://web.media.mit.edu/~lieber/Publications/Treaty-of-Orlando-Treaty-Text.pdf ; PDF; Accessed July 2022

[^ClassVsPrototype]: "Classes vs. Prototypes, Some Philosophical and Historical Observations"; Antero Taivalsaari; Apr 22, 1996; https://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.56.4713&rep=rep1&type=pdf ; PDF; Accessed July 2022
