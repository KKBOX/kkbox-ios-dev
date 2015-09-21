CATransaction
-------------

CATransaction 在 Core Animation framework 中主要扮演了「整體舞台設定」
的角色。當我們在改變某個 CALayer 的 animatable 的屬性的時候，預設都會
產生 0.25 秒的動畫，但很多時候我們想要改變這個動畫的時間長度，或是，當
我們剛建立某些 layer，要把這些 layer 放上畫面的時候，我們還不希望改動
這些 layer 時會產生動畫；我們這時候就會使用 CATransaction。

使用 CATransaction 的方式就是把我們想做特別設定的動畫 code，用
CATransaction 的 class method 前後包起來。比方說，我們現在希望不要產生
動畫，便可以這麼寫：
