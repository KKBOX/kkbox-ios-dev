Delegate 與 Protocol
====================

相信在閱讀這一章之前，你應該已經寫過一些簡單的 iOS程式，知道如果我們想
要一個像是系統設定（Setting.app）那樣的表格介面，我們會建立
`UITableViewController` 的 subclass，這個 controller 的 view 是
`UITableView`。

當我們想要設定表格中的內容，像是這個表格中有多少個 section、每個
section裡頭有多少 row、每個 row 裡頭又是哪些內容…我們不是直接呼叫
`UITableView`的 method，像是呼叫 `[myTableView setSectionCount:3]` 或
是`[myTableView setRowCount:3 atSection:4]`，而是去實作
`UITableViewController` 裡頭幾個像是 template method 的東西，像
`numberOfSectionsInTableView:`、`tableView:numberOfRowsInSection:`等等。

為什麼不是直接去改變 view，而是 controller 要準備一些不知道會被誰呼叫
的method？理由是：`UITableViewController` 是 `UITableView` 的 data
source與 delegate。那，什麼是 delegate？

用我的話來說，delegate 就是 **將眾多的 callback，集中在一個物件上** 。

從其他平台來看 Objective-C 的 Delegate
--------------------------------------

Delegate 算是入門 Objective-C的另外一個障礙，但了解之後就會知道其實很
簡單，一開始可能覺得並不好懂的主要原因是，delegate的這套作法，跟其他平
台在做同樣的事情的時候，作法比較不一樣。

如果你曾經在其他的平台上開發過應用程式，像是微軟的 .Net平台的話，可以
發現，在 C\# 語言中在講一個 class 有哪些成員的時候，會包括properties、
methods 以及 events；但是我們在講 Objective-C 的 class時，並不會提到
events，C\# 中使用 events 在做的事情，在 Objective-C中，我們往往使用
target/action 與 delegate 實作。 [^1]

雖然在 Mac OS X 與 iOS 上分別有 `NSEvent` 與 `UIEvent`，但是這邊的
events 與 .Net 裡頭講的 events 又是兩件事情：`NSEvent`是用來描述鍵盤按
了哪個按鍵，滑鼠移到了什麼位置，而 `UIEvent`則用來描述觸控事件以及耳機
上的播放控制按鈕等等，單純用來描述從硬體輸入了什麼事件，透過作業系統傳
到我們的應用程式中。而C\# 裡頭所稱的 events，則是用來處理 callback。

所謂 callback 是指，當我們呼叫了一個 function 或 method之後，可能會花
上許多時間，或是計算的是大量資料，或是需要透過網路連線，所以我們並不馬
上要求得到回傳的結果，而是等到一段時間之後，計算結果才會透過另外一個
function/method 傳回來。

現在絕大多數的應用程式開發環境都採用 MVC架構，我們將物件分成三類：
model、view、controller，雖然在不同的平台上往往實作出來的結果不太一樣，
像是在微軟的平台上，往往把window （或是 form）當做是 controller 使用，
由 window物件負責管理放在這個 window 上面所有的 control 元件，但是 Mac
OS X 上面window 並不拿來當做 controller，而是被當成 view，controller
在 window之外，而且一個 controller 也可以控制多個 window…但都大抵如此。

在 MVC 的架構中，我們通常會先建好 controller class，然後加入 model 與
view，變成 controller 的成員變數；因為 model 與 view 是 controller的成
員變數，所以 controller 可以直接呼叫 model 與 view，那麼，當 model與
view 發生了變化，要回來通知 controller，我們也可以稱之為callback，例如，
controller建立了一個按鈕，但是直到按鈕被點選之後，controller 才負責做
事。

在 C\# 中，我們要處理一個 event，就要提供一個 event handler，我們現在
要處理的是 Click：

``` csharp
private void InitializeComponent()
{
    this.button1 = new System.Windows.Forms.Button();
    this.button1.Click += Button1_Click;
}

private void Button1_Click(object sender, System.EventArgs e)
{
}
```

如果只是單純的點選事件的話，我們會用 target/action 實作。但，對照 .Net
framework 裡頭，events 不只是 click 而已，還可能會有 double click、
triple click、quadruple click…，如果是 C\# 裡頭，會這麼寫：

``` csharp
this.button1.Click += Button1_Click;
this.button1.DoubleClick += Button1_DoubleClick;
this.button1.TripleClick += Button1_TripleClick;
this.button1.QuadrupleClick += Button1_QuadrupleClick;
```

變成 Objective-C 的話就可能變成：

``` objc
[button1 setTarget:self];
[button1 setAction:@selector(click:)];
[button1 setDoubleTarget:self];
[button1 setDoubleAction:@selector(doubleClick:)];
[button1 setTripleTarget:self];
[button1 setTripleAction:@selector(tripleClick:)];
[button1 setQuadrupleTarget:self];
[button1 setQuadrupleAction:@selector(QuadrupleClick:)];
```

這樣寫起來實在很讓人煩躁（雖然 `NSTableView` 也的確有
`setDoubleAction:` …），所以，當這樣的東西一多，在 Objective-C語言裡頭，
會直接準備好一個物件，這個物件準備好了所有可以呼叫的method，這個物件就
叫做 delegate，而這些可以呼叫的 method 的集合，叫做protocol。準備好這
個物件之後，我們就不用呼叫這麼多`setDoubleAction:`、`setTripleAction:`，
只要呼叫 `setDelegate:`。

設計 Protocol 與實作 Delegate 的方式
------------------------------------

我們來用 delegate 的想法來實作前面提到的狀況。

### 宣告 Protocol 與 Delegate 的方式

我們先來建立一個 `NSButton` 的 subclass，叫做 `MyButton`， `MyButton`
的delegate 必須實作 `MyButtonDelegate` 這個 protocol。.h 檔案中宣告如
下：

``` objc
#import <Cocoa/Cocoa.h>

@class MyButton;

@protocol MyButtonDelegate
- (void)myButtonDidBecomeClicked:(MyButton *)button;
- (void)myButtonDidBecomeDoubleClicked:(MyButton *)button;
- (void)myButtonDidBecomeTripleClicked:(MyButton *)button;
- (void)myButtonDidBecomeQuadrupleClicked:(MyButton *)button;
@end

@interface MyButton : NSButton
{
    id <MyButtonDelegate> delegate;
}
@property (weak, nonatomic) id <MyButtonDelegate> delegate;
@end
```

逐行解釋這個 header 裡頭的內容：

1.  `#import <Cocoa/Cocoa.h>` ：因為 NSButton 是在 AppKit
    裡頭，所以我們必須呼叫對應的 header。
2.  `@class MyButton;` ：因為在我們接下來的 protocol 宣告中，會用到
    `MyButton` 這個 class，但是 `MyButton` 其實是宣告在
    `MyButtonDelegate` 的下面，所以我們需要先預先宣告 `MyButton` 這個
    class 的存在。
3.  從 `@protocol MyButtonDelegate` 開始，就是在宣告 `MyButtonDelegate`
    這個 protocol 裡頭的四個 method。
4.  接下來宣告 `MyButton` 這個 class，裡頭有一個叫做 delegate 的變數。

### 實作 Delegate Methods

假如我們有一個叫做 `MyController` 的 controller 物件，要成為
`MyButton`的 delegate，我們會這麼做。首先是 .h 部分：

``` objc
@interface MyController : NSObject <MyButtonDelegate>
{
    IBOutlet MyButton *myButton;
}
@end
```

我們要先宣告 `MyController` 有實作 `MyButtonDelegate` 這個
protocol，如此一來，假如 `MyController` 漏了實作哪些定義在
`MyButtonDelegate` 裡頭的
method，在編譯的時候就會跳出警告，要求我們修正，如果我們還是不實作的話，執行時，就會發生找不到
selector 對應的實作的錯誤而 crash。

如果 `MyController` 又是其他物件的 delegate的話，我們可以在這段用大於
與小於間包起來的宣告，繼續加入其他的 protocol的名稱，例如
`<MyButtonDelegate, AnotherProtocol>`。至於一個物件是否有實作某個
protocol，我們可以用 `conformsToProtocol:` 檢查。

在 `MyController` 的實作中，我們就只要將 `myButton` 的 delegate設成自
己，然後實作該實作的 method。我個人不太喜歡在 protocol的宣告中出現大量
註解，因為在實作 protocol 的時候，最方便的方式就是直接把protocol 宣告
的 method 複製貼上，接著逐一把肉放進 protocol定義的骨幹裡；如果當中出
現註解，複製貼上之後，還要把這些註解刪掉，其實還頂煩人。

``` objc
@implementation MyController

- (void)awakeFromNib
{
    [myButton setDelegate:self];
}

#pragma mark - MyButtonDelegate

- (void)myButtonDidBecomeClicked:(MyButton *)button
{
}
- (void)myButtonDidBecomeDoubleClicked:(MyButton *)button
{
}
- (void)myButtonDidBecomeTripleClicked:(MyButton *)button
{
}
- (void)myButtonDidBecomeQuadrupleClicked:(MyButton *)button
{
}

@end
```

我們在這邊透過 `setDelegate:` 指定 delegate，如果我們把 delegate宣告成
是一個 IBOutlet 的話，也可以直接在 Interface Builder 中連結。

### Delegate Methods 是怎麼被呼叫的？

`MyButton` 是怎樣呼叫 delegate 的呢？其實很簡單。

``` objc
@implementation MyButton

- (void)mouseDown:(NSEvent *)theEvent
{
    switch ([theEvent clickCount]) {
        case 1:
            [delegate myButtonDidBecomeClicked:self];
            break;
        case 2:
            [delegate myButtonDidBecomeDoubleClicked:self];
            break;
        case 3:
            [delegate myButtonDidBecomeTripleClicked:self];
            break;
        case 4:
            [delegate myButtonDidBecomeQuadrupleClicked:self];
            break;
        default:
            break;
    }
}

@synthesize delegate;
@end
```

注意事項
--------

在上面的範例中，我們看到了設計 delegate 與 protocol 應該注意的地方：

### Delegate 物件不應該指定 Class

我們將 delegate 物件宣告成 `id <MyButtonDelegate> delegate`，意思就是
不需要管這個物件屬於哪個 class，只要是個 Objective-C物件即可，但是這個
物件必須實作 `MyButtonDelegate` protocol。

我們其實可以將 delegate 物件是那個 class 寫死，例如把 `MyButton` 的
delegate 的 class 指定成`MyController`，但這樣做非常不好，如此一來，就
只有 `MyController`可以使用 `MyButton`，其他 controller 都無法使用，就
大大減少了重複使用`MyButton` 的彈性。

Delegate 這種設計方式，也方便我們在同時開發 Mac OS X 與 iOS跨平台專案
時共用程式碼，我們在撰寫某個 model 物件的時候，只使用Foundation 或是其
他兩個平台都有的framework，至於與平台相依的部份，就放進 delegate 中，
然後在 Mac OS X 與iOS 上各自實作 delegate 物件。

總之， *在實作 delegate 的時候，delegate 屬於哪個 class並不重要，重要
的是 delegate 物件有沒有實作我們想要呼叫的 method。*

### Delegate 屬性應該要用 Weak，而非 Strong

在使用 property 語法的時候，如果這個 property 是 Objective-C物件，我們
照理說應該要設定成 strong 或 retain，但是遇到的是 delegate，我們應該設
成 weak 或 assign。

原因是：需要設計 delegate 物件的這個物件，往往是其 delegate物件的成員變
數。在我們的例子中，`MyButton` 的 instance 是 `myButton`，是
`MyController` 的成員變數，自己可能已經被 `MyController` retain了一份。
如果 `MyButton` 又 retain了一次 `MyController`，就會出現循環 retain 的
問題—我已經被別人 retain，我又把別人 retain 一次。

如此，會造成我們會無法釋放 `MyController`：在該釋放 `MyController`的時
候，`MyController` 還是被自己的成員變數 retain，`MyController`得要走到
`dealloc` 才會釋放 `myButton`，但是自己卻因為被 `myButton` 給retain 起
來，而始終走不到 `dealloc`。

#### Delegate Method 的命名方式

Delegate method 的命名有個鮮明的特色，就是這個 method至少會傳入一個參
數，就是把到底是誰呼叫了這個 delegate method傳遞進來。同時，這個
method 也往往以傳入的 class名稱開頭，讓我們可以辨識這是屬於哪個 class
的 delegate method。以`UITableViewDelegate` 為例，假如我們在 iOS的表格
中，選擇了某一列，就會呼叫

``` objc
- (void)tableView:(UITableView *)tableView
        didSelectRowAtIndexPath:(NSIndexPath *)indexPath
```

Method 的名稱就以「tableView」開頭，讓我們知道這是屬於 Table View 的
delegate method，然後第一個參數把這個 Table View 的 iinstance傳入，接
下來才傳入到底是哪一列被選起來的資訊。

至少把是誰呼叫了這個 delegate method 傳入的理由很簡單。以我們的
`MyController` 為例，這個 controller 可能有好幾個 `MyButton`，而這些
`MyButton` 全都把 delegate 指到同一個 controller 上，那麼，controller
就需要知道到底是被哪個 button 呼叫。判斷方式只要簡單比對指標就好了：

``` objc
- (void)myButtonDidBecomeQuadrupleClicked:(MyButton *)button
{
    if (button == myButton1) {
    }
    else if (button == myButton2) {
    }
}
```

Data Source 與 Delegate 的差別？
--------------------------------

我們現在可以來看看 `UITableView` 與 `UITableViewController`是怎麼運作
的。 `UITableViewController` 在 `loadView` 中建立了一個`UITableView`
的 instance，指定成是自己的 view，同時將這個 view 的delegate 與 data
source 設定成自己。一個 class 可以根據需要，將 delegate拆成好幾個，以
`UITableView` 來說，跟表格中有什麼資料有關的，就放在 data source 中，
其餘的 method 放在 delegate 中。

我們在 Mac OS X 會用到的最龐大的 UI 元件，莫過於 `WebView`，雖然在 iOS
上 `UIWebView` 被閹割到只剩下四個 delegate method，但是 Mac OS X上足足
有五大類 delegate method，網頁頁框的載入進度、個別圖片檔案的載入進度、
下載檔案的 UI呈現、該不該開新視窗或是新分頁、沒有安裝 Java 或是 Flash
要怎麼呈現、用JavaScript 跳出 alert 該怎麼呈現…都是一堆 delegate
method。

假如先不管 `UITableView` 怎麼重複使用 `UITableViewCell` 的機制（這個機
制還頂複雜），我們要更新 `UITableView` 的資料時，先指定data source 物
件後，要呼叫一次 `reloadData` 。 `reloadData`可能是這樣寫的：

``` objc
- (void)reloadData
{
    NSInteger sections = 1;
    if ([dataSource respondsToSelector:
        @selector(numberOfSectionsInTableView:)]) {
        sections = [dataSource numberOfSectionsInTableView:self];
    }
    for (NSInteger section = 0; section < sections; section++) {
        NSInteger rows = [dataSource tableView:self
                         numberOfRowsInSection:section];
        for (NSInteger row = 0; row < rows; row++) {
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:row
                                                        inSection:section];
            UITableViewCell *cell = [dataSource tableView:self
                                    cellForRowAtIndexPath:indexPath];
            // Do something with the cell...
        }
    }
}
```

我們注意到幾件事情：首先，因為 `numberOfSectionsInTableView:` 被定義成
是optional 的 delegate method，delegate 不見得要實作，所以我們會用
`respondsToSelector:` 檢查是否有實作。我們可以在 protocol的宣告中，指
定某個 delegate method 是 required 或是optional，如果不特別指定的話，
預設都是 required。我們簡單看一下`UITableViewDataSource` 就知道如何定
義 required 與 optional 的 delegate method。

``` objc
@protocol UITableViewDataSource<NSObject>
@required
- (NSInteger)tableView:(UITableView *)tableView
  numberOfRowsInSection:(NSInteger)section;
- (UITableViewCell *)tableView:(UITableView *)tableView
  cellForRowAtIndexPath:(NSIndexPath *)indexPath;
@optional
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;
// ....
@end
```

另外，就是定義在 data source 的 method，是在 `reloadData` 中被呼叫，因
此我們可以知道 `UITableView` 的 data source 與 delegate的最大差別：我
們絕對不可以在 data source 定義的 method 中呼叫`reloadData`，不然就會
進入無窮迴圈！

Formal Protocol 與 Informal Protocol
------------------------------------

`@protocol` 這個關鍵字是在 Objective-C 2.0 之後出現的，在這之前要定義
protocol，則是寫成 NSObject 的 category，前者叫做 formal protocol，後
者則稱為 informal protoco。UIKit 問世時就採用 Objective-C 2.0 語法，至
於 Mac OS X，蘋果在 2008 年開始大幅改寫 Foundation 與AppKit，現在
（2012 年）絕大多數可以看到的 protocol，都是 formal protocol，但如果你
在 maintain 一份稍微有點歷史的程式，或是在蘋果少數的API 中，還是可以看
到 informal protocol。

在 Core Animation 裡頭，就可以看到`CALayerDelegate`、`CALayoutManager`、
`CAAnimationDelegate`，都還是informal protocol。其中 `CALayerDelegate`、
`CALayoutManager`兩者之間還夾著 `CAAction` 這個 formal protocol—在兩個
informal protocol中間夾著一個 formal protocol，實在讓人很反感—為什麼不
一起改掉呢？至於`CAAnimationDelegate` 也很怪異：CAAnimation 的
delegate 不是用assign，而是會 retain 起來。

無所不在的 Delegate
-------------------

由於在 Objective-C 語言中，delegate 相當於 event handler的用途，所以，
當你在其他平台中看到 event handler 用得多頻繁，就等於delegate 用得多頻
繁。舉例來說：

-   在使用 `NSURLConnection` 抓取網路上的資料的時候，無論收到了 HTTP
    response code、是否連線失敗、是否連線結束…都是透過 delegate 回傳。
-   在使用 Core Location 的時候，如果 `CLLocationManager`
    找到了我們的所在位置，或是發現我們正在移動，也都會透過 delegate
    通知。
-   當我們要使用手機拍照、傳送簡訊或是電子郵件等等，當照片拍完，會用
    delegate 回傳 image 物件，簡訊或是電子郵件傳送成功，也會用 delegate
    告訴我們執行完畢。

甚至，當我們在寫一個 iOS 程式的第一步，其實都是在實作一個 delegate
method。我們在 Xcode 裡頭開了一個新專案之後，下一步往往是實作
`application:didFinishLaunchingWithOptions:` 這個method，但是要了解整
個程式的進入點，我們要從 `main.m`來看。裡頭通常只有簡短的幾行：

``` objc
int main(int argc, char *argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, nil));
    }
}
```

一個 iOS 程式是從 `main` 這個 function 開始，接著透過呼叫
`UIApplicationMain` 建立 `UIApplication` 這個 Singleton 物件。
`UIApplication` 用來代表一個應用程式的基本狀態，包括 icon上面該顯示多
少則 push notification的數量、支援水平還是垂直畫面、是否顯示狀態列等，
當 `UIApplication`物件被建立起來後，就要通知它的delegate—程式已經開啟
了，請進行下一步，這個 delegate method 就是
`application:didFinishLaunchingWithOptions:`，我們在這邊建立基本的
view controller 與 window，顯示出來。

也就是說，當我們在開始寫第一行 iOS 程式的時候，我們就起碼需要了解什麼
是Singleton 和 delegate，但是在了解之後，想要知道 Mac OS X 與 iOS中眾
多的元件該如何使用，以及怎樣用比較好的方式設計自己的元件，就不是問題了。

其他平台上所謂的 Delegate
-------------------------

在其他平台中，也用到了 delegate 這個詞，但是意義不太一樣。

### Design Pattern 中所講的 Delegate

就我的理解，Design Pattern 中所講的 Delegate Pattern，比較像是做一個
Wrapper，有一個 class 在實作 method 時，其實是直接把這個 method的實作
傳遞到自己的成員變數物件的實作上。以 Objective-C語言實作會像這樣：

首先產生一個內部的物件，叫做 `MyInnerClass`：

``` objc
@interface MyInnerClass : NSObject
- (void)doSomething;
@end
@implementation MyInnerClass
- (void)doSomething
{
    NSLog(@"Do something");
}
@end
```

然後， `MyClass` 會把該做的事情，都交給 `MyInnerClass`：

``` objc
@interface MyClass : NSObject
{
    MyInnerClass *innerObject;
}
- (void)doSomething;
@end
@implementation MyClass
- (void)dealloc
{
    [innerObject release];
    [super dealloc];
}
- (id)init
{
    self = [super init];
    if (self) {
        innerObject = [[MyInnerClass alloc] init];
    }
    return self;
}
- (void)doSomething
{
    [innerObject doSomething];
}
@end
```

在 Cocoa Framework 中，會比較像是 `NSButton` 與 `NSButtonCell`的關係。
你或許會問，為什麼 Objective-C 裡頭的 delegate 與 Design Pattern裡頭講
的 Delegate Pattern 意義不一樣？為什麼 Objective-C不按照這套用法？但其
實是， Objective-C 使用 delegate 這個觀念，早於Design Pattern 成書。

### C\# 中所謂的 Delegate

C\# 語言中也有 delegate 這個關鍵字，不過用途卻是處理 anonymous
function，以我們上面的例子，我們打算用 C\# 增加按鈕被點選的 event
handler，原本是這麼寫：

``` c#
private void InitializeComponent()
{
    this.button1 = new System.Windows.Forms.Button();
    this.button1.Click += Button1_Click;
}

private void Button1_Click(object sender, System.EventArgs e)
{
}
```

在 C\# 2.0 可以寫成這樣： [^2]

``` c#
private void InitializeComponent()
{
    this.button1 = new System.Windows.Forms.Button();
    this.button1.Click += delegate(object sender, System.EventArgs e) {
    // Do something here.
    };
}
```

關於在 Objective-C 語言中怎麼使用 anonymous function，我們會在接下來的
章節，講 block 的時候討論。

我曾經犯過的低級錯誤
--------------------

最後來提一個我之前花了半個月才找出問題在哪的 bug。問題出在 protocol不
該這麼設計。我之前寫了一個 class，這個 class 有兩個 method： `begin`與
`stop`，按下 `begin` 的時候要開始做一件事情，之後想要停止，就呼叫
`stop`，要開始或要結束的時候，都會通知 delegate。程式大概是這樣：

``` objc
@class MyClass;
@protocol MyClassDelegate <NSObject>
- (void)myClassWillBegin:(MyClass *)myClasss;
- (void)myClassDidBegin:(MyClass *)myClasss;
- (void)myClassWillStop:(MyClass *)myClasss;
- (void)myClassDidStop:(MyClass *)myClasss;
@end

@interface MyClass : NSObject
{
    id <MyClassDelegate> delegate;
}
- (void)begin;
- (void)stop;
@property (assign, nonatomic) id <MyClassDelegate> delegate;
@end

@implementation MyClass

- (void)begin
{
    [delegate myClassWillBegin:self];
    // Do something
    [delegate myClassDidBegin:self];
}
- (void)stop
{
    [delegate myClassWillStop:self];
    // Do something
    [delegate myClassDidStop:self];
}
@synthesize delegate;
@end
```

這個 protocol 有什麼問題呢？就像前面提到，在 `UITableView` 的 data
source 的 method 裡頭不該呼叫 `reloadData` 一樣，這邊的幾個 delegate
method 的實作裡頭，也都不該隨意的呼叫 `begin` 與 `stop` ，而我在
`myClassWillBegin:`裡頭想要做一些檢查，如果在某些條件下，這件事情不該
跑起來，而應該停止，所以我在`myClassWillBegin:` 裡頭呼叫了`stop`。但這
麼做，並不會讓這件事情結束，因為 `begin` 這個 method 在對delegate 呼叫
完 `myClassWillBegin:`之後，程式還是會繼續往下走，所以還是把 begin 整
個做完了。

這個 protocol 應該這麼設計：

``` objc
@class MyClass;
@protocol MyClassDelegate <NSObject>
- (BOOL)myClassShouldBegin:(MyClass *)myClasss;
- (void)myClassDidBegin:(MyClass *)myClasss;
- (BOOL)myClassShouldStop:(MyClass *)myClasss;
- (void)myClassDidStop:(MyClass *)myClasss;
@end

@interface MyClass : NSObject
{
    id <MyClassDelegate> delegate;
}
- (void)begin;
- (void)stop;
@property (assign, nonatomic) id <MyClassDelegate> delegate;
@end
@implementation MyClass

- (void)begin
{
    if (![delegate myClassShouldBegin:self]) {
        return;
    }
    // Do something
    [delegate myClassDidBegin:self];
}
- (void)stop
{
    if (![delegate myClassShouldStop:self]) {
        return;
    }
    // Do something
    [delegate myClassDidStop:self];
}
@synthesize delegate;
@end
```

相關閱讀：
---------

- [蘋果官方文件 Working with Protocols](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/ProgrammingWithObjectiveC/WorkingwithProtocols/WorkingwithProtocols.html#//apple_ref/doc/uid/TP40011210-CH11-SW1)


[^1]: 我在寫這章的時候，一直在想拿 C\# 到底是不是好主意，畢竟想要學 Objective-C 語言者，不見得都有 C\# 的基礎。之所以以 C\# 舉例，原因是這份資料其實是來自於當初我個人在公司內部的教材，而公司當時進來的新人之前是寫 C\# 的。

[^2]: 參見 <http://msdn.microsoft.com/en-us/library/bb882516.aspx>
