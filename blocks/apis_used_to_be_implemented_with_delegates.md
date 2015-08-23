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
	[UIView setAnimationDidStopSelector:
	  @selector(animationDidStop:finished:context:)];
	self.subview.frame = CGRectMake(10, 10, 100, 100);
	[UIView commitAnimations];
}

- (void)animationDidStop:(NSString *)animationID
                finished:(NSNumber *)finished
                 context:(void *)context
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
NSArray *sortedArray = [array sortedArrayUsingComparator:
    ^NSComparisonResult(id obj1, id obj2) {
	return [obj1 compare:obj2];
}];
```
