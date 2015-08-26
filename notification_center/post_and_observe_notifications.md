接收與發送 Notification
-----------------------

### 接收 Notification

一個通知分成幾個部分

1. **object**: 發送者，是誰送出了這個通知
2. **name**: 這個通知叫做什麼名字
3. **user info**: 這個通知還帶了哪些額外資訊

所以，當我們想要監聽某個通知的時候，便是指定要收聽由誰所發出、哪個名字
的通知，並且指定負責處理通知的 selector，以前面處理 locale 改變的例子來
看，我們就會寫出這樣的 code：

``` objc
- (void)viewDidLoad
{
	[super viewDidLoad];
	[[NSNotificationCenter defaultCenter] addObserver:self
	  selector:@selector(localeDidChange:)
	  name:NSCurrentLocaleDidChangeNotification
	  object:nil];
}

- (void)localeDidChange:(NSNotification *)notification
{
   // 處理 locale 改變的狀況
}

- (void)dealloc
{
	[[NSNotificationCenter defaultCenter] removeObserver:self];
}
```

意思就是，我們要指定 name 為 NSCurrentLocaleDidChangeNotification 的通
知，交由 `localeDidChange:` 處理。在這邊 object 設定為 nil，代表不管是
哪個物件送出的，只要符合 NSCurrentLocaleDidChangeNotification 的通知，
我們統統都處理。

每個通知當中，還可能會有額外的資訊，會夾帶在 NSNotification 物件的
userInfo 屬性中，userInfo 是一個 NSDictionary。

像是我們如果讓某個 text field 變成了 first responder，那麼，在 iOS 上，
就會出現螢幕鍵盤，而當螢幕鍵盤出現的時候，我們往往要調整畫面的 layout，
而螢幕鍵盤在不同輸入法下的大小不一樣，像是跟英文鍵盤比較起來，中文輸入
鍵盤上面往往會有一塊組字區，而造成中文鍵盤比英文鍵盤大。在螢幕鍵盤升起
來的時候，我們會收到 UIKeyboardWillShowNotification 這項通知，而這項通
知就會用 userInfo 告訴我們鍵盤尺寸與位置。

如果我們在 `-addObserver:selector:name:object:` 裡頭，把 name 指定成
nil，就代表我們想要訂閱所有的通知，通常不太會有這種情境，不過有時候你想
要知道系統內部發生了什麼事情，可以用這種方式試試看。

當我們不需要繼續訂閱某項通知的時候，記得對 Notification Center 呼叫
`-removeObserver:`，以上面的程式為例，我們在 add observer 的時候傳入了
self，在 remove observer 的時候，就要傳入 self。我們通常在 dealloc 的
時候停止訂閱。

在 iOS 4 與 Mac OS X 10.6 之後，我們可以使用
`-addObserverForName:object:queue:usingBlock:` 這組使用 block 語法的
API 訂閱通知，由於是傳入 block，所以我們就不必另外準備一個 selector，
可以將處理 notification 的程式與 add observer 的這段呼叫寫在一起。而
remove observer 的寫法也會不太一樣：
`-addObserverForName:object:queue:usingBlock:` 會回傳一個 observer 物
件，我們想要停止訂閱通知的時候，是對 `-removeObserver:` 傳入之前拿到的
observer 物件。範例如下。

Add observer 的時候：

``` objc
self.observer = [[NSNotificationCenter defaultCenter]
	addObserverForName:NSCurrentLocaleDidChangeNotification
	object:nil
	queue:[NSOperationQueue mainQueue]
	usingBlock:^(NSNotification *note) {
	// 處理 locale 改變的狀況
}];
```

Remove observer 的時候：

``` objc
[[NSNotificationCenter defaultCenter] removeObserver:self.observer];
```

### 發送 Notification


至於要發送 notification，則是在建立了 notification 物件之後，對
NSNotificationCenter 呼叫 `-postNotification:` 即可。

這三組 method 都可以用來發送 notification。

``` objc
- (void)postNotification:(NSNotification *)notification;
- (void)postNotificationName:(NSString *)aName
                      object:(id)anObject;
- (void)postNotificationName:(NSString *)aName
                      object:(id)anObject
                    userInfo:(NSDictionary *)aUserInfo;
```

### Notification 與 Threading

當我們訂閱某個 notification 之後，我們並不能夠保證負責處理
notification 的 selector 或 block 會在哪個 thread 執行：這個
notification 是在哪條 thread 送出的，負責接受的 selector 或是 block，
就會在哪條 thread 執行。

在慣例上，絕大多數的 notification 都會在 main thread 送出，之所以說
「絕大多數」，就是因為有例外：像是在 iOS 上，如果我們接上耳機、拔除耳
機，或是將音樂透過 AirPlay 送到 Apple TV 的時候，系統會透過
AVAudioSessionRouteChangeNotification 告訴我們音訊輸出設備改變了[^1]，
這個通知就會發生在背景，而不是 main thread。

不過，當我們在撰寫自己的程式，要發送 notification 的時候，為了考慮其他
開發者會預期在 main thread 收到 notification，所以我們也就在 main
thread 發送 notification，像是透過 GCD，把 postNotification 的呼叫送到
`dispatch_get_main_queue()` 上。



[^1]: 參見蘋果官方文件 Responding to Route Changes - https://developer.apple.com/library/ios/documentation/Audio/Conceptual/AudioSessionProgrammingGuide/HandlingAudioHardwareRouteChanges/HandlingAudioHardwareRouteChanges.html

