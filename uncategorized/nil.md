NULL、nil、Nil…
---------------

在 Objective-C 語言中，有很多個代表「沒有東西」的東西，一開始也很容易
搞混。包括：

- NULL
- nil
- Nil
- NSNull
- NSNotFound

### NULL

NULL 其實並不算是 Objective-C 的東西，而是屬於 C 語言。NULL 就是 C 語
言當中的空指標，是指向 0 的指標。絕大多數狀況下，nil、Nil 與 NULL 可以
代替使用，但是語意上，當某個 API 想要你傳入某個指標（void *）時，而不
是 id 型別時，雖然你可以在這種狀況下傳入 Objective-C 物件指標，也就是
可以傳入 nil，但是傳入 NULL 意義會比較清楚。

像建立 NSTimer 時，API 在 userInfo 這邊要求的是 id，我們傳入 nil 會比
較好：

``` objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)seconds
	target:(id)target
	selector:(SEL)aSelector
	userInfo:(id)userInfo
	repeats:(BOOL)repeats
```

而像 UIView 的 `beginAnimations:context` 的定義是：

```
+ (void)beginAnimations:(nullable NSString *)animationID context:(void *)context;
```

傳入 NULL 就會比較好。


### nil


### Nil


### NSNull


### NSNotFound
