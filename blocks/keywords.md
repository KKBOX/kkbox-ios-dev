__block 關鍵字
--------------

在一個 block 裡頭如果使用了在 block 之外的變數，會將這份變數先複製一份
再使用，也就是說，在沒有特別宣告的狀況下，對我們目前所在的 block 來說，
所有外部的變數都是唯讀，只能讀取，不能變更。至於 block 裡頭用到的
Objective-C 物件，則都會被多 retain 一次。

如果我們想要讓某個 block 可以改動某個外部的變數，我們就要在這個需要可
以被 block 改動的變數前面，加上 __block 關鍵字。

像這樣是不合法的程式：

``` objc
int i = 1;
void (^block)(void) = ^{
	i = i + 1;
};
```

應該寫成：

``` objc
__block int i = 1;
void (^block)(void) = ^{
	i = i + 1;
};
```

__weak 關鍵字
-------------

在使用了 block 之後，記憶體管理會變得非常複雜，所以最好是在開啟了 ARC
自動記憶體管理之後再使用 block。不過，即使開啟了 ARC，還是可能會遇到循
環 retain 的問題。

由於 block 中用到的 Objective-C 物件都會被多 retain 一次，這邊所指的
Objective-C 物件也包含 self，所以，假使有個物件的 property 是一個block，
而這個 block 裡頭又用到了 self，就會遇到循環 retain 而無法釋放記憶體的
問題：self 要被釋放才會去釋放這個 property，但是這個 property 作為
block 又 retain 了 self 導致 self 無法被釋放。

下面這段 code 就有循環 retain 的問題：

``` objc
@interface MyClass : NSObject
- (void)doSomthing;
@property (copy, nonatomic) void (^myBlock)(void);
@end

@implementation MyClass

- (instancetype)init
{
	self = [super init];
	if (self) {
		self.myBlock = ^ {
			[self doSomthing];
		};
	}
	return self;
}
- (void)doSomthing
{
}
@end
```

如果我們不想讓 self 被 myBlock 給 retain 起來，我們就要把 self 變成
weak reference 再傳入到 block 中。像是改成這樣：

``` objc
__weak MyClass *weakSelf = self;
self.myBlock = ^ {
	[weakSelf doSomthing];
};
```
