CALayer
-------

在 iOS 上如果我們要開始使用 Core Animation，會從某個 viwe 上面的
CALayer 開始著手。Mac 上的 NSView 預設是沒有 CALayer 的，如果想在 Mac
上使用 Core Animation，我們需要先把 NSView 的 `wantsLayer` 設定成 YES，
然後自己建立一個 CALayer 物件，設成 NSView 的 layer property。

我們在這邊先講 iOS 上 UIView 裡頭的 CALayer。

### CALayer 與 UIView 的關係

任何一個 view 都是兩種性質組成的，就是可以操作，以及可以被看到。一個
view 可以被操作，像是在 iOS 上按鈕可以收到觸控事件、在 Mac 上可以被滑
鼠點選，這是一個 view 作為 responder 的部份。

至於一個 view 呈現出來的外觀，不同於 Mac 上 NSView 本身具有繪圖的責任，
在 iOS 上，UIView 的外觀呈現，都是由 Core Animation 實作。我們看到的
UIView 的樣子，其實是裡頭的 CALayer 的樣子。

在 iOS 上，我們可以使用一系列的 UIView 的 class method 產生動畫，像是
呼叫 `+animateWithDuration:delay:options:animations:completion:` 這一
系列 method，其實並不是 View 本身產生動畫，而是 View 裡頭的 CALayer 物
件的功勞。或這麼說：其實 UIView 裡頭的 CALayer 本身就具有產生動畫的能
力，而改動 CALayer 的任何屬性，都會產生 0.25 秒的動畫，只是我們平常在
設定 UIView 的時候，UIKit 的設計是刻意把動畫觀關閉了。

我們在前面幾節的練習中，像我們在寫貪食蛇這個練習的時候，知道如果要改變
一個 view 的外觀，可以透過在 UIView 的 subclass 中 override 掉
`drawRect:` 達成。—UIView 的外觀難道不是由 `drawRect:` 的實作決定的嗎？
跟 CALayer 有什麼關係呢？

`drawRect:` 其實是個 delegate call，用途不是繪製 view，而是繪製
CALayer 的內容。

每個 UIView，都是屬於自己專屬的 CALayer 物件的 delegate，當我們要重繪
某個 view 的內容時，其實是叫 CALayer 重繪，重繪 CALayer 時會呼叫到
CALayer 的`-drawInContext:`，在這個 method 中，CALayer 可以自己決定怎
麼繪製內容，或是去問 delegate 該怎麼畫，而去問了 CALayer delegate 的
`-drawLayer:inContext:`。

在 UIView 的 `-drawLayer:inContext:` 的實作中，便呼叫了 `drawRect:`，
因此， `drawRect:` 繪製的內容，放到 CALayer 上。

所以我們可以知道幾件事情：雖然要設定一個 layer 的內容，最簡單的方法是
直接對 layer 設定 contents。contents 的型別是 id，不過在 iOS 上我們會
設成 CGImageRef（在 Mac 上則要設成 NSImage 物件）；CALayer 有很多的
subclass，我們也可以按照這些subclass 的要求設定。不過，如果我們自己
subclass 了 CALayer 的話，也可以自己override `-drawLayer:inContext:`。

另外，CALayer 並不是 responder。於是當我們遇到了很複雜的畫面，畫面中有
許多不同的元素要一直變化、移動的時候，我們不妨考慮讓 app 中出現是許多
layer，而不是 view，因為每多一個 view，就會在 responder chain 當中出現
一個 responder，因此會影響每一輪 run loop 的速度。一個 layer 上面可以
繼續增加 layer，就像 view 可以呼叫 `addSubview:` 一樣，CALayer 也有對
應的 `addSublayer:`。

最重要的是，如果你自己建立了 CALayer 物件，千萬不要把 delegate 指到某
個 UIView 上，UIView 已經是自己的 layer 的 delegate 了！

### 畫面截圖

在 iOS 7 之前，我們往往會利用 view 的內容是由 layer 繪製這點，產生某個
view 截圖。我們只要要求某個 view 的 layer 再對某個 graphic context 畫
一次圖，然後把繪製的內容放進一個 UIImage 物件即可。

``` objc
@implementation UIView(MyExtensions)
- (UIImage *)imageOfCurrentContent
{
	UIGraphicsBeginImageContextWithOptions(self.bounds.size, YES, [UIScreen mainScreen].scale);
	[self.layer renderInContext:UIGraphicsGetCurrentContext()];
	UIImage *viewImage = UIGraphicsGetImageFromCurrentImageContext();
	UIGraphicsEndImageContext();
	return viewImage;
}
@end
```

產生畫面的截圖這點在製作很多 UI 時非常有用，像我們想要某個 view A 變形、
然後另外一個 view B 出現這樣的轉場效果，中間的變化往往不是對 A 與 B 這
兩張圖片做操作，而是對 A 與 B 的畫面截圖做影像相關的處理，比起直接改變
兩個 view，讓 view 不斷重繪，處理兩張圖片的速度會快上許多。製作 UI 時
經常需要許多的障眼法。

在 iOS 7 之後我們通常不會用這種方式產生畫面截圖，原因是 iOS 7 之後
UIView 有新的 API `drawViewHierarchyInRect:afterScreenUpdates:` 可以使
用。新的 API 與透過 CALayer 繪圖的差別是，iOS 7 之後的 UI 設計大量使用
半透明毛玻璃效果的 view，而用 CALayer 截出的圖片無法抓到這部份，而新的
API 就蘋果的說法，可以抓到無論是 UIKit、Quartz、OpenGL ES、SpriteKit
等這種繪圖系統產生的畫面。

相關說明請參見蘋果官方文件：
[Technical Q&A QA1817 View Snapshots on iOS 7](https://developer.apple.com/library/ios/qa/qa1817/_index.html)

### 設定 CALayer 的基本樣式與屬性

CALayer 有許多的屬性與 UIView 相同，像是 frame、bounds，很多時候可以把
CALayer 想像成是 UIView 使用；不過，CALayer 有一些 UIView 所沒有的屬性
可以設定，只要對每個 view 的 layer 設定這些屬性，就可以改變原本只靠
UIView 的屬性沒有辦法達到的視覺效果。

比方說，UIView 沒有跟邊框相關的屬性，但是 CALayer 則可以設定
`cornerRadius`、`borderWidth`、`borderColor`，我們可以很快就設定好
layer 的邊框寬度與顏色，甚至可以設定圓角效果。

另外，UIView 也沒有陰影可以設定，但是對 CALayer 則可以設定
`shadowOpacity`、`shadowRadius`、`shadowOffset` 與 `shadowColor`，用來
設定陰影的顏色、位置、尺寸大小與透明度等。不過，我們會建議另外設定
`shadowPath`，用一個 CGPath 描述陰影的外框範圍，設上去之後的效能會快許
多。

前面提到，絕大多數的 CALayer 的屬性在設定之後，會產生 0.25 的動畫效果，
這種因為改變屬性而產生的動畫，蘋果的術語叫做 Implicit Animations；打開
CALayer.h，只要看到註解裡頭提到某個屬性是屬於 Animatable，就是會產生動
畫效果的屬性。

因為改變任何屬性都會產生動畫，所以，當我們建立了 layer 之後，通常會先
設好 frame，才把 layer 加到 super layer 上，不然，如果先加到 super
layer，才去改變 frame，就會產生很奇怪的動畫效果。

CALayer 在建立完畢之後，預設都是一倍解析度，所以在 Retina Display 的裝
置上，看起來都會糊糊的（尤其是使用 CATextLayer 這個用來顯示文字內容的
layer，更容易凸顯解析度的不足），所以需要告訴 CALayer 應該要用怎樣的解
析度，方法是透過設定 `contentsScale` 屬性。在 iOS 上，我們通常設成
UIScreen 的scale。

``` objc
layer.contentsScale = [UIScreen mainScreen].scale;
```

在 Mac 上這件事情會變得比較複雜：一個 layer 該用怎樣的解析度，會跟這個
layer 放在哪個 NSWindow 上有關，一台 Mac 可能本身的螢幕具有 Retina
Display，但是另外用 mini display port 等介面外接了其他不是 Retina
Display 的螢幕或投影機，而一個 window 可能會被拖放到不同的 screen 上，
所以，當某個 window 移動到某個 screen 上之後，上面的 layer 也要跟著反
應，把這些 layer 設定成對應的解析度。我們稍後說明。

### 實作 CALayer drawInContext: 需要注意的地方

如果我們打算自己實作 `drawInContext:`，就需要注意一下，我們平常在UIKit
用到的許多跟繪圖相關的功能，都是對 Core Graphics 的 current context 操
作（也就是呼叫 `UIGraphicsGetImageFromCurrentImageContext()` 回傳的
CGContextRef），但是在實作 `drawInContext:` 的時候，是要把圖片繪製到指
定的 context 裡頭；所以我們要先把傳入了 context 變成 current context。

要把某個 context 變成 current context，只要呼叫
`UIGraphicsPushContext()` 即可，不過，當我們離開 `drawInContext:` 的時
候，要記得呼叫 `UIGraphicsPopContext()`，還原到原本的設定。

``` objc
- (void)drawInContext:(CGContextRef)ctx
{
	UIGraphicsPushContext(ctx);
	// Your drawing code here.
	UIGraphicsPopContext();
}
```

了解 CALayer 之後，下一步就是要讓 CALayer 動起來。我們在下一節要討論的
就是讓 CALayer 移動的 CAAnimation。而 CALayer 其實有許多 subclass，我
們在後面也會繼續討論一些常用 CALayer subclass 的功能。
