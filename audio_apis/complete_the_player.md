打造 Player 的完整功能
----------------------

在開發音樂 App 的時候，如何使用底層 Audio API，以及管好 Audio Session
會是最困難的部份，接下來還要做一些事情補完整個 App 的功能，但相對來說難
度容易許多。

### 音量控制

在一套音樂播放軟體中，我們往往需要能夠調整系統音量的 UI 介面。其實我們
可以對 Audio Queue 或是 AUGraph API 設置音量，像 Audio Queue 可以用
AudioQueueSetParameter 設定 kAudioQueueParam\_Volume，AUGraph 則可以對
mixer 設定 kMultiChannelMixerParam\_Volume；不過目前可以看到所有的音樂
App，只要出現調整音量的 UI，都不是調整這些內部元件的音量，而是整體的系
統音量，大概是因為從第一支 iPhone 問世起，內建的音樂 App 就有調整系統
音量的介面建立的設計慣例。

我們可以使用 Media Player Framework 中找到 MPVolumeView，這個 view 裡
頭除了音量控制條之外，旁邊還會顯示跟 AirPlay 相關的控制按鈕。使用
MPVolumeView 會是提供音量調整介面最簡單的方法。

如果我們不想用 MPVolumeView，而是想設計自己的音量控制 UI，我們可以從
MPMusicPlayerController 裡頭找到 volume 這個屬性。
MPMusicPlayerController 是一個用來在我們的 App 中，播放內建音樂 App 裡
頭的歌曲的 class，不少運動 App 使用了這個 class，讓用戶可以在選擇記錄
運動的時候，一邊選擇自己的運動歌曲，至於調整音量這個功能會放在這個
class 裡頭，說起來也頂奇怪的，想起來裝置的音量好像放在 UIDevice 還比較
有道理。

MPMusicPlayerController 的 volume 選項在 iOS 7 的時候也被標成
deprecated，也沒有對應的其他 API，看來蘋果不太喜歡我們做自己的音量條，
只希望我們使用 MPVolumeView。

另外，當系統音量改變的時候，我們會收到
MPMusicPlayerControllerVolumeDidChangeNotification 通知。

### 遙控器

我們之前在 [Responder](../responder/application.md) 這一章裡頭已經講過
如何處理遙控器事件—在 iOS 7.1 之後，我們會偏好使用
MPRemoteCommandCenter 這組 API，然後對 MPRemoteCommandCenter 提供的許
多 command，如 playCommand、stopCommand 等，設置 target/action。

不過，如果你所開發的音樂 App 除了播放音樂之外，有一天有人就突發奇想想
在裡頭加入用 AVPlayer 播放 MV 與 Live 直播功能，還可以用 web view 看網
路文章，文章裡頭還嵌入了 YouTube 影片，你除了會遇到 Audio Session 的問
題：自己 App 內部的 audio player 與 video player 互相 interrupt，還會
遇到你幫 audio player 設好的 MPRemoteCommandCenter 設定，被這些 video
player 搶走了。

因此，我們需要在 AVPlayer 與 web view 裡頭的影片結束的時候，重設
MPRemoteCommandCenter 的設定。要知道 AVPlayer 與 MPMoviePlayer 結束的
事件還算簡單，只要接收 AVPlayerItemDidPlayToEndTimeNotification 與
MPMoviePlayerPlaybackDidFinishNotification 等通知即可，但是 web view
裡頭的 YouTube 影片關閉卻沒有什麼明確的通知。

目前想到的解法是這樣：我們聽取某個 UIWindow 關閉的通知
UIWindowDidBecomeHiddenNotification，然後去檢查這個 window 的 subview
中是否包含 AVPlayerView，如果有的話，那大概就是播放 YouTube 影片的
video player 的 window 吧…。

### MPNowPlayingInfoCenter

MPNowPlayingInfoCenter 是讓我們設置「現正播放」資訊的 class，我們只要
對 MPNowPlayingInfoCenter 的 singleton 物件，設定 nowPlayingInfo 屬性，
我們就可以將目前正在播放的歌曲名稱、封面圖等，顯示在 iOS 裝置的 lock
screen 上，在 AirPlay 的時候也會投放到 AppleTV 上，當我們用 Lighting
等介面將 iOS 裝置連接到一些車用音響系統的時候，也可以在車用音響的面板
上顯示歌曲資訊。

nowPlayingInfo 是一個 NSDictionary，裡頭用到的 key 多半定義在
MPMediaItem 裡頭，MPNowPlayingInfoCenter 這個 class 裡頭多定義了幾個額
外的 key。使用 MPNowPlayingInfoCenter 的時候要注意幾點：

1. 雖然你看 nowPlayingInfo 的 property 定義成 copy，但當你設定了
   nowPlayingInfo 之後，其實 MPNowPlayingInfoCenter 會開另外一條背景
   thread，在背景 enumerate 這個 dictionary。所以，如果你建立了一個
   mutable array，把這個 mutable array 設定成 nowPlayingInfo，然後繼續
   改動這個 mutable array，你會遇到一邊 enumerate 一邊改動而造成的
   crash。
2. 在設定 MPMediaItemPropertyPersistentID 的時候，這個 ID 應該要是
   NSNumber，但如果你不小心把這個 ID 設成了字串，絕大多數狀況下沒事，
   但是當你接上一些車用音響的時候，卻可能把這些車用音響系統搞當。這是
   我們拿著筆記型電腦在地下停車場修了兩個小時 Bug 後得到的心得。
