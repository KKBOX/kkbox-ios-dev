練習：將 Web Service API 包裝成 SDK Part 2
------------------------------------------

### 練習範圍

- Delegate

### 練習目標

一個 Web Serive SDK 大概可以分成兩種模式，一種是這個 library 發送出去
的連線本身不會互相干擾，另外一種則是會互相取消。

比方說，我們現在在 UI上有一個搜尋功能，用戶打了一個關鍵字之後，前一個
搜尋結果還沒有回傳，用戶又打了一個關鍵字，這個時候我們就必須先把前一個
已經發送出去的連線取消掉。原因是：我們並不能夠保證每個連線的時間都是一
樣的，如果第一個連線花了更多的時間才抓回資料，第二個連線都結束了一陣子
之後，第一個連線才回來，那麼，最後用戶看到的是第一個連線的結果，而這並
不符合用戶的期待。

我們現在要寫的就是一套會互相取消連線的 SDK。

### 練習內容

[httpbin.org](http://httpbin.org) 是一個讓人練習 HTTP 連線的沙箱，我們
要練習寫一個 httpbin.org 的 Web Service SDK。這個服務有很多 API
endpoint，會回傳 JSON、HTML 或圖片格式的資料，我們要使用以下這些 API：

- http://httpbin.org/get
- http://httpbin.org/post
- http://httpbin.org/image/png

包裝成像是以下的 method

``` objc
- (void)fetchGetResponse;
- (void)postCustomerName:(NSString *)name;
- (void)fetchImageWithCallback;
```

我們的連線物件只有一個單一的 delegate，當這些 method 的連線完成之後，
都會將資料透過一個我們設計好的 protocol 傳給 delegate 物件。在呼叫這些
method 的時候，如果發現有任何還在進行中的連線，都要先取消原本的連線，
才發送新的連線。

這個練習一樣要寫單元測試。
