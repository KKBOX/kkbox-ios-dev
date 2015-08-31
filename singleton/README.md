再談 Singleton
==============

我們現在用 Objective-C 實作 Singleton 的時候，大概都是按照 Mike Ash 的
建議—參見
*[Friday Q&A 2009-10-02: Care and Feeding of Singletons](https://mikeash.com/pyblog/friday-qa-2009-10-02-care-and-feeding-of-singletons.html)*─
使用 GCD 裡頭的 `dispatch_once` 實作。大概像這樣：

``` objc
@interface MyClass : NSObject
+ (instancetype) sharedInstance;
@end

@implementation MyClass

+ (instancetype) sharedInstance
{
	static MyClass *instance = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		instance = [[MyClass alloc] init];
	});
	return instance;
}
@end
```

之所以這麼寫的原因，是為了避免在多重 thread 的環境下，shared instance
可能會被重複建立的問題。我們來看如果只用 if 語法實作的 singleton：

``` objc
@implementation MyClass
+ (instancetype) sharedInstance
{
	static MyClass *instance = nil;
	if (!instance) {
		instance = [[MyClass alloc] init];
	}
	return instance;
}
@end
```

在 instance 這個變數還沒有建立的狀況下，假如就已經有多個 thread 同時進
入了前面這段程式的 if 判斷式當中，那麼，每個進入這段程式的 thread，都
會分別執行到 `instance = [[MyClass alloc] init];` 這一行，而重複建立
instance。我們對於Singleton 物件的要求是：這個 Class 只會建立一個
instance，在所有地方呼叫 `+sharedInstance`，也都只該拿到同一個物件，但
是在這種寫法中，不同 thread 呼叫 `+sharedInstance`，就可能會拿到不同的
物件。

由於 `dispatch_once` 裡頭的 block 只會執行一次，我們便透過這個特性，保
證 `instance = [[MyClass alloc] init];` 只會被呼叫到一次。
