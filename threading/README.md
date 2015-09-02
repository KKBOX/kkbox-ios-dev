Threading
=========

在前一章提到，iOS 有一項系統限制：如果我們的程式中某個操作超過一定時間，
那麼系統就會認為我們的應用程式沒有回應，而會強制關閉我們的 App，當 App
被強制關閉的時候，外觀上會非常像是當機。如果是在 Mac 上，就會看到滑鼠
圖示變成沙灘球不斷旋轉，最後 App 一樣被強制關閉。

但是我們的 App 往往會有許多需要花時間的操作，像是去網路上抓取資料，或
是 local 的檔案處理。所以，在這些場合，我們需要將工作丟到背景 thread
執行，在工作完成之後、或是需要更新進度的時機，才告訴 main thread 更新
UI。

以網路連線來說，我們會避免使用 NSData 或 NSString 的
`-initWithContentsOfURL:` 這個 API，而使用 NSURLSession 或
NSURLConnection 發送非同步的連線，NSURLSession 與 NSURLConnection在做
的事情，便是將抓取資料這件工作放在其他 thread 中執行，然後在必要的時候
callback —在這邊我們要順道注意一下，其實像 NSURLSession 的 data task
的 callback block，也是在背景 thread 中執行。

在 iOS 與 Mac OS X 上，我們可以呼叫低階的 POSIX thread，不過既然有比較
高階的 API，我們自然會選擇使用高階 API。我們通常在 iOS 與 Mac OS X 上
使用三種方式處理 Multi-thread 的問題，分別是：

- Perform Selector
- GCD （Grand Centeral Dispatch）
- NSOperation 與 NSOperationQueue
