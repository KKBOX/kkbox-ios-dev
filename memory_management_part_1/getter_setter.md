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
