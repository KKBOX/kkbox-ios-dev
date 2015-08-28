NSCoding 的常見用途
-------------------

### XIB/Storyboard

我們在實作一個 UIView 的 subclass 的時候，會注意到，如果我們希望在
initialize 這個 view 的時候，就要做一些事情，不但要 override 掉
`initWithFrame:`，也要 override `initWithCoder:`。

如果這個 view 是我們用 code建立的，那麼就會呼叫到 `initWithFrame:`，但，
如果我們是在 Interface Builder 裡頭，用圖形化工具建立了一個 view，那個，
當這樣的 view 在執行的時候，則會走進 `initWithCoder:` 這一段的實作。

我們在開始接觸 iOS 開發的時候，大概就會先學習如何使用 Interface
Builder 拉出想要的介面，產生出 XIB 或是 Storyboard 檔案。XIB 與
Storyboard 在我們撰寫程式的期間，是 XML 格式的檔案，當我們編譯 App 的
時候，Xcode 會將 XIB 與 Storyboard 編譯成 binary 格式的 data，分別是
NIB 與 storyboardc 檔案，而這些 data 其實就是序列化過的 Objective-C
view 物件。

在一些其他開發平台上（像是使用 Visual Studio 拉出 Windows Form 應用程
式）使用視覺化開發工具的時候，這些工具在做的事情，是把從拉出來的介面產
生程式碼；不過在 Xcode 中編輯 XIB 檔案做的事情不一樣，是先產生出序列化
後的檔案，然後再執行的時候，讀取這些檔案，將 data 轉成 view 物件。這個
流程正是使用 NSCoding protocol，於是從 NIB/storyboardc 讀出我們的 view
的時候，所呼叫的便是 `initWithCoder:`—我們可以從 UIView 的 interface中，
看到 UIView 實作了 NSCoding protocol。

我們在 Xcode 2 左右的年代（大概是 Mac OS X 10.4 左右）開發 Mac App 時，
我們在 Xcode 中其實是直接編輯 NIB 檔案，到了 Xcode 5 與 Mac OS X 10.5
之後才出現使用 XML 格式的 XIB 檔案。這個轉變跟當時 SVN 等版本管理系統
的出現有關，在版本管理系統中編輯 binary 格式的檔案，會難以 diff、merge
以及處理版本衝突，所以蘋果便從 binary 格式換成文字格式的檔案。

### NSUserDefaults

如果在我們的 App 中，我們想要儲存一些偏好設定，那麼最好用的選擇莫過於
Cocoa/Cocoa Touch Framework 本身就提供的 NSUserDefaults 物件。操作
NSUserDefaults 與操作 NSDictionary 差不多，我們只要指定特定的 key，就
可以將設定值存入 NSUserDefaults 中。

NSUserDefaults 支援 NSString、NSArray、NSDictionary、 NSData 以及 int、
double、float 等型別的資料。但，如果是我們自己定義的 Class，或是許多其
他的 Class，會無法存入 NSUserDefaults 中，我們會需要先透過 NSCoding 轉
換成 NSData 後存入，在取出的時候，也要多做一次 unarchive。

比方說，如果我們的 App 的某個地方可以設定顏色，我們想把 UIColor 變成設
定值，UIColor 就是一種無法直接存入 NSUserDefaults 的物件。所以我們想把
UIColor 存入 NSUserDefaults，就得這麼寫：

``` objc
UIColor *color = [UIColor colorWithHue:1.0 saturation:0.5 brightness:0.5 alpha:1.0];
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:color];
[[NSUserDefaults standardUserDefaults] setObject:data forKey:@"color"];
```

### Document-based App

### Copy and Paste

### State Preservation and Restoration
