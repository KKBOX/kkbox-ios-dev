Crash 的類型
------------

在蘋果官方文件
[Technical Note TN2151 Understanding and Analyzing iOS Application Crash Reports](https://developer.apple.com/library/ios/technotes/tn2151/_index.html)
上，可以看到完整的錯誤說明，當中最常見的是 Bad Memory
Access（EXC\_BAD\_ACCESS / SIGSEGV / SIGBUS） 與 Abnormal Exit
（EXC\_CRASH / SIGABRT）這兩項。如果遇到了在這之外的錯誤，可以參考前述
蘋果文件。

### Bad Memory — 記憶體錯誤

在 ARC 問世之後，這樣的問題已經少了很多，不然在 iOS 5 之前，記憶體錯誤
幾乎佔所有 crash 的最大宗。記憶體錯誤代表的是我們嘗試使用一個不正確的
記憶體指標，在 crash log 的 Exception Sub-code 這段，會出現當時嘗試使
用的記憶體位置。

最常見的記憶體問題就是一個 Objective-C 物件的 retain 與release 不成對，
一個物件已經 retain count 為 0 了，我們還繼續要求這個物件 release；或
是一個變數在 release 的時候沒有指向 nil，所以這個變數所指向的 物件已經
retain count 為 0 了，我們還嘗試呼叫，於是呼叫到錯誤的記憶體。此外也包
含 C 的記憶體錯誤，像還沒有 alloc 一塊記憶體就先呼叫。

雖然有了 ARC 之後記憶體問題少很多，但還是會發生。我們在
[記憶體管理 Part 1](memory_management_part_1/README.md) 與
[記憶體管理 Part 2 - ARC](memory_management_part_2/README.md) 討論了不
少相關議題，在這邊就不重複。

要修正記憶體管理問題，找到 crash 發生在哪一行是第一步，可以找到是哪個
物件、或是哪塊記憶體出問題，不過要修正的不見得就是直接發生 crash 的那
一行：一個物件或一塊記憶體產生之後，往往會在很多地方使用過，所以
retain、release 不成對的狀況很有可能發生在 crash 的地方之前。

例如，我們現在寫一個手動管理記憶體的 UIViewController，有個叫做 button
的成員變數，我們在 `loadView` 的地方我們手動寫了一行
`button = [UIButton buttonWithType:UIButtonTypeCustom]`，之後把 button變成
self.view 的 subview。在這個 UIViewController 的 `dealloc` 的地方，
我們寫了 `[button release]`，結果發生了crash，要修正的可能就不是這行
`[button release]`，而是一開始要把 button retain 一份，寫成
`button = [[UIButton buttonWithType:UIButtonTypeCustom] retain]`。

要找到記憶體在什麼地方不成對，可以用 Instrument 的 Zombie 這項設定做
profiling。

### Abnormal Exit - 發生了 Exception

只要程式中有地方發生了 NSException throw，或是沒有達到 NSAssert 的條件，
就會觸發這種錯誤，前一節的 NSNull 問題就是這種。遇到這種錯誤，首先要看
的不是 crash 的 thread，要去看「Last Exception Backtrace」，以及
console 上的訊息。

常見 Exception 包括：

#### 找不到 selector

出現這種錯誤的時候，會跳出「unsupported selector」錯誤訊息。

這種錯誤的原因是，我們期待操作的物件，與實際上拿到的物件不一樣。我們想
要一個 Array的時候可能拿到字串，想要拿到字串的時候卻拿到 NSNull，或是
我們期待的是一個 mutable 的物件，結果拿到的卻是 immutable 的。於是，這
個物件沒有我們期待的 selector 可以使用。

說起來這個問題是 Objective-C 這個語言天生的問題：所有的物件都可以 cast
成 id，然後一個物件放進 array 或 dictionary 拿出來之後，也無法確實確認
是哪種型別。以下面這行 code 來說：

``` objc
NSString *s = [aDict objectForKey:@"key"];
```

我們根本不能相信 s 一定是 NSString，所以就會寫一堆這樣的 code：

``` objc
NSString *s = [aDict objectForKey:@"key"];
if ([s isKindOfClass:[NSString class]]) {
   // 繼續做事
}
```

如果不這麼寫，就有可能發生 crash。真的要解決問題，第一個方法就是，我們
以後就別寫 Objective-C 了，直接改寫 Swift，因為在 Swift 語法中會強迫我
們做這件事情，我們在 Swift 中可能會寫出大量的 if let 語法：

```
if let s = aDict["key"] as? NSString {
   // 繼續做事
}
```

另外一個方式是，我們盡量避免直接使用 NSArray 或 NSDictionary 當 model，
而是在這些物件上另外包裝一層我們自己的 model 物件，在想要取用某個
property 的時候，這個 model class 會做好型別的判斷，確實回傳符合型別的
物件。像 GitHub 推出的 open source 專案
[Mantle](https://github.com/Mantle/Mantle) 就可以幫助我們撰寫這類的
model 物件。

#### nil 的操作

#### UIKit 中的 assertion
