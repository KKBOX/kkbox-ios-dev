Mac 上的其他 Notification Center
--------------------------------

在 iOS 上面我們通常只會用到 NSNotificationCenter，特別是
NSNotificationCenter 的 defaultCenter：不過，在 Mac OS X 上，我們還有
其他的 notification center 可以使用。

### NSDistributedNotificationCenter

蘋果在 iOS 上的限制較為嚴格，一直以來都想辦法禁止跨 App 之間的通訊
（IPC，Inter-Process Communication）。不過自從 Mac OS X 出現以來，
Cocoa Framework 就有 Distributed Objects 這套 IPC 機制，讓不同 App 之
間可以傳遞 Objective-C 物件，後來更推出了 XPC，可以在不同 App 之間傳遞
block。

NSDistributedNotificationCenter 就是在 Distributed Objects 技術上建立
的 notification center，也就是，如果你對
NSDistributedNotificationCenter 發送了通知，便可以讓其他的 App 收到來
自你目前所在 App 送出的通知。

### NSWorkSpace 的 Notification Center

NSWorkSpace 這個物件在 Mac 上代表的是 Mac 的桌面環境。如果你想要要求
Mac OS X 開啟另外一個 App，處理某個檔案或 URL（在 iOS 上我們會要求
UIApplication 來 openURL:，但是在 Mac 上則是交由 NSWorkSpace 處理），
或是取得某個檔案在 Finder 裡頭的代表圖示…等，就會用到 NSWorkSpace。

跟 NSWorkSpace 相關的通知，像是某個 App 是否被成功開啟、你的 Mac 電腦
是否離開了休眠…等等，都不會透過 NSNotificationCenter 的defaultCenter，
而是要透過 `[[NSWorkSpace sharedWorkspace] notificationCenter]` 這邊的
notification center，我們要選擇正確的 notification center 做 add
observer，才能正確收到通知。
