GCD（Grand Central Dispatch）
----------------------------

如果我們有一件工作，想要在某條指定的 thread 上執行，現在最簡單的方法大
概就是呼叫 GCD。GCD 其實包含相當多的 API，是一群 C function 的組合 ，
其中，我們最常用的是`dispatch_async`。

### dispatch_async

`dispatch_async` 這個 function，可以讓我們選擇要在哪個指定的 thread 上，
用非同步的方式執行一個 block。比方說，我們現在在前景，但是想要在背景執
行一件工作，就會這麼寫：

``` objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	[someObject doSomethingHere];
});
```

`dispatch_get_global_queue` 這個 function 會讓系統根據目前的狀況，在適
當時機建立一條 thread，第一個參數是這條 thread 執行工作的優先程度，優
先程度會從 2 到 -2 安排，2 為最重要，-2 為最不重要；至於第二個參數則是
保留參數，目前都沒有作用，直接填 0 即可。

如果我們已經在背景了，想要在 main thread 執行工作，那麼，就把
`dispatch_get_global_queue` 換成 `dispatch_get_main_queue`

``` objc
dispatch_async(dispatch_get_main_queue(), ^{
	[someObject doSomethingHere];
});
```

我們經常會先讓某個工作在背景執行，執行完畢之後，再繼續在 main thread
更新 UI，讓用戶知道這件工作已經執行完畢，我們便可以組合前面兩個呼叫：

``` objc
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
	[someObject doSomethingInBackground];
	dispatch_async(dispatch_get_main_queue(), ^{
		[someObject doSomethingOnMainThread];
	});
});
```

如果我們想要讓好幾件工作都在背景執行，而每件工作並非平行執行，而是一件
工作做完之後，再繼續下一件，我們便可以使用 serial 的 queue。像這樣：

``` objc
dispatch_queue_t serialQueue = \
    dispatch_queue_create("com.kkbox.queue", DISPATCH_QUEUE_SERIAL);

dispatch_async(serialQueue, ^{
    [someObject doSomethingHere];
});

dispatch_async(serialQueue, ^{
    [someObject doSomethingHereAsWell];
});
```

### dispatch_sync

不同於 `dispatch_async` 會做平行處理，呼叫 `dispatch_sync` 的時候，則
是會先把 `dispatch_sync` 的這個 block 做完之後，才繼續執行到程式的下一
行。

我們在呼叫 `dispatch_sync` 的時候要特別注意：如果我們已經在某一條
thread 中，而呼叫 `dispatch_sync` 時所傳入的 thread 就是目前所在的
thread，那麼會造成程式執行時卡死。比方說，我們已經在 main thread 了，
但我們卻呼叫：

``` objc
dispatch_sync(dispatch_get_main_queue(), ^{
    [someObject doSomethingHere];
});
```

這段程式就會卡住。我們可以用 NSThread 的一些 method 檢查我們目前正在哪
條 thread，例如使用 `+isMainThread` 檢查是否是 main thread。

### 其他一些好用的 API

和 `dispatch_async` 與 ``dispatch_sync` 相較，底下這些 API 會比較少用，
但是可以解決不少麻煩的問題。

#### dispatch_once

`dispatch_once` 保證某個 block 只會被執行一次，現在大家最常使用這個特
性實作 singleton。我們在「[再談 Singleton](singleton/README.md)」這一
章當中也提過。

#### dispatch_after

可以延後執行某個 block 在某個指定的 dispatch queue 上執行，我們可以用
這個 function 代替 timer。

``` objc
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
	[someObject doSomething];
});
```

#### dispatch_apply

如果我們想要重複執行某個 block，就可以考慮使用 `dispatch_apply`。
`dispatch_apply` 有三個參數，第一個參數是要執行的次數，第二個參數則是
要在哪個 dispatch queue 上執行：就像前面提到的，如果想要平行執行，就呼
叫 `dispatch_get_global_queue`，如果想要依序執行，就建立一個 serial 的
dispatch queue。
