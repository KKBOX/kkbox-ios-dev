iOS 與 Mac OS X 的 Audio API 概觀
---------------------------------

我們知道串流音樂的播放流程之後，現在就來看一下 iOS 與 Mac OS X 上有哪
些適合的 Audio API。我們大概分成兩群

不適合用在音樂播放的：

- System Sound
- OpenAL

可以用用在音樂播放的：

- NSSound
- QuickTime
- AVFoundation
- Audio Queue
- Audio Unit Graph

### System Sound Services

在 iOS 上，如果我們想要發出簡短的系統提示音效，像是按了某個按鈕要發出
的聲響，會選擇使用 System Sound Services。

System Sound Services 提供了一些 C API 可以讓我們播放小於三十秒的 CAF
（Core Audio Format）檔案。只要透過 `AudioServicesCreateSystemSoundID`
載入指定的檔案 URL、建立 SystemSoundID 後，就可以用

``` objc
static SystemSoundID soundFileObject;
if (!soundFileObject) {
		CFBundleRef mainBundle;
		mainBundle = CFBundleGetMainBundle();
		CFURLRef soundFileURLRef;
		soundFileURLRef = CFBundleCopyResourceURL(mainBundle, CFSTR("sound"), CFSTR("aiff"), NULL);
		AudioServicesCreateSystemSoundID(soundFileURLRef, &soundFileObject);
		CFRelease(soundFileURLRef);
}
AudioServicesPlaySystemSound(soundFileObject);
```

此外，當我們想要讓 iPhone 出現像來電時的震動效果，也是呼叫 System
Sound Services，因為震動效果其實被定義成一種系統音效。我們想讓手機震動，
就呼叫：

``` objc
AudioServicesPlaySystemSound (kSystemSoundID_Vibrate)
```

只能播放 30 秒的未壓縮格式，顯然沒有辦法滿足我們想要播放 MP3、AAC 這些
格式的需求。另外 System Sound Services 發出的聲音有個特點：只會從 iOS
裝置上的本機播放出來，就算用鏡像的 AirPlay 模式連到了 AppleTV 上，還是
只會從本機播放。

### OpenAL

OpenAL 是一套開放規格，用途是產生在遊戲當中提供立體的音效。當我們在玩
一款第一人稱射擊遊戲的時候，如果感受到敵人的腳步聲由遠而近，或是可以用
槍聲的來源感受到敵人的位置，這種效果便可以透過 OpenAL 達成。

最近似乎大家都很少直接呼叫 OpenAL API，許多的遊戲引擎都在 OpenAL 的基
礎上又另外架設了一套音效引擎，就連蘋果自己的 SpriteKit 裡頭都有播放音
效用的 API。

OpenAL 一樣是 C API，一樣只能夠播放未壓縮的音檔，播放流程是先將整個音
檔的資料載入到 buffer 中，然後將 buffer 丟入 source 播放，然後可以對
source 做像是位置等設定。

如果要拿 OpenAL 播放 MP3…也不是不行，我們可以先把 MP3 檔案載入到記憶體
中，用 converter 將整個 MP3 檔案轉成 LPCM 格式，如果是三分鐘的歌，就把
30MB 的資料載入到記憶體中，聽起來就不太可行。

播放串流音樂的時候，我們只會把少量的資料轉換成 LPCM 格式播放，等前一包
資料播放完畢後，再繼續提供下一包的資料。那我們有沒有辦法用這個原則先把
整個檔案切成幾個小包，然後交給 OpenAL 播放呢？實際上不太可行，因為
OpenAL 裡頭缺乏音效播放完畢的 callback 機制，不會通知我們哪個音效播放
完畢，當然也可以用 timer 算好時間播放某個 buffer，但我們前面也提到了
timer 的原理，知道 timer 其實並不精確。

### NSSound、QuickTime、AV Foundation

NSSound、QuickTime 與 AV Foundation 是 Mac OS X 與 iOS 上的高階 API，
只要提供檔案的遠端或本機 URL，便可以直接開始播放，

NSSound 主要是用在 Mac 上用來播放簡短的提示音效用的 API—當然，當我們只
想要在 Mac 上發出一個用來提示錯誤用的聲響，也可以簡單呼叫 `NSBeep()`就
可以了—雖然定位很接近 System Sound Services，但 NSSound 其實可以載入多
種格式、而且檔案長度較長的音檔，我們還可以知道播放的檔案長度，要求指定
的播放時間位置（random seek），還可以調整音量、要求一直 loop 重播等。

雖然 NSSound 可以說是一個完整的 Audio Player，不過定位上還是偏向用來播
放系統提示音效在蘋果為了 API 一致、將 AV Foundation 從 iOS port 回 Mac
之前，在 Mac 上播放各種媒體檔案，會更常使用 QuickTime API 裡頭的
QTMovie 這個 Class。QTMovie 可以做到 Mac OS X 系統中 QuickTime Player
可以做到的事情，除了播放 audio 之外，還包括播放甚至編輯 QuickTime 影片
的功能。不過，蘋果在 Mac OS X 10.9 中就將 QTMovie 標成 deprecated。

在 Mac 與 iOS 上的 AV Foundation 不完全相同。在 AV Foundation 裡頭有三
個直接與 Audio 播放相關的 class，按照出現的時間排列，分別為
AVAudioPlayer、AVPlayer 與 AVAudioEngine。

AVAudioPlayer 是在 iPhoneOS 2.2 上推出的，大概是在 iPhoneOS SDK 問世後
半年左右出現的 API。用 AVAudioPlayer 相當適合用在像是播放遊戲背景音樂
等應用上，這個 class 明顯的優點是可以播放多種格式的檔案，但也有兩個明
顯的缺點。

其一是 AVAudioPlayer 只能夠播放位在本機的檔案，而無法指定放在網路上的
URL 播放，所以，直到 iOS 4 AVPlayer 推出之前，在播放網路上的音檔時，如
果不想要用底層的 C API，要不就是先把整個檔案抓下來之後用 AVPlayer 播放，
要不就是用 UIWebView 開啟，但整個畫面都會變成 web view 的播放畫面。
KKBOX 的 iOS 版本是在 2009 年一月，大約 iPhoneOS 2.1 版左右的時候推出，
於是只能選擇更底層的 API。

AVAudioPlayer 的另外一個缺點則是，蘋果在 iOS 4 開始支援背景執行，其中
支援背景 audio 播放，但 AVAudioPlayer 無法在背景播放 audio。


### Audio Queue

### Audio Unit Graph
