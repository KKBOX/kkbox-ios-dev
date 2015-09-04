NSOpertation 與 NSOperationQueue
--------------------------------

GCD 雖然好用，但是 GCD 的介面讓我們不太容易取消已經排程、或是已經在背
景執行的作業，如果我們有中途取消某個背景作業的需要，使用 NSOperation
與 NSOperation Queue 會是更好的選擇。

NSOperation 是一個用來封裝一項作業的 Objective-C class，這樣的物件稱為
operation，在建立了一個 operation 後，便可以將 operation 丟入 opeation
queue（型別為 NSOperationQueue）中排程，讓 opeation queue 決定在適當的
時機、系統可以負荷的狀況下，執行我們排入排程的工作。

無論是 NSOperationQueue 與 NSOperation，都有與取消工作相關的 API 可以
呼叫。我們也可以設定每個 opeation 的優先程度，以及不同 operation 之間
的相依關係（dependency），要求一件工作完成之後，才可以繼續下一件工作。

### 建立 NSOperationQueue

假如我們有一個 Class，裡頭有一個 operation queue，只要呼叫 `alloc` 與
`init` 便可以建立。

``` objc
#import <Foundation/Foundation.h>

@interface Test : NSObject
@property (nonatomic, strong) NSOperationQueue *queue;
@end

@implementation Test

- (instancetype)init
{
	self = [super init];
	if (self) {
		self.queue = [[NSOperationQueue alloc] init];
		self.queue.maxConcurrentOperationCount = 2;
	}
	return self;
}

@end
```

我們可以透過 `maxConcurrentOperationCount` 這個 property，設定
NSOperationQueue 可以同時平行執行幾件工作，如果超過 1，就代表允許平行
執行，如果剛好是 1 的話，就代表在這個 queue 當中的所有工作都會依次執行。
預設值是 -1（NSOperationQueueDefaultMaxConcurrentOperationCount），意
思是讓系統自己決定最多可以同時建立多少 thread。

我們可以對 NSOperationQueue 呼叫 `addOperation:` 加入 operation，用
`cancelAllOperations` 取消所有排程中的作業。至於已經在執行中的作業，我
們就得對特定的 operation 呼叫 `cancel` 了。

### 建立 NSOperation

在 Cocoa/Cocoa Touch Framework 中，已經存在兩個 NSOperation 的subclass：
NSBlockOperation 與 NSInvocationOperation。NSBlockOperation可以讓你把
一個 block 封裝成 NSOperation，至於 NSInvocationOperation 則是用來封裝
NSInvocation。

一般來說，除非是像前面說的，你希望這些工作在排程中就可以取消，或是要特
別指定 operation 之間的相依關係，不然，要在背景執行某個 block 或是
invocation，其實使用 GCD API 會更容易。

我們可能會更常建立自己的 NSOperation subclass，處理更複雜的背景工作。

比方說，我們現在要開發一套食譜 App，這套 App 可以讓用戶在本機的編輯介
面中編好一份食譜後上傳，上傳後要清除本機的暫存檔，這份食譜可能會包含一
份包含標題、內文的 JSON 檔案，還有一張圖片，所以上傳食譜這份工作就包含
上傳 JSON 文件與圖片兩件工作，而我們也希望可以在上傳的過程中隨時取消，
讓用戶繼續編輯再重新上傳—這種比較複雜卻又帶有次序性質的工作，就是很適
合 NSOperation 的舞台。

要 subclass 一個 NSOperation，最重要的就是要 override 掉 main 這個
method，main 這個 method 裡頭代表的是這個 operation 要做什麼事情。我們
現在可以來寫我們的 operation：

``` objc
@interface RecipetUploadOperation : NSOperation
@property (nonatomic, strong) UIImage *image;
@property (nonatomic, strong) NSString *JSON;
@end

@implementation RecipetUploadOperation
- (void)main
{
	@autoreleasepool {
    // 1. Upload image
	// 2. Upload JSON
	}
}
@end
```

在 main 裡頭，我們也要建立 auto release pool。

接下來我們會遇到一個問題：在上傳照片與 JSON 檔案的時候，我們會呼叫
NSURLSession 的相關 API，這些 API 都是非同步的，但是在 main 這個method
裡頭，如果不做特別的處理，還沒等到連線回應，main 就已經執行結束了。我
們必須要想辦法停在 main 中，等待連線 API 的回應。

### 在 Operation 中等待與取消

要在 operation 的中途停下來等候回應，我們大致上有兩種作法，一種是在
operation 當中執行 NSRunloop，另外一種則是使用 GCD 當中的 semaphore。

#### NSRunloop

在有 GCD 之前，我們希望一個 operation 可以在一個地方停下來等候其他事情
發生，作法會是在這條 thread 裡頭執行 run loop。

前一章提到，run loop 就是那個「之所以 GUI 程式會一直執行，而不會像某個
function 或 method 從頭到尾跑完就結束」的迴圈。在 iOS 或 Mac OS X App
中，除了在 main thread 會執行最主要的 run loop
（`[NSRunloop mainRunLoop]`）之外，每個thread/operation 裡頭，也會有屬
於各自的 run loop，只要呼叫 `[NSRunloop currentRunLoop]`，呼叫的就是屬
於當前 thread 自己的 run loop—所以我們要注意，雖然在不同的 thread 中，
我們呼叫的都是 `[NSRunloop currentRunLoop]`，但這個
`+currentRunLoop`這個 class method 回傳的並不視同一個物件。另外，
NSRunloop 不可以手動建立，我們只能使用系統提供的 run loop 物件。

我們希望能夠在這個 operation 執行到一半的時候可以被取消，要取消一條
operation，便是呼叫 NSOperation 的 `cancel` 這個 method，因為我們
subclass 了 NSOperation，改變了 operation 裡頭做的事情，那麼也就得
override 掉 `cancel`：當我們的 operation 在跑 run loop 時，我們的
`cancel` 必須要能夠通知 run loop 停止。

當一條 thread 在跑自己的 run loop 之後，如果不同 thread 之間想要互相溝
通，那我們就必須在當前的 thread 建立 NSPort 物件，並且將 NSPort 物件註
冊到 run loop 內，才能讓訊息傳遞到 run loop 裡頭。所以，當外部要求對
port 呼叫 `invalidate` 的時候，就會讓 run loop 收到訊息，停止繼續跑，
繼續執行 `-main` 這個 method 接下來的動作。

NSPort 也有對應的 Core Foundation 實作，像
[CFMessagePort](https://developer.apple.com/library/mac/documentation/CoreFoundation/Reference/CFMessagePortRef/)
等，不過在 iOS 7 之後我們沒辦法在這個地方使用 CFMessagePort。從 iOS 7
之後，呼叫 `CFMessagePortCreateLocal` 或 `CFMessagePortCreateRemote`
這些建立CFMessagePort 的 function 都無法建立物件，只會回傳 NULL（可以
參見CFMessagePort 的 reference），蘋果不允許我們使用 CFMessagePort 的
原因是，CFMessagePort 不但可以傳遞訊息到其他 thread 的 run loop 上，甚
至可以傳到其他 process 的 run loop 上，而 iOS 政策上禁止 process 互相
溝通。

在 iOS 7 剛問世的時候，蘋果又完全沒有說清楚這件事，只忙著宣傳 iOS 7 的
扁平化新設計。我們為了 CFMessagePort 的這項改變，還在 WWDC 2013 會場上
跑了兩天的 Lab。

範例程式如下：

``` objc
@interface RecipetUploadOperation : NSOperation
{
	NSPort *port;
	BOOL runloopRunning;
}
@property (nonatomic, strong) UIImage *image;
@property (nonatomic, strong) NSString *JSON;
@end

@implementation RecipetUploadOperation

- (void)main
{
	@autoreleasepool {
		[someAPI uploadImageData:UIImagePNGRepresentation(self.image) callback:^ {
			[self quitRunLoop];
		}];
		[self doRunloop];
		if (self.isCancelled) {
			return;
		}
		[someAPI uploadJSON:self.JSON callback:^ {
			[self quitRunLoop];
		}];
		[self doRunloop];
	}
}

- (void)doRunloop
{
	runloopRunning = YES;
	port = [[NSPort alloc] init];
	[[NSRunLoop currentRunLoop] addPort:port forMode:NSRunLoopCommonModes];
	while (runloopRunning && !self.isCancelled) {
		@autoreleasepool {
			[[NSRunLoop currentRunLoop] runUntilDate:[NSDate dateWithTimeIntervalSinceNow:0.5]];
		}
	}
	port = nil;
}

- (void)quitRunLoop
{
	[port invalidate];
	runloopRunning = NO;
}

- (void)cancel
{
	[super cancel];
	[self quitRunLoop];
}

@end
```

#### GCD Semaphores

有了 GCD 之後，很多事情都變得簡單許多。當我們想要在執行到一半的時候暫
停下來，現在可以選擇建立 semaphore，接著：

- 只要對 semaphore 呼叫 `dispatch_semaphore_wait`，程式就會在這個地方
  暫停等候。
- 對已經在等候中的 semaphore，再呼叫 `dispatch_semaphore_signal`，發送
  signal，程式就會繼續往下運作。

範例程式如下：

``` objc
@import UIKit;

@interface RecipetUploadOperation : NSOperation
@property (nonatomic, strong) UIImage *image;
@property (nonatomic, strong) NSString *JSON;
@property (nonatomic, strong) dispatch_semaphore_t semaphore;
@end

@implementation RecipetUploadOperation
- (void)main
{
	@autoreleasepool {
		self.semaphore = dispatch_semaphore_create(0);
		[someAPI uploadImageData:UIImagePNGRepresentation(self.image) callback:^ {
			dispatch_semaphore_signal(self.semaphore);
		}];
		dispatch_semaphore_wait(self.semaphore, DISPATCH_TIME_FOREVER);
		if (self.cancelled) {
			return;
		}
		self.semaphore = dispatch_semaphore_create(0);
		[someAPI uploadJSON:self.JSON callback:^ {
			dispatch_semaphore_signal(self.semaphore);
		}];
		dispatch_semaphore_wait(self.semaphore, DISPATCH_TIME_FOREVER);
	}
}

- (void)cancel
{
	[super cancel];
	dispatch_semaphore_signal(self.semaphore);
}

@end
```
