# 實戰：背景作業執行太久

這是 Facebook iOS App 的 crash log。我們看到 BKProcessAssertion 這個關
鍵字，就可以大致知道，這個 crash 跟背景作業有關。

從 iOS 4 開始，iOS 支援背景作業，背景作業分成時間有限與時間無限兩種。
時間無限是指必須要在 Info.plist 指定特定 background mode 的那種，像
KKBOX 是一個可以在背景播放音樂的 App，就是因為設定了 audio 這種
background mode。

而其實所有的 App 在不用設定 background mode 的狀況下，在進入背景的時候，
都還可以用 UIApplication 的
`beginBackgroundTaskWithName:expirationHandler:` 執行一個 block，用這
個 block 把想做的事情做完，但是這個 block 有時間限制，在 iOS 7 之前最
長十分鐘，iOS 7 之後只有三分鐘，如果超過時間，就會被系統強制結束。被強
制結束的時候是什麼樣子呢？就是這個 BKProcessAssertion 了。

在蘋果官方文件
[App Programming Guide for iOS - Background Execution](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html)
中，大概示範了 `beginBackgroundTaskWithName:expirationHandler:` 的用法，
我們就用蘋果的 Sample code 說明問題出在哪裡。

``` objc
- (void)applicationDidEnterBackground:(UIApplication *)application
{
    bgTask = [application beginBackgroundTaskWithName:@"MyTask" expirationHandler:^{
        // 在這邊你可以寫想要背景執行的 code
	    // 但事情做太久，還是會被強制結束
        // 強制結束的時候會產生

	    [application endBackgroundTask:bgTask];
        bgTask = UIBackgroundTaskInvalid;
    }];

    // Start the long-running task and return immediately.
    dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
        [application endBackgroundTask:bgTask];
        bgTask = UIBackgroundTaskInvalid;
    });
}
```

不過 Facebook 這個 crash log 有個地方還頂值得玩味。出問題的 thread 是
第四條叫做 fbtelephonycache 的 thread，telephony 看名字是跟撥打電話有
關，而我自己想不出來 Facebook 跟撥打電話有什麼關係，也搞不清楚這條
thread 在 cache 些什麼。
