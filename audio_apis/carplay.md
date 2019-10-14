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

### 再談 MPRemoteCommandCenter

一般來說，我們至少會實作以下的 MPRemoteCommandCenter 指令：

* playCommand：開始播放。
* pauseCommand：暫停播放。
* stopCommand：完全停止播放。Pause 與 Stop 的差別在於，Pause 只是停止目前正在播
  放用的 Audio Graph/AVAudioEngine/Audio Queue，但是 Stop 會完全放開目前播放器元
  件參考到的歌單/歌曲物件。
* togglePlayPauseCommand：檢查目前是否正在播放，播放中就執行 pause，反之則執行 play。

然後以下幾個指令需要特別注意：

* changeRepeatModeCommand：修改循環模式，包括循環播放、不循環播放、單手循環播放
* changeShuffleModeCommand：修改播放模式，包括循序播放、隨機播放…等

在 MPRemoteCommandCenter 當中絕大多數的指令，都是用戶真的做了手動操作、在各種地
方按下按鈕之後才觸發，但是 changeShuffleModeCommand 不一樣，如果我們實作了
changeShuffleModeCommand，在用戶接上了一般的車用音響之後，就會被直接呼叫一次。

iPhone 除了支援 CarPlay 車用音響之外，也支援更早之前的車用音響。蘋果在 2001 年就
推出了最早的 iPod，在 iPhone 推出之前，就已經有一套讓車用音響支援 iPod 的協定，
所以，在這樣的車用音響上，iPhone 會被當成是一支 iPod，也就是說，
MPRemoteCommandCenter的指令，其實不只會用在 CarPlay 車機上，也會用在非 CarPlay車
機上。所以，當我們在實作 changeShuffleModeCommand 與 changeShuffleModeCommand 的
時候，必須參考從外部傳進來的新狀態。像是：

先指定 changeRepeatModeCommand 與 changeShuffleModeCommand 的 target/action：

``` swift
center.changeRepeatModeCommand.addTarget(self, action: #selector(changeRepeatMode(_:)))
center.changeShuffleModeCommand.addTarget(self, action: #selector(changeShuffleMode(_:)))
```

實作方式：

``` swift
@objc func changeRepeatMode(_ event: MPChangeRepeatModeCommandEvent) -> MPRemoteCommandHandlerStatus {
    let type = event.repeatType
    /// 使用傳入的 repeatType
    return .success
}

@objc func changeShuffleMode(_ event: MPChangeShuffleModeCommandEvent) -> MPRemoteCommandHandlerStatus {
    var type = event.shuffleType
    /// 使用傳入的 shuffleType
    return .success
	}

```


