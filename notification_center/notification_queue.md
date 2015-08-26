Notification Queue
------------------

有的時候，我們的程式可能會在很短的時間送出大量的 notification，而造成
資源的浪費或效能問題。

以 KKBOX 來說，我們在歌單中的歌曲物件發生改動的時候，會透過
notification 更新 UI：一首歌曲可能會出現在多個歌單中，而我們可能用了很
多不同的 UI 物件來呈現不同張歌單，因此，像一首歌曲的播放次數改變、如這
首歌被多播放了一次，歌曲物件就透過 notification center，告訴每個跟歌單
UI 相關的物件重新讀取歌單資料。

照理說，只要有一首歌曲改變，就該發出這種通知，但假如我們現在做的事情是
歌單同步—把另外一台裝置上的歌單資料，同步到我們這台裝置上，那麼改動的
就不只是一首歌曲，而是一大批的歌曲，如果有十首歌，就送出了十次通知；但
是，其實 UI 只需要改動一次就好了，沒有重複更新十次 UI 的必要。

這時候我們就該用 NSNotificationQueue。我們可以把 NSNotificationQueue
想成 notification 的發送端與 notification center 之間的一個 buffer，這
個 buffer 可以讓我們暫緩送出 notification，而在一段緩衝期之內，決定我
們是否要合併通知。以前面的例子來看，我們就可以先把原本預計的十次通知先
放進 NSNotificationQueue 當中，然後讓 NSNotificationQueue 幫我們把十次
通知合併成只有一次通知。

我們要先建立一個 NSNotificationQueue 物件：

``` objc
notificationQueue = [[NSNotificationQueue alloc]
initWithNotificationCenter:[NSNotificationCenter defaultCenter]];
```

再來我們發送通知的程式原本像這樣：

``` objc
NSNotification *n = [NSNotification
    notificationWithName:@"KKSongInfoDidChangeNotification"
    object:self];
[[NSNotificationCenter defaultCenter] postNotification:n];
```

改寫成這樣：

``` objc
NSNotification *n = [NSNotification
    notificationWithName:@"KKSongInfoDidChangeNotification"
    object:self];
[notificationQueue enqueueNotification:n
	postingStyle:NSPostASAP
	coalesceMask:NSNotificationCoalescingOnName | NSNotificationCoalescingOnSender
	forModes:nil];
```

我們在這邊傳入了 `NSNotificationCoalescingOnName` 與
`NSNotificationCoalescingOnSender`，代表的就是請 notification queue 合
併名稱相同、發送者也相同的通知。
