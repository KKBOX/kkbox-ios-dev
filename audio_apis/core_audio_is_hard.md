Core Audio 到底難在哪？
----------------------

相較於 iOS 問世的前幾年，後來出現了像 AVAudioEngine，以及
[The Amazing Audio Engine](http://theamazingaudioengine.com/) 與
[AudioKit](http://audiokit.io/) 等專案，會讓做 audio 的相關開發容易一
些，如果現在想要做一些新的 audio 應用，是可以先從這些第三方 library 下
手，可以免除掉不少直接使用像 AUGraph、AudioUnit 痛苦的地方。

使用 AUGraph 這些 audio API 最痛苦的地方，在於在開發過程中難免會有錯誤，
而發生錯誤時讓人完全難以理解。

每次呼叫一個 API 的時候，會回傳一種叫做OSStatus 的錯誤代碼，OSStatus
也就是 32 位元整數，有時候我們可以把這個整數轉換成四個 char，然後這四
個 char 可能會有一些意義，可以讓你大概判讀可能的原因。

Core Audio API 在錯誤代碼的解釋方面很少，由於相對於其他種類的 App 開發，
Audio App 可能是種比較冷門的領域，坊間講 Core Audio 開發的書籍目前也只
看到 2012 年 Chris Adamson 所做的 *Learning Core Audio* 那麼一本，當你
遇到錯誤的時候，連網路上的討論也找不到幾篇。而有些時候，蘋果對錯誤代碼
的說明，也跟你實際遇到的狀況完全沒有關係。

比方說，如果你還沒呼叫 `AUGraphInitialize`，就呼叫了 `AUGraphStart`，
那麼 `AUGraphStart` 就會回傳錯誤代碼
kAUGraphErr_CannotDoInCurrentContext （-10863）。要修正這個問題，是記
得要去呼叫一次 `AUGraphInitialize`，可是蘋果的說明文件是這麼說的：

> To avoid spinning or waiting in the render thread (a bad idea!),
> many of the calls to AUGraph can return:
> kAUGraphErr_CannotDoInCurrentContext. This result is only generated
> when you call an AUGraph API from its render callback....

…有很多的 API 可能會回傳 kAUGraphErr_CannotDoInCurrentContext，避免在
AUGraph 的 render threading 發生卡死的狀況。這種狀況只有當你在 render
thread 裡頭呼叫了 AUGraph API 才會發生…

可是我們明明是在呼叫 `AUGraphStart` 的時候就發生錯誤了，連 render
thread 都還沒有進去。這個說明跟我們遇到的狀況完全無關啊！

或，我們寫好了前一節的 player 之後，想繼續玩玩看有哪些 effect node 可
以使用，我們就把原本 EQ node 裡頭這行
`EQUnitDescription.componentSubType = kAudioUnitSubType_AUiPodEQ` 換成
`EQUnitDescription.componentSubType = kAudioUnitSubType_Reverb2`，想試
試看 iOS 上的 reverb 效果如何，結果在設定 effect node 的輸入格式的時候，
發生 kAudioUnitErr_FormatNotSupported 錯誤（-10868），為什麼呢？你可能
要花上很長一段時間在網路上搜尋，才在某個論壇的某篇文章知道，iOS 上的
reverb2 這個 effect node 只接受浮點格式的 LPCM 資料，但我們現在卻是使
用 16 位元整數，因此會在設定輸入格式的時候失敗。

可是，你去翻蘋果官方文件，卻也根本找不到 reverb2 只能夠接受哪些輸入格
式的相關說明，唯一跟 kAudioUnitSubType_Reverb2 相關的說明就只有兩行：
- iPhone only. A reverb for iOS。
