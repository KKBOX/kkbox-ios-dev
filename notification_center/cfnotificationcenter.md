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
