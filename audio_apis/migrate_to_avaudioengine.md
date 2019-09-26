從 Audio Unit Processing Graph 到 AVAudioEngine
----

如果你用 AUGraph 開發了一個播放器，要移轉到 AVAudioEngine 的話，除了絕大多數的 C
API 會變成 Objective-C 之外，就是這兩種 API 中所提供的各種音訊效果不盡相同。

我們可以從 AVAudioEngine 的介面設計中，看到 AVAudioEngine 試圖抹去 macOS、iOS、
tvOS 等平台上的差異。像是：我們在前面提到，在 iOS/tvOS 上，我們用到了 Remote IO
輸出音訊，但 Remote IO 其實不只用在輸出而已，Remote IO 的 bus 0 的用途是用在輸
出，而bus 1 則是用在輸入，我們在使用麥克風錄製音訊的時候，是用 Remote IO 的 bus
1 傳到 AUGraph 中，但是在 macOS 上，輸出時使用的是 kAudioUnitType\_Output 以及
subtype kAudioUnitSubType\_HALOutput，在輸入時，則要考慮到 macOS 裝置可能有多個
麥克風（用 USB 外接了多個麥克風）、也可能一個麥克風都沒有（像是 Mac Mini）。不
過，在 AVAudioEngine 中，都統一抽象化成 input node 與 output node。

所以，一些原本在 AUGraph 中可以使用的 Audio Unit，在改用 AVAudioEngine 開發播放
器的時候，就可能找不到對應的 AVAudioUnit class：因為某些 Audio Unit 只有 iOS有，
但是 macOS 沒有，或反之。AVAudioEngine 所提供的，是多個不同的蘋果平台上都提供的
效果。

以 EQ 等化器來說，蘋果原本在 iOS 上提供的是 kAudioUnitSubType\_AUiPodEQ，在macOS
上提供 kAudioUnitSubType\_GraphicEQ，這兩種等化器的差異是，
kAudioUnitSubType\_AUiPodEQ 只提供一些蘋果用在 iPod 上的一些 preset，像是低音強
化、高音強化等，但是 kAudioUnitSubType\_GraphicEQ 則提供了 10 或 31 band 的設
置，可以自由調整不同頻段聲音的音量。在 AVAudioEngine 相關 API 中，只有
AVAudioUnitEQ，對應到 kAudioUnitSubType\_GraphicEQ，換言之，我們不能在
AVAudioEngine 中使用對應到 kAudioUnitSubType\_AUiPodEQ 的 AVAudioUnit，蘋果也在
iOS 13 當中將 kAudioUnitSubType\_AUiPodEQ 列為 deprecated。
