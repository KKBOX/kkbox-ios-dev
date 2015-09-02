Application
-----------

Mac OS X 與 iOS 上的 application 不太相同，在 Mac OS X 上是
NSApplication，在 iOS 上則是 UIApplication，但基本上都負責相同的工作：
把來自外部的種種傳遞給內部，包括硬體事件，與其他各種系統事件。

硬體事件會被傳遞給 window，而其他系統事件，像 App被開啟或關閉、被推到
前景或背景、收到 push notification…等等，則是會轉發給 application 的
delegate。

由於 application 位在 responder chain 的最底層，每一個 view 與 window
都不處理的時候，才會丟給 application 處理，所以如果我們希望處理一些會
影響整個 App 行為的事件的時候，就適合由 application 這一層處理。

比方說，KKBOX 是一個音樂 App，所以我們會希望用戶可以透過藍芽耳機或線控
耳機上的按鈕，切換 KKBOX 當中的歌曲，換到前一首或下一首歌曲。在
iOS 7.1 之前，我們要處理線控耳機的事件，會選擇實作 UIResponder
protocol 中的 `-remoteControlReceivedWithEvent:`。因為換歌這件事情應該
是對整個 KKBOX 的操作，無論放在哪個 view 或 view controller 都不適合，
所以應該要放在 application 這一層。

要讓 application 這一層可以做額外的事情，我們首先要建立自己的
UIApplication subclass：

``` objc
@interface KKApplication : UIApplication
@end
```

然後，在 main.m 裡頭，告訴 `UIApplicationMain`，我們應該要使用
KKApplication，而不是原本的 UIApplication 的實作：

``` objc
int main(int argc, char * argv[]) {
    @autoreleasepool {
    return UIApplicationMain(argc, argv,
  	  NSStringFromClass([KKApplication class]),
	  NSStringFromClass([AppDelegate class]));
    }
}
```

我們就可以在 KKApplication 處理事件了：

``` objc
@implementation KKApplication
- (void)remoteControlReceivedWithEvent:(UIEvent *)theEvent
{
	if (theEvent.type == UIEventTypeRemoteControl) {
		switch(theEvent.subtype) {
			case UIEventSubtypeRemoteControlPlay:
				break;
			case UIEventSubtypeRemoteControlPause:
				break;
			case UIEventSubtypeRemoteControlStop:
				break;
			case UIEventSubtypeRemoteControlTogglePlayPause:
				break;
			case UIEventSubtypeRemoteControlNextTrack:
				break;
			case UIEventSubtypeRemoteControlPreviousTrack:
				break;
		    ...
			default:
				return;
		}
	}
}
@end
```

當然，如果我們想要開始接收來自耳機的事件，我們還要對 UIApplication 的
singleton 物件呼叫 `-beginReceivingRemoteControlEvents:`。

雖然跟 application 這一層無關，不過提到了耳機線控，就得提一下。蘋果在
推出 iOS 7.1 的時候，同時推出了 Car Play 功能，Car Play 允許用戶在車用
音響的介面上控制 iOS App，由於車用音響的畫面較大，所以，除了可以用來切
換前後首歌曲之外，蘋果還加入了可以對歌曲評分，表示喜歡或不喜歡等功能，
於是整個改寫了處理耳機線控的這一塊，推出 MPRemoteCommandCenter 這個
class。

從 MPRemoteCommandCenter 的 singleton 物件 `sharedCommandCenter` 上，
我們可以拿到許多種不同的 MPRemoteCommand，然後對 MPRemoteCommand 設定
target/action。我們之前想要開始播放，會在
`-remoteControlReceivedWithEvent:` 裡頭處理
UIEventSubtypeRemoteControlPlay，現在會改成向 MPRemoteCommandCenter 要
求 `playCommand`，然後指定 target/action，例如：

``` objc
[[MPRemoteCommandCenter sharedCommandCenter].playCommand addTarget:self action:@selector(play:)];
```
