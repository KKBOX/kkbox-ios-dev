Selector 有什麼用途？
---------------------

我們會在幾個地方應用 selector：

### Target/Action pattern

Selector 的主要用途，就是實作 target/action。相信你應該試過在 Xcode中
建立一個新專案之後，可能在 Interface Builder 中建立了一個 UIButton或是
NSButton 物件，然後將按鈕連接到 controller 中宣告成 IBAction 的method
上，這時候，我們的 Controller 就是 Button 的 target，而要求controller
執行的 method，就叫做 action。

我們在 Interface Builder裡頭做的事情，也可以透過程式碼做到。而如果我們
想要設計一套系統 Framework裡頭所沒有的客製 UI 元件，第一步就是要了解怎
麼實作 target/action。

在 UIKit 中的 Target/Action 稍微複雜一些，因為同一個按鈕可以一次連接好
幾個 target 與 action，我們在這邊使用 AppKit 示範—在 Mac 上，一次只會指
定單一的 target 與 action。如果想要產生一個按鈕或是其他的 custom
control，我們會繼承自 NSView，然後建立兩個成員變數：target 與 action，
action 是一個 selector。

``` objc
@interface MyButton : NSView
{
    id target;
    SEL action;
}
@property (assign) IBOutlet id target;
@property (assign) SEL action;
@end

@implementation MyButton
- (void)mouseDown:(NSEvent *)e
{
    [super mouseDown:e];
    [target performSelector:action withObject:self];
}
@synthesize target, action;
@end
```

我們在這邊將 target 的型別設定為 id，代表的是任意 Objective-C物件的指
標，如同前面提到，Controller 到底是什麼class，在這邊並不重要，而且我們
也不該將 target 的 class寫死，因為如此一來，就變成只有某些 Controller
才能使用這個按鈕。

我們接著在 `mouseDown:` 中，要求 target 執行之前傳入的 action，由於
selector 是字串，是可以傳遞的參數，所以也就可以成為按鈕的成員變數。

我們接下來也可以使用程式碼連結 target 與 action，在 Controller
的程式中，只要這麼寫即可：

``` objc
[(MyButton *)button setTarget:self];
[(MyButton *)button setAction:@selector(clickAction:)];
```

把要做什麼事情當做參數傳遞，每個語言都有不同的作法。 Objective-C用的是
拿字串來尋找對應的實作 function 指標，在 C語言裡頭就會直接傳遞指標，一
些更高階的語言或著是把一段程式碼當做是字串傳遞，要使用的時候再去
evaluate這段程式碼字串，或是一段程式碼本身就是一個物件，所以可以把程式
碼當做物件傳遞，我們稱之為「匿名函式」（Anonymous Function ），現在
Objective-C 也有匿名函式，叫做block，不過，對這個 1983年誕生的語言來說，
這是很晚近才有的功能，我們會稍晚討論。

### 檢查 method 是否存在

前面提到，我們有可能會呼叫到並不存在的method，如果這麼做就會產生錯誤。
但我們有時候會遇到的狀況是：我們並不確定某些method 到底有沒有實作，如
果有，就呼叫，如果沒有，就略過或是使用其他的method。

這種狀況最常遇到的就是顧及向下相容。比方說，在 iOS 4 之後，才開始支援
Retina Display，我們在繪圖的 code中要決定現在應該繪製怎樣精細程度的圖
片，需要知道目前用的是傳統的一倍品質，還是Retina Display 的兩倍品質，
就要去問 `UIScreen` 的 `scale`屬性。但是，當我們開始支援 iOS 4 的時候，
可能還要顧及 iOS 3的使用者，導致我們不能夠貿然直接呼叫 `scale`（當然，
如果你的應用程式都只支援最新版本的作業系統，那是再幸福不過的事），而是
要去檢查這個屬性是否存在，如果沒有，就代表使用者的作業系統是iOS 4之前
的版本，我們只需要提供一倍品質的圖片就可以了。在未來，只要遇到向下相容，
我們就還是得處理這樣的狀況。

另外，雖然蘋果只允許 iOS上面的應用程式只能夠是單一的執行檔，不能夠在執
行時載入其他的binary，但是在 Mac OS X 上面卻可以載入 loadable bundle，
或是在應用程式中放置 private framework，一個物件的某些 method可以不在
主程式中，而是在 plug-in 中實作。我們也要做這樣的檢查。

檢查某個物件是否實作了某個 method，只要呼叫 `respondsToSelector:`
就可以了：

``` objc
BOOL scale = 1.0;
if ([[UIScreen mainScreen] respondsToSelector:@selector(scale)]) {
    scale = [UIScreen mainScreen].scale;
}
```

在其他程式語言中，也需要這樣檢查 method 是否存在嗎？在 Ruby 語言中，有
類似的 `respond_to?` 語法，至於 Python，我們或著可以用 `dir`這個
funciton 檢查某個物件的全部 attribute 中是否存在對應到某個 method的
key，但是更常見的作法就是使用 try…catch 語法，如果遇到某個 method可能
不存在，就包在 try…catch 的 block 中，像是：

``` python
try:
    myObject.doSomething()
except Exception, e:
    print "The method does not exist."
```

在 Objective-C 中，同樣也有 try…catch 語法，在許多語言中，善用
try…catch，也可以將程式寫得清楚有條理，但是我們並不鼓勵在 Objective-C
語言中使用。原因與 Objective-C 的記憶體管理機制有關，如果大量使用
try…catch，會導致記憶體漏水（Memory Leak）。

Objective-C 本身並不算有記憶體回收機制（Garbage Collection，以下簡稱GC）
的語言，雖然在 Mac OS X 10.5 的時代，蘋果嘗試在 Objective-C 上實作GC，
但是成果實在不甚理想，如果貿然在 Mac OS X 上大量使用GC，實際運作會有嚴
重的記憶體漏水問題；蘋果在推出 iOS之後，也不敢將這套機制用在行動裝置上，
而是在 iOS 5 時放棄在 runtime管理記憶體，而是推出 ARC（Automatic
Reference Counter），在 compile time時決定什麼時候應該釋放記憶體。

由於傳統的 Objective-C 記憶體管理大量使用一套叫做 auto-release的機制—
雖然說是auto，其實也沒多自動，頂多算是半自動—將一些應該要釋放的物件延
遲釋放，在這一輪runloop 中先不釋放，而是到了下一輪 runloop開始時才釋放
這些記憶體。如果使用 try…catch 捕捉例外錯誤，就會跳出原本的runloop，而
導致應該釋放的記憶體沒被釋放。

我們接下來還會在
[記憶體管理 Part 1](memory_management_part_1/README.md) 與
[Responder](responder/README.md) 討論這個部分。

### Timer

NSObject 除了 `performSelector:` 這個 method 之外，同樣以
performSelector 開頭的，還有好幾組 API 可以呼叫，例如
`-performSelector:withObject:afterDelay:`
，就可以讓我們在一定的秒數之後，才要求某個 method 執行。

``` objc
[self performSelector:@selector(doSomething) withObject:nil afterDelay:1.0];
```

如果時間還不到已經預定要執行的時間，method還沒有執行，我們也可以反悔，
取消剛才預定要執行的 method，只要呼叫
`cancelPreviousPerformRequestsWithTarget:` 即可。如以下範例：

``` objc
[NSObject cancelPreviousPerformRequestsWithTarget:self];
```

`performSelector:withObject:afterDelay:` 的效果相當於產生 `NSTimer`物
件，當我們想要延遲呼叫某個method，或是要某件事情重複執行，都可以透過建
立 `NSTimer`物件達成，要使用 timer，我們也必須使用 selector 語法。

我們先定義一個 timer 要做的事情：

``` objc
- (void)doSomething:(NSTimer *)timer
{
    // Do something
}
```

然後透過 `doSomething:` 的 selector 建立 timer

``` objc
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                          target:someObject
                          selector:@selector(doSomething:)
                          userInfo:nil
                          repeats:YES];
```

除了透過指定 target 與 selector 之外，還可以透過指定 `NSInvocation`呼
叫建立 `NSTimer` 物件；`NSInvocation` 其實就是將 target/action以及這個
action 中要傳遞給 target 的參數這三者，再包裝成一個物件。呼叫的method
是 `scheduledTimerWithTimeInterval:invocation:repeats:`。

透過建立 `NSInvocation` 物件建立 timer 的方式如下。

``` objc
NSMethodSignature *sig = [MyClass instanceMethodSignatureForSelector:
                                  @selector(doSomething:)];
NSInvocation *invocation = [NSInvocation invocationWithMethodSignature:sig];
[invocation setTarget:someObject];
[invocation setSelector:@selector(doSomething:)];
[invocation setArgument:&anArgument atIndex:2];
NSTimer *timer = [NSTimer scheduledTimerWithTimeInterval:1.0
                          invocation:invocation
                          repeats:YES];
```

請注意，在呼叫 `NSInvocation` 的 `setArgument:atIndex`的時候，我們要傳
遞的參數，要從 2 開始，因為在這邊我們要想成，這是給`objc_msgSend` 呼叫
用的參數，在 0 的參數是物件的 self，位在 1 的則是selector。

### 接收 NSNotification

我們稍晚才會討論 NSNotification 以及 NSNotificationCenter，不過在
這邊先簡單提到：如果我們要接收 NSNotification，我們也要在開始訂閱通
知的時候，指定要由哪個 selector 處理這個通知。詳見 [Notification Center](notification_center/README.md) 這一章。

### 在某個 Thread 執行 method

除了已經提到的 `-performSelector:withObject:afterDelay:` 之外，
`NSObject` 還有好幾個 method，是讓指定的 selector 丟到某個 Thread執行，
包括：

-   `-performSelectorOnMainThread:withObject:waitUntilDone:modes:`
-   `-performSelectorOnMainThread:withObject:waitUntilDone:`
-   `-performSelector:onThread:withObject:waitUntilDone:modes:`
-   `-performSelector:onThread:withObject:waitUntilDone:`
-   `-performSelectorInBackground:withObject:`

假如有一件事情—在這邊叫做doSomething—會執行太久，我們可以將這件事情丟
到背景，也就是另外建立一條Thread 執行：

``` objc
[self performSelectorInBackground:@selector(doSomething) withObject:nil];
```

注意，在背景執行時，這個 method 的內部需要建立自己的 Auto-Release Pool。

執行完畢後，我們可以透過
`-performSelectorOnMainThread:withObjectwaitUntilDone:` ，通知主
Thread我們已經把事情做完了。像是，如果我們要轉換一個比較大的檔案，就可
以在背景實際轉檔，轉完之後，再告訴主Thread，在 UI 上跳出提示視窗，提示
使用者已經轉檔完畢。

``` objc
- (void)doSomthing
{
    @autoreleasepool {
        // Do something here.
        [self performSelectorOnMainThread:@selector(doAnotherThing)
              withObject:nil
              waitUntilDone:NO];
    }
}
```

### Array 排序

我們今天想要對 `NSArray` 做排序，就得要告訴這個 Array怎樣比較裡頭每個
東西的大小，所以我們需要把怎麼比較大小這件事情傳遞到array 上。 Cocoa
Framework 提供三種方式排序Array，我們可以把怎麼比大小寫成 C Function，
然後傳遞 C Function的指標，現在也可以傳遞 Block，而如果 Array 裡頭的物
件有負責比較大小的method 的話，我們也可以透過 selector 指定要用哪個
method 排序。

`NSString`、`NSDate`、`NSNumber` 以及 `NSIndexPath`，都提供
`compare:`這個 method，假如有一個 array 裡頭都是字串的話，我們就可以使
用`compare:` 排序， `NSString` 用來比較大小順序的 method與選項（像是是
否忽略大小寫，字串中如果出現數字，是否要以數字的大小排列而不是只照字元
順序…等等），其中最常用的，該是`localizedCompare:` ，這個 method會參考
目前使用者所在的系統語系決定排序方式，像是簡體中文語系下用拼音排序，繁
體中文語系下用筆劃排序…等等。

我們使用 `sortedArrayUsingSelector:` 產生重新排序的新 Array，如果是
`NSMutableArray`，則可以呼叫 `sortUsingSelector:`

``` objc
NSArray *sortedArray = [anArray sortedArrayUsingSelector:
                                 @selector(localizedCompare:)];
```

我們也可以透過傳遞 selector，要求 Array 裡頭每一個物件都執行一次指定的
method。

``` objc
[anArray makeObjectsPerformSelector:@selector(doSomething)];
```

### 代替 if...else 與 switch…case

因為 selector 其實就是 C 字串，除了可以當做參數傳遞之外，也可以放在
array 或是 dictionary 裡頭。有的時候，如果你覺得寫一堆 if…else 或是
switch…case 太過冗贅，例如，原本我們可能這麼寫：

``` objc
switch(condition) {
    case 0:
        [object doSomething];
        break;
    case 1:
        [object doAnotherThing];
        break;
    default:
        break;
}
```

如果沒有什麼會超過邊界的問題的話，其實可以考慮搭配 Xcode 4.4 之後所提供
的 literal 新寫法 [^1] ，看起來就精簡一些。

``` objc
[object performSelector:NSSelectorFromString(@[@"doSomething",
    @"doAnotherThing"][condition])];
```

我們可以使用 `NSStringFromSelector` ，將 selector 轉換成 `NSString`
，反之，也可以使用 `NSSelectorFromString` 將 `NSString` 轉成 selector。

### …呼叫 Private API

Objective-C 裡頭其實沒有真正所謂的 private method，一個物件實作了那些
method，即使沒有 import 對應的header，我們都呼叫得到。系統裡頭許多原本
就內建的 class，有一些 header並沒有宣告的 method，但是從一些相關網站或
是其他管道，我們就是知道有這些method，先不管究竟是什麼原因，我們有的時
候就是想要呼叫看看，這時候我們往往會用`performSelector:` 呼叫。原因也
很簡單：因為我們沒有 header。

但我們並不建議做這樣的事情：今天一個 method 沒有被放在 header裡頭，就
代表在作業系統改版的時候，系統可能把整個底層的實作換掉，這個method 可
能就此消失，而造成系統升級之後，因為呼叫不存在的 method而造成應用程式
crash。而如果你打算寫一套 iOS 應用程式，在 AppSrtore上架販售，蘋果的審
查過程中就會拒絕使用 private API 的軟體。


[^1]: 參見 <http://developer.apple.com/library/mac/#documentation/DeveloperTools/Conceptual/WhatsNewXcode/Articles/xcode_4_4.html>
