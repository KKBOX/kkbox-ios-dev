CarPlay
-------

蘋果在 iOS 7.1 開始推出 CarPlay 功能。CarPlay 的使用體驗是由 CarPlay 專屬的車用
音響系統以及 iOS 兩者共同創造的，CarPlay 車用音響會有一個小型的觸控螢幕，當用戶
的 iPhone連接到音響系統後，就會把這個小螢幕變成另外一個 iPhone 的延伸螢幕，在這
個螢幕上，會用圖示、文字都比較大的 UI 設計，列出 iOS 的一些基本 App，包括電話、
簡訊、音樂、地圖等，由於介面中的元素較大，加上可以用 Siri 語音操作，就讓用戶在開
車的時候，比較方便操作這些功能，也更能夠顧及行車安全。

![carplay.jpg])(carplay.jpg)

蘋果開放軟硬體廠商開發 CarPlay 相關應用，如果你是汽車、或是汽車音響廠商，你可以
針對車輛的功能開發專屬的 App，像是可以用 CarPlay 操作開關車窗、調整空調等，至於
一般會在 App Store 上的 App，也只開放了導航與音樂服務兩種類型。如果你想要讓你的
App 出現在 CarPlay 的延伸螢幕中，在 code sign 的時候還需要加上額外的
entitlement，這個 entitlement 需要額外向蘋果申請，也大概會是一般開發者進入
CarPlay 開發的門檻。

蘋果不允許音樂類型的 App，像 KKBOX，在實作 CarPlay 功能的時候，客製自己的使用者
UI，而是只能夠用一種階層式的方式瀏覽 App 提供的內容，從中挑選想要播放的歌曲/歌
單。也就是說，蘋果設計好了 CarPlay 上的音訊類型 App 的 UI，第三方 App 只能夠提供
一種階層式的資料，讓 iOS 把我們提供的資料填入到 CarPlay UI 裡頭。蘋果提供了一個
叫做 MPPlayableContentManager 的 class，我們可以指定 MPPlayableContentManager 的
data source 與 delegate，透過 data source 與 delegate 提供資料。

### 實作 CarPlay 功能

要讓用戶可以完整使用 CarPlay，我們需要…

* 實作 MPRemoteCommandCenter
* 實作 MPPlayableContentManager 的 data source 與 delegate
* 在放時在 MPNowPlayingInfoCenter 填入歌曲資訊

這個部分其實蘋果並沒有說得很清楚，在指定 MPPlayableContentManager 的 data source
與 delegate 之前，必須先設定 MPRemoteCommandCenter 當中的指令，不然，即使設定了
MPPlayableContentManager 的 data source，MPPlayableContentManager 也不會開始向
data source 要求資料。倒是 MPNowPlayingInfoCenter 可以稍晚設定。

### MPRemoteCommandCenter
