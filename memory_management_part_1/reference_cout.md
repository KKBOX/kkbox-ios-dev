Reference Count/Retain/Release
------------------------------

Objective-C 語言裡頭每個物件，都是指向某塊記憶體的指標，在 C語言當中，
你會使用像是 `malloc` 、 `calloc` 這些 function配置記憶體，用完之後，
就呼叫 `free`釋放記憶體。但是我們如何知道一塊記憶體被多少地方用到，之
後這些地方又不再用到呢？所以在Objective-C語言發展之初，就建立了一套計
算有多少地方用到某個物件的簡單機制，叫做*reference count*，意義非常簡
單：只要一個物件被某個地方用到一次，這個地方就對這個物件加一，反之就減
一，如果數字減到變成了零，就該釋放這塊記憶體。

一個物件如果使用 `alloc` 、 `init` …產生，初始的 reference count 為
1。接著，我們可以使用 `retain` 增加 reference count。

``` objc
[anObject retain]; // 加 1
```

反之就用 `release` ：

``` objc
[anObject release]; // 減 1
```

我們可以使用 `retainCount` 檢查某個物件被 retain 了多少次。

``` objc
NSLog(@"Retain count: %d", [anObject retainCount]);
```

有了基本概念之後，我們就可以看出以下程式有什麼問題

``` objc
id a = [[NSObject alloc] init];
[a release];
[a release];
```

因為在第二行，a所指向的記憶體已經被釋放了，所以第三行想要再釋放一次，
就會造成錯誤。同樣的：

``` objc
id a = [[NSObject alloc] init];
id b = [[NSObject alloc] init];
b = a;
[a release];
[b release];
```

在第三行中，由於 b 指向了 a 原本所指向的記憶體，但是 b原本所指向的記憶
體卻沒有釋放，同時再也沒有任何變數指向 b原本指向的記憶體，因此這塊記憶
體就發生了記憶體漏水。接著，在第四行呼叫`[a release]` 時，這塊記憶體就
已經被放掉了，但是由於 a 與 b都已經指向了同一塊記憶體，所以第五行的
`[b release]`也是操作同一塊記憶體，於是會發生 `EXC_BAD_ACCESS` 錯誤。
