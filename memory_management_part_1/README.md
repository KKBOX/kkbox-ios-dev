記憶體管理 Part 1
=================

記憶體管理是一件極其簡單又極其麻煩的事情。

記憶體管理在做什麼？
--------------------

說它簡單，因為所謂的記憶體管理不過就是兩件事情：一塊記憶體我們是要用，
還是不要用；該用的時候就用，不用的時候就釋放。麻煩的地方就在於我們很容
易疏忽，而造成用與不用不成對。這就很像開車，開車也不過就是前進與煞車，
但是天天路上都會發生車禍。

不成對，就會造成兩種結果：如果用完之後不釋放記憶體，就會造成軟體佔用了
一堆沒有用到的記憶體，記憶體用量愈來愈大，造成記憶體用盡，在iOS 上系統
會強制終止我們的應用程式，這種狀況叫做記憶體漏水（Memory Leak）。

反之，如果一塊記憶體已經被釋放掉了，我們卻還以為這塊記憶體還存在我們可
以呼叫的物件，所以當我們嘗試呼叫的時候，才發現這塊記憶體該存在的物件已
經不在了，這種狀況叫做over-release 或是 invalid memory reference，會造
成應用程式 crash，crash log 上面會告訴你錯誤類型是 `EXC_BAD_ACCESS` 。

處理 over-release 可能是初學者開始學習 Mac OS X 與 iOS平台的第一個障礙
（我覺得通常第二個障礙是搞不懂 delegate是什麼）：一開始寫程式，程式卻
一直莫名其妙的crash，於是前往網路論壇求救，但是通常也不會有多少人幫忙，
因為網路上的其他同行通常都樂於回答問題，但懶得幫別人修bug。

過去三十年學界與業界也一直努力解決記憶體管理的問題，畢竟在寫軟體的時候，
不把力氣放在處理程式邏輯問題，而是這種瑣碎的困擾，對於工程師的生產力是
一大傷害，主要提出的解決方案是記憶體自動回收（Garbage Collection，GC），
在軟體執行時，如果發現已經沒有任何一個變數指向某塊記憶體，就代表這塊記
憶體再也用不到，於是開始回收這塊記憶體。90年代之後誕生的程式語言，幾乎
都有 GC 機制。

蘋果曾經推出 Mac OS X 10.5 時，在 Mac OS X 上實作了GC，同時也帶動一些
使用 GC 的動態語言開始與 Cocoa Framework 橋接，誕生了PyObjC、MacRuby、
RubyCocoa、JSTalk 專案，我們因此也可以使用Python、Ruby、JavaScript 等
語言，直接撰寫 Mac OS X應用程式。但是，蘋果所實作的 GC，問題不小。

但我過去參與一項專案，在 Mac OS X 10.6 上面以 GC開發一個軟體，結果可說
是淒慘無比：整個 Cocoa Framework 是架構在很多 C library 上的，而底層許
多 library 其實並不支援 GC，我們發現只要在 main thread 之外用到像是
`NSDateFormatter` 、 `NSNumberFormatter` 這些class（這些 class 架構在
libICU 上，是 IBM 的一項 Open Source專案，用來處理多國語系的格式問題，
同時也是一套 Regular Expression引擎），GC 就完全沒有作用，記憶體狂漏不
止。蘋果在推出 iOS之後，也始終不在 iOS 上實作 GC。

隨著蘋果在 Compiler 的投資逐漸展露成果，逐步將 Compiler 從 GCC 換成
LLVM，最後決定改變技術方向，從另外一種方向來解決將記憶體管理自動化的問
題，就是在iOS 5 上推出的 ARC（Automatic Reference Counting），不在
runtime回收記憶體，而是在編譯程式的時候，自動幫你加上與記憶體釋放有關
的程式碼。

雖然 ARC推出的目的就是希望你以後再也不要為記憶體管理問題煩惱，但問題是，
實際上，你還是得知道記憶體管理是怎麼運作的。—記得有人打比方說，GC是自
排車，而 Objective-C的記憶體管理就像開手排車，要自己知道怎麼進檔對檔，
我覺得，加上 ARC呢，其實還是手排車，只是你訓練了一隻猴子幫你在副駕駛座
幫你打檔，而你接下來，還得要學會怎麼調教這一隻猴子。

Reference Count/Retain/Release
------------------------------

Objective-C 語言裡頭每個物件，都是指向某塊記憶體的指標，在 C語言當中，
你會使用像是 `malloc` 、 `calloc` 這些 function配置記憶體，用完之後，
就呼叫 `free`釋放記憶體。但是我們如何知道一塊記憶體被多少地方用到，之
後這些地方又不再用到呢？所以在Objective-C語言發展之初，就建立了一套計
算有多少地方用到某個物件的簡單機制，叫做*reference count*，意義非常簡
單：只要一個物件被某個地方用到一次，這個地方就對這個物件加一，反之就減
一，如果數字減到變成了零，就該釋放這塊記憶體。

一個物件如果使用 `alloc` 、 `init` …產生，初始的 reference count 為
1。接著，我們可以使用 `retain` 增加 reference count。

``` objc
[anObject retain]; // 加 1
```

反之就用 `release` ：

``` objc
[anObject release]; // 減 1
```

我們可以使用 `retainCount` 檢查某個物件被 retain 了多少次。

``` objc
NSLog(@"Retain count: %d", [anObject retainCount]);
```

有了基本概念之後，我們就可以看出以下程式有什麼問題

``` objc
id a = [[NSObject alloc] init];
[a release];
[a release];
```

因為在第二行，a所指向的記憶體已經被釋放了，所以第三行想要再釋放一次，
就會造成錯誤。同樣的：

``` objc
id a = [[NSObject alloc] init];
id b = [[NSObject alloc] init];
b = a;
[a release];
[b release];
```

在第三行中，由於 b 指向了 a 原本所指向的記憶體，但是 b原本所指向的記憶
體卻沒有釋放，同時再也沒有任何變數指向 b原本指向的記憶體，因此這塊記憶
體就發生了記憶體漏水。接著，在第四行呼叫`[a release]` 時，這塊記憶體就
已經被放掉了，但是由於 a 與 b都已經指向了同一塊記憶體，所以第五行的
`[b release]`也是操作同一塊記憶體，於是會發生 `EXC_BAD_ACCESS` 錯誤。

Auto-Release
------------

如果我們今天有一個 method，會回傳一個 Objective-C 物件，假使寫成這樣：

``` objc
- (void)one
{
    return [[NSNumber alloc] initWithInt:1];
}
```

那麼，每次用到了由 one 這個 method產生出來的物件，用完之後，都需要記住
要 release這個物件，很容易造成疏忽。慣例上，我們會讓這個 method 回傳
auto-release的物件。像是寫成這樣：

``` objc
- (void)one
{
    return [[[NSNumber alloc] initWithInt:1] autorelease];
}
```

所謂的 auto-release 其實也沒有多麼自動，而是說，在這一輪 runloop中我們
先不釋放這個物件，讓這個物件可以在這一輪 runloop中都可以使用，但是先打
上一個標籤，到了下一輪 runloop 開始時，讓 runtime判斷有哪些前一輪
runloop 中被標成是 auto-release 的物件，這個時候才減少retain count 決
定是否要釋放物件。

我們在這邊遇到了一個陌生的名詞： Runloop ，同一個概念在 Windows平台或
許會稱為 Message Loop 。Runloop就是事件的循環，我們先來想一個簡單的問
題：當我們在開發 Mac OS X 或是 iOS軟體的時候，程式進入點是在 main.m 裡
頭，執行 `main()` ，為什麼 `main()`不會跑完馬上結束，而是會讓應用程式
可以繼續執行？

原因是，在 `main()`中，我們會建立一個不斷執行的迴圈，在每一輪迴圈中的
開始，會從硬體收取硬體事件，像是鍵盤、滑鼠、觸控事件等，在發現硬體事件
之後，應用程式首先將事件送給對應到這個事件所在位置的視窗（`NSWindow`
物件或是 `UIWindow`物件），接著視窗物件會將事件送給負責處理的 UI元件，
這個目前負責應該處理事件的最上層元件，我們則稱之為 First Responder。這
樣的迴圈，就叫做 Runloop 。

在建立 Foundation 物件的時候，除了可以呼叫 `alloc` 、 `init` 以及
`new`之外（ `new` 這個 method 其實就相當於呼叫了 `alloc` 與 `init`；比
方說，我們呼叫 `[NSObject new]` ，就等同於呼叫了`[[NSObject alloc]
init]` 。），還可以呼叫另外一組與物件名稱相同的method。

以 `NSString` 為例，有一個叫做 `initWithString` 的 instance method，就
有一個對應的 class method 叫做 `stringWithFormat` ，使用這一組method，
就會產生 auto-release 的物件。也就是說，呼叫了
`[NSString stringWithFormat:...]` ，相當於呼叫了`[[[NSString alloc]
initWithFormat:...] autorelease]` 。使用這一組method，可以讓程式碼較為
精簡。

基本原則
--------

先整理一下我們已經學到的事情：

-   如果是 `init` 、 `new` 、 `copy` 這些 method
    產生出來的物件，用完就該呼叫 `release` 。
-   如果是其他一般 method 產生出來的物件，就會回傳 auto-release
    物件、或是 `Singleton` 物件（稍晚會解釋什麼是
    Singleton），就不需要另外呼叫 `release` 。

而呼叫 retain 與 release 的時機包括：

-   如果是在一般程式碼中用了某個物件，用完就要 release 或是
    auto-release。
-   如果是要將某個 Objective-C
    物件，變成是另外一個物件的成員變數，就要將物件 retain 起來。但是
    delegate 物件不該 retain，我們稍晚會討論什麼是 delegate。
-   在一個物件被釋放的時候，要同時釋放自己的成員變數，也就是要在實作
    dealloc 的時候，釋放自己的成員變數。

要將某個物件設為另外一個物件的成員變數，需要寫一組
getter/setter。我們接下來要討論怎麼寫 getter/setter。

Getter/Setter 與 Property 語法
------------------------------

Getter 就是用來取得某個物件的某個成員變數的 method，setter則是用來設定
成員變數。如果某個成員變數是 C 的型別，像是int，我們可以這麼寫。假使我
們有個 Class 叫做 MyClass，成員變數是number：

``` objc
@interface MyClass:NSObject
{
    int number;
}
- (int)number;
- (void)setNumber:(int)inNumber;
@end
```

我們建立了 setter 叫做 `setNumber:` ，而 getter 叫做 `number`
。請注意，在其他語言的慣例中，getter 可能會取名叫做 `getNumber`，但是
Objective-C 語言的慣例則是只取 `number` 這樣的名稱。實作則是：

``` objc
- (int)number
{
    return number;
}
- (void)setNumber:(int)inNumber
{
    number = inNumber;
}
```

如果是 Objective-C物件，我們則要將原本成員變數已經指向的記憶體位置釋放，
然後將傳入的物件retain 起來。可能像這樣：

``` objc
- (id)myVar
{
    return myVar;
}
- (void)setMyVar:(id)inMyVar
{
    [myVar release];
    myVar = [inMyVar retain];
}
```

假如今天我們在開發的應用程式裡頭用到了很多個 thread，而在不同的 thread
中，同時會用到 myVar，這麼寫其實並不安全：在某個 thread 中呼叫了
`[myVar release]` 之後，到 mvVar 指定到 inMyVar 的位置之間，假使另外一
個thread 剛好用到了 myVar，這時候 myVar剛好指到了一塊已經被釋放的記憶
體，於是就造成了 `EXC_BAD_ACCESS` 錯誤。

要避免這種狀況，一種方法是加上一些 lock，讓程式在呼叫 `setMyVar:`的時
候，不讓其他 thread 可以使用myVar；另外一種簡單的方法是，只要一直不要
讓 myVar指定到可能被釋放的記憶體位置。我們可以這麼改寫：

``` objc
- (void)setMyVar:(id)inMyVar
{
    id tmp = myVar;
    myVar = [inMyVar retain];
    [tmp release];
}
```

我們先將 myVar 原本指向的記憶體位置，暫存在一個變數中，接著直接將
myVar指到傳入的記憶體位置，接著再釋放 tmp變數中所記住的、原本的記憶體
位置。由於每次都要這麼寫，寫久了會覺得麻煩，通常會寫成一個macro，或是
直接使用 Objective-C 2.0 裡頭的 property 語法。

相信在開始學習開發 Mac OS X 與 iOS程式的時候，大部分書籍一開始就示範如
何使用 property語法。也就是，像我們上面的例子，用 property 語法可以寫
成：

``` objc
@interface MyClass:NSObject
{
    id myVar;
    int number;
}
@property (retain, nonatomic) id myVar;
@property (assign, nonatomic) int number;
@end

@implementation MyClass
- (void)dealloc
{
    [myVar release];
    [super dealloc];
}
@synthesize myVar;
@synthesize number;
@end
```

我們在這邊使用了 `@synthesize`語法，在編譯我們的程式的時候，其實就會被
編譯成我們在上面所寫的getter/setter，而我們想要設定 myVar 的內容時，除
了可以呼叫 `setMyVar:`之外，也可以呼叫 dot 語法，像是 `myObject.myVar
= someObject` 。

我們需要注意，在釋放記憶體的時候， `myVar = nil` 與 `self.myVar =
nil`這兩段程式是不一樣的，前者只是單純的將 myVar 的指標指向nil，但是並
沒有釋放原本所指向的記憶體位置，所以會造成記憶體漏水，但後者卻等同於呼
叫`[self setMyVar:nil]` ，會先釋放 myVar 原本指向的位置，然後將 myVar
設成nil。

在這邊先補充一下，在 Xcode 4.4 之後，如果用了 property語法，我們甚至不
用宣告對應的成員變數，compiler在編譯程式的時候，會自動補上 myVar 與
number 需要對應的成員變數。

偶而我們可以在網路上面聽到一些聲音，認為初學者應該避免使用property，主
要原因除了上述 `myVar = nil` 與 `self.myVar = nil`這兩者容易搞混之外，
property 的 dot 語法，又與 C 的 structure語法相同，在還不熟悉的狀況下，
很容易讓初學搞錯哪些是property，哪些又是屬於一個 structure。

例如，我們想要知道一個 view 的 x 座標是在哪裡，會寫出像
`self.view.frame.origin.x` 這種程式，就需要知道，`view` 是 `self` 的
property， `frame` 也是 `view` 的 property，但是 `x` 卻是 `origin` 這
個`CGPoint` 裡頭的變數，而 `origin` 也是 `frame` 這個 `CGRect`裡頭的變
數，但是初學的時候很容易搞混。

我們想要取得 x，可以寫成 `self.view.frame.origin.x` ，但想要設定 x的位
置，如果這麼寫：

``` objc
self.view.frame.orgin.x = 0.0;
```

程式會發生編譯錯誤。 `self.view.frame.origin.x` 其實會被編譯成
`[[self view] frame].origin.x` ，這沒問題，但是如果要改變 view 的frame，
我們還是要透過 `setFrame:` ，所以即使只是要改變 x座標的位置，我們還是
得要這麼寫：

``` objc
CGRect originalFrame = self.view.frame;
originalFrame.origin.x = 0.0;
self.view.frame = originalFrame;
```

其實這兩點要搞清楚並不困難，如果因為這些緣故而不使用 property
語法，實在有些因噎廢食，因為使用 property
語法，可以大幅精簡程式碼。而由於前述 Xcode 4.4 可以在宣告 property
之後，由 compiler 自動補上對應的成員變數的特性，也不致於會搞錯
`myVar = nil` 與 `self.myVar = nil` 的差別—因為我們並沒有宣告 `myVar`
，如果寫成前者，編譯時會產生錯誤。

`UIViewController` 的記憶體管理議題
-----------------------------------

這部份主要提到的是 iOS 6 之前的狀況，後來其實可以不用理會這部份的內容。

### Lazy Loading

由於 iOS 裝置的資源相較於 Desktop平台，並不充裕，為了在資源有限的裝置
上有效利用記憶體，以及讓啟動程式的速度變得更快，相較於Mac OS X，iOS 大
量使用 **Lazy Loading** ：*我們要去使用某個物件的時候，我們才去建立那
個物件* 。 [^1]

我們在開發 iOS 應用程式的時候，最常使用的 `UIViewController` ，就是使
用Lazy Loading 好例子。 `UIViewController`負責管理在應用程式中每個會用
到的畫面，最主要的屬性就是 `view`，當我們在透過 `alloc` 、 `init` 或
`initWithNibName:bundle:` 建立一個View Controller 的時候，並不會馬上建
立 view，而是當我們呼叫 `view`這個屬性的時候才會建立。我們以下面的程式
為例：

``` objc
// 建立 MyViewController 的 instance，這時候還沒有建立 view
MyViewController *controller = [[MyViewController alloc]
    initWithNibName:NSStringFromClass([MyViewController class]) bundle:nil];
// 在被加入到 navigation stack 的時候，會去呼叫 [controller view]
// 這時候 view 才被建立起來
[navigationController pushViewController:controller animated:YES];
[controller release];
```

用 Lazy Loading 的方式實作一個 getter的方式大致如下。在我們自己的程式
中，想要有效使用記憶體，我們也可以嘗試這麼寫。

``` objc
- (UIView *)view
{
    if (!_view) {
        _view = [[UIView alloc]
            initWithFrame:[UIScreen mainScreen].bounds];
    }
    return view;
}
```

不過，`UIViewController` 在還沒有 view，而要去建立 view的時候，會呼叫
的其實是 `loadView` 這個 method，在 view成功載入之後，則會呼叫
`viewDidLoad` 。我們雖然不知道蘋果到底是怎麼實作`UIViewController` ，
但不外乎類似這樣：

``` objc
- (UIView *)view
{
    if (!_view) {
        [self loadView];
        if (_view) {
            [self viewDidLoad];
        }
    }
    return view;
}
```

所以，如果你有天不小心寫出像下面的程式碼，就會進入無窮迴圈：因為呼叫
`[self view]` 的時候發現沒有 view，就會呼叫 `loadView` ，但
`loadView`又去呼叫 `[self view]` 。

``` objc
- (void)loadView
{
    [self view];
}
```

### 記憶體不足警告（Memory Warnings）

在 Desktop作業系統中，如果實體記憶體不足，應用程式使用的記憶體量，超過
實體記憶體的數量，這時候作業系統會自動將記憶體中的部分資料，存入磁碟的
虛擬記憶體（Virtaul Memory）當中，需要使用的時候，再從虛擬記憶體中載回
實體記憶體，Mac OS X就有這樣的機制。

iOS在發展之初到現在，都沒有虛擬記憶體，而是會在記憶體快要用完的時候，
對應用程式發出記憶體不足警告，要求釋放一些可以暫時不需要用到的物件，讓
應用程式可以有足夠的記憶體繼續運作。如果無視記憶體警告，繼續放任記憶體
用量成長，系統最後便會強制要求終止應用程式。

在記憶體不足的時候，除了會對 `UIApplication` 的 delegate 呼叫
`applicationDidReceiveMemoryWarning:` 之外，也會對系統中所有的
`UIViewController` 呼叫 `didReceiveMemoryWarning`。我們雖然知道要實作
這些method，但是一定會這麼一問：我們之所以會將資料載入到記憶體中，就代
表我們會用到這些資料，這些東西都是有用的，哪有應該釋放的道理？

我們不妨看看蘋果自己認為什麼是該釋放的物件。從 iOS 2 到 iOS 5，在發生
記憶體警告的時候，系統除了發出通知之外，還會多做一件事情，就是是放不在
最前景的view controller 的 view 物件。

所謂不在最前景的 view controller 像這樣：假如我們今天有一個 tab bar
controller，tab bar 裡頭有四個項目，對應到四個 view controller，但是其
實只會顯示一個，那麼，在發生記憶體警告的時候，其他三個view controller
的 view 就可以先放掉；在 navigation controller 的navigation stack 裡頭，
也只有最上面的 view controller的畫面需要顯示，我們也可以釋放其他 view
controller 的 view。這些 view controller 的 view 既然被釋放了，但是沒
關係，因為 `view` 屬性是透過 Lazy Loading 實作的，下次我們要切換到這些
畫面的時候，view 又會被重新載入。

因為 `UIViewController` 在 iOS 2 到 iOS 5是這麼實作的，所以我們需要注
意幾個重要事項：

#### 釋放 IBOutlet 的時機

如果我們將一個 `UIViewController` 的 view 放在 XIB檔案中載入，我們會將
像是按鈕等 UI 元件，透過 IBOutlet 連接到我們的 view controller 的成員
變數上，而不同於 Mac OS X，iOS 的 IBOutlet 是會被retain 起來的，所以我
們在不用到這些 IBOutlet的時候，也要釋放記憶體；而釋放記憶體的時機就不
只是寫在 dealloc裡頭，在收到記憶體警告，view 被釋放的時候，IBOutlet也
要一起釋放—要不然，到了下一次呼叫 loadView 的時候，所有的 IBOutlet又從
XIB 檔案中載入了一次，就造成了記憶體漏水。

在 iOS 2 的時候，由於系統並不會通知 view什麼時候被釋放，當時的作法是要
在我們的 `UIViewController` subclass 中override 掉 `setView:` 這個
method，如果傳入的參數是 nil，就代表 view被釋放。程式大致如下：

``` objc
- (void)setView:(UIView *)inView
{
    [super setView:inView];
    if (inView == ni) {
        self.button = nil; // 釋放 IBOutlet
        self.textView = nil;
    }
}
```

在 iOS 3 之後，蘋果則要求我們將釋放 IBOutlet 的相關程式碼，放在一個
template method 中，叫做 `viewDidUnload` 。我們會這麼寫：

``` objc
- (void)viewDidUnload
{
    [super viewDidUnload];
    self.button = nil;
    self.textView = nil;
}
```

請注意：我們在 `viewDidUnload` 中寫了與釋放 IBOutlet相關的程式碼，但是
記得在 dealloc 的時候還是要再做一次。

蘋果推出 iOS 6 時，或許是認為像是 iPhone 5這樣的裝置在可用資源上遠遠超
越過去的硬體，因此不再需要刻意釋放`UIViewController` 的 view 了，因此
即使我們實作了 `viewDidUnload`，也不會被呼叫到。如果你覺得蘋果過去這一
套機制仍然可取，或是你已經有一個做了幾年的iOS 專案，想要在 iOS 6上繼續
維持過去的記憶體管理策略，我們可以透過下面的範例，自己釋放 view以及呼
叫 `viewDidUnload` 。

``` objc
- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
    if (self.isViewLoaded && !self.view.window) {
        self.view = nil;
        if ([self respondsToSelector:@selector(viewDidUnload)]) {
            [self viewDidUnload];
        }
    }
}
```

#### `viewDidLoad` 不是 `UIViewController` 的 Initializer

雖然我們在開始使用某個 view controller 之前，一定會呼叫到一次
`viewDidLoad` ，我們也通常會在這個地方，做一些初始化這個 view
controller的事情，但是並不代表 `viewDidLoad` 這個 method 的定位就是用
來初始化`UIViewController` 。

由於 view controller 可能會被重複釋放與載入 view， `viewDidLoad`也會被
重複呼叫，如果我們只在這編寫初始化 view controller的程式碼的話，那麼，
當我們遇到記憶體警告，再回到這個 view controller之後，我們預期可以看到
之前離開這個 view controller的狀態，但是看到的卻是重設到初始狀態。所以，
在 view裡頭的內容被改變的時候，view controller 或許要隨時，或是在
`viewDidUnload` 被呼叫到的時候，記住 view 裡頭的狀態，而在
`viewDidLoad`的時候還原到 `viewDidUnload` 之前的狀態。

#### iOS 如何知道哪個 View Controller 位在最上層？

那麼，view controller自己怎麼知道自己位在最上層或不是，在收到記憶體警
告的時候，該不該將 view釋放呢？其實很簡單：view controller 被放到最上
層時，會被呼叫到`viewWillAppear:` 以及 `viewDidAppear:` ，離開最上層時，
會呼叫`viewWillDisappear:` 與 `viewDidDisappear:` 。

我們通常會在 view controller 的 view正要被顯示出來之前做一些事情，比方
說，如果我們的畫面中有一個 table view，我們或許希望每次被顯示出來的時
候，都更新一下 table view的內容，於是在我們自己的 view controller
subclass 裡，override 了`viewWillAppear:` ，呼叫了一次 table view 的
`reloadData`。但我們在做這樣的事情的時候，千萬要記住，必須保留 super的
實作，這樣才能夠保證這套釋放與還原 view 的機制運作正常。

假如我們自己要寫一套像是 `UINavigationController` 或是
`UITabBarController` 這類的 class，我們也需要記住呼叫
`viewWillAppear:`這些 method。簡單範例如下：

``` objc
- (void)pushViewController:(UIViewController *)inController
                  animated:(BOOL)animated
{
    [inController view];
    [inController viewWillAppear:animated];
    [[viewControllers lastObject] viewWillDisappear:animated];
    [[[self.contentView subviews] lastObject] removeFromSuperView];
    inController.view.frame = self.contentView.bounds;
    [self.contentView addSubview:inController.view];
    [inController viewDidAppear:animated];
    [[viewControllers lastObject] viewDidDisappear:animated];
    [viewControllers addObject:inController];
}
```

[^1]: 也可以參見 Wikipedia 上的說明 <http://en.wikipedia.org/wiki/Lazy_loading>
