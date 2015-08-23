Formal Protocol 與 Informal Protocol
------------------------------------

`@protocol` 這個關鍵字是在 Objective-C 2.0 之後出現的，在這之前要定義
protocol，則是寫成 NSObject 的 category，前者叫做 formal protocol，後
者則稱為 informal protoco。UIKit 問世時就採用 Objective-C 2.0 語法，至
於 Mac OS X，蘋果在 2008 年開始大幅改寫 Foundation 與AppKit，現在
（2012 年）絕大多數可以看到的 protocol，都是 formal protocol，但如果你
在 maintain 一份稍微有點歷史的程式，或是在蘋果少數的API 中，還是可以看
到 informal protocol。

在 Core Animation 裡頭，就可以看到`CALayerDelegate`、`CALayoutManager`、
`CAAnimationDelegate`，都還是informal protocol。其中 `CALayerDelegate`、
`CALayoutManager`兩者之間還夾著 `CAAction` 這個 formal protocol—在兩個
informal protocol中間夾著一個 formal protocol，實在讓人很反感—為什麼不
一起改掉呢？至於`CAAnimationDelegate` 也很怪異：CAAnimation 的
delegate 不是用assign，而是會 retain 起來。
