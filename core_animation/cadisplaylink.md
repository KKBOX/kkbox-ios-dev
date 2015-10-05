CADisplayLink
-------------

我們在寫貪食蛇的練習的時候，曾經使用 NSTimer 定時要求 view 重繪（對
view呼叫 setNeedsDisplay），而達到讓 view 產生讓蛇移動的動畫。不過，如
果我們希望有更順暢的動畫，我們會選擇 Core Animation，而不是這種讓 view
一直重繪的動畫。

原因是，在 timer 所處在的 run loop 中，因為要處理多種不同的輸入，所以
timer 的最小週期是在 50 到 100 毫秒之間（相當於 0.05 到 0.1 秒），一秒
鐘之內，頂多只能夠跑 20 次 NSTimer。[^1]

如果我們希望在螢幕上看到流暢的動畫，那麼，一秒鐘就必須要有 60 格，每格
之間的間距 0.016 秒左右，NSTimer 拿不到這個速度。而 Core Animation 有
另外一個 timer 物件，叫做 CADisplayLink。

CADisplayLink 也是要註冊在 runloop 中，但不像 NSTimer。NSTimer 需要在
上一次 run loop 整個跑完之後才會驅動，而註冊了 CADisplayLink 之後，只
要螢幕需要重繪，就會呼叫 CADisplayLink 所指定的 selector。所以我們可以
看到像 Cocos2D 這類的遊戲引擎，便是使用 CADisplayLink 作為 animator，
每格 60 秒重繪一次畫面。

像 Cocos2D 這類遊戲引擎是使用 OpenGL 繪製畫面，不過，其實我們也可以在
CADisplayLink 指定的 selector 中，透過 CATransaction 關閉 Implicit
Animation 後（呼叫 CATransaction 的 `setDisableActions:` 傳入 YES），
然後改變 CALayer 的屬性，如此一來，也可以產生許多的 CALayer 動畫效果。

[^1]: 請參見 NSTimer 的文件 https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSTimer_Class/
