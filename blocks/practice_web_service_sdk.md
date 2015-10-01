練習：將 Web Service API 包裝成 SDK
-----------------------------------

### 練習範圍

- Block
- 網路連線
- JSON 格式處理

### 練習目標

在屬於 Mobile Internet 的時代裡，我們在寫的手機 App 往往不會是像貪食蛇
這樣的單機遊戲，更有可能是透過網路連線，抓取或上傳資料，讓用戶可以提供
源源不絕的資訊，並且讓用戶與用戶之間溝通。換言之，手機 App 往往就是一
個 Internet Client，KKBOX、甚至 KKBOX 公司內的其他產品線，也是這樣的軟
體。

在寫這樣的 App 的時候，我們通常會把整個 App 所有跟網路連線相關的部份集
中在一起，比方說，我們把前往某個網路服務所有的網路連線呼叫，都放在同一
個 Class 裡頭，不同的 Web API 變成不同的 method，而不是 App 裡頭每個個
別的地方發送連線。最後寫出來的程式就會很像我們可以看到的 Facebook SDK
等 Web Service SDK 或是 library。

這麼寫有很多好處：因為所有的連線呼叫都在一起，所以，哪天我們想寫一個
Mac 版本的時候，就可以把這整塊程式搬到 Mac 上，而不用 Mac 版另外再寫一
次。我們也可以對整個 library 寫單元測試，知道我們設置的連線方式是否正
確，以及 server 是否發生異常。

我們要注意：雖然像 NSString、NSData 都有 `initWithContentsOfURL:` 這些
method，可以直接傳入網路上的 URL，從網路上抓取資料，但我們應該避免在
GUI app 裡頭使用這些 API，因為這些 API 會卡住 UI。我們建議使用
NSURLSession 或 NSURLConnection 等物件發送非同步的連線；由於
NSURLConnection 在 iOS 9 deprecate，我們特別建議使用 NSURLSession，而
且 NSURLSession 的 callback 用的都是本章介紹的 block。

### 練習內容

[httpbin.org](http://httpbin.org) 是一個讓人練習 HTTP 連線的沙箱，我們
要練習寫一個 httpbin.org 的 Web Service SDK。這個服務有很多 API
endpoint，會回傳 JSON、HTML 或圖片格式的資料，我們要使用以下這些 API：

- http://httpbin.org/get
- http://httpbin.org/post
- http://httpbin.org/image/png

包裝成像是以下的 method

``` objc
- (void)fetchGetResponseWithCallback:(void(^)(NSDictionary *, NSError *))callback;
- (void)postCustomerName:(NSString *)name callback::(void(^)(NSDictionary *, NSError *))callback;
- (void)fetchImageWithCallback:(void(^)(UIImage *, NSError *))callback;
```

1. 收到的 JSON 資料要轉成 NSDictionary 物件，請查詢
   NSJSONSerialization 的文件
2. 收到的 image data 要轉成 UIImage 物件
3. 處理 http://httpbin.org/post 這支 API 的時候，我們只 post
   `custname`，像 `custname=kkbox`
4. 這個 library 的每個連線之間都不會互相影響

我們寫完這個 library 之後，也要寫單元測試。但是非同步的單元測試跟我們
前面寫過的不太一樣：要測試非同步的 API，我們必須要在 test case 裡頭等
待測試回應，不然就會在連線還沒完成之前，test case 就結束了。

我們以前遇到非同步的 test case，需要自己跑 Run Loop，或是用 GCD 的
`dispatch_semaphore_t`，可以參考 [Run Loop](responder/run_loop.md) 與
[NSOperation 與 NSOperationQueue](threading/nsoperation_and_nsoperationqueue.md)
這兩章後面的章節。不過，在 Xcode 6 之後，比較容易的作法是使用 XCUnit
提供的 XCTestExpectation；如何在 Xcode 6 中做非同步的測試，請參考

- [Testing in Xcode 6](https://developer.apple.com/videos/wwdc/2014/?id=414)
- [XCTest​Case/XCTest​Expectation/measure​Block()](http://nshipster.com/xctestcase/)
- [Asynchronous Testing With Xcode 6](https://www.bignerdranch.com/blog/asynchronous-testing-with-xcode-6/)

請不要忘了 AAA 原則：

- Arrange: 連線應該要成功，而且應該要正確抓回 dictionary 或 image 物件，
  dictionary 裡頭也應該要有預期的 key，value 也是預期的型別
- Act: 呼叫 method，發送連線
- Assert: 驗證呼叫的結果是否符合預期

另外，如果你打算把這個 class 寫成 singleton 物件，請先跳到後面閱讀
[再談 Singleton](design_patterns/singleton.md) 這一章。
