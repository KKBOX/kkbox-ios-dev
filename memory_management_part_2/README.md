記憶體管理 Part 2
=================

我們在這一章當中主要討論 ARC。

在前一章提到，由於 ARC 是透過靜態分析，在 Compile Time 決定應該要在程
式碼的那些地方加入 retain、release，所以，要使用 ARC 基本上相當簡單，
就是先把原本要手動管理記憶體的地方，把 retain、release 都拿掉，在
dealloc 的地方，也把 `[super dealloc]` 拿掉。但有時候，ARC 也會把
retain、release 加錯地方。

ARC 可能會錯誤釋放記憶體的時機
------------------------------

在大概 iOS 5 到 iOS 6 的時代，寫出這樣程式，你會收到 Bad Access 的錯誤
而造成 crash：

``` objc
#import <QuartzCore/QuartzCore.h>

@implementation ViewController
- (CGColorRef)redColor
{
	UIColor *red = [UIColor redColor];
	CGColorRef colorRef = red.CGColor;
	return colorRef;
}

- (void)viewDidLoad
{
	[super viewDidLoad];
	CGColorRef red = [self redColor];
	self.view.layer.backgroundColor = red;
}
@end
```

之所以會發生這種錯誤，就在於 Compiler 所認定的「已經不需用使用某個記憶
體，因此可以釋放」的時機點有問題。以上面的程式來說，釋放記憶體的時機應
該是在 `self.view.layer.backgroundColor = red;` 這一行之後，但是有一段
時間，Compiler 卻認為是在 `return colorRef` 這一行之前，red 這個
UIColor 物件就已經沒有被使用而該被釋放，但釋放了 red，就會造成 red 物
件裡頭所包含的 CGColor 也被釋放，因此回傳了已經被釋放了 colorRef 變數
而造成 Bad Access。

``` objc
#import <QuartzCore/QuartzCore.h>

@implementation ViewController
- (CGColorRef)redColor
{
	UIColor *red = [UIColor redColor];
	CGColorRef colorRef = red.CGColor;
	// Compiler 可能會在這邊自動產生 [red release]
	return colorRef;
}

- (void)viewDidLoad
{
	[super viewDidLoad];
	CGColorRef red = [self redColor];
	self.view.layer.backgroundColor = red;
	// 正確釋放記憶體的時機應該是這邊
}
@end
```

這個問題可以參考
[ARC Best Practices](http://amattn.com/p/arc_best_practices.html) 乙文。

Toll-Free Bridged
-----------------


https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html


Unsafe-Unretained
-----------------
