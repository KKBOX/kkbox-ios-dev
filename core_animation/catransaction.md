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

``` objc
[CATransaction begin];
[CATransaction setDisableActions:YES];
// 原本的動畫 code
[CATransaction commit];
```

`[CATransaction setDisableActions:YES]` 這行也可以寫成
`[CATransaction setValue:@(YES) forKey:kCATransactionDisableActions]`，
意思是一樣的。

至於我們想要改變所有 property 動畫的時間，也是一樣的作法。比方說，我們
想把 0.25 秒的動畫改成三秒：

``` objc
[CATransaction begin];
[CATransaction setAnimationDuration:1.0];
[CATransaction commit];
```

在使用 CATransaction 的時候，我們應該避免巢狀呼叫，例如下面這個例子：

``` objc
[CATransaction begin];
[CATransaction setDisableActions:NO];
// 一些動畫 code

[CATransaction begin];
[CATransaction setDisableActions:YES];
// 原本的動畫 code
[CATransaction commit];

[CATransaction commit];
```

在我們的經驗中，這樣寫會讓畫面產生非常不自然的閃爍。
