什麼時候應該要使用 Category
---------------------------

如果想要擴充某個 class 功能，增加新的成員變數與 method，我們又沒有這個
class 的程式碼，正規作法就是繼承、建立新的subclass。那，我們需要在不用
繼承，就直接增加 method這種作法的重要理由，就是我們想要擴充的 class 很
難繼承。

我能想到的，大概有幾種狀況：

1. Foundation 物件
2. 用 Factory Method Pattern 實作的物件
3. Singleton 物件
4. 在專案中出現次數已經多不勝數 的物件。

### Foundation 物件

Foundation 裡頭的基本物件，像是 NSString、NSArray、NSDictionary 等
Class 的底層實作，除了可以透過 Objective-C的介面呼叫之外，也可以透過另
外一個 C 的介面，叫做 Core Foundation，像NSString 其實會對應到 Core
Foundation 裡頭的 CFStringRef，NSArray 對應到 CFArrayRef，而你甚至可以
直接把 Foundation 物件 cast成 Core Foundation 的型別，當你遇到一個需要
傳入 CFStringRef 的function的時候，只要建立 NSString 然後 cast 成
CFStringRef 傳入就可以了。

所以，當你使用 alloc、init 產生一個 Foundation物件的時候，其實會得到一
個同時有 Foundation 與 Core Foundation 實作的subclass，而實際產生出來
的物件，往往與你的認知會有很大的差距，例如，我們以為 NSMutableString
繼承自 NSString，但建立 NSString ，呼叫alloc、init 的時候，我們真
正拿到的是 \_\_NSCFConstantString，而建立 NSMutableString ，拿到
\_\_NSCFString，而 \_\_NSCFConstantString 其實繼承自 \_\_NSCFString！

我們來寫點程式檢查 Foundation 物件其實屬於哪些 Class：

``` objc
#define CLS(x) NSStringFromClass([x class])
NSLog(@"NSString:%@", CLS([NSString string]));
NSLog(@"NSMutableString:%@", CLS([NSMutableString string]));
NSLog(@"NSNumber:%@", CLS([NSNumber numberWithInt:1]));
#undef CLS
```

執行結果： :

    NSString:__NSCFConstantString
    NSMutableString:__NSCFString
    NSNumber:__NSCFNumber

因此，當我們嘗試建立 Foundation 物件的 subclass 之後，像是繼承
NSString，建立我們自己的 MyString，假如果我們並沒有 override
原本關於建立 instance 的 method，我們也不能保證，建立出來的就是
MyString 的 instance。

### 用 Factory Method Pattern 實作的物件

Wikipedia 上對 Factory Method Pattern 的解釋是：

> ...the factory method pattern is a creational pattern which uses
> factory methods to deal with the problem of creating objects without
> specifying the exact class of object that will be created.

翻譯成中文：Factory Method Pattern 是一套用來解決不用特別指定是哪個
class，就可以建立物件的方法。比方說，某個 class底下，其實有一堆
subclass，但對外部來說並不需要確實知道這些 subclass而是只要 **對最上層
的class，輸入指定的條件，就會從挑選一個符合指定條件的 subclass、建立
instance 回傳** 。

在 UIKit 中，UIButton 就是個好例子。我們在建立 UIButton 物件的時候，並
不是呼叫 `init` 或是 `initWithFrame:`，而是呼叫 UIButton 的 class
method：`buttonWithType:`，透過傳遞按鈕的 type建立按鈕物件。在大多數狀
況下，會回傳 UIButton 物件，但假如我們傳入的type 是
`UIButtonTypeRoundedRect`，卻會回傳繼承自 UIButton 的
`UIRoundedRectButton`。

檢查一下：

``` objc
#define CLS(x) NSStringFromClass([x class])
NSLog(@"UIButtonTypeCustom %@",
    CLS([UIButton buttonWithType:UIButtonTypeCustom]));
NSLog(@"UsIButtonTypeRoundedRect %@",
    CLS([UIButton buttonWithType:UIButtonTypeRoundedRect]));
#undef CLS
```

輸出結果： :

    UIButtonTypeCustom UIButton
    UIButtonTypeRoundedRect UIRoundedRectButton

我們想要擴充 `UIButton`，但拿到的卻是 `UIRoundedRectButton`，而
`UIRoundedRectButton` 卻無法繼承，因為這個物件不在公開的 header中，我
們也不能夠保證以後傳入 `UIButtonTypeRoundedRect` 就一定會拿到
`UIRoundedRectButton`。如此一來，就造成我們難以繼承 `UIButton`。

或這麼說：假使今天我們的需求是想要改動某個上層的 class，讓底下所有的
subclass 也都增加了一個新的 method，我們又無法改動這個上層 class的程式，
就會採用 category。比方說，我們今天希望所有的 `UIViewController`都有一
個新 method，如此我們整個應用程式中每個 `UIViewController` 的subclass
都可以呼叫這個 method，但，我們就是無法改動`UIViewController`。

### Singleton 物件

**Singleton** 物件是指： **某個 class 只有、也只該有一個instance，每次
都只對這個 instance 操作，而不是建立新的 instance** 。像UIApplication、
NSUserDefault、NSNotificationCenter 以及 Mac OS X上的 NSWorkSpace 等，
都採用 singleton 設計。

之所以說 singleton 物件很難繼承，我們先來看怎麼實作singleton：我們會有
一個 static的物件，然後每次都回傳這個物件。宣告部分如下：

``` objc
@interface MyClass : NSObject
+ (MyClass *)sharedInstace;
@end
```

實作部分：

``` objc
static MyClass *sharedInstace = nil;

@implementation MyClass
+ (MyClass *)sharedInstace
{
    return sharedInstace ?
           sharedInstace :
           (sharedInstace = [[MyClass alloc] init]);
}
@end
```

其實現在 Singleton 大多會使用 GCD 的 `dispatch_once` 實作，但是在我們還
沒有提到 GCD 之前，我們先使用這樣的寫法。我們會在討論 Threading 的時候
繼續討論 GCD，至於用 GCD 實作 Singleton 的細節，請參見
[再談 Singleton](../design_patterns/singleton.md) 這一章。

我們如果 subclass 了 MyClass，卻沒有 override 掉`sharedInstace`，那麼，
`sharedInstace` 回傳的還是 MyClass 的 singleton instance。而想要
override 掉 `sharedInstace` 又不見得這麼簡單，因為這個method 裡頭很可
能又做了許多其他事情，很可能會把一些 initiailize時該做的事情，反而放在
這邊做（這不是很好的作法，但就是可能發生）。例如MyClass 可能這麼寫：

``` objc
+ (MyClass *)sharedInstace
{
    if (!sharedInstace) {
        sharedInstace = [[MyClass alloc] init];
        [sharedInstace doSomething];
        [sharedInstace doAnotherThine];
    }
    return sharedInstace;
}
```

如果我們並沒有 MyClass 的程式碼，這個 class 是在其他的 library 或是
framework 中，我們直接 override 了`sharedInstace`，就很有可能有事情沒
做，而產生不符合預期的結果。

### 在專案中出現次數已經多不勝數

隨著專案不斷成長，某些 class已經頻繁使用到了到處都是，而我們現在需求改
變，必須要增加新的method，我們卻也沒有力氣可以把所有用到的地方統統換成
新的subclass。Category 就是解決這種狀況的救星。
