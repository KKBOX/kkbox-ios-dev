記憶體管理 Part 2
=================

我們在這一章當中主要討論 ARC。

在前一章提到，由於 ARC 是透過靜態分析，在 Compile Time 決定應該要在程
式碼的那些地方加入 retain、release，所以，要使用 ARC 基本上相當簡單，
就是先把原本要手動管理記憶體的地方，把 retain、release 都拿掉，在
dealloc 的地方，也把 `[super dealloc]` 拿掉。

但是，有了 ARC，並不代表在開發 iOS 或 Mac OS X App 的時候，就不需要了
解記憶體管理，我們雖然很多程式會使用 Objetive-C 語言開發，但是還是會經
常用到 C 語言，我們還是得要了解 C 語言裡頭的記憶體管理。而且，有時候，
ARC 也會把 retain、release 加錯地方。

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
	self.timer = [NSTimer scheduledTimerWithTimeInterval:1.0
		target:self
		selector:@selector(timer:)
		userInfo:nil
		repeats:YES];
}

@end
```

要修正這個問題，我們應該改成，在 viewDidDisappear: 的時候，就要停止
timer。

Toll-Free Bridged
-----------------

前面在講 Category 的時候提到，Foundation Framework 裡頭的每個物件，都
有對應的 C 實作，這一層 C 的實作叫做 Core Foundation，當我們在使用
Core Foundation 裡頭的 C 型態時，像是 CFString、CFArray 等，我們可以讓
這些型態變成可以接受 ARC 的管理。這種讓 C 型態也可以被當做 Objetive-C
物件，接受 ARC 的記憶體管理的方式，叫做 Toll-Free Bridged。

Toll-Free Bridged 有三個語言關鍵字： `__bridge`、 `__bridge_retained`、
以及 `__bridge_transfer`。我們直接翻譯蘋果官網的定義：

* `__bridge` 會把 Core Foundation 的 C 資料型態轉換成 Objetive-C 物件，但
  是不會多做 retain 與 release。
* `__bridge_retained` 會把 Core Foundation 的 C 資料型態轉換成
  Objetive-C 物件，並且會做一次 retain，但是之後必須由我們自己手動呼叫
  CFRelease，釋放記憶體。
* `__bridge_transfer` 會把 Core Foundation 物件轉換成 Objective-C 物件，
  並且會讓 ARC 主動添加 retain 與 release。

不見得每個 Core Foundation 型態都有辦法轉換成 Objective-C 物件。請參閱
蘋果官方的
[詳細說明](https://developer.apple.com/library/ios/documentation/CoreFoundation/Conceptual/CFDesignConcepts/Articles/tollFreeBridgedTypes.html)
。

其他
----

Objetive-C 語言有了 ARC 之後，除了禁止使用 retain、release 這些關鍵字
之外，也禁止了一些我們在 ARC 之前的程式寫作方式，包括我們不可以把
Objective-C 物件放進 C Structure 裡頭，Compiler 會告訴我們語法錯誤。

在有 ARC 之前，我們之所以會把 Objective-C 物件放進 C Structure 裡，大
概會有幾個目的，其一是，假如我們有某個 Class 有很多成員變數，那我們可
能會想以下這種寫法將成員變數分成群組：

``` objc
@interface MyClass : NSObject
{
	struct {
		NSString *memberA;
		NSString *memberB;
	} groupA;

	struct {
		NSString *memberA;
		NSString *memberB;
	} groupB;
}
@end
```

這樣，如果我們想要使用 groupA 裡頭的 memberA，可以用
`self.groupA.memberA` 呼叫。

另外一種目的，則是有時候，我們可能會想要刻意隱藏某個 Objective-C Class
裡頭有哪些成員變數。像下面這段 code 裡頭，我們原本有一個 Class 叫做
MyClass，裡頭有 privateMemberA 與 privateMemberB 兩個成員變數，原本應
該直接寫在 MyClass 的宣告裡頭，但是我們卻刻意把這兩個成員變數包
進_Privates 這個 C Structure 裡頭，而原本放在 MyClass 成員變數宣告的地
方，指剩下了一個叫做 privates 的指標，光看到這個指標，讓人難以理解這個
Class 裡頭到底有什麼東西。

``` objc
@interface MyClass : NSObject
{
	void *privates;
}
@end

typedef struct {
	NSString *privateMemberA;
	NSString *privateMemberB;
} _Privates;

@implementation MyClass

- (void)dealloc
{
	_Privates *privateMembers = (_Privates *)privates;
	[privateMembers->privateMemberA release];
	[privateMembers->privateMemberB release];
	free(privates);
	privates = NULL;
	[super dealloc];
}

- (instancetype)init
{
	self = [super init];
	if (self) {
		privates = calloc(1, sizeof(_Privates));
		_Privates *privateMembers = (_Privates *)privates;
		privateMembers->privateMemberA = @"A";
		privateMembers->privateMemberB = @"B";
	}
	return self;
}
@end
```

這種寫法其實是種程式碼保護的技巧，主要在防範
[class-dump](http://stevenygard.com/projects/class-dump/)，或是從
class-dump 衍生出的
[class-dump-z](https://code.google.com/p/networkpx/wiki/class_dump_z)
這些工具。class-dump 可以從編譯好的 Binary 中還原出每個 class 的header，
當我們從 class-dump 抽出別人的 App 的 header，看出有哪些 Class，每個
Class 有哪些成員變數、有哪些 method，也就可以看出整個 App 的架構大致如
何。這種寫法就是讓別人用 class-dump 倒出我們 App 的 header 時，不會太
容易可以了解我們一些重要的 Class 是如何運作。

怎樣做逆向工程不是這份文件的重點。總之，有了 ARC 之後，我們都無法繼續
使用以上兩種的程式寫作方式。
