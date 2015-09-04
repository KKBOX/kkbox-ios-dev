## 實戰：因為 Category 造成的 Crash

KKBOX 在 2014 年十月推出 iOS 版本 6.0.26 版本，推出之後，就收到不少客
訴反應應用程式在執行到特定的地方—瀏覽線上精選畫面—時會發生 crash。這件
事情非常奇怪，因為這個版本在我們的開發到 QA 驗證的過程中，從來就沒有在
這個地方發生過crash，如果按照客訴在電話中的描述，我們也完全無法重現問
題。

所幸我們可以收集到來自用戶的 crash report。

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
KKExploreCardCollectionItemCell 是線上精選頁面 collection view 裡頭使
用到的一個 cell，符合「瀏覽線上精選」會 crash 這條描述。

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

`colorWithHexString:` 是 KKBOX 使用到的一個 UIColor category，可以根據
傳入的色碼（Hex code）產生 UIColor 物件，平常使用都沒問題，為什麼會在
用戶的環境裡頭變成 nil？

還記得我們在第一章就提到，在 Objective-C 裡頭，由於 Objective-C 的動態
特性，所以每一個 method 都有可能在 run time 被換掉？由於用戶 JB 過，所
以可以載入額外的 library，很有可能在用戶所載入的眾多 library 中（至於
具體來說是那一個呢？鬼才曉得），也有名稱一樣叫做 `colorWithHexString:`
的 method，把我們原本的實作換掉了，更換之後的 `colorWithHexString:` 實
作並不認得 @"#888888"，回傳 nil，接著我們又把 nil 插入 NSDictionary。

你可能聽說過，JB 會造成系統不穩定，這就是 JB 造成系統不穩定的好例子：
因為 JB 之後，額外安裝的 library 會導致軟體本身的行為改變，超過了開發
人員的預料，於是不會 crash 的地方也 crash 了。

那我們可以做什麼呢？我們可以把自己這份 `colorWithHexString:` 換個名字，
避免與別人載入的 UIColor category 同名，所以現在許多人也建議在
category 名稱前方加上自己的 prefix，我們可能要改名叫做
`kk_colorWithHexString:`，但，這麼做的目的是為了避免讓 App 在用戶 JB
過的環境下 crash，著實讓人產生強烈的無力感。

而這種問題在測試階段是無法發現的。我們不可能測試所有 JB 之後的各種環境
的組合。
