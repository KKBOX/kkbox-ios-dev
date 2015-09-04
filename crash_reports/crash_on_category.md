## 實戰：因為 Category 造成的 Crash

KKBOX 在 2014 年十月推出 iOS 版本 6.0.26 版本，推出之後，就收到不少客
訴反應應用程式在執行到特定的地方時會發生 crash。這件事情非常奇怪，因為
這個版本在我們的開發到 QA 驗證的過程中，從來就沒有在這個地方發生過
crash，如果按照客訴在電話中的描述，我們也完全無法重現問題。

所幸我們可以收集到來自用戶的 crash log。

看到這份 crash log 的第一印象就是用戶做了 JB。用戶用的機種明明就是
iPhone 5S，應該要執行 armv7s 的 library，但是卻載入了一大堆 armv6 的
library，而你看到 CydiaSubstrate 的時候，心底也有個底了。

接著來看 crash 類型，是在
`-[__NSPlaceholderDictionary initWithObjects:forKeys:count:]` 裡頭發生
了 exception，\_\_NSPlaceholderDictionary 是 NSDictionary 的內部實作，
當我們在建立 NSDictionary 的時候，Foundation 其實會回傳的是另外一個介
面相同的 subclass 回來。至於 NSDictionary 會產生的 exception 幾乎都跟
nil 有關—嘗試把 nil 插入到 NSDictionary 裡頭造成的。

我們收集到了 exception 發生時的 console log：

> *** Terminating app due to uncaught exception
> 'NSInvalidArgumentException', reason: '***
> -[__NSPlaceholderDictionary initWithObjects:forKeys:count:]: attempt
> to insert nil object from objects[2]'

我們嘗試建立 Dictionary 的時候，第三筆傳遞進去的資料（objects[2]）是
nil。我們繼續根據解開來的位置，檢查一下
KKExploreCardCollectionItemCell.m 的第 159 行，`drawRect:` 這個 method。

以下程式從 142 行開始

``` objc
 - (void)drawRect:(CGRect)rect
 {
	UIBezierPath *path = [UIBezierPath bezierPathWithRoundedRect:self.bounds cornerRadius:3.0];
	[[UIColor whiteColor] set];
	[path fill];
	[path addClip];

	NSMutableParagraphStyle *paragraphStyle = [[NSMutableParagraphStyle alloc] init];
	paragraphStyle.alignment = NSTextAlignmentLeft;

	if (self.subtitle.length) {
		CGSize titleSize = [self.title boundingRectWithSize:CGSizeMake(self.frame.size.width - 22, 20.0) options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:14.0], NSParagraphStyleAttributeName: paragraphStyle} context:nil].size;
		CGRect titleRect = CGRectMake(11.0, CGRectGetMaxY(imageFrame) + 10.0, titleSize.width, titleSize.height);
		[self.title drawWithRect:titleRect options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:14.0], NSParagraphStyleAttributeName: paragraphStyle, NSForegroundColorAttributeName: [UIColor blackColor]} context:nil];

		CGSize subtitleSize = [self.subtitle boundingRectWithSize:CGSizeMake(self.frame.size.width - 22, 20.0) options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:12.0], NSParagraphStyleAttributeName: paragraphStyle} context:nil].size;
		CGRect subtitleRect = CGRectMake(11.0, self.frame.size.height - 10.0 - subtitleSize.height, subtitleSize.width, subtitleSize.height);
		[self.subtitle drawWithRect:subtitleRect options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:12.0], NSParagraphStyleAttributeName: paragraphStyle, NSForegroundColorAttributeName: [UIColor colorWithHexString:@"#888888"]} context:nil];
	}
	else {
		CGSize titleSize = [self.title boundingRectWithSize:CGSizeMake(self.frame.size.width - 22, 40.0) options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:14.0], NSParagraphStyleAttributeName: paragraphStyle} context:nil].size;
		CGRect titleRect = CGRectMake(11.0, CGRectGetMaxY(imageFrame) + 11.0, titleSize.width, titleSize.height);
		[self.title drawWithRect:titleRect options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:14.0], NSParagraphStyleAttributeName: paragraphStyle, NSForegroundColorAttributeName: [UIColor blackColor]} context:nil];
	}
}
```

當中第 159 行是

```
[self.subtitle drawWithRect:subtitleRect options:NSStringDrawingTruncatesLastVisibleLine | NSStringDrawingUsesLineFragmentOrigin attributes:@{NSFontAttributeName: [UIFont systemFontOfSize:12.0], NSParagraphStyleAttributeName: paragraphStyle, NSForegroundColorAttributeName: [UIColor colorWithHexString:@"#888888"]} context:nil];
```

在這一行中，我們要把 subtitle 這個字串畫到畫面中，我們傳入了三個樣式設
定：字體要是 12 point 大小、一個特定的段落樣式、以及將顏色設定成
「#888888」，傳入三個樣式這件事情，符合前面「建立一個 Dictionary」的條
件，那麼，object[2] 就會是 `[UIColor colorWithHexString:@"#888888"]`
傳回的結果了。

`[UIColor colorWithHexString:@"#888888"]` 為什麼會變成 nil？
