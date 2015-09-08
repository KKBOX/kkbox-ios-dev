常見 Crash 的類型
-----------------

在蘋果官方文件
[Technical Note TN2151 Understanding and Analyzing iOS Application Crash Reports](https://developer.apple.com/library/ios/technotes/tn2151/_index.html)
上，可以看到完整的錯誤說明，當中最常見的是 Bad Memory
Access（EXC\_BAD\_ACCESS / SIGSEGV / SIGBUS） 與 Abnormal
Exit（EXC\_CRASH / SIGABRT）這兩項。如果遇到了在這之外的錯誤，可以參考
前述蘋果文件，尤其是像錯誤代碼為 00000020 這類的「其他錯誤」，大概也就
只有這篇文件可以參考，去 Stack Overflow 也不見得可以找到答案。

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
以後就別寫 Objective-C 了，直接改寫 Swift，一方面 Swift 的 array 與
dictionary 可以透過 Generics 語言特性指定裡頭的物件型態，再來 Swift 語
法中會經常強迫我們確認物件型別，在 Swift 中，我們可能會寫出大量的 if
let 語法：

```
if let s = aDict["key"] as? NSString {
   // 繼續做事
}
```

再蘋果在 WWDC 2015 中，宣布 Objective-C 也可以選用 Generics 語法，應該
也會有一些幫助。

另外一個方式是，我們盡量避免直接使用 NSArray 或 NSDictionary 當 model，
而是在這些物件上另外包裝一層我們自己的 model 物件，在想要取用某個
property 的時候，這個 model class 會做好型別的判斷，確實回傳符合型別的
物件。像 GitHub 推出的 open source 專案
[Mantle](https://github.com/Mantle/Mantle) ，就可以幫助我們撰寫這類的
model 物件，在這個專案的設計中，透過大量的 tranformer 物件，讓每個
property 都轉出正確的形態。

#### nil 的操作

無論是對 NSMutableArray 或 NSMutableDictionary 插入 nil，都會發生
crash。要避免這個問題，就是在做插入的動作之前，都先檢查一下現在要插入
的物件是否是 nil。

要不然就是改寫 Swift：Swift 語法特別強調一個變數是否可以指向 nil，這項
特性叫做 Optional，一個可以指向 nil 的變數必須設成 Optional，也就是變
數後方必須加上一個問號，而這個變數以後每次出現，後方都一定會出現問號與
驚嘆號。

我們不打算在這邊講解太多 Swift，有些人說，蘋果推出 Swift 這門新語言，
語法比較簡潔所以適合新手入門，這種說法就見仁見智，Swift 從 WWDC 2014推
出之後語法一直變化，而且一直新增語言關鍵字，而問號驚嘆號更經常讓人眼花
撩亂。不過，相較於 Objective-C，因為對 nil 與物件型態的強調，Swift會是
一門更安全的語言。

#### Out of Bounds

如果一個 array 只有兩筆資料，但我們卻去要第三筆資料，就會產生 out of
bounds 錯誤。

#### 一邊 enumerate 一邊改動 array

假如我們一邊 enumerate 一個 array，一邊改動它，就會跳出 exception。像
我們想要把一個 mutable 的字串 array 中，長度小於 3 的字串都拿掉，如果
像以下這種寫法就會 crash：

``` objc
for (NSString *s in array) {
	if ([s length] < 3) {
	  [array removeObject:s]
	}
}
```

我們可以先把想刪除的物件先放到另外一個 array 中，再告訴原本的 array 要
刪除哪些東西。

``` objc
NSMutableArray *arrayToDelete = [NSMutableArray array];
for (NSString *s in array) {
	if ([s length] < 3) {
		[arrayToDelete addObject:s];
	}
}
[array  removeObjectsInArray:arrayToDelete];
```

不過，如果我們想做的事情是想把一些東西從某個 array 濾掉，也可以考慮改
用 NSPredicate。上面的 code 其實意思也就是：把長度大於 2 的字串留下來。

``` objc
NSPredicate *predicate = [NSPredicate predicateWithFormat:@"SELF.length > 2"];
[a filterUsingPredicate:predicate];

```

#### UIKit 中的 assertion

UIKit 中有不少跟資料一致性相關的 assertion。當我們要求一個 table view
刪除或加入某些列、同時帶有動畫效果的時候（透過呼叫
`-insertRowsAtIndexPaths:withRowAnimation:` 與
`-deleteRowsAtIndexPaths:withRowAnimation:` 這些 method），如果 table
view 的 data source 沒有對應的變化—像原本 table view 裡頭有六列，我們
要求刪除一列，但 table view 的 data source 並沒有變成五列，那麼就會造
成 table view crash。

所以在遇到經常變動的 model 的時候，我們需要考慮關閉動畫效果。以 KKBOX
的歌單功能來說，我們除了可以讓用戶手動編輯歌單之外，歌單的內容也可能因
為背景的同步作業、或是下載歌曲的狀態改變而更動；如果在 table view 中出
現動畫的時候，發生這些狀況，就會 crash。

此外，在使用 UIKit 的各種元件的時候，我們要對 0.25 秒這個時間保持敏感，
絕大多數在 UIKit 中的動畫效果都是 0.25 秒，像上面提到的 table view 新
增或刪除 row 的動畫、UINavigationController push 或 pop view
controller 的動畫，鍵盤升起的動畫，以及 present modal view 的動畫（這
個在 iOS 7 之後倒是有一些改變）等等。如果在一個動畫執行到一半的時候，
我們的 App 又要做一件跟這件動畫相反的事情（像 navigation controller
push 的動畫還沒做完，我們就叫它 pop），狀況好一點，是 view heirarchy
會變亂，畫面變得亂七八糟，狀況不好就是直接 crash 了。
