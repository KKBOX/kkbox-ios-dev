基本原則
--------

先整理一下我們已經學到的事情：

-   如果是 `init` 、 `new` 、 `copy` 這些 method
    產生出來的物件，用完就該呼叫 `release` 。
-   如果是其他一般 method 產生出來的物件，就會回傳 auto-release
    物件、或是 `Singleton` 物件（稍晚會解釋什麼是
    Singleton），就不需要另外呼叫 `release` 。

而呼叫 retain 與 release 的時機包括：

-   如果是在一般程式碼中用了某個物件，用完就要 release 或是
    auto-release。
-   如果是要將某個 Objective-C
    物件，變成是另外一個物件的成員變數，就要將物件 retain 起來。但是
    delegate 物件不該 retain，我們稍晚會討論什麼是 delegate。
-   在一個物件被釋放的時候，要同時釋放自己的成員變數，也就是要在實作
    dealloc 的時候，釋放自己的成員變數。

要將某個物件設為另外一個物件的成員變數，需要寫一組
getter/setter。我們接下來要討論怎麼寫 getter/setter。
