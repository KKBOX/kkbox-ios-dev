Notification Center
===================

Notification Center 是在 Cocoa/Cocoa Touch Framework 中，物件之間可以
不必互相知道彼此的存在，也可以互相傳遞訊息、交換資料/狀態的機制。

我們可以把 Notification Center 想像是一種廣播系統。當一個物件 A 的狀態
發生改變，而有多個物件需要知道這個物件發生改變的狀況下，物件 A 不必直接
對這些物件發出呼叫，而是告訴一個廣播中心說：「我的狀態改變了」，至於其
他需要聽取狀態的物件呢，也只要對這個廣播中心訂閱（subscribe）指定的通知，
所以當物件 A 發出通知的時候，這個廣播中心就會通知有訂閱通知的其他物件。
這個廣播中心，就是 Notification Center。

我們經常使用 Notification Center 處理來自作業系統的事件。假如我們現在
寫了一個日記軟體，這個軟體裡頭已經有很多 view，每個 view 裡頭都有一篇
日記，每篇日記上都有該篇日記的撰寫日期與時間。我們通常會使用
NSDateFormatter，使用系統偏好設定中的語系（Locale）設定，將日期轉成符
合語系設定的字串顯示，那麼，當用戶調整了系統偏好設定，像是把中文改成英
文，那麼，我們原本用中文顯示的日期，也應該馬上變成用英文顯示─我們該怎
麼做呢？

我們最常使用的通知中心是 NSNotificationCenter 這個 class，我們也通常使
用這個 class 的 singleton 物件 default center（也就是說，其實
Notification Center 有好幾個，不過我們最常使用的還是這個）。當系統語系
改變的時候，Notification Center 就會發出叫做
NSCurrentLocaleDidChangeNotification 的這項通知。所以，我們所有要顯示
日期的畫面物件，都應該要訂閱這個通知，在收到通知的時候，就要重新產生日
期字串。

