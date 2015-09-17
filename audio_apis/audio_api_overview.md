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
- Audio Unit Processing Graph

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
裝置上的本機播放出來，就算用鏡像的 AirPlay 模式連到了 Apple TV 上，還
是只會從本機播放。

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
AVAudioPlayer、AVPlayer 與 AVAudioEngine。我們先來看 AVAudioPlayer 與
AVPlayer。

AVAudioPlayer 是在 iPhoneOS 2.2 上推出，大概是在 iPhoneOS SDK 問世後半
年左右出現的 API。用 AVAudioPlayer 相當適合用在像是播放遊戲背景音樂等
應用上，這個 class 明顯的優點是可以播放多種格式的檔案，但也有兩個明顯
的缺點。

其一是 AVAudioPlayer 只能夠播放位在本機的檔案，而無法指定放在網路上的
URL 播放，所以，直到 iOS 4 AVPlayer 推出之前，在播放網路上的音檔時，如
果不想要用底層的 C API，要不就是先把整個檔案抓下來之後用 AVPlayer 播放，
要不就是用 UIWebView 開啟，但整個畫面都會變成 web view 的播放畫面。

AVAudioPlayer 的另外一個缺點則是，蘋果在 iOS 4 開始支援背景執行，其中
一項背景模式是支援背景 audio 播放，但 AVAudioPlayer 並不支援，無法在背
景播放 audio。

iOS 4 時推出的 AVPlayer 是一個好用的元件，AVPlayer 支援多種現在流行的
檔案格式，不但可以用來播放音訊，也可以播放影片，只要指定要播放的
URL（包括本機與遠端），便可以開始播放，如果要播放的是影片，則可以搭配
AVPlayerLayer 顯示畫面，也支援背景播放。

在前一節提到播放網路串流音訊的流程，AVAudioPlayer 與 AVPlayer 都做完裡
頭的六個步驟；在絕大多數場合中，AVPlayer 可以滿足播放的需求，不過，如
果你有以下需求：

* 因為商業上的需求，我們要播放的檔案經過加密，在播放的過程中需要先解
  密才能播放。
* 我們想要提供更多的播放效果，像是增加迴音、升降 key、支援 EQ 等化器等…。

那我們就只能夠使用更底層的 Audio API 了。

### Audio Queue

Audio Queue 是 Mac OS X 與 iOS 上用來播放與錄製音訊的 C API。先看播放
的部份，對照前一節提到的播放步驟，Audio Queue 可以幫我們簡化最後兩步：
我們只要一開始指定好 Audio Queue 是哪一種是檔案格式，之後只要提供這種
格式的資料就可以播放，我們不用自己動手把原本的 MP3 或 AAC 格式轉換成
LPCM，但前面分析出 packet 這步還是得自己來。

Audio Queue 是
[Audio Toolbox Framework](https://developer.apple.com/library/mac/documentation/MusicAudio/Reference/CAAudioTooboxRef/)
的一部分。Audio Toolbox 是一個不算小的 Framework，裡頭包含我們前面提到
的 System Sound、後面要提到的 Audio Unit Processing Graph 外，也包含我
們在播放過程中會用到的 parser 與 converter，parser 包括 Audio File
Services 與 Audio File Stream Services 等，至於 Audio Converter
Services 便是 converter。

Audio Queue API 主要由兩個主要的資料類型組成：Audio Queue 與 Audio
Queue Buffer。我們不妨把 Audio Queue 想像成是一個水池，而每個 Audio
Queue Buffer 則是許許多多的水桶，在要求 Audio Queue 開始播放音訊之後，
Audio Queue 這個水池一開始是乾的，所以我們要提第一桶水桶，把水桶裡頭的
水倒進水池裡頭（這一步叫做 enqueue buffer）。接著，這個水池會因為水慢
慢地被用掉，於是慢慢變乾，但是在完全乾枯之前會通知我們（callback）水快
沒了，所以跟我們要下一桶水；接著就是這樣的步驟不斷循環。

至於錄音，則是將前述步驟整個倒轉過來，AUdio Queue 與 Audio Queue
Buffer 仍然扮演水池與水桶的角色，只是我們要先把一個空水桶放進乾掉的水
池裡頭，當錄音的資料進來的時候，水會裝進這個水池裡頭唯一的水桶裡，當一
個水桶快要裝滿時，Audio Queue 就會通知我們趕快把裝滿水的水桶拿出去存檔，
同時再拿一個空的水桶進來裝水。然後不斷循環這個步驟。

在 iOS 上使用 Audio Queue API 的時候，還要搭配正確的 Audio Session。
Audio Session 是一種用來描述我們的 App 打算怎麼使用 Audio 的 API，要正
確設定 Audio Session 的類型，並且讓 Audio Session 變成 active，系統才
會允許我們做一些我們想做的事情，像我們必須告訴系統我們是媒體播放的 App，
系統才會允許我們執行背景播放，要告訴系統我們是可以播放與錄音的軟體，才
有辦法使用麥克風錄音。

在使用 Audio Queue API 播放音訊的時候，我們要稍微注意一下對播放時間的
控制。如果我們想要知道一首歌現在播放到哪個時間，一般在 player 的 UI 上
我們大概只需要精確到秒就好了，而我們最小可以到達的單位則是 packet 的大
小，透過我們送出了多少 packet 而推算出播放時間。

我們在建立 Audio Queue Buffer 的時候，通常會建立比較大的 buffer，可能
會是半秒、一秒甚至更多秒數的 buffer，假如我們送出了一個一秒鐘的 buffer，
在播放這一秒的時候，其實我們不太能精確掌握「我們播放到了這一秒鐘的哪個
地方」。Audio Queue 雖然有兩個跟播放時間相關的 C function，這兩個
function 回傳的時間通常也不是很精確。

由於 Audio Queue 只要拿到 AAC、MP3 資料就可以播放，而如果我們想要做一
些音效的處理，會是在 LPCM 資料這一層做。KKBOX 最早使用 Audio Queue 開
發播放器，但由於會員一直要求我們能夠提供 EQ 等化器等功能，所以後來重新
開發了使用 Audio Unit Processing Graph API 為基礎的播放器。

### Audio Unit Processing Graph

如果我們想要對音訊播放擁有最完整的控制，那我們最後的選擇，就是最底層的
Audio Unit Processing Graph 這層 C API。上面提到的不少 API，像 OpenAL、
Audio Queue 以及 AVFoudation 等，也是在這一層 API 上架構的。

我們先來解釋一個名詞：Core Audio。Core Audio 是蘋果的整個 audio 的架構
底層，我們前面所講的 Audio Toolbox framework 的 API，是屬於 Core Audio
的應用層，Audio Unit Processing Graph 算是應用層的最底層，已經距離操控
硬體不遠了。

此外還有我們平常比較不會接觸到的硬體與 codec 這幾塊，在 iOS上我們完全
無法修改，但是在 Mac OS X 上，如果我們做了什麼新的 audio 硬體，或是要
讓 Mac 的 player 支援某些新的檔案格式，開發者還是可以在 Core Audio 架
構上寫新的 driver 或 codec，像我們想在 Mac 上播放微軟的 WMA，或是
RealPlayer 的格式的話，就得安裝額外的 codec。至於用來控制各種外部的電
子樂器，則會用到 Core MIDI framework，這也是 Core Audio 的一部分。

Audio Unit Processing Graph 是一個可以處理播放與錄音的 API，這層把
audio 播放的處理過程，抽象化變成一個個的組件，我們可以透過組合這些組件
創造我們想要的錄製與播放效果。Audio Unit Processing Graph API 裡頭大概
有三個主要的角色：

- AUNode 或 AudioComponent
- AudioUnit
- AUGraph

我們不妨想像我們現在身處在演唱會的舞台上，有錄製歌聲與樂器的麥克風，而
從麥克風到輸出到音響之間，還串接了大大小小的效果器，在這個過程中，無論
是麥克風、音響或是效果器，都是不同的 AUNode。AUNode 是這些器材的實體，
而我們要操控這些器材、改變這些器材的效果屬性，就會需要透過每個器材各自
的操控介面，這些介面便是AudioUnit，最後構成整個舞台，便是 AUGraph。

AUNode 與 AudioComponent 的差別在於，其實像上面講到的各種器材，除了可
以放在 AUGraph 使用之外，也可以單獨使用，比方說我們有台音響，我們除了
把音響放在舞台上使用外，也可以單獨拿這台音響輸出音樂。當我們要在
AUGraph 中使用某個器材，我們就要使用 AUNode 這種形態，單獨使用時，就使
用 AudioComponent。但無論是操作 AUNode 或 AudioComponent，都還是得透過
AudioUnit 這一層操作介面。

AUNode 與 AudioComponent 分成好幾類，包括輸入、輸出、混音、效果處理、
格式轉換等等，彼此之間可以互相串接。輸入裝置包括像麥克風輸入或 MIDI 樂
器，效果處理則包括像EQ 等化器、殘響（reverb）、改變音調（pitch）等；至
於這邊所謂的格式轉換，是指在不同的 LPCM 格式之間轉換，在這一層 API 中
只支援 LPCM 格式，但LPCM 之間又有很多種，不見得每個 node 都支援所有的
LPCM 格式，像 reverb效果的 effect node 就只支援浮點數，所以要讓音訊資
料通過這個 node 之前，就需要先轉換成浮點數格式的 LPCM 資料。

每個 AudioUnit 都有各自的輸入與輸出，在串接的時候，就是從某個
AudioUnit 的輸出，串接到另外一個 AudioUnit 的輸入，這種輸入輸出的端子
叫做 bus，而每個 AudioUnit 最少會有一個輸入與輸出的 bus，也可能會有多
個 bus。以 mixer 來說，就會有多個 bus，當我們從兩個輸入 bus 將資料送到
同一個 mixer 上時，就可以產生混音效果。

在 Mac OS X 上，我們通常會使用 default output 作為音訊播放的最終輸出的
AudioUnit，以 default input 作為錄音的起點。在 iOS 上則有一個特別的
AudioUnit，叫做 Remote IO，這個 AudioUnit 同時代表 iOS 的輸出與輸入裝
置。Remote IO 有兩個 bus，bus 0 就是 iOS 的預設輸出，bus 1 則是輸入，
所以我們在 iOS 上播放音樂，就是往 Remote IO 的 bus 0 傳送資料。

Remote IO 的 bus 1 預設是關閉的，當我們要錄音的時候，我們必須先告訴
Remote IO 把 bus 1 變成 enable，但我們要做這件事情的時候，我們不但要獲
得使用者給予我們使用麥克風的授權，還要設定正確的 Audio Session。我們會
在後面說明 Audio Session。

在使用 Audio Unit Processing Graph API 的時候，我們經常需要設定 render
callback function。以錄音來說，當我們從 Remote IO 的 bus 1 收到資料後，
想要儲存檔案，我們並沒有一種叫做「存檔」的 AudioUnit，而是我們要對某個
AudioUnit 設定 callback function，綁定某個 bus，在這個 function 中撰寫
存檔的程式。以播放來說，當我們告訴 AUGraph 或 Remote IO 開始播放，我們
也要設定 render callback function，提供用來播放用的資料。

由於這邊只支援 LPCM 格式，因此我們在播放 MP3 或 AAC 資料之前，還得有一
個將 MP3 或 AAC 轉換成 LPCM 格式的 converter。總之，我們提到播放網路串
流音樂有六個步驟，當我們用到這一層 API 的時候，這六個步驟都得自己來了。

講完 AUGraph 會比較容易理解 AVFoundation 中最新的 audio player：
AVAudioEngine，AVAudioEngine 是 Objective-C API，用 Objective-C 物件把
AUGraph API 多包裝了一層，AVAudioEngine 裡頭的 AVAudioPlayerNode、
AVAudioUnitEffect 等等，都可以找到對應的 C API，但是高階許多。不過，從
AVAudioPlayerNode 的設計來看，AVAudioEngine 看起來很容易處理本機檔案，
只要傳入一個 file URL 就可以輕鬆播放，並且加入各種效果。不過，如果是網
路串流，看起來我們還是得自己轉成 PCM Buffer 送給 AVAudioPlayerNode。
