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

Unsafe-Unretained
-----------------

另外，ARC 有時候會在一些地方沒做 retain，結果卻又自動多做了一次release
最後導致 Bad Access 的錯誤。我們在講 Selector 的時候提到，我們可以將
target/action 與必要的參數合起來變成另外一種物件，叫做 NSInvocation，
在 ARC 環境下從 NSInvocation 拿出參數時，就必須要額外注意記憶體管理問
題。

比方說，我們現在要把對 UIApplication 要求開啟指定 URL 這件事情，變成一
個 Invocation。

``` objc
NSURL *URL = [NSURL URLWithString:@"http://kkbox.com"];
NSMethodSignature *sig = [UIApplication instanceMethodSignatureForSelector:
						  @selector(openURL:)];
NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:sig];
[invocation setTarget:[UIApplication sharedApplication]];
[invocation setSelector:@selector(openURL:)];
[invocation setArgument:&URL atIndex:2];
```

但假如我們用以下這段 code 的方式，把 invocation 拿出參數的物件的時候，
就會遇到 Bad Access 錯誤：

``` objc
NSURL *arg = nil;
[invocation getArgument:&arg atIndex:2];
NSLog(@"arg:%@", arg);
// 在這邊會 crash
```

之所以會 crash 的原因是，我們在透過 `getArgument:atIndex:` 拿出參數的
時候，`getArgument:atIndex:` 並不會幫我們把 arg 多 retain 一次，而到了
用 NSLog 印出 arg 之後，ARC 認為我們已經不會用到 arg 了，所以就對 arg
多做了一次 release，於是 retain 與 release 就變得不成對。

我們要解決這個問題的方法是要把 arg 設為 Unsafe Unretained，讓 arg 這個
Objetive-C 物件的指標不被 ARC 管理，要求 ARC 不要幫這個物件做任何自動
的 retain 與 release。我們在這邊會用上 `__unsafe_unretained` 關鍵字。
程式會寫成這樣：

``` objc
__unsafe_unretained NSURL *arg = nil;
[invocation getArgument:&arg atIndex:2];
NSLog(@"arg:%@", arg);
```

循環 Retain
-----------

Toll-Free Bridged
-----------------


https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html
