Blocks
======

蘋果在 Mac OS X 10.6 與 iOS 4 之後導入 Block 語法，之後就大幅改變了撰
寫 Objectice-C 語言的方式。

Block 是 Cocoa/Cocoa Touch Framework 中的匿名函式（Anonymous Functions）
的實作。所謂的匿名函式，就是一段 **具有物件性質的程式碼**，這一段程式
碼可以當做函式執行，另一方面，又可以當做物件傳遞；因為這種特性，最常使
用block 的時機，就是拿 block 實作 callback。

在有 block 之前，在 Cocoa/Cocoa Touch Framework 上要處理 callback，最
常見的就是使用 delegate（此外也可以使用比較具有 C 語言風格的方式，傳遞
callback function 的 pointer，或是使用 target/action pattern）。在 iOS
4 有了 block 之後，可以看到蘋果自己便大幅改寫了 UIKit 等 Framework 的
API，把原本使用 delegate 處理 callback 的地方，都大幅換成了 block。

Block 的語法
------------

一直以來還是有不少人不滿 block 語法，甚至有人成立了一個叫做
[fuckingblocksyntax.com](http://fuckingblocksyntax.com) 的網站。這個網
站的網域名稱不怎麼優雅，不過裡頭倒是清楚整理了我們應該如何宣告 block。

將 block 宣告成變數（local variable）的語法是：

``` objc
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};
```

宣告成 Objective-C property 的語法是：

``` objc
@property (nonatomic, copy) returnType (^blockName)(parameterTypes);
```

宣告成 method 的參數（method parameter）的語法是：

``` objc
- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;
```

在執行某個需要傳入 block 當做參數的 method 的時候，則是用這以下方式呼
叫。這也是絕大多數用 block 當做 callback 的處理方式：

``` objc
[someObject someMethodThatTakesABlock:^returnType (parameters) {...}];
```

把一種 block 宣告成 typedef：

``` objc
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^returnType(parameters) {...};
```

Block 如何代替了 Delegate
-------------------------

要了解在哪些場合使用 block，我們不妨先看一下蘋果自己如何使用 block。

首先是 UIView 動畫。當我們在某個 view 上面改變了某些 subview 的位置與
大小，或是改變了這個 view 的一些屬性，像是背景顏色等，一般來說不會產生
動畫效果，我們的改動會直接生效，但是我們也可以產生一段動畫效果，這種動
畫我們稱為 UIView Animation。（UIView Animation 其實底層是透過 Core
Animation 完成的，但我們稍晚才會討論 Core Animation。）

像是，我們想要改動某個 subview 的 frame：

``` objc
self.subview.frame = CGRectMake(10, 10, 100, 100);
```

在 iOS 4 之前，我們會使用 UIView 的 `+beginAnimations:context:` 與
`+commitAnimations` 兩個 class method，把原本的 code 包起來，那麼，在
這兩個 class method 之間的程式碼就會產生動畫效果。

``` objc
[UIView beginAnimations:@"animation" context:nil];
self.subview.frame = CGRectMake(10, 10, 100, 100);
[UIView commitAnimations];
```

如果我們想要在這段動畫結束的時候做一件事情，像是執行另外一個動畫，我們
應該怎麼做呢？iOS 4 之前唯一的方法就是透過 UIView 的 delegate，我們在
執行動畫之前，需要先設定好 delegate，以及要執行 delegate 上的哪個
selector。像是：

``` objc
- (void)moveView
{
	[UIView beginAnimations:@"animation" context:nil];
	[UIView setAnimationDelegate:self];
	[UIView setAnimationDidStopSelector:@selector(animationDidStop:finished:context:)];
	self.subview.frame = CGRectMake(10, 10, 100, 100);
	[UIView commitAnimations];
}

- (void)animationDidStop:(NSString *)animationID finished:(NSNumber *)finished context:(void *)context
{
	// do something
}
```

可以看到，如果使用 delegate pattern，一段連續的流程，會分散在很多不同
的 method 中。有了 block 語法之後，我們可以將「動畫該做什麼」與「動畫
完成之後要做什麼」，寫成一段集中的程式碼。像是：

``` objc
- (void)moveView
{
	[UIView animateWithDuration:0.25 animations:^{
		self.subview.frame = CGRectMake(10, 10, 100, 100);
	} completion:^(BOOL finished) {
		// Do something
	}];
}
```

另外像 NSArray 裡頭的物件排序，以往我們必須以 C function pointer 或是
selector 形式傳入用來比較物件大小的 comparator。像是：

``` objc
NSArray *array = @[@1, @2, @3];
NSArray *sortedArray = [array sortedArrayUsingSelector:@selector(compare:)];
```

也可以改用 block 語法：

``` objc
NSArray *array = @[@1, @2, @3];
NSArray *sortedArray = [array sortedArrayUsingComparator:^NSComparisonResult(id obj1, id obj2) {
	return [obj1 compare:obj2];
}];
```


什麼時候該用 Blocks？什麼時候該用 Delegate？
--------------------------------------------




Block 作為 Objective-C 物件
---------------------------

前面提到，Block 其實可以當成是一種物件，由於 Objective-C 物件需要做記
憶體管理，因此，如果你還在手動管理記憶體，在建立 Block 的時候，也必須
手動管理 Block 的記憶體。我們可以使用 `Block_Copy()` 和
`Block_Release()` 這兩個 C Function




相關閱讀：
---------

- [蘋果官方文件 Blocks Programming Topics](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Blocks/Articles/00_Introduction.html#//apple_ref/doc/uid/TP40007502-CH1-SW1)
