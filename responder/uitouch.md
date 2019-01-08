UITouch
-------

在觸控事件發生之後，我們會從 UIEvent 中，收到代表觸控事件的 UITouch 物
件。

UITouch 起初是個非常單純的物件，我們頂多只會使用 `-locationInView:` 判
斷觸控事件發生在 view 的哪個位置上（CGPoint），以及用 `-tapCount` 知道
碰觸了幾下，以及用 `-timestamp` 知道觸控事件的時間。

但隨著 iOS 不斷演進，UIEvent 與 UITouch 也變得愈來愈複雜，尤其是在 iOS
9 推出之後，突然一次出現了非常多新的 API。

### 觸控螢幕的掃描速率

在 iOS 9 之後，增加了 `coalescedTouchesForTouch` 這個屬性，主要原因是
iPad Air 2 硬體效能的提升。

當我們看到 `touchesBegan:withEvent:` 這些 API，大概可以想到，我們會在
每一輪 runloop 收到一次 touch 物件，所以觸控螢幕在接收 touch 物件的速
度，大概會是跟 runloop 的速度差不多。於是，當開發者在開發一些在 iPad上
的繪圖應用的時候，往往就覺得在 iPad 上會有延遲的現象：手指已經在螢幕上
畫過去了，但是畫面卻是慢慢地更新。

蘋果在 iPad Air 2 的觸控螢幕上加快了對觸控事件的掃描頻率，比起之前的
iPad 快上一倍，但是，run loop 的速度並沒有改變，因此，在 iOS 9 的 API
中，會把這些比以前來得多的觸控事件，變成 UIEvent 物件的
`coalescedTouchesForTouch:` method，在這個 method 中，可以拿到更多的
UITouch，可以讓我們在搭配 iOS 9 的 iPad Air 2 上抓到更多touch 物件，繪
製更精細的線條。

UIEvent 同時也多了一組叫做 `predictedTouchesForTouch:` 的 method，預測
下一個觸控事件可能出現的位置，因此，即使這個觸控事件還沒有發生，但我們
便可以偷吃步先去做繪圖相關的工作，讓畫面看起來即時更新。

相關說明請參見 WWDC 2015 影片
[WWDC 2015 Advanced Touch Input on iOS](https://developer.apple.com/videos/wwdc/2015/?id=233)。

### 3D Touch

蘋果在 iPhone 6S 上加入了 3D Touch 功能，除了在 App 這層加了 shortcut、
在 view controller 這層加入了 peek and pop 功能（實作
UIViewControllerPreviewingDelegate protocol）之外，便是 UITouch 物件本
身也加入了 `force` 與 `maximumPossibleForce` 等屬性，用來判斷觸控的力
道。

在 iOS 9 與 iPhone 6S 上，`touchesMoved:withEvent:` 的行為也發生了變化，
原本只有 Touch 事件的 X 軸或 Y 軸有改變的時候，系統才會觸發
`touchesMoved:withEvent:`，但是在有了 3D Touch 之後，觸控壓力的改變，也
會觸發 `touchesMoved:withEvent:`，換言之，這個 method 被呼叫的時候，我
們不能夠假設用戶的手指真的移動了位置，很有可能只是壓力的改變而已。要知
道手指的位置是否真的移動了，我們需要另外比對 UITouch 物件的 X 軸或 Y 軸
的位置，不然就可能會把單點誤判成 Swipe。這部份說明請參見
 [iOS 9.1 Release Note](https://developer.apple.com/library/prerelease/ios/releasenotes/General/RN-iOSSDK-9.1/index.html)。

當然，如果只是要知道用戶是否單點在一個位置上，用 UIGestureRecognizer 還
是比較簡單，也是比較保險的方法。

### Apple Pencil

在 iOS 9 推出的同時，Apple 同時宣布了 iPad Pro 這條產品線，在 iPad Pro
上可以使用 Apple Pencil 這款輸入裝置做更精密的手寫。於是，蘋果在
iOS 9.1 SDK 中增加不少與 Apple Pencil 相關的 API。摘錄 iOS 9.1 SDK 的
release note 部分如下：

- UITouch 增加了 `type` 屬性，用來判斷這個觸控事件是來自於直接、間接的
  觸控，或是來自於 Apple Pencil
- UITouch 物件原本就有 `-locationInView:` 與 `-previousLocationInView`
  這兩個 method 表示觸控發生的位置，在使用 Apple Pencil 的時候，我們可
  以透過 `-preciseLocationInView:` 與 `precisePreviousLocationInView:`
  這兩個 method，知道更精細的觸控位置。
- `altitudeAngle`、`azimuthAngleInView:` 與 `azimuthUnitVectorInView:`
  可以讓你知道 Apple Pencil 的高度與方位。
- UIEvent 中的 `predictedTouchesForTouch:` method 只預測了下一個觸控事
  件可能的位置，但有了 Apple Pencil 之後，我們還會想要預測接下來 Apple
  Pencil 的高度、方位與壓力等資訊。我們便可以透過
  `estimatedProperties` 與 `estimatedPropertiesExpectingUpdates` 這些
  屬性取得。
