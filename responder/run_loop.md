
Run loop
--------

我們來問一個很簡單的問題：我們所寫過的程式，大多都是從頭到尾、一行一行
往下執行，執行完畢，程式就結束；那麼，一個 GUI 應用程式—無論是我們現在
正在學習的 iOS 與 Mac OS X、還是其他平台—為什麼不是打開之後一路執行到
底結束，而是會停留在螢幕中等待我們操作？

原因很簡單，因為一個 GUI 應用程式開始執行之後，就會不斷執行一個迴圈，
直到用戶決定要關閉這個應用程式的時候，才會關閉這個迴圈。這樣的迴圈在
Windows 平台上叫做 message loop，在 iOS 與 Mac OS X 上叫做 run loop，
而這個迴圈當中所做的，就是收取與分派事件。

每一輪 run loop 的時間並不固定，會與這一輪 run loop 裡頭做了多少事情相
關，比方說，如果我們的畫面複雜，在 App 中同時有很多 view，那麼這一輪
runloop 就得要花上比較多的時間尋找 first responder；而像我們在 UI 上放
了一個按鈕，然後按鈕按下去要做一些事情，全都會算入到這一輪 run loop 的
時間。如果我們的程式做了一件很花時間的事情，讓這一輪 runloop 執行非常
久，就會導致「應用程式介面沒有回應」這種狀態，當介面卡住一段時間，應用
程式就會被系統強制關閉。

Timer 也是倚靠 run loop 運作的。當我們建立了一個 NSTimer 物件之後，下
一步就是要把 timer 物件註冊到 run loop 當中，如果只建立了 NSTimer 物件，
像是只呼叫了 `alloc`、`init`，這個 timer 並不會有作用，而呼叫
`scheduledTimerWithTimeInterval:target:selector:userInfo:repeats:` 會
在建立 NSTimer 物件之外，同時將 timer 加入到 run loop 中。

Timer 運作的原理是，在每一輪 run loop 裡頭，會檢查是否已經到了某個
timer 所指定的時間，如果到了，就執行 timer 所指定的 selector。所以我們
可以知道幾件事：

1. 由於每一輪 runloop 的時間不一定，所以我們其實也不能夠期待 timer 會
   在非常精確的時間執行。前一輪 runloop 如果做了很花時間的事情，就會影
   響到原本應該要執行的 timer 實際執行的時間。
2. 雖然並沒有所謂的最小的時間單位這件事情，但是 timer 的時間間隔一定會
   有一個上限，我們不可能建立比 run loop 的頻率還要更頻繁的 timer。

我們在講 selector 與記憶體管理的時候也提到，runloop 的其中一項功能還包
含管理 auto-release 物件。Mac OS X 與 iOS 早期並沒有自動化的記憶體管理，
當時會使用一套叫做 auto-release 的半自動機制方便管理記憶體，在每一輪
run loop 中，如果某些物件只有在這一輪 run loop 中有用，之後就應該釋放，
我們可以先把物件放進 auto-release pool 裡頭，等到這一輪 run loop 的時
候，再把 auto-release pool 倒空。

講到這裡，我們可以來談 iOS 與 Mac OS X 的程式進入點到底在哪裡。我們在
寫第一個 iOS App 的時候，可能第一個改寫的地方是
`-application:didFinishLaunchingWithOptions:`，就以為這個 method 是
iOS App 的程式進入點。但 iOS App 的程式進入點其實就跟所有的 C 語言程式
一樣，是 `main()`。我們來看 main.m 裡頭寫了什麼。

``` objc
int main(int argc, char * argv[]) {
	@autoreleasepool {
	    return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
	}
}
```

在進入 `-application:didFinishLaunchingWithOptions:` 之前，會

1. 建立 auto-release pool
2. 呼叫 UIApplicationMain，而這個 function 會
3. 建立 UIApplication 這個 singleton 物件
4. 開始執行 run loop
5. 這些步驟完畢後，代表 app 已經開始執行，所以
6. 對 UIApplication 的 delegate 呼叫 `-application:didFinishLaunchingWithOptions:`

在 Cocoa 與 Cocoa Touch 應用程式中，我們會使用 CFRunloop 與 NSRunloop
等物件，描述 runloop。
