GCD（Grand Central Dispatch）
----------------------------

如果我們有一件工作，想要在某條指定的 thread 上執行，現在最簡單的方法大
概就是呼叫 GCD。

### dispatch_async

GCD 其實包含相當多的 API，我們最常用的是`dispatch_async`，這個
function 可以讓我們選擇要在哪個指定的 thread 上，用非同步的方式執行一
個 block。比方說，我們現在在前景，但是想要在背景執行一件工作，就會這麼
寫：

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

