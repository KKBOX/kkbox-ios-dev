什麼時候該用 Blocks？什麼時候該用 Delegate？
--------------------------------------------

即使 block 可以大幅度取代 delegate 處理 callback，但是從蘋果自己的 API
設計中可以看到，並不是所有的 delegate 都被 block 取代，在 Cocoa 與
Cocoa Touch framework 中，仍然大幅度使用 delegate。那麼，我們就要問：
當我們在設計 API 的時候，什麼狀況下應該使用 block？什麼時候又該使用
delegate？

通常的區分方式是：如果一個 method 或 function 的呼叫只有單一的
callback，那麼就使用 block，如果可能會有多個不同的 callback，那麼就使
用 delegate。

這麼做的好處是：當一個 method 或 function 呼叫會有多種 allback 的時候，
很有可能某些 callback 是沒有必要實作的。

如果使用 delegate 實作，那麼，在 delegate 需要實作的 protocol 中，我們
可以用 `@required` 與 `@optional` 關鍵字區分哪些是一定需要實作的
delegate method。

但相對的，用 block 處理 callback，就會很難區分某個 block 是否是必須要
實作：在Xcode 6.3 之前，Objective-C 並沒有 `nullable` 、`nonnull` 等關
鍵字，讓我們知道某個 property、或某個 method 要傳入的 block 可不可以是
nil，我們也往往搞不清楚在這些地方傳入 nil，會不會發生什麼危險的事情。

舉個例子。在 iOS 7 之後，蘋果鼓勵開發者使用 NSURLSession 處理網路連線，
NSURLSession 就充分表現了「單一 callback 用 block、多重 callback 用
delegate」這一點。

假如我們現在想要把 KKBOX 的官網首頁抓下來，我們只要建立一個
NSURLSessionDataTask 物件，一般來說，我們只需要處理「這個連線做完事情
的下一步該做什麼」，所以一般也只需要實作這個 task 的 completion
handler，就是傳入網路連線結束之後要執行的 block；一般連線結束，大概就
是成功抓到資料或是連線失敗兩種狀況，所以我們可以透過 data 與 error這兩
物件判斷是哪種狀況：失敗的話，error 就不會是 nil，我們就要處理 error，
反之就要處理 data。

``` objc
NSURL *URL = [NSURL URLWithString:@"http://kkbox.com"];
NSURLRequest *request = [NSURLRequest requestWithURL:URL];
NSURLSessionDataTask *task = [[NSURLSession sharedSession]
  dataTaskWithRequest:request
    completionHandler:^(NSData *data, NSURLResponse *response, NSError *error) {
	if (error) {
		// handle error
		return;
	}
	// handle data
}];
[task resume];
```

但，NSURLSession 本身也還是具有 delegate。我們在發送連線的時候，除了處
理連線結束要做什麼之外，有時候也可能會想處理在連線中途所發生的其他狀況，
像是：HTTP 連線收到 302 轉址、遇到有問題的 SSL 憑證、server 要求用戶輸
入帳號密碼，這些狀況我們要不要提示使用者？或，如果這是一個傳遞大檔、很
花時間的連線，我們有沒有必要顯示連線進度條？這些狀況還是會傳遞給
NSURLSession 的 delegate，而如果我們要處理這些狀況，就要實作以下這些
delegate methods。

``` objc
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
	willPerformHTTPRedirection:(NSHTTPURLResponse *)response
	newRequest:(NSURLRequest *)request
	completionHandler:(void (^)(NSURLRequest *))completionHandler;
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
	didReceiveChallenge:(NSURLAuthenticationChallenge *)challenge
	completionHandler:(void (^)(NSURLSessionAuthChallengeDisposition disposition, NSURLCredential *credential))completionHandler;
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
	needNewBodyStream:(void (^)(NSInputStream *bodyStream))completionHandler;
- (void)URLSession:(NSURLSession *)session task:(NSURLSessionTask *)task
	didSendBodyData:(int64_t)bytesSent
	totalBytesSent:(int64_t)totalBytesSent
	totalBytesExpectedToSend:(int64_t)totalBytesExpectedToSend;
```
