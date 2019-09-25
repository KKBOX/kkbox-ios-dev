AVAudioEngine
-----

AVAudioEngine 是蘋果在 iOS 8 時推出的音訊處理相關 API。

相較於 Core Audio/Audio Graph/Audio Unit 這些 C API，AVAudioEngine 相對比較高
階：我們可以把 AVAudioEngine 想像成是把 Core Audio API 再用 Objective-C 物件包裝
一層，對一些麻煩、重複的工作做一些抽象化，不過，基本上還是要有一些跟音訊處理相關
的背景知識，才有辦法操作 AVAudioEngine。

由於只是包裝一層，所以你還是可以從這些介面中，接觸到原本的 C API，像是
`AVAudioUnit` 這個 class 有個屬性，叫做 `audioUnit`，就可以拿到 `AudioUnit` 型態
的資料；在 `AVAudioBuffer` 中，也可以拿到屬於 C API 的 `AudioBufferList` 資料。

在蘋果剛推出 AVAudioEngine 的時候，個人感覺是，蘋果主要希望讓像是遊戲或混音軟體
的廠商用比較高階的 API，輕鬆製作有多的不同 input 時的混音應用，所以一開始也只有
提供 `AVAudioFile` 與 `AVAudioPCMBuffer` 這些 API，讓你透過 `AVAudioPCMBuffer`從
各種不同的音檔中讀出 PCM buffer，然後丟進 `AVAudioEngine` 混音。在這個時候，還缺
少了處理串流的資料的編碼/解碼的相關 API，像是 `AVAudioCompressedBuffer` 與
`AVAudioConverter`，都是後面幾代 iOS 才陸續出現。在我們的經驗中，如果你想要盡可
能使用 AVAudioEngine API 實作一個串流播放器的話，你會需要用到 iOS 11 以上的
API。

## AVAudioEngine 與 Audio Unit Processing Graph 的關係

我們前面講過 Audio Unit Processing Graph API，所以我們可以了解 AVAudioEngine API
與 Core Audio之間的一些對應關係：

AVAudioEngine 包裝了 AUGraph：在使用 AVAudioEngine API 時，AVAudioEngine 這個
class 本身扮演了 AUGraph 的角色。我們在使用 AUGraph 的時候，當中所有的 AUNode都
要自己手動建立，使用 AVAudioEngine 則比較輕鬆：AVAudioEngine 本身就提供幾個基本
的節點，像是 `inputNode`、`outputNode`、`mainMixerNode` 等，只要直接呼叫
AVAudioEngine的這幾個 property，就會用 lazy 的方式產生出幾個基本的節點。

但，如果你要手動增加其他的節點，還是可以自己建立想要的 AUAudioNode 的 subclass，
像是 EQ 等化器、Reverb 殘響效果…等等，然後透過 AVAudioEngine 的 `-attachNode:`
加入節點，以及用 `-connect:to:fromBus:toBus:format:` 將節點串連起來。
AVAudioEngine `-attachNode:`對應到 AUGraph 的 `AUGraphAddNode`，
`-connect:to:fromBus:toBus:format:` 則對應到`AUGraphConnectNodeInput`。

`AUAudioNode` 包裝了 `AUNode` ，像是輸入、輸出、混音，以及播放過程當中的我們想要
加入的各種效果，都是各自的 AUAudioNode。以播放來說，我們會特別注意
`AVAudioPlayerNode`：`AVAudioPlayerNode` 是 `AVAudioEngine` 的播放資料來源，我們
要播放音訊，首先要建立自己的 `AVAudioPlayerNode`，然後連接到 main mixer node 之
類的節點上。

在使用 AUGraph API 的時候，我們的作法是準備好 render callback function，當
AUGraph 需要資料的時候，從 render callback function 中提供資料。至於在使用
AVAudioEngine 時，我們則是反過來，主動透過 schedule… 開頭的一系列 method，或是提
供一個 Audio File，或是把 PCM 資料包在 `AVAudioPCMBuffer` 物件裡頭，餵入
AVAudioPlayerNode。

如果我們想要播放的是直接從網路抓取的音樂資料，而且是壓縮音檔，那麼就必須把 MP3、
AAC 之類的檔案，手動轉換成 PCM 資料。在 AVAudioEngine API 中，有兩種不同的buffer
物件：`AVAudioPCMBuffer` 與 `AVAudioCompressedBuffer`，兩者都繼承自
`AVAudioBuffer`。

`AVAudioPCMBuffer` 用來包裝 PCM 格式的資料，而如果是壓縮音檔，就得先裝在
`AVAudioCompressedBuffer` 這個容器中，然後，你可以用`AVAudioConverter` 轉換格
式，以播放的情境來說，你就往往需要把`AVAudioCompressedBuffer` 轉換成
``AVAudioCompressedBuffer`，在錄音的情境下則反之。

我們知道，並不是只有一種格式的 PCM，而可以是 16 位元整數、32 位元整數或 32 位元
浮點數…等等。所以我們可以看到 `AVAudioPCMBuffer` 有三個屬性：
`floatChannelData`、`int16ChannelData` 與 `int32ChannelData`，裡頭分別是不同型態
的 PCM 資料。請注意，在三個屬性當中，一次只會有一個屬性中會有資料，例如，如果你
用的是 16 位元整數的 PCM 資料，那麼就只有 `int16ChannelData` 會拿到資料，其他兩
個屬性當中都會是空的。

## 用 AVAudioEngine 實作播放器

用 AVAudioEngine 實作一個播放器的流程大概是：

* 建立 `AVAudioEngine` 的 instance
* 將各種音效處理的 AVAudioUnit 物件加入到 `AVAudioEngine` 中，這時候會呼叫到
  `-attachNode:`、`-connect:to:fromBus:toBus:format:`…等等
* 建立 `AVAudioPlayerNode` 的 instance，連接到 main mixer node 上
* 發送網路連線抓取音檔
* 將音檔的 packet 儲存起來
* 要求 `AVAudioEngine` 與 `AVAudioPlayerNode` 開始播放
* 將壓縮音檔的 packet 轉換成 `AVAudioPCMBuffer`，餵入 `AVAudioPlayerNode`。

