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

ARC 也不會排除循環 Retain（Retain Cycle）的狀況，遇到了循環 Retain，還
是會造成記憶體漏水。循環 Retain 就是，A 物件本身 retain 了 B 物件，但
是 B 物件又 retain 了 A 物件，結果我們要在釋放 A 的時候才有辦法釋放 B，
但是 B 又得要在 B 被釋放的時候才會釋放 A，最後導致 A 與 B 都沒有辦法被
釋放。這種狀況通常最可能出現在：

1. 把 delegate 設為 strong reference，我們會在討論 delegate 的時候繼續
   討論這個狀況。
2. 某個物件的某個 property 是一個 block，但是在這個 block 裡頭把物件自
   己給 retain 了一份。我們會在討論 block 的時候討論這個狀況。
3. 使用 timer 的時候，到了 dealloc 的時候才停止 timer。

假如我們現在有一個 view controller，我們希望這個 view controller 可以
定時更新，那麼，我們可能會使用
`+scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:` 這
個 method 建立 timer 物件，指定定時執行某個 selector。我們要特別注意，
在建立這個 timer 的時候，我們指定給 timer 的 target，也會被 timer
retain 一份，因此，我們想要在 view controller 在 dealloc 的時候，才停
止 timer 就會有問題：因為 view controller 已經被 timer retain 起來了，
所以只要 timer 還在執行，view controller 就不可能走到 dealloc 的地方。

``` objc
#import <UIKit/UIKit.h>

@interface ViewController : UIViewController
@property (strong, nonatomic) NSTimer *timer;
@end

@implementation ViewController

- (void)dealloc
{
	[self.timer invalidate];
}

- (void)timer:(NSTimer *)timer
{
	// Update views..
}

- (void)viewDidLoad
{
	[super viewDidLoad];
	self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0 target:self selector:@selector(timer:) userInfo:nil repeats:YES];
}

@end
```

要修正這個問題，我們應該改成，在 viewDidDisappear: 的時候，就要停止
timer。

Toll-Free Bridged
-----------------


https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html
