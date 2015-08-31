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


相關閱讀
--------

- WWDC 2013 Fixing Memory Issues
