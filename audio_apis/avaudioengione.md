AVAudioEngine
-----

AVAudioEngine 是蘋果在 iOS 8 時推出的音訊處理相關 API，相較於 Core Audio/Audio
Unit 這些 C API，AVAduioEngine 相對比較高階：我們可以把 AVAudioEngine 想像成是把
Core Audio API 再用 Objective-C 物件包裝一層，對一些麻煩、重複的工作做一些抽象
化，不過，基本上還是要有一些跟音訊處理相關的背景知識，才有辦法操作
AVAudioEngine。

在蘋果剛推出 AVAudioEngine 的時候，個人感覺是，蘋果主要希望讓像是遊戲或混音軟體
的廠商用比較高階的 API，輕鬆製作有多的不同 input 時的混音應用，所以一開始也只有
提供 `AVAudioFile` 與 `AVAudioPCMBuffer` 這些 API，讓你透過 `AVAudioPCMBuffer`從
各種不同的音檔中讀出 PCM buffer，然後丟進 `AVAudioEngine` 混音，於是缺少了處理串
流的資料的編碼/解碼的相關 API，像是 `AVAudioCompressedBuffer` 與
`AVAudioConverter，`都是後面幾代 iOS 才出現的。所以，如果你想要用AVAudioEngine
API 實作一個串流播放器，你會需要用到 iOS 11 以上的 API。

## AVAudioEngine 與 Audio Unit Processing Graph 的關係

我們前面講過 Audio Unit Processing Graph API，所以我們可以了解 AVAudioEngine API
與 Core Audio之間的一些對應關係：

AVAudioEngine 包裝了 AUGraph：在使用 AVAudioEngine API 時，AVAudioEngine 這個
class 本身扮演了 AUGraph 的角色。我們在使用 AUGraph 的時候，當中所有的 AUNode都
要自己手動建立，使用 AVAudioEngine 則比較輕鬆：AVAudioEngine 本身就提供幾個基本
的節點，像是 `inputNode`、`outputNode`、`mainMixerNode` 等，只要直接呼叫
AVAudioEngine的這幾個 property，就會用 lazy 的方式產生出幾個基本的節點。

但，如果你要手動增加其他的節點，還是可以自己建立想要的 AUAudioNode 的 subclass，
然後透過 AVAudioEngine 的 `-attachNode:` 加入節點，以及用
`-connect:to:fromBus:toBus:format:` 將節點串連起來。AVAudioEngine `-attachNode:`
對應到 AUGraph 的 `AUGraphAddNode`，`-connect:to:fromBus:toBus:format:` 則對應到
`AUGraphConnectNodeInput`。

AUAudioNode 包裝了 AUNode，像是輸入、輸出、混音，以及播放過程當中的我們想要加入
的各種效果，都是各自的 AUAudioNode。以播放來說，我們會特別注意
AVAudioPlayerNode，AVAudioPlayerNode 是 AVAudioEngine 的播放資料來源，我們要播放
音訊，首先要建立自己的 AVAudioPlayerNode，然後把這個 AVAudioPlayerNode 連接到之
類的節點上，