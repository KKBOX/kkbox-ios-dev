Perform Selector
----------------

NSOperation 與 NSOperationQueue 是在 Mac OS X 10.5/ iPhoneOS 2.0 的時
候推出的功能，GCD 則是在 Mac OS X 10.6/iOS 4 的時候推出，在這之前，使
用 NSObject 的 performSelector 系列的 API 會是處理 threading 時比較容
易的作法。就像我們之前在 Selector 這一章提到的，我們可以使用以下這些
API，將某些工作放在指定的 Thread 執行：

- -performSelectorOnMainThread:withObject:waitUntilDone:modes:
- -performSelectorOnMainThread:withObject:waitUntilDone:
- -performSelector:onThread:withObject:waitUntilDone:modes:
- -performSelector:onThread:withObject:waitUntilDone:
- -performSelectorInBackground:withObject:

作為比較早期的 API，和 GCD/NSOperationQueue 比較起來，這幾組 API 的最
大缺點便在於不會幫你管理應該要建立多少 thread，全部都得要自己手動管理。
一台機器有其硬體效能的上限，能夠開出多少 thread 也有其上限，如果我們建
立超過硬體效能上限數量的 thread，最糟的狀況會造成整台機器癱瘓。

假如我們現在有一百份工作要在背景執行，如果我們使用 GCD 或是
NSOperationQueue，我們可以把這些工作寫成 block 或是 NSOperation，然後
丟到 GCD 或 NSOperationQueue 中，GCD 或 NSOperationQueue 會根據目前的
硬體效能決定 Thread 的數量，假如目前使用的機器只適合建立三條背景
thread，那麼，就只會建立三條背景 thread，然後將這一百份工作分批放在這
三條 thread 中執行。

但，如果你寫了一個迴圈，在迴圈中呼叫了一百次
`-performSelectorInBackground:withObject:`，那麼，就真的會建立一百個
thread！如果我們想要避免這種狀況，那麼，在使用
`-performSelectorInBackground:withObject:` 時，就有必要自己寫一個
queue，將工作指派在特定的 thread 上。那麼，還是使用 GCD 或是
NSOperationQueue 會比較容易些。

在呼叫 `-performSelector:onThread:withObject:waitUntilDone:modes:` 與
`-performSelector:onThread:withObject:waitUntilDone:` 的時候，我們要特
別注意一下傳入的 NSThread 物件。假如我們在 thread 這個參數傳遞 nil，那
麼，就會造成程式卡在這一行，而不會繼續執行。
