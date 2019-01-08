進階的 Accessibility 設定
-----------------------

### 可以調整數值的元件

如果我們寫了一個客製化元件是個 slider，可以讓用戶調整裡頭的數值，那麼，
在 VoiceOver 打開的時候，用戶就可以用向上或向下 swipe 的手勢操作。要實
作這樣的行為，就要實作 `accessibilityIncrement` 與
`accessibilityDecrement`。

在以下的範例中，我們寫了一個叫做 KKProgressSlider（繼承自 UIControl），
實作了 `accessibilityIncrement` 與`accessibilityDecrement，這兩個
method 都會改變 value，增加或減少 5，觸發之後，就會告訴 target 執行對
應到 UIControlEventValueChanged 的 action。

``` objc
- (BOOL)isAccessibilityElement
{
	return YES;
}

- (UIAccessibilityTraits)accessibilityTraits
{
	return UIAccessibilityTraitAdjustable | UIAccessibilityTraitUpdatesFrequently;
}

- (NSString *)accessibilityLabel
{
	return LFLSTR(@"Playing progress");
}

- (void)_updateValueByAccessibility
{
	if (value < self.minimumValue) {
		value = self.minimumValue;
	}
	if (value > self.maximumValue) {
		value = self.maximumValue;
	}
	for (id target in [[self allTargets] allObjects]) {
		NSArray *actions = [self actionsForTarget:target forControlEvent:UIControlEventValueChanged];
		for (NSString *action in actions) {
			[self sendAction:NSSelectorFromString(action) to:target forEvent:nil];
		}
	}
}

- (void)accessibilityIncrement
{
	value += 5.0;
	[self _updateValueByAccessibility];
}

- (void)accessibilityDecrement
{
	value -= 5.0;
	[self _updateValueByAccessibility];
}
```

### UIAccessibilityContainer

很多時候，在使用者介面中的重要文字與圖片，並不是放在直接放在某個UIView
裡頭，當你使用 Core Animation 製作 UI 的時候，就會把很多東西放在
CALayer 裡－某個 layer 裡頭的 content 屬性可能是一張重要的圖，或是用了
CATextLayer 呈現文字。

CALayer 物件本身沒辦法處理無障礙資源。想要讓系統知道某個 layer 的存在
與其所代表的意義，就必須要用一個 container 物件處理，這個 container 通
常是這個 layer 的 super layer 所在的 view，實作
UIAccessibilityContainer protocol。

需要注意，自己實作了UIAccessibilityContainer protocol 之後，會完全改變
這個 view 裡頭的無障礙支援的行為，所以，如果原本這個 view 的 subviews
中還有許多其他的UIView 物件，如果我們自己實作的
UIAccessibilityContainer 並沒有處理到這些 subview，這些 subview 原本具
有的無障礙支援也就會隨之失效。

我們可以針對每一個 CALayer 物件，產生一個對應的 UIAccessibilityElement
物件，然後透過 setAccessibilityFrame: 指定對應的 CALayer 在畫面上的範
圍。

需要注意的是，這邊所指定的位置，必須是這個 layer 在整個應用程式畫面中
的位置，而不是 layer 原本的 frame 屬性，layer 本身的 frame 屬性只是相
對於上一層的 layer （super layer）的位置而已。我們在這裡可以透過呼叫
convertRect:fromLayer:，轉換出目前 layer，與應用程式啟動時，我們所產生
的那個 UIWindow 物件上的 layer，兩者之間的位置關係，這個 CGRect 才是我
們想要的資訊。

當 layer 在畫面上的位置有改變時，最後記得要發一次
UIAccessibilityLayoutChangedNotification。

比方說，我們寫了一個 UIView 的 subclass，裡頭使用 Core Animation Layer
顯示許多圖片，叫做 KKPhotoGridView，這個 view 設計了一個 data source，
每次呼叫 reload data 的時候，都會跟 data source 索取 KKPhotoItemLayer
物件，KKPhotoItemLayer 有個屬性叫做 accessibilityElement，屬於
UIAccessibilityElement。

```
@interface KKPhotoItemLayer : CALayer
@property (strong, nonatomic) UIAccessibilityElement *accessibilityElement;
@end
```

KKPhotoGridView 的 reload data 可以這麼寫：跟 data source 要了 layer
之後，除了把 layer 加入到 layerArray 之外，同時也建立了
UIAccessibilityElement 物件，變成是 layer 的 accessibilityElement 屬性。

```
NSUInteger itemsCount = [photoGridDataSource numberOfItemsInPhotoGridView:self];
for (NSUInteger index = 0; index < itemsCount; index++) {
	KKPhotoItemLayer *itemlayer = [photoGridDataSource
		photoGridView:self layerForItemAtIndex:index];
	UIAccessibilityElement *anElement = [[UIAccessibilityElement alloc]
		initWithAccessibilityContainer:self];
	anElement.accessibilityLabel = LFLSTR(@"Photo");
	anElement.accessibilityIdentifier = [NSString stringWithFormat:@"Photo %lu", (unsigned long)index];
	anElement.accessibilityTraits = UIAccessibilityTraitButton;
	itemlayer.accessibilityElement = anElement;
	[layerArray addObject:itemlayer];
}
UIAccessibilityPostNotification(UIAccessibilityLayoutChangedNotification, nil);
```

最後實作 UIAccessibilityContainer

``` objc
- (BOOL)isAccessibilityElement
{
	return NO;
}

- (NSInteger)accessibilityElementCount
{
	return [layerArray count];
}

- (id)accessibilityElementAtIndex:(NSInteger)index
{
	KKPhotoItemLayer *aLayer = layerArray[index];
	CGRect frame = aLayer.frame;
	aLayer.accessibilityElement.accessibilityFrame = [[UIApplication sharedApplication].keyWindow convertRect:frame fromView:self];
	return aLayer.accessibilityElement;
}

- (NSInteger)indexOfAccessibilityElement:(id)element
{
	NSInteger index = 0;
	for (KKPhotoItemLayer *aLayer in layerArray) {
		if ([aLayer.accessibilityElement isEqual:element]) {
			return index;
		}
		index++;
	}
	return NSNotFound;
}
```
