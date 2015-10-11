CAAnimation
-----------

要讓一個 CALayer 動起來，除了透過改變 layer 的 animanatable 的屬性外，
就是建立 CAAnimation 物件，然後對 layer 呼叫 `-addAnimation:forKey:`。
我們通常會選擇一種 CALayer 的 subclass，製作我們想要的動畫效果。

### CATransition

我們先從 Core Animation 內建的轉場效果 CATransition 講起（請不要跟
CATransaction 搞混），因為 CATransition 大概是最容易上手，而且可以馬上
有成就感的動畫。CATransition 通常用在兩個 view 之間的切換；UIView 本身
就有定義一些跟轉場有幾個以 transition 開頭的 class method，像是

- `+ transitionWithView:duration:options:animations:completion:`
- `+ transitionFromView:toView:duration:options:completion:`

而這些 method 可以設定的 tansition option 包括：

- `UIViewAnimationOptionTransitionNone`
- `UIViewAnimationOptionTransitionFlipFromLeft`
- `UIViewAnimationOptionTransitionFlipFromRight`
- `UIViewAnimationOptionTransitionCurlUp`
- `UIViewAnimationOptionTransitionCurlDown`
- `UIViewAnimationOptionTransitionCrossDissolve`
- `UIViewAnimationOptionTransitionFlipFromTop`
- `UIViewAnimationOptionTransitionFlipFromBottom`

也就是說，UIView 本身就已經提供了四種方向的翻轉（上下左右）、淡入淡出
與翻頁幾種轉場效果。其實這些效果都是用 Core Animation 實作的，而
CATransition 所提供的效果也遠比這些多，只要我們知道怎樣使用
CATransition，就可以使用更多的轉場效果。

我們來寫一個簡單的 view controller：這個 view controller 的 view 上面
我們額外建立一個 CALayer，然後加了一個按鈕，這個按鈕按下去之後的
action 是讓這個 layer 在紅色與綠色之間切換。

ViewController.h

``` objc
@import UIKit;
@import QuartzCore;

@interface ViewController : UIViewController
- (IBAction)toggleColor:(id)sender;
@end
```

ViewController.m

``` objc
#import "ViewController.h"

@interface ViewController ()
@property (strong, nonatomic) CALayer *aLayer;
@property (assign, nonatomic) BOOL isGreen;
@end

@implementation ViewController
- (void)viewDidLoad
{
	[super viewDidLoad];
	self.aLayer = [[CALayer alloc] init];
	self.aLayer.frame = CGRectMake(50, 50, 100, 100);
	self.aLayer.backgroundColor = [UIColor redColor].CGColor;
	[self.view.layer addSublayer:self.aLayer];
}

- (IBAction)toggleColor:(id)sender
{
	self.isGreen = !self.isGreen;
	self.aLayer.backgroundColor = self.isGreen ? [UIColor greenColor].CGColor : [UIColor redColor].CGColor;
}
@end
```

<iframe width="350" height="621"
src="https://www.youtube.com/embed/h5PfrQ3OevA?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

原本這個版本只會產生 0.25 秒的屬性變化，使用的是淡入淡出效果。我們來加
入 `kCATransitionMoveIn` 這個轉場效果看看，就會變成新的畫面從上方推入。

``` objc
- (IBAction)toggleColor:(id)sender
{
	self.isGreen = !self.isGreen;
	self.aLayer.backgroundColor = self.isGreen ? [UIColor greenColor].CGColor : [UIColor redColor].CGColor;
	CATransition *transition = [[CATransition alloc] init];
	transition.type = kCATransitionMoveIn;
	transition.subtype = kCATransitionFromRight;
	[self.aLayer addAnimation:transition forKey:@"tansition"];
}
```

<iframe width="350" height="621"
src="https://www.youtube.com/embed/EtrvXUZJhjY?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

CATransition 可以應用的地方不只是切換背景顏色而已，無論是改變這個
layer 的任何 animatable 的屬性，像是改變 contents，或是在這個 layer 中
增加或移除 sublayer，用 `addAnimation:forKey:` 加入了 CATransition 物
件之後，都可以產生轉場效果。

如果我們想要像 UIView 的
`+transitionWithView:duration:options:animations:completion:` 那樣，是
在某個 view 上面增加或移除 subview 的時候產生轉場效果，用 CATransition
的作法也差不多，我們先寫一好要某個 view A 新增或移除 subview，然後把
CATransition 加到 view A 的 layer 上即可。

蘋果在 iOS 上的 CATransition 的 public header 中定義了幾個 type：

- `kCATransitionFade`：淡入淡出動畫
- `kCATransitionMoveIn`：新的畫面從上方移入的效果
- `kCATransitionPush`：推擠效果，很像 navigation controller 的換頁
- `kCATransitionReveal`：原本的畫面從上方移出的效果

不過…其實蘋果有不少 private API 可以用。比方說，我們可以把 type 指定成
立體塊狀翻轉效果。

``` objc
transition.type = @"cube";
```

<iframe width="350" height="621"
src="https://www.youtube.com/embed/YI1wTK1xbok?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

在 http://iphonedevwiki.net/index.php/CATransition 這一頁上可以看到整
理好的 private API 可以呼叫。雖然蘋果的政策是禁止在上架的 app 中呼叫
private API，如果用了可能會被 reject…不過印象中其實有很多 app 都用到這
些 private API。

如果對內建的這些效果，甚至 private 的效果都不滿意的話，iOS 5 之後，
CATransition 還有一個叫做 filter 的屬性，我們可以在這個屬性上加上
CIFilter 物件，客製更多的轉場效果。

另外要注意，在我們呼叫 `addAnimation:forKey:` 的時候，如果加入的是個
CATransition 動畫，無論使用了怎樣的名稱當做 key，key 都會是 transition。

### CAPropertyAnimation

CAPropertyAnimation 便是透過設定某個 CALayer 的屬性產生動畫。前面提到，
只要修改 layer 的 animatable 屬性會自定產生動畫效果，不過，跟使用
CAPropertyAnimation 的狀況不太一樣，我們對某個 layer 加入了
CAPropertyAnimation 之後，雖然會產生動畫，但是就只有產生動畫而已，
layer 屬性原本的值並不會因此改變，用蘋果的術語，這種動畫叫做 Explicit
Animations。

CAPropertyAnimation 是一層介面，我們通常使用的是 CAPropertyAnimation
的 subclass CABasicAnimation。設定 CABasicAnimation 的時候，主要會設定
以下屬性：

1. `fromValue`: 要讓某個屬性產生變化的動畫時的初始值
2. `toValue`: 要讓某個屬性產生變化的動畫時結束的數值
3. `byValue`: 要讓某個屬性產生變化的動畫時，介於開始與結束的中間值，但
   是很多時候可以不用特別設定，設成 nil 即可

然後有一些設定是定義在 CAMediaTiming protocol 中，像是：

1. `duration`: 這個動畫要花上多少時間
2. `repeatCount`: 我們要執行這個動畫幾次，如果只要跑一次這個動畫，設定
   成 1 即可；如果我們想要這個動畫一直跑的話，不妨就把這個動畫設成
   `NSNotFound`，`NSNotFound` 就是整數的最大值。

比方說，當我們想要讓某個 layer 一直不停的旋轉，我們可以修改
`transform.rotation.x`、`transform.rotation.y`、`transform.rotation.z`
等，要求這個 layer 是按照 x、y 還是 z 軸旋轉，我們可以讓 fromValue 設
成 0，代表是初始還沒旋轉的狀態，至於 toValue 設成 M_PI * 2，代表要旋轉
360 度。像以下這段程式：

``` objc
[super viewDidLoad];
self.aLayer = [[CALayer alloc] init];
self.aLayer.frame = CGRectMake(50, 50, 100, 100);
self.aLayer.backgroundColor = [UIColor redColor].CGColor;
CABasicAnimation *rotateAnimation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
rotateAnimation.fromValue = @0.0f;
rotateAnimation.toValue = @(M_PI * 2);
rotateAnimation.autoreverses = YES;
rotateAnimation.repeatCount = NSUIntegerMax;
rotateAnimation.duration = 2.0;
[self.aLayer addAnimation:rotateAnimation forKey:@"x"];
[self.view.layer addSublayer:self.aLayer];
```

效果如下：

<iframe width="350" height="621"
src="https://www.youtube.com/embed/3sRDqSIK-nM?rel=0&amp;showinfo=0"
frameborder="0" allowfullscreen></iframe>

### CAKeyframeAnimation

使用 CAKeyframeAnimation 與 CABasicAnimation 的主要差別在於，我們想要
透過改變某個屬性產生動畫時，不是設定初始值與結束值，而是使用一個貝茲曲
線描述（是 CGPath）。所以，當我們希望動畫不只是直線前進，而是按照某種
曲線移動的時候，就可以使用 CAKeyframeAnimation。

### CAAnimationGroup

一個 CALayer 可以同時執行多個 CAAnimation，當我們加入了一個CAAnimation
之後，就會立刻執行這個動畫。而我們也可以把很多個 animation 物件包裝成
群組（像是 layer 一邊移動位置一邊翻轉），方法就是建立 CAAnimationGroup
物件，然後把想要變成群組的其他動畫，變成 array，設定成
CAAnimationGroup 的 `animations` property。

``` objc
CAAnimationGroup *group = [CAAnimationGroup animation];
group.duration = 0.5;
group.animations = @[positionAnimation, flipAnimation];
group.delegate = self;

[aLayer addAnimation:group forKey:@"group"];
```
