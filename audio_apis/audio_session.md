Audio Session
-------------

Audio Session 是一個只在 iOS 上，Mac OS X 沒有的 API，用途是用來描述目
前的 App 打算如何使用 audio，以及我們的 App 與其他 App 之間在 audio 這
部份應該是怎樣的關係。在 iPhoneOS SDK 問世的時候，只有 C API，到了
iPhoneOS 3.0 之後開始有 Objective-C API，放在 AVFoundation framework
中。

### 決定 Audio Session 的 Category

要讓我們的 App 在 audio 的表現上正常，我們必須要先選擇正確的 Audio
Session category，然後將 Audio Session 設定成 active。

``` objc
NSError *audioSessionError = nil;
[[AVAudioSession sharedInstance] setCategory:AVAudioSessionCategoryPlayback error:&audioSessionError];
[[AVAudioSession sharedInstance] setActive:YES error:&audioSessionError];
```

Audio Session 的 category 包括：

- AVAudioSessionCategoryAmbient
- AVAudioSessionCategorySoloAmbient
- AVAudioSessionCategoryPlayback
- AVAudioSessionCategoryRecord
- AVAudioSessionCategoryPlayAndRecord
- AVAudioSessionCategoryAudioProcessing
- AVAudioSessionCategoryMultiRoute

前兩種屬於「就算把聲音整個關了，其實也不影響 App 的功能」的 App，比方
說各式各樣的手機遊戲：就算沒有聲音，我們還是有辦法玩神魔之塔、Angry
Bird 或是 Candy Crush，不像 KKBOX 是一種音樂播放軟體，如果聽不到聲音，
KKBOX 就等於沒有功能可言。

設了這種 category 之後，只要 App 退到背景、鎖上靜音鎖或是進入 lock
screen，App 就不會發出聲音。至於 AVAudioSessionCategoryAmbient 與
AVAudioSessionCategorySoloAmbient 的差別就是是否允許可以與其他 App 混
音，前者可以，後者不行，如果你的遊戲只有簡單的音效，像揮劍、開槍的聲音，
沒有其他背景音樂，就不妨設定成 AVAudioSessionCategoryAmbient，讓用戶可
以一邊玩遊戲，一邊聽 KKBOX。但如果你的遊戲有自己的背景音樂，不希望受到
干擾，那就設成 AVAudioSessionCategorySoloAmbient，只要開始遊戲背景音樂，
就會通知系統，要求像 KKBOX 這類背景播放音樂的 App 停止播放。

像 KKBOX 這類的媒體播放軟體，則應該將 Audio Session category 設定成
AVAudioSessionCategoryPlayback。這類的軟體可以在靜音鎖鎖上、進入 lock
screen 播放聲音，也可以在背景播放。這種軟體在有來電、鬧鐘響起的時候，
會收到 Interrupt 的通知，也必須處理這種通知。

如果你只要做一個錄音軟體，就設定成 AVAudioSessionCategoryRecord，至於
AVAudioSessionCategoryPlayAndRecord 則是用在像 Skype 這類的網路電話軟
體，設定之後，一樣是在靜音鎖鎖上與 lock screen 時可以使用錄製功能。

最後兩種很少見，在網路上可以看到的資源也很少。根據蘋果文件，
AVAudioSessionCategoryAudioProcessing 是用在純粹只使用蘋果的 Audio API
做音訊的處理，但是沒有任何的輸入與輸出，而
AVAudioSessionCategoryMultiRoute 的用途則是強制將 audio 從指定的硬體上
輸入與輸出。

### 決定 Audio Session Category 的時機

我們要設置正確的 Audio Sesion Category，才能夠正確使用 audio，不然在呼
叫後續的 audio API 時就會遇到錯誤。所以我們設定 Audio Session Category
的時機，必須要在呼叫所有後續的 audio API 之前。

像如果我們沒有把 Audio Session Category 設定成
AVAudioSessionCategoryRecord 或 AVAudioSessionCategoryPlayAndRecord，
當我們想要使用作為 Input 使用的 Audio Queue，或是使用 RemoteIO 的 bus
1 時，就會遇到錯誤。

而如果我們沒有把 Audio Session 設定成 AVAudioSessionCategoryPlayback，
就在背景呼叫與播放相關的 Audio Queue API，或是用 AUGraphOpen 建立
AUGraph，都會遇到錯誤。

在 iOS 8 之前，我們沒有什麼機會可以在背景開啟 App ，然後讓 App繼續待在
背景，從來不出現在前景。iOS 8 之前，App 開啟之後一定會先待在前景，用戶
也得在前景選擇一首歌曲播放，所以我們可以在讓用戶在前景開始播放後，再設
定 Audio Session Category。不過，iOS 8 之後有兩種方式會讓 App 在背景開
啟，一種是 iOS 8 的 Interactive Notification，用戶可以直接在
Notification 的畫面中讓 App 做某件事情，而不用將 App 叫到前景，另外一
種方式則是讓 Apple Watch 呼叫我們的 App。

為了確保 Audio Session Category 就在呼叫其他 audio API 之前被設定，我
們就得在很早的地方就先設好 Audio Session Category，像是 App 一啟動時，
或是在建立我們的 player class 的時候。

說到背景執行，其實 iOS 還有另外一項跟背景相關的限制，就是 iOS App 不可
以在背景使用 OpenGL API，只要一在背景呼叫了 OpenGL API，就會馬上因為
exception 而 crash。所以，如果你寫了一個一個可以在背景播放 audio 的音
樂 App，就不該在 UI 上使用 OpenGL。

KKBOX 在 2013 年的時候，暫時做了一個實驗性的功能，可以在 KKBOX 的 iOS
App 這個播放器裡頭玩一個像是 Tap-Tap Revenge 的音樂節奏遊戲，因為這個
遊戲是在以 OpenGL 為基礎的遊戲引擎上開發，所以，KKBOX 平時的 Audio
Session Category 是 AVAudioSessionCategoryPlayback，但是進入節奏遊戲模
式後，就必須切換到 AVAudioSessionCategorySoloAmbient，避免因為在背景呼
叫 OpenGL 而 crash。

### Audio Session Category Options

在設了 Audio Session Category 之後，還有許多細項可以設定，包括
Category Options 與 Mode。先講 Category Options 的部份，共有以下幾個
option

- AVAudioSessionCategoryOptionMixWithOthers
- AVAudioSessionCategoryOptionDuckOthers
- AVAudioSessionCategoryOptionAllowBluetooth
- AVAudioSessionCategoryOptionDefaultToSpeaker

在 KKBOX 的開發過程中，AVAudioSessionCategoryOptionDuckOthers 對我們來
說影響比較大；並不是 KKBOX 使用了這種 option，而是當其他 App 設定了這
種 Catetgory Option 之後，KKBOX 會受到影響。這個 Category Option 只要
一設定，就會讓系統中其他 App 發出的聲音的音量都減半，主要是一些車用導
航軟體會這樣設定 audio：當一套導航軟體會使用語音導引駕駛如何開車的時候，
就會要求其他 App 降低音量，讓駕駛可以清楚聽到導航語音。

我們在 KKBOX 曾經收到這樣的客訴：用戶只要開啟了 Papago 的導航，就沒有
辦法聽清楚 KKBOX 的音樂了。面對這樣的客訴，我們也只能夠請客戶去找
Papago，因為當 Papago 這樣設定 Audio Session Category Option 後，KKBOX
的音量一定會減半。



### Audio Session Mode

### Interrupt

### Audio Route
