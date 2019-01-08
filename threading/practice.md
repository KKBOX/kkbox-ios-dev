練習：一個發送多個連線的 Operation
----------------------------------

### 練習範圍

- NSOperation
- GCD

### 練習目標

我們前面在講 block 的時候，嘗試寫過 [httpbin.org](http://httpbin.org)
這個服務的 SDK（見 [練習：將 Web Service API 包裝成 SDK](../blocks/practice_web_service_sdk.md)）。

在前面的練習中，我們每個 method 都只會發送一個連線，但是在現實的軟體開
發過程中，我們往往會想做一些比較複雜的工作，而這樣的工作需要發送好幾個
連線，每個連線之間有相依關係，而這樣的工作也可以中途取消，取消的時候，
要同時取消所有的連線。

我們現在就要練習寫這樣的程式。

### 練習內容

1. 先拿出我們在
   [練習：將 Web Service API 包裝成 SDK](../blocks/practice_web_service_sdk.md)
   當中完成的作業。
2. 寫一個叫做 HTTPBinManager 的 singleton 物件。
3. 在這個 HTTPBinManager 中，增加一個 NSOperationQueue 的成員變數
4. 寫一個叫做 HTTPBinManagerOperation 的 NSOperation subclass，
   HTTPBinManagerOperation 使用 delegate 向外部傳遞自己的狀態。
   HTTPBinManagerOperation 裡頭的 `main` method 依序要執行：
   - 對我們之前寫的 SDK 發送 `fetchGetResponseWithCallback:` 並等候回
     應。
   - 如果前一步成功，先告訴 delegate 我們的執行進度到了 33%，如果失敗
     就整個取消作業，並且告訴 delegate 失敗。delegate method 要在 main
     thread 當中執行。
   - 對我們之前寫的 SDK 發送 `postCustomerName:callback:` 並等候回應。
   - 如果前一步成功，先告訴 delegate 我們的執行進度到了 66%，如果失敗
     就整個取消作業，並且告訴 delegate 失敗。delegate method 要在 main
     thread 當中執行
   - 對我們之前寫的 SDK 發送 `fetchImageWithCallback:` 並等候回應。
   - 如果前一步成功，先告訴 delegate 我們的執行進度到了 100%，並且告訴
     delegate 執行成功，並回傳前面抓取到的兩個 NSDcitionary 與一個
     UIImage 物件；如果失敗就整個取消作業，並且告訴 delegate 失敗。
     delegate method 要在 main thread 當中執行。
5. 這個 operation 要實作 `cancel`，發送 `cancel` 時，要立刻讓operation
   停止，包括清除所有進行中的連線。
6. HTTPBinManager 要加入一個叫做 `executeOperation` 的 method，這個
   method 首先會清除 operation queue 裡頭所有的 operation，然後加入新
   的 HTTPBinManagerOperation。
7. HTTPBinManagerOperation 的 delegate 是 HTTPBinManager。
   HTTPBinManager 也有自己的 delegate，在 HTTPBinManagerOperation 成功
   抓取資料、發生錯誤的時候，HTTPBinManager 也會將這些事情告訴自己的
   delegate。
8. 撰寫單元測試。
9. 寫一個 UI，上面有一個按鈕與進度條，按鈕按下後，就會執行
   HTTPBinManager 的 `executeOperation`，然後進度條會顯示
   HTTPBinManagerOperation 的執行進度。
