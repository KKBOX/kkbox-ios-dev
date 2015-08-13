Category
========

基本定義
--------

一句話解釋什麼是 category：**不用繼承物件，就直接增加新的 method，或是
override 原本的 method**。

前一章提到，Objective-C 語言中，每個 class 有哪些 method，都是在 runtime
時加入的，我們可以透過 runtime 提供的一個叫做 `class_addMethod` 的
function，加入對應某個 selector 的實作。而在 runtime 時想要加入新的
method，使用 category 會是更容易理解與寫作的方法，因為可以使用與宣告
class 時差不多的語法，同時也以一般實作 method 的方式，實作我們要加入的
method。

什麼時候應該要使用 Category
---------------------------

如果想要擴充某個 class 功能，增加新的成員變數與 method，我們又沒有這個
class 的程式碼，正規作法就是繼承、建立新的
subclass。那，我們需要在不用繼承，就直接增加 method
這種作法的重要理由，就是我們想要擴充的 class 很難繼承。

我能想到的，大概有幾種狀況：1. Foundation 物件、2. 用 Abstract Factory
Pattern 實作的物件、3. Singleton 物件、4.
在專案中出現次數已經多不勝數的物件。

### Foundation 物件

Foundation 裡頭的基本物件，像是 `NSString`、`NSArray`、`NSDictionary`
等等 Class 的底層實作，除了可以透過 Objective-C
的介面呼叫之外，也可以透過另外一個 C 的介面，叫做 Core Foundation，像
`NSString` 其實會對應到 Core Foundation 裡頭的 `CFStringRef`，`NSArray`
對應到 `CFArrayRef`，而你甚至可以直接把 Foundation 物件 cast 成 Core
Foundation 的型別，當你遇到一個需要傳入 `CFStringRef` 的 function
的時候，只要建立 `NSString` 然後 cast 成 `CFStringRef` 傳入就可以了。

所以，當你使用 alloc、init 產生一個 Foundation
物件的時候，其實會得到一個同時有 Foundation 與 Core Foundation 實作的
subclass，而實際產生出來的物件，往往與你的認知會有很大的差距，例如，我們以為
`NSMutableString` 繼承自 `NSString`，但建立 `NSString` ，呼叫
alloc、init 的時候，我們真正拿到的是 `__NSCFConstantString`，而建立
`NSMutableString`，拿到 `__NSCFString`，而 `__NSCFConstantString`
其實繼承自 `__NSCFString`！

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
`NSString`，建立我們自己的 `MyString`，假如果我們並沒有 override
原本關於建立 instance 的 method，我們也不能保證，建立出來的就是
`MyString` 的 instance。

### 用 Abstract Factory Pattern 實作的物件

在 Design Pattern 中所謂的 Abstract Factory Pattern，就是在某個 class
底下，其實有一堆 subclass，但是這樣的狀況往往會讓人不知道底下這一堆
subclass 個別的用途，所以在最上層提供了一個介面，我們只要 *對最上層的
class，輸入指定的條件，就會從挑選一個符合指定條件的 subclass、建立
instance 回傳* 。

在 UIKit 中，`UIButton` 就是個好例子。我們在建立 `UIButton`
物件的時候，並不是呼叫 `init` 或是 `initWithFrame:`，而是呼叫 `UIButton`
的 class method：`buttonWithType:`，透過傳遞按鈕的 type
建立按鈕物件。在大多數狀況下，會回傳 `UIButton` 物件，但假如我們傳入的
type 是 `UIButtonTypeRoundedRect`，卻會回傳繼承自 `UIButton` 的
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
`UIRoundedRectButton` 卻無法繼承，因為這個物件不在公開的 header
中，我們也不能夠保證以後傳入 `UIButtonTypeRoundedRect` 就一定會拿到
`UIRoundedRectButton`。如此一來，就造成我們難以繼承 `UIButton`。

或這麼說：假使今天我們的需求是想要改動某個上層的 class，讓底下所有的
subclass 也都增加了一個新的 method，我們又無法改動這個上層 class
的程式，就會採用 category。比方說，我們今天希望所有的 `UIViewController`
都有一個新 method，如此我們整個應用程式中每個 `UIViewController` 的
subclass 都可以呼叫這個 method，但，我們就是無法改動
`UIViewController`。

### Singleton 物件

*Singleton* 物件是指： *某個 class 只有、也只該有一個
instance，每次都只對這個 instance 操作，而不是建立新的 instance* 。像
`UIApplication`、`NSUserDefault`、`NSNotificationCenter` 以及 Mac OS X
上的 `NSWorkSpace` 等，都採用 singleton 設計。

之所以說 singleton 物件很難繼承，我們先來看怎麼實作
singleton：我們會有一個 static
的物件，然後每次都回傳這個物件。宣告部分如下：

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

我們如果 subclass 了 MyClass，卻沒有 override 掉
`sharedInstace`，那麼，`sharedInstace` 回傳的還是 MyClass 的 singleton
instance。而想要 override 掉 `sharedInstace` 又不見得這麼簡單，因為這個
method 裡頭很可能又做了許多其他事情，很可能會把一些 initiailize
時該做的事情，反而放在這邊做（這不是很好的作法，但就是可能發生）。例如
MyClass 可能這麼寫：

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
framework 中，我們直接 override 了
`sharedInstace`，就很有可能有事情沒做，而產生不符合預期的結果。

### 在專案中出現次數已經多不勝數

隨著專案不斷成長，某些 class
已經頻繁使用到了到處都是，而我們現在需求改變，必須要增加新的
method，我們卻也沒有力氣可以把所有用到的地方統統換成新的
subclass。Category 就是解決這種狀況的救星。

實作 Category
-------------

Category 的語法很簡單，一樣是用 @interface 關鍵字宣告 header，在
@implementation 與 @end 關鍵字當中的範圍是實作，然後在原本的 class
名稱後面，用中括弧表示新增的 category 名稱。

舉例來說，我們今天雖然寫的是 Objective-C 語言，但是想要變得更像 Small
Talk 一點，所以我們不想用 `NSLog`
印出某個物件的資料，而是每個物件都有個把自己印出來的 method，所以我們對
NSObject 建立了一個叫做 SmallTalkish 的 category。

``` objc
@interface NSObject (SmallTalish)
- (void)printNl;
@end

@implementation NSObject (SmallTalish)
- (void)printNl
{
    NSLog(@"%@", self);
}
@end
```

如此一來，每個物件都增加了 `printNl` 這個 method。可以這麼呼叫：

``` objc
[myObject printNl];
```

前一章提到，我們在排序一個裡頭都是字串的 Array 的時候，可以呼叫
`localizedCompare:`，但，假如我們希望所有的字串都一定要用中文筆劃順序排序，我們可以寫一個自己的
method，例如 `strokeCompare:`。

``` objc
@interface NSString (CustomCompare)
- (NSComparisonResult)strokeCompare:(NSString *)anotherString;
@end

@implementation NSString (CustomCompare)
- (NSComparisonResult)strokeCompare:(NSString *)anotherString
{
    NSLocale *strokeSortingLocale = [[[NSLocale alloc]
              initWithLocaleIdentifier:@"zh@collation=stroke"]
              autorelease];
    return [self compare:anotherString
                 options:0
                   range:NSMakeRange(0, [self length])
                  locale:strokeSortingLocale];
}
@end
```

在存檔的時候，檔名的慣例是原本的 class 名稱加上 category
的名稱，中間用加號連接，以我們剛剛建立的 CustomCompare
為例，存檔時就要存成 NSString+CustomCompare.h 以及
NSString+CustomCompare.m。

Category 還可以有什麼用途？
---------------------------

除了幫原有的 class 增加新 method，我們也會在幾種狀況下使用 category。

### 將一個很大的 Class 切成幾個部分

由於我們可以在建立 class 之後，繼續透過 category 增加
method，所以，假如一個 class 很大，裡頭有數十個
method，實作上千行，我們就可以考慮將這個 class 的 method 拆分成若干個
category，讓整個 class 的實作分開在不同的檔案中，方便知道某一群的 method
屬於什麼用途，也方便日後維護。

切開一個很大的 class 可以收到的好處包括：

#### 跨專案

如果你手上同時有好幾個專案，我們在進行專案的時候，由於之前寫的程式碼可以重複使用，造成每個專案可能共用同一個
class，但是每個專案又不見得都會用到這個 class
裡頭全部的實作，我們就可以考慮將只屬於某個專案的實作，拆分到一個
category 中。

#### 跨平台

做為寫 Objective-C
語言的工程師，我們非常有可能會遇到跨平台開發的需求，如果我們某段程式碼只有用到
Mac OS X 與 iOS 都有的 library 與 framework 的話，我們的程式就可以同時在
Mac OS X 與 iOS 使用。當我們打算在 Mac OS X 與 iOS 共用同一個
class，我們就可以考慮將跨平台的部份與平台相依的部份拆開，將只屬於某個平台的部份拆成另外一個
category，以蘋果自己的例子來說，在 Mac OS X 與 iOS 上都有
`NSString`，但由於兩個平台在繪圖方面的實作有所不同，所以在繪製字串的部份，就被拆分到
`NSStringDrawing` 與 `UIStringDrawing` 這些 category 中。

### 替換原本的實作

由於一個 class 有哪些 method，是在 runtime
時加入的，所以除了可以加入新的 method 之外，假如我們嘗試再加入一個
selector 與已經存在的 method 名稱相同的實作，我們可以把已經存在的 method
的實作，換成我們要加入的實作。這麼做在 Objective-C
語言中是完全合法的，如果 category 裡頭出現了名稱相同的 method，compiler
還是容許編譯成功，只會跳出簡單的警告訊息。

在實務上，這麼做卻非常危險，假如我們自己寫了一個
class，我們又另外寫了一個 category 置換其中的
method，當我們日後想要修改這個 method 的內容，很容易忽略在 category
中的同名 method，結果就是不管我們怎麼改動原本 method
裡頭的程式，結果卻是什麼改變都沒有。

我自己曾經犯過一個低級錯誤：在開發時我建立了另外一個 git
分支，在新分支中，我覺得某個 class 太大，於是將部分 method
拆到了另外一個 category 中，但是開發主線卻又在修改這個
class，結果造成合併分支的時候，就變成原本的 class 與 category
中出現了相同的 method，花了半天的時間才找到問題出在哪裡。

除了某一個 category 中可以出現與原本 class 中名稱相同的
method，我們甚至可以在好幾個 category 中，都出現名稱相同的
method，哪一個 category 在執行時被最後載入，就會變成是這個 category
中的實作。那麼，如果有多個 category，我們怎麼知道哪一個 category
會被最後載入呢？Objective-C runtime 並不保證 category
的載入順序，所以我們必須嚴格避免寫出這種程式。

Extensions
----------

Objective-C 語言中有一項叫做 extensions 的設計，也可以用來拆分一個很大的
class，語法與 category 非常相似，但是不太一樣。在語法上，extensions
像是一個沒有名字的 category，而 extensions 定義的 method，需要放在原本的
class 實作中。以下是一個使用 extensions 的例子：

``` objc
@interface MyClass : NSObject
@end

@interface MyClass()
- (void)doSomthing;
@end

@implementation MyClass
- (void)doSomthing
{
}
@end
```

在 `@interface MyClass ()`
這段宣告中，我們並沒有在括弧中定義任何名稱，接著，`doSomthing`
又是直接在 `MyClass` 中實作。 extensions 可以有幾個用途：

### 拆分 Header

如果我們就是打算實作一個很大的 class，但是覺得 header
裡頭已經列出了太多的 method，我們可以將一部分 method 搬到 extensions
這邊的定義。

### 管理 Private Methods

這其實是更常見的用途。我們在寫一個 class 的時候，內部有一些 method
不需要、我們也不想要放在 public header 中，但是如果不將這些 method 放在
header 裡頭，又會出現一個困擾：在 Xcode 4.3 之前，如果這些 private
method 在程式碼中不放在其他 method 前面，其他的 method 在呼叫這些 method
的時候，compiler
會不斷跳出警告，而這種無關緊要的警告一多，我們往往會忽視真正重要的警告。[^1]

想要避免這些警告，要不就是把 private method
都放在最前面，但這樣並不能完全解決問題，因為 private method
之間也會相互呼叫，花時間確認每個 method
之間的呼叫順序並不是很經濟的事；要不就是都用 `performSelector:`
呼叫，這樣問題更大，就像前面提到，在 method 改名、呼叫 refactoring
工具的時候，這樣非常危險。

蘋果提供的建議是，我們在 .m 或 .mm 檔案開頭的地方宣告一個 extensions，將
private method 都放在這個地方，如此一來，其他 method 就可以找到 private
method 的宣告。在 Xcode 4 所提供的 file template 中，如果你選擇建立一個
`UIViewController` 的 subclass，就可以看到在 .m
檔案的最前面，幫你預留了一塊 extensions 的宣告。

Category 是否可以增加新的成員變數或屬性？
-----------------------------------------

因為 Objective-C 物件會被編譯成 C 的 structure，我們雖然可以在 category
中增加新的 method，但是我們卻不能夠增加新的成員變數。

在 Mac OS X 10.6 與 iOS 4 之後，蘋果提出一套叫做 Associated Objects
的辦法，讓我們可以在 category 中增加新的
getter/setter，觀念差不多是：既然我們可以用一張表格記錄一個 class 有哪些
method，那，我們不就也可以另外建一張表格，記錄有哪些物件與這個 class
相關？

要使用 Associated Objects，我們需要匯入 `objc/runtime.h`，然後呼叫
`objc_setAssociatedObject` 建立 setter，用 `getAssociatedObject` 建立
getter，呼叫時要傳入：我們要讓哪個物件與哪個物件之間建立關連，關連時使用的是那一個
key（型別為 C 字串）。在以下的範例中，我們在 `MyCategory` 這個 category
裡，增加一個叫做 myVar 的 property。

``` objc
#import <objc/runtime.h>

@interface MyClass(MyCategory)
@property (retain, nonatomic) NSString *myVar;
@end

@implementation MyClass
- (void)setMyVar:(NSString *)inMyVar
{
    objc_setAssociatedObject(self, "myVar",
           inMyVar, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (NSString *)myVar
{
    return objc_getAssociatedObject(self, "myVar");
}
@end
```

在 `setMyVar:` 中呼叫 `objc_setAssociatedObject` 時，所最後一個參數
`OBJC_ASSOCIATION_RETAIN_NONATOMIC`，是用來決定要用哪一種記憶體管理策略，管理我們傳入的參數，在我們的例子中，我們傳入的是
`NSString`，是一個 Objective-C 物件，所以我們必須要 retain
起來。這邊可以傳入的參數還可以是
`OBJC_ASSOCIATION_ASSIGN`、`OBJC_ASSOCIATION_COPY_NONATOMIC`、`OBJC_ASSOCIATION_RETAIN`
以及 `OBJC_ASSOCIATION_COPY`，與 property
語法使用的記憶體管理方式一致。而當我們的 `MyClass` 物件在 dealloc
的時候，所有透過 `objc_setAssociatedObject` 而 retain
起來的物件，也都會被一併釋放。

雖然我們不能在 category 中增加成員變數，但是卻可以在 extensions
中宣告。在 Xcode 4.2 之後，我們可以這麼寫：

``` objc
@interface MyClass()
{
    NSString *myVar;
}
@end
```

我們甚至可以將成員變數直接放在 `@implementation` 的程式區塊中：

``` objc
@implementation MyClass
{
    NSString *myVar;
}
@end
```

[^1]: 請見 [Objective-C Feature Availability Index](https://developer.apple.com/library/ios/#releasenotes/ObjectiveC/ObjCAvailabilityIndex/_index.html#//apple_ref/doc/uid/TP40012243)
