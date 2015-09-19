Core Animation
==============

<iframe width="420" height="315"
src="https://www.youtube.com/embed/g2CQCG0kmew" frameborder="0"
allowfullscreen></iframe>

https://speakerdeck.com/dryman/mastering-core-animation

Core Animation 是在 iOS 上負責一般 UI 繪圖與動畫的 framework。當你想要
讓你的 app 有更豐富的動畫效果，但是你並不打算做套 3D 遊戲、還沒打算使
用 OpenGL 或 Metal 這些 3D 繪圖的 framework 的話，你就應該先看一下
Core Animation。

Core Animation 大概分成三種主要的 class：

- CALayer
- CAAnimation
- CATransaction



CALayer 與 UIView 的關係
------------------------

任何一個 view 都是兩種性質組成的，就是可以操作，以及可以被看到。一個
view 可以被操作，像是在 iOS 上按鈕可以收到觸控事件、在 Mac 上可以被滑
鼠點選，這是一個 view 作為 responder 的部份；至於一個 view 呈現出來的
外觀，在 iOS 上則是由 Core Animation 實作。

在 iOS 上，每個 UIView 都有一個屬於 Core Animation framework 的
CALayer 物件，負責呈現這個 view 的外觀。

我們在前面幾節的練習中，像我們在寫貪食蛇這個練習的時候，知道如果要改變
一個 view 的外觀，可以透過在 UIView 的 subclass 中 override 掉
`drawRect:` 達成。不過，`drawRect:` 其實是個 delegate call，用途不是繪
製 view，而是繪製 CALayer 的內容：每個UIView 都是屬於自己專屬的
CALayer 物件的 delegate，當我們要重繪某個view 的內容時，其實是叫
CALayer 重繪，而 CALayer 可以自己決定怎麼繪製內容，或是去問 delegate
該怎麼畫，在這個狀況下，由於 UIView 是 CALayer的 delegate，於是就呼叫
到了 `drawRect:`，把 `drawRect:` 繪製的內容，放到 CALayer 上。

因此，當我們遇到了很複雜的畫面，畫面中有許多不同的元素要一直變化、移動
的時候，我們不妨考慮讓 app 中出現是許多 layer，而不是 view，因為每多一
個 view，就會在 responder chain 當中出現一個 responder，因此會影響每一
輪 run loop 的速度。一個 layer 上面可以繼續增加 layer，就像 view 可以
呼叫 `addSubview:` 一樣，CALayer 也有對應的 `addSublayer:`。
