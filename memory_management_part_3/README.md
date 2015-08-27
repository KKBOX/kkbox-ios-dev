記憶體管理 Part 3
=================

本章主要討論跟 UIViewController 相關的記憶體相關問題，嚴格說起來比較像
是在討論 UIViewController 的 life cycle。

總之，我們要回答的問題是─當我們建立了一個 UIViewController 之後，Xcode
給我們的 template 中，會叫我們去實作一個叫做
`didReceiveMemoryWarning:` 的 method，然後你可能從一些相關文件上知道，
當系統記憶體不夠的時候，我們應該要在這個 method 裡頭釋放一些記憶體，那
麼，有哪些記憶體是應該要釋放的？我們應該怎麼實作這個 method？

記憶體不足警告（Memory Warnings）
--------------------------------

在 Desktop 作業系統中，如果實體記憶體不足，應用程式使用的記憶體量，超過
實體記憶體的數量，這時候作業系統會自動將記憶體中的部分資料，存入磁碟的
虛擬記憶體（Virtaul Memory）當中，需要使用的時候，再從虛擬記憶體中載回
實體記憶體，Mac OS X就有這樣的機制。

iOS在發展之初到現在，都沒有虛擬記憶體，而是會在記憶體快要用完的時候，
對應用程式發出記憶體不足警告，要求釋放一些可以暫時不需要用到的物件，讓
應用程式可以有足夠的記憶體繼續運作。如果無視記憶體警告，繼續放任記憶體
用量成長，系統最後便會強制要求終止應用程式。

在記憶體不足的時候，除了會對 UIApplication 的 delegate （就是所謂的
AppDelegate）呼叫`applicationDidReceiveMemoryWarning:` 之外，也會對系
統中所有的UIViewController 呼叫 `didReceiveMemoryWarning:`。如果我們想
要知道哪些記憶體是可以在 `didReceiveMemoryWarning:` 釋放的，不妨先回顧
一下在 iOS 6 之前，iOS 是怎麼做的—

從 iOS 問世到 iOS 5，只要發生記憶體不足，就會把所有不在最前景的 View
Controller 的 view 釋放掉。因為這些 View Controller 的 view 並不在畫面
上，用戶根本看不到，所以暫時先放掉也沒有關係。

iOS 6 之前記憶體不足時系統主動釋放 View 的行為
----------------------------------------------

所謂不在最前景的 view controller 就是：假如我們今天有一個 tab bar
controller，tab bar 裡頭有四個項目，對應到四個 view controller，但是其
實只會顯示一個，那麼，在 iOS 6 之前，只要發生記憶體警告的時候，其他三
個view controller 的 view 就會被釋放。

在 navigation controller 的navigation stack 裡頭，也只有最上面的 view
controller的畫面需要顯示，其他 view controller 的 view 也可以被釋放。

所以，如果你曾經在 iOS 6 之前的環境上開發過 iOS App，可能會遇到一個奇
怪的 bug：你把一些狀態直接記錄在 view 裡頭，像是改變了一些 label 裡頭
的文字，但是繼續做了一些操作，然後回到這個 view 之後，發現 view 莫名其
妙的回復到初始值，原本放在 label 的文字不見了，其實就是遇到了記憶體警
告的結果。

UIViewController 與 View 的關係
-------------------------------

`UIViewController`負責管理在應用程式中每個會用到的畫面，最主要的
property 就是 `view`，而這個 property 是使用**Lazy Loading** pattern
實作。Lazy Loading 就是：**我們要去使用某個物件的時候，我們才去建立那
個物件**，避免在物件初始時就建立了所有的property，而達到讓初始物件這個
動作加速的效果。 [^1]

當我們在透過 `alloc` 、 `init` 或`initWithNibName:bundle:` 建立 View
Controller 的時候，並不會馬上建立 view，而是當我們呼叫 `view` 這個屬性
的時候才會建立。我們以下面的程式為例：

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

在 iOS 6 之前，如果某個 View Controller 不在最上層，發出記憶體警告時，
系統就會通知這些 View Controller 把 view 指向 nil；而當我們再次需要使
用這個 View Controller 的時候，就會因為呼叫到 `view`，而把 view 重新載
入回來。

所以我們要注意，`viewDidLoad` 並不是 `UIViewController` 的
Initializer，—雖然我們在開始使用某個 view controller 的時候，一定會呼
叫到一次`viewDidLoad` ，我們也通常會在這個地方，做一些初始化這個 view
controller的事情—但 `viewDidLoad` 是有機會在 View Controller 的 Life
Cycle 中被重複呼叫好幾遍—在建立了 view 之後，view 也可以再次指向 nil，
所以 view controller 可能會被重複釋放與載入 view， `viewDidLoad`也會被
重複呼叫。

所以在 iOS 6 之前，你曾經遇到某個 View Controller 回復到初始值這樣的問
題，就是：原本有狀態的 view 因為記憶體警告被釋放了，而我們如果在
`viewDidLoad` 再次被呼叫的時候，沒有正確還原狀態，自然只有初始狀態的 view。

### iOS 如何知道哪個 View Controller 位在最上層？

那麼，view controller自己怎麼知道自己位在最前景呢？其實很簡單：view
controller 被放到最上層時，會被呼叫到 `viewWillAppear:` 以及
`viewDidAppear:` ，離開最上層時，會呼叫 `viewWillDisappear:` 與
`viewDidDisappear:` 。

只有呼叫過 `viewWillAppear:` 以及 `viewDidAppear:`，而沒有呼叫過
`viewWillDisappear:` 與 `viewDidDisappear:` 的 View Controller，就是位
在最前景的 View Controller。

我們經常會 override `viewWillAppear:` 這些 method，在做 override 的時
候，應該要呼叫一次 super 的實作，因為 super 的 `viewWillAppear:` 這些
method 其實作了一些必要的事情，在 iOS 6 之前是用來確保哪些 view 該被釋
放，雖然蘋果推出 iOS 6 時，或許是認為像是 iPhone 5這樣的裝置在可用資源
上遠遠超越過去的硬體，因此不再刻意釋放 view，但呼叫一下 super 的實作還
是比較保險。

所以我們應該要在 `didReceiveMemoryWarning:` 做什麼？
----------------------------------------------------

從過去的經驗來看，不在最上層的 view 其實可以釋放，在 iOS 6 之後，當我
們遇到記憶體不足時，我們也可以選擇性的決定要不要釋放 view，像 web view
這種記憶體怪物在沒用到的時候實在應該要放掉。

以下是蘋果的範例程式：

``` objc
- (void)didReceiveMemoryWarning
{
    [super didReceiveMemoryWarning];
    if ([self.view window] == nil) {
        self.view = nil
	}
}
```

相關閱讀
--------
- [Resource Management in View Controllers](https://developer.apple.com/library/ios/featuredarticles/ViewControllerPGforiPhoneOS/ViewLoadingandUnloading/ViewLoadingandUnloading.html)

[^1]: 也可以參見 Wikipedia 上的說明 <http://en.wikipedia.org/wiki/Lazy_loading>
