# You Don't Know JS Yet: Get Started - 2nd Edition
# 附錄 B：練習，練習，再練習！

在本附錄中，我們將探索一些練習題及其建議的解答。這些只是為了讓你*開始*練習本書中的概念。

## 練習比較

讓我們練習使用值型別和比較（第 4 章，支柱 3），其中需要涉及強制轉型。

`scheduleMeeting(..)` 應該接受一個開始時間（以 24 小時制格式作為字串 "hh:mm"）和一個會議持續時間（分鐘數）。如果會議完全落在工作日內（根據 `dayStart` 和 `dayEnd` 指定的時間），它應該回傳 `true`；如果會議違反了工作日的界限，則回傳 `false`。

```js
const dayStart = "07:30";
const dayEnd = "17:45";

function scheduleMeeting(startTime,durationMinutes) {
    // ..TODO..
}

scheduleMeeting("7:00",15);     // false
scheduleMeeting("07:15",30);    // false
scheduleMeeting("7:30",30);     // true
scheduleMeeting("11:30",60);    // true
scheduleMeeting("17:00",45);    // true
scheduleMeeting("17:30",30);    // false
scheduleMeeting("18:00",15);    // false
```

請先嘗試自己解決這個問題。考慮等式和關係比較運算子的用法，以及強制轉型如何影響這段程式碼。一旦你有了可以運作的程式碼，將你的解答與本附錄末尾「建議解答」中的程式碼進行*比較*。

## 練習閉包

現在讓我們練習閉包（第 4 章，支柱 1）。

`range(..)` 函式接受一個數字作為第一個引數，代表所需數字範圍的第一個數字。第二個引數也是一個數字，代表所需範圍的結尾（包含）。如果省略第二個引數，則應該回傳另一個期望該引數的函式。

```js
function range(start,end) {
    // ..TODO..
}

range(3,3);    // [3]
range(3,8);    // [3,4,5,6,7,8]
range(3,0);    // []

var start3 = range(3);
var start4 = range(4);

start3(3);     // [3]
start3(8);     // [3,4,5,6,7,8]
start3(0);     // []

start4(6);     // [4,5,6]
```

請先嘗試自己解決這個問題。

一旦你有了可以運作的程式碼，將你的解答與本附錄末尾「建議解答」中的程式碼進行*比較*。

## 練習原型

最後，讓我們來練習 `this` 和透過原型連結的物件（第 4 章，支柱 2）。

定義一台有三個轉軸的吃角子老虎機，每個轉軸可以個別地 `spin()`（旋轉），然後 `display()`（顯示）所有轉軸的當前內容。

單個轉軸的基本行為定義在下面的 `reel` 物件中。但吃角子老虎機需要個別的轉軸——委派到 `reel` 的物件，且每個都有自己的 `position` 屬性。

一個轉軸只*知道如何* `display()` 它當前的槽位符號，但吃角子老虎機通常每個轉軸顯示三個符號：當前槽位（`position`）、上方一個槽位（`position - 1`）和下方一個槽位（`position + 1`）。所以顯示吃角子老虎機最終應該顯示一個 3 x 3 的槽位符號網格。

```js
function randMax(max) {
    return Math.trunc(1E9 * Math.random()) % max;
}

var reel = {
    symbols: [
        "♠", "♥", "♦", "♣", "☺", "★", "☾", "☀"
    ],
    spin() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        this.position = (
            this.position + 100 + randMax(100)
        ) % this.symbols.length;
    },
    display() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        return this.symbols[this.position];
    }
};

var slotMachine = {
    reels: [
        // this slot machine needs 3 separate reels
        // hint: Object.create(..)
    ],
    spin() {
        this.reels.forEach(function spinReel(reel){
            reel.spin();
        });
    },
    display() {
        // TODO
    }
};

slotMachine.spin();
slotMachine.display();
// ☾ | ☀ | ★
// ☀ | ♠ | ☾
// ♠ | ♥ | ☀

slotMachine.spin();
slotMachine.display();
// ♦ | ♠ | ♣
// ♣ | ♥ | ☺
// ☺ | ♦ | ★
```

請先嘗試自己解決這個問題。

提示：

* 使用 `%` 模數運算子，在循環存取轉軸上的符號時包裝 `position`。

* 使用 `Object.create(..)` 來建立物件並將其原型連結到另一個物件。一旦連結，委派允許物件在方法呼叫期間共享 `this` 上下文。

* 不要直接修改 reel 物件來顯示三個位置中的每一個，你可以使用另一個臨時物件（再次使用 `Object.create(..)`），該物件有自己的 `position`，從而進行委派。

一旦你有了可以運作的程式碼，將你的解答與本附錄末尾「建議解答」中的程式碼進行*比較*。

## 建議解答

請記住，這些建議解答僅僅是：建議。解決這些練習題有許多不同的方式。將你的方法與你在這裡看到的進行比較，並考慮每種方法的優缺點。

「比較」（支柱 3）練習的建議解答：

```js
const dayStart = "07:30";
const dayEnd = "17:45";

function scheduleMeeting(startTime,durationMinutes) {
    var [ , meetingStartHour, meetingStartMinutes ] =
        startTime.match(/^(\d{1,2}):(\d{2})$/) || [];

    durationMinutes = Number(durationMinutes);

    if (
        typeof meetingStartHour == "string" &&
        typeof meetingStartMinutes == "string"
    ) {
        let durationHours =
            Math.floor(durationMinutes / 60);
        durationMinutes =
            durationMinutes - (durationHours * 60);
        let meetingEndHour =
            Number(meetingStartHour) + durationHours;
        let meetingEndMinutes =
            Number(meetingStartMinutes) +
            durationMinutes;

        if (meetingEndMinutes >= 60) {
            meetingEndHour = meetingEndHour + 1;
            meetingEndMinutes =
                meetingEndMinutes - 60;
        }

        // re-compose fully-qualified time strings
        // (to make comparison easier)
        let meetingStart = `${
            meetingStartHour.padStart(2,"0")
        }:${
            meetingStartMinutes.padStart(2,"0")
        }`;
        let meetingEnd = `${
            String(meetingEndHour).padStart(2,"0")
        }:${
            String(meetingEndMinutes).padStart(2,"0")
        }`;

        // NOTE: since expressions are all strings,
        // comparisons here are alphabetic, but it's
        // safe here since they're fully qualified
        // time strings (ie, "07:15" < "07:30")
        return (
            meetingStart >= dayStart &&
            meetingEnd <= dayEnd
        );
    }

    return false;
}

scheduleMeeting("7:00",15);     // false
scheduleMeeting("07:15",30);    // false
scheduleMeeting("7:30",30);     // true
scheduleMeeting("11:30",60);    // true
scheduleMeeting("17:00",45);    // true
scheduleMeeting("17:30",30);    // false
scheduleMeeting("18:00",15);    // false
```

----

「閉包」（支柱 1）練習的建議解答：

```js
function range(start,end) {
    start = Number(start) || 0;

    if (end === undefined) {
        return function getEnd(end) {
            return getRange(start,end);
        };
    }
    else {
        end = Number(end) || 0;
        return getRange(start,end);
    }


    // **********************

    function getRange(start,end) {
        var ret = [];
        for (let i = start; i <= end; i++) {
            ret.push(i);
        }
        return ret;
    }
}

range(3,3);    // [3]
range(3,8);    // [3,4,5,6,7,8]
range(3,0);    // []

var start3 = range(3);
var start4 = range(4);

start3(3);     // [3]
start3(8);     // [3,4,5,6,7,8]
start3(0);     // []

start4(6);     // [4,5,6]
```

----

「原型」（支柱 2）練習的建議解答：

```js
function randMax(max) {
    return Math.trunc(1E9 * Math.random()) % max;
}

var reel = {
    symbols: [
        "♠", "♥", "♦", "♣", "☺", "★", "☾", "☀"
    ],
    spin() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        this.position = (
            this.position + 100 + randMax(100)
        ) % this.symbols.length;
    },
    display() {
        if (this.position == null) {
            this.position = randMax(
                this.symbols.length - 1
            );
        }
        return this.symbols[this.position];
    }
};

var slotMachine = {
    reels: [
        Object.create(reel),
        Object.create(reel),
        Object.create(reel)
    ],
    spin() {
        this.reels.forEach(function spinReel(reel){
            reel.spin();
        });
    },
    display() {
        var lines = [];

        // display all 3 lines on the slot machine
        for (
            let linePos = -1; linePos <= 1; linePos++
        ) {
            let line = this.reels.map(
                function getSlot(reel){
                    var slot = Object.create(reel);
                    slot.position = (
                        reel.symbols.length +
                        reel.position +
                        linePos
                    ) % reel.symbols.length;
                    return slot.display();
                }
            );
            lines.push(line.join(" | "));
        }

        return lines.join("\n");
    }
};

slotMachine.spin();
slotMachine.display();
// ☾ | ☀ | ★
// ☀ | ♠ | ☾
// ♠ | ♥ | ☀

slotMachine.spin();
slotMachine.display();
// ♦ | ♠ | ♣
// ♣ | ♥ | ☺
// ☺ | ♦ | ★
```

本書到此結束。但現在是時候尋找真正的專案來練習這些概念了。持續寫程式吧，因為這是最好的學習方式！
