在 Mac 上使用 Core Animation
---------------------------

前面提到，一個 layer 應該要怎樣設定 content scale（layer 的解析度、裡
頭的內容的精細程度），會跟這個 layer 出現在哪個 window 上有關。在 iOS
上我們通常只需要問 UIScreen 的 main screen 即可，因為 iOS 的 key
window 就只會出現在 main screen 上，而我們通常可以假設 iOS 的外接螢幕
裝置都不會有 retina display。

至於在 Mac 上，由於一台 Mac 可能本身的螢幕具有 Retina Display，但是另
外用 mini display port 等介面外接了其他不是 Retina Display 的螢幕或投
影機，而一個 window 可能會被拖放到不同的 screen 上，所以，當某個
window 移動到某個 screen 上之後，上面的 layer 也要跟著反應，把這些
layer 設定成對應的解析度。

比方說，我們一開始有一個 Window，出現在一台 Retina Display 的 MacBook
Pro 的主要螢幕上，這個 Window 有個 layer，那麼這個 layer 的 content
scale 就要設定成兩倍。但接下來，我們把這個 Window 移動到沒有 Retina
Display 的外接螢幕上，layer 的 content scale 設成兩倍並沒有意義，所以
應該改回變成一倍解析度。

在Mac 上，我們不是去問 screen 的 scale，而是問目前所在 window 的
`backingScaleFactor` 屬性，而當 window 的 `backingScaleFactor` 屬性改
變時，會發送 NSWindowDidChangeBackingPropertiesNotification 通知。

我們通常會在擁有某個 layer 的 view 上面偵測 window 解析度的改變，在
view被加到 window 上時，會呼叫到 NSView 的 `viewDidMoveToWindow`，至於
什麼時候 view 會從 window 上移除呢？就是呼叫 `removeFromSuperview` 的
時候了。所以，我們 override 掉這兩個 method，收聽
NSWindowDidChangeBackingPropertiesNotification 的通知，把目前 window
的 `backingScaleFactor`，設成我們的 layer 的 content scale。

程式碼如下：

``` objc
@implementation MYView

- (void)viewDidMoveToWindow
{
	[[NSNotificationCenter defaultCenter] addObserver:self selector:@selector(scaleDidChange:) name:NSWindowDidChangeBackingPropertiesNotification object:[self window]];
	[self _updateContentScale];
}

- (void)removeFromSuperview
{
	[[NSNotificationCenter defaultCenter] removeObserver:self name:NSWindowDidChangeBackingPropertiesNotification object:[self window]];
	[super removeFromSuperview];
}

- (void)scaleDidChange:(NSNotification *)n
{
	[self _updateContentScale];
}

- (void)_updateContentScale
{
	if (![self window]) {
		return;
	}

	CGFloat scale = [[self window] backingScaleFactor];
	[myLayer setContentsScale:scale];
}
@end
```
