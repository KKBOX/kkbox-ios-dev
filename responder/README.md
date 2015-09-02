Responder
=========

當我們在 iPhone、iPad 等 iOS 裝置上，用手指按到一個按鈕上的時候，事實
上，我們並不是真的按在一個按鈕上，而是按在螢幕上。是觸控螢幕的硬體接收
了我們的輸入之後，再將我們的觸控輸入送到軟體中，最後營造了「我們的手指
按到了按鈕上」的錯覺。

不同於其他的開發平台，在 iOS 與 Mac OS X 上，事件（Event）只用來表達來
自硬體的各種輸入行為。在 iOS 上的 UIEvent 包含了觸控輸入、藍芽耳機遙控
換歌等，Mac OS X 上的 NSEvent 則包括了鍵盤、滑鼠事件。

事件的傳遞
----------

在 iOS 裝置上，當硬體發生觸控事件，到我們的按鈕發生反應之間，事實上經
歷了：

- 硬體把事件傳到我們的 App 中，交由 UIApplication 物件分派事件
- UIApplication 把事件傳送到 Key Window 中，接著由 Key Window 負責分派
  事件
- Key Window 開始尋找在 View Heirarchy 中最上層的 view controller 與
  view，然後，發現最上層的 view 是我們的按鈕
- 觸發按鈕的 target/action

![事件的傳遞](responder.png)

事件從 application 傳遞到 window，從 window 傳遞到對應的 view 之上的流
程，如果我們反過來看，就會變成「誰最後應該負責處理事件」—如果有個 view
該處理，就會是 view 處理，不然就會 fallback 到 window，window 不處理又
會 fallback 到 application上，這個負責處理事件的物件，叫做 first
responder，而這種一環又一環尋找誰該處理事件的鎖鏈，叫做 Responder
Chain。每個可以處理事件的物件，都要實作 NSResponder 或 UIResponder
protocol。

而這個流程，會在 runloop 當中不斷循環。

Run loop
--------

我們來問一個很簡單的問題：我們所寫過的程式，大多都是從頭到尾、一行一行
往下執行，執行完畢，程式就結束；那麼，一個 GUI 應用程式—無論是我們現在
正在學習的 iOS 與 Mac OS X、還是其他平台—為什麼不是打開之後一路執行到
底結束，而是會停留在螢幕中等待我們操作？

原因很簡單，因為一個 GUI 應用程式開始執行之後，就會不斷執行一個迴圈，
直到用戶決定要關閉這個應用程式的時候，才會關閉這個迴圈。這樣的迴圈在
Windows 平台上叫做 message loop，在 iOS 與 Mac OS X 上叫做 run loop，
而這個迴圈當中所做的，就是收取與分派事件。

每一輪 run loop 的時間並不固定，會與這一輪 run loop 裡頭做了多少事情相
關，比方說，如果我們的畫面複雜，在 App 中同時有很多 view，那麼這一輪
runloop 就得要花上比較多的時間尋找 first responder；而像我們在 UI 上放
了一個按鈕，然後按鈕按下去要做一些事情，全都會算入到這一輪 run loop 的
時間。如果我們的程式做了一件很花時間的事情，讓這一輪 runloop 執行非常
久，就會導致「應用程式介面沒有回應」這種狀態，當介面卡住一段時間，應用
程式就會被系統強制關閉。

Timer 也是倚靠 run loop 運作的。當我們建立了一個 NSTimer 物件之後，下
一步就是要把 timer 物件註冊到 run loop 當中，如果只建立了 NSTimer 物件，
像是只呼叫了 `alloc`、`init`，這個 timer 並不會有作用，而呼叫
`scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:` 會
在建立 NSTimer 物件之外，同時將 timer 加入到 run loop 中。

Timer 運作的原理是，在每一輪 run loop 裡頭，會檢查是否已經到了某個
timer 所指定的時間，如果到了，就執行 timer 所指定的 selector。所以我們
可以知道幾件事：

1. 由於每一輪 runloop 的時間不一定，所以我們其實也不能夠期待 timer 會
   在非常精確的時間執行。前一輪 runloop 如果做了很花時間的事情，就會影
   響到原本應該要執行的 timer 實際執行的時間。
2. 雖然並沒有所謂的最小的時間單位這件事情，但是 timer 的時間間隔一定會
   有一個上限，我們不可能建立比 run loop 的頻率還要更頻繁的 timer。



Application
-----------


Window
------


相關閱讀
--------

- [About Events in iOS](https://developer.apple.com/library/ios/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40009541-CH1-SW1)
- [UIResponder Class Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIResponder_Class/)
- [UIEvent Class Reference](https://developer.apple.com/library/ios/documentation/UIKit/Reference/UIEvent_Class/)
- [NSResponder Class Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSResponder_Class/)
- [NSEvent Class Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ApplicationKit/Classes/NSEvent_Class/)
- [NSRunLoop Class Reference](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSRunLoop_Class/)
- [Technical Q&A QA1693 Synchronous Networking On The Main Thread](https://developer.apple.com/library/ios/qa/qa1693/_index.html)
