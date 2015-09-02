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
