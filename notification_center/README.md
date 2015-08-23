Notification Center
===================

Notification Center 是在 Cocoa/Cocoa Touch Framework 中，物件之間可以
不必互相知道彼此的存在，也可以互相傳遞訊息、交換資料/狀態的機制。

我們可以把 Notification Center 想像是一種廣播系統。當一個物件 A 的狀態
發生改變，而有多個物件需要知道這個物件發生改變的狀況下，物件 A 不必直接
對這些物件發出呼叫，而是告訴一個廣播中心說：「我的狀態改變了」，至於其
他需要聽取狀態的物件呢，也只要對這個廣播中心訂閱（subscribe）指定的通知，
所以當物件 A 發出通知的時候，這個廣播中心就會通知有訂閱通知的其他物件。
這個廣播中心，就是 Notification Center。

我們經常使用 Notification Center 處理來自作業系統的事件。假如我們現在
寫了一個日記軟體，這個軟體裡頭已經有很多 view，每個 view 裡頭都有一篇
日記，每篇日記上都有該篇日記的撰寫日期與時間。我們通常會使用
NSDateFormatter，使用系統偏好設定中的語系（Locale）設定，將日期轉成符
合語系設定的字串顯示，那麼，當用戶調整了系統偏好設定，像是把中文改成英
文，那麼，我們原本用中文顯示的日期，也應該馬上變成用英文顯示─我們該怎
麼做呢？

我們最常使用的通知中心是 NSNotificationCenter 這個 class，我們也通常使
用這個 class 的 singleton 物件 default center（也就是說，其實
Notification Center 有好幾個，不過我們最常使用的還是這個）。當系統語系
改變的時候，Notification Center 就會發出叫做
NSCurrentLocaleDidChangeNotification 的這項通知。所以，我們所有要顯示
日期的畫面物件，都應該要訂閱這個通知，在收到通知的時候，就要重新產生日
期字串。

接收 Notification
-----------------

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

發送 Notification
-----------------

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

Notification 與 Threading
-------------------------

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

Notification Queue
------------------

有的時候，我們的程式可能會在很短的時間送出大量的 notification，而造成
資源的浪費或效能問題。

以 KKBOX 來說，我們在歌單中的歌曲物件發生改動的時候，會透過
notification 更新 UI：一首歌曲可能會出現在多個歌單中，而我們可能用了很
多不同的 UI 物件來呈現不同張歌單，因此，像一首歌曲的播放次數改變、如這
首歌被多播放了一次，歌曲物件就透過 notification center，告訴每個跟歌單
UI 相關的物件重新讀取歌單資料。

照理說，只要有一首歌曲改變，就該發出這種通知，但假如我們現在做的事情是
歌單同步—把另外一台裝置上的歌單資料，同步到我們這台裝置上，那麼改動的
就不只是一首歌曲，而是一大批的歌曲，如果有十首歌，就送出了十次通知；但
是，其實 UI 只需要改動一次就好了，沒有重複更新十次 UI 的必要。

這時候我們就該用 NSNotificationQueue。我們可以把 NSNotificationQueue
想成 notification 的發送端與 notification center 之間的一個 buffer，這
個 buffer 可以讓我們暫緩送出 notification，而在一段緩衝期之內，決定我
們是否要合併通知。以前面的例子來看，我們就可以先把原本預計的十次通知先
放進 NSNotificationQueue 當中，然後讓 NSNotificationQueue 幫我們把十次
通知合併成只有一次通知。

我們要先建立一個 NSNotificationQueue 物件：

``` objc
notificationQueue = [[NSNotificationQueue alloc]
initWithNotificationCenter:[NSNotificationCenter defaultCenter]];
```

再來我們發送通知的程式原本像這樣：

``` objc
NSNotification *n = [NSNotification
    notificationWithName:@"KKSongInfoDidChangeNotification"
    object:self];
[[NSNotificationCenter defaultCenter] postNotification:n];
```

改寫成這樣：

``` objc
NSNotification *n = [NSNotification
    notificationWithName:@"KKSongInfoDidChangeNotification"
    object:self];
[notificationQueue enqueueNotification:n
	postingStyle:NSPostASAP
	coalesceMask:NSNotificationCoalescingOnName | NSNotificationCoalescingOnSender
	forModes:nil];
```

我們在這邊傳入了 `NSNotificationCoalescingOnName` 與
`NSNotificationCoalescingOnSender`，代表的就是請 notification queue 合
併名稱相同、發送者也相同的通知。

Mac 上的其他 Notification Center
--------------------------------

在 iOS 上面我們通常只會用到 NSNotificationCenter，特別是
NSNotificationCenter 的 defaultCenter：不過，在 Mac OS X 上，我們還有
其他的 notification center 可以使用。

### NSDistributedNotificationCenter

蘋果在 iOS 上的限制較為嚴格，一直以來都想辦法禁止跨 App 之間的通訊
（IPC，Inter-Process Communication）。不過自從 Mac OS X 出現以來，
Cocoa Framework 就有 Distributed Objects 這套 IPC 機制，讓不同 App 之
間可以傳遞 Objective-C 物件，後來更推出了 XPC，可以在不同 App 之間傳遞
block。

NSDistributedNotificationCenter 就是在 Distributed Objects 技術上建立
的 notification center，也就是，如果你對
NSDistributedNotificationCenter 發送了通知，便可以讓其他的 App 收到來
自你目前所在 App 送出的通知。

### NSWorkSpace 的 Notification Center

NSWorkSpace 這個物件在 Mac 上代表的是 Mac 的桌面環境。如果你想要要求
Mac OS X 開啟另外一個 App，處理某個檔案或 URL（在 iOS 上我們會要求
UIApplication 來 openURL:，但是在 Mac 上則是交由 NSWorkSpace 處理），
或是取得某個檔案在 Finder 裡頭的代表圖示…等，就會用到 NSWorkSpace。

跟 NSWorkSpace 相關的通知，像是某個 App 是否被成功開啟、你的 Mac 電腦
是否離開了休眠…等等，都不會透過 NSNotificationCenter 的defaultCenter，
而是要透過 `[[NSWorkSpace sharedWorkspace] notificationCenter]` 這邊的
notification center，我們要選擇正確的 notification center 做 add
observer，才能正確收到通知。

CFNotificationCenter
--------------------

CFNotificationCenter 是 NSNotificationCenter 在 Core Foundation 中的 C
實作，一般來說，如果我們有比較高階的 API 可以使用的話，我們會盡量避免
使用比較低階的 API，所以，只要有 NSNotificationCenter 可以使用的場合，
我們應該不會用到 CFNotificationCenter。

比較有可能用到 CFNotificationCenter 的場合，大概是 iOS 8 之後，Hosting
App 與 Extension 之間的溝通。iOS 8 之後推出了 Extension，可以允許開發
者撰寫 Today Widget、Share Widget 以及模擬鍵盤等功能，Applw Watch 上的
Watch App 也屬於 Extension；每個 Extension都是額外可以讓作業系統載入的
Bundle，Extension 與我們原本的 App（便是 Hosting App）之間，可以用
Shared Data 共用資料，但是當 App 發生改變要通知 Extension，卻沒有什麼
比較直接的辦法。在蘋果有新的 API 之前，我們就會倚賴透過
`CFNotificationCenterGetDarwinNotifyCenter()` 取得的 darwin
notification center 發送通知。

而即使我們可以發送通知，CFNotificationCenter 用起來也不是很方便，主要
原因是 CFNotificationCenter 不像 NSNotificationCenter，在傳遞通知的時
候可以夾帶 user info。再來，就是像前面說的， CFNotificationCenter 是 C
API，而我們會盡量希望使用比較高階的 API。

所以，KKBOX 在開發 Watch App 的時候，就在 CFNotificationCenter 上面又
簡單架構了一層 Objective-C API，介面類似 NSNotificationCenter，叫做
KKWatchAppNotificationCenter。程式碼如下：

KKWatchAppNotificationCenter.h

``` objc
@import Foundation;

@interface KKWatchAppNotificationCenter : NSObject
+ (instancetype)sharedCenter;
- (void)postNotification:(NSString *)key;
- (void)addTarget:(id)target selector:(SEL)selector name:(NSString *)notification;
- (void)removeObserver:(NSObject *)observer;
@end
```

KKWatchAppNotificationCenter.m

``` objc
#import "KKWatchAppNotificationCenter.h"

#define LFSuppressPerformSelectorLeakWarning(doPerformSelector) \
do { \
_Pragma("clang diagnostic push") \
_Pragma("clang diagnostic ignored \"-Warc-performSelector-leaks\"") \
doPerformSelector; \
_Pragma("clang diagnostic pop") \
} while (0)

@interface KKWatchAppNotificationCenter ()
{
	NSMutableDictionary *notificationKeys;
}
@end

@implementation KKWatchAppNotificationCenter

+ (instancetype)sharedCenter
{
	static KKWatchAppNotificationCenter *sharedRegister = nil;
	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		sharedRegister = [[KKWatchAppNotificationCenter alloc] init];
	});
	return sharedRegister;
}

- (instancetype)init
{
	self = [super init];
	if (self) {
		notificationKeys = [[NSMutableDictionary alloc] init];
	}
	return self;
}

- (void)postNotification:(NSString *)key
{
	CFNotificationCenterRef const center = CFNotificationCenterGetDarwinNotifyCenter();
	CFDictionaryRef const userInfo = NULL;
	BOOL const deliverImmediately = YES;
	CFNotificationCenterPostNotification(center,
  	  (CFStringRef)key, NULL, userInfo, deliverImmediately);
}

- (void)addTarget:(id)target selector:(SEL)selector name:(NSString *)notification
{
	if (!target) {
		return;
	}

	if (!selector) {
		return;
	}

	if (!notification || ![notification length]) {
		return;
	}

	NSMutableArray *a = [notificationKeys objectForKey:notification];
	BOOL needRegisterNotification = NO;

	if (!a) {
		a = [NSMutableArray array];
		needRegisterNotification = YES;
	}

	for (NSDictionary *d in a) {
	  if (d[@"target"] == target &&
    	NSSelectorFromString(d[@"selector"]) == selector) {
			return;
		}
	}

	NSDictionary *d = @{@"target": target, @"selector": NSStringFromSelector(selector)};
	[a addObject:d];

	if (needRegisterNotification) {
		[self registerNotification:notification];
		[notificationKeys setObject:a forKey:notification];
	}
}

- (void)removeObserver:(NSObject *)observer
{
	NSMutableArray *notificationsToDelete = [[NSMutableArray alloc] init];
	for (NSString *notification in notificationKeys) {
		NSMutableArray *a = notificationKeys[notification];
		NSMutableArray *objectsToDelete = [NSMutableArray array];
		for (NSDictionary *d in a) {
			id target = d[@"target"];
			if (target == observer) {
				[objectsToDelete addObject:d];
			}
		}
		[a removeObjectsInArray:objectsToDelete];
		if (![a count]) {
			[notificationsToDelete addObject:notification];
		}
	}

	for (NSString *notification in notificationsToDelete) {
		[self unregisterNotification:notification];
		[notificationKeys removeObjectForKey:notificationKeys];
	}
}

- (void)registerNotification:(NSString *)key
{
	CFNotificationCenterRef const center = CFNotificationCenterGetDarwinNotifyCenter();
	CFNotificationSuspensionBehavior const suspensionBehavior = CFNotificationSuspensionBehaviorDeliverImmediately;
	CFNotificationCenterAddObserver(center,
		(__bridge const void *)(self),
		KKWatchAppNotificationRegisterCallback,
		(CFStringRef)key, NULL, suspensionBehavior);
}

- (void)unregisterNotification:(NSString *)key
{
	CFNotificationCenterRef const center = CFNotificationCenterGetDarwinNotifyCenter();
	CFNotificationCenterRemoveObserver(center,
	  (__bridge const void *)(self),
	  (CFStringRef)key, NULL);
}

void KKWatchAppNotificationRegisterCallback(CFNotificationCenterRef center,
  void * observer, CFStringRef name, void const * object, CFDictionaryRef userInfo)
{
	KKWatchAppNotificationCenter *self = (__bridge KKWatchAppNotificationCenter *)observer;
	NSString *notification = (__bridge NSString *)name;

	NSArray *a = [self->notificationKeys objectForKey:notification];
	for (NSDictionary *d in a) {
		id target = d[@"target"];
		SEL action = NSSelectorFromString(d[@"selector"]);
		LFSuppressPerformSelectorLeakWarning([target performSelector:action withObject:nil]);
	}
}

@end
```

相關文章
--------

- [Notification Programming Topics](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Notifications/Introduction/introNotifications.html#//apple_ref/doc/uid/10000043i)
- [Introduction to Workspace Services](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/Workspace/introduction.html#//apple_ref/doc/uid/10000100i)

[^1]: 參見蘋果官方文件 Responding to Route Changes - https://developer.apple.com/library/ios/documentation/Audio/Conceptual/AudioSessionProgrammingGuide/HandlingAudioHardwareRouteChanges/HandlingAudioHardwareRouteChanges.html
