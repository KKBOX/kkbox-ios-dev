實戰：JB 造成的 Exception
-------------------------

這是 KKBOX 的 crash report，裡頭說，在 main thread 發生了一個
exception，但是在 main thread 裡頭完全沒有呼叫到 KKBOX 的 code，但是出
現了 ActionMenu.dylib 與 Dial.dylib。這些都是 JB 之後才會出現的
library，所以一眼就可以看出；用戶 JB 了他的 iPhone。

Exception 發生在 ActionMenu.dylib 的 0x00327900，我們不清楚這行程式做
了什麼而造成 KKBOX crash，不過，就算知道了也沒什麼幫助，因為這個問題整
個在我們的守備範圍之外，我們能做的就是請客服通知用戶解除安裝
ActionMenu，或是像 ActionMenu 的作者回報，不會是由我們出面修正
ActionMenu 的問題。

大概在 iOS 5 到 iOS 6 左右的時間，因為 JB 造成的問題也形成我們在客服上
不小的負擔，有大量的客訴其實都是用戶自己破壞了軟體的穩定。最近一兩年有
比較好一些。
