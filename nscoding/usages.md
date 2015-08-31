NSCoding 的常見用途
-------------------

我們會在以下這些場合用到 NSCoding：

### XIB/Storyboard

我們在實作一個 UIView 的 subclass 的時候，會注意到，如果我們希望在
initialize 這個 view 的時候，就要做一些事情，不但要 override 掉
`initWithFrame:`，也要 override `initWithCoder:`。

如果這個 view 是我們用 code建立的，那麼就會呼叫到 `initWithFrame:`，但，
如果我們是在 Interface Builder 裡頭，用圖形化工具建立了一個 view，那個，
當這樣的 view 在執行的時候，則會走進 `initWithCoder:` 這一段的實作。

我們在開始接觸 iOS 開發的時候，大概就會先學習如何使用 Interface
Builder 拉出想要的介面，產生出 XIB 或是 Storyboard 檔案。XIB 與
Storyboard 在我們撰寫程式的期間，是 XML 格式的檔案，當我們編譯 App 的
時候，Xcode 會將 XIB 與 Storyboard 編譯成 binary 格式的 data，分別是
NIB 與 storyboardc 檔案，而這些 data 其實就是序列化過的 Objective-C
view 物件。

在一些其他開發平台上（像是使用 Visual Studio 拉出 Windows Form 應用程
式）使用視覺化開發工具的時候，這些工具在做的事情，是把從拉出來的介面產
生程式碼；不過在 Xcode 中編輯 XIB 檔案做的事情不一樣，是先產生出序列化
後的檔案，然後再執行的時候，讀取這些檔案，將 data 轉成 view 物件。這個
流程正是使用 NSCoding protocol，於是從 NIB/storyboardc 讀出我們的 view
的時候，所呼叫的便是 `initWithCoder:`—我們可以從 UIView 的 interface中，
看到 UIView 實作了 NSCoding protocol。

我們在 Xcode 2 左右的年代（大概是 Mac OS X 10.4 左右）開發 Mac App 時，
我們在 Xcode 中其實是直接編輯 NIB 檔案，到了 Xcode 3 與 Mac OS X 10.5
之後才出現使用 XML 格式的 XIB 檔案。這個轉變跟當時 SVN 等版本管理系統
的出現有關，在版本管理系統中編輯 binary 格式的檔案，會難以 diff、merge
以及處理版本衝突，所以蘋果便從 binary 格式換成文字格式的檔案。

### NSUserDefaults

如果在我們的 App 中，我們想要儲存一些偏好設定，那麼最好用的選擇莫過於
Cocoa/Cocoa Touch Framework 本身就提供的 NSUserDefaults 物件。操作
NSUserDefaults 與操作 NSDictionary 差不多，我們只要指定特定的 key，就
可以將設定值存入 NSUserDefaults 中。

NSUserDefaults 支援 NSString、NSArray、NSDictionary、 NSData 以及 int、
double、float 等型別的資料。但，如果是我們自己定義的 Class，或是許多其
他的 Class，會無法存入 NSUserDefaults 中，我們會需要先透過 NSCoding 轉
換成 NSData 後存入，在取出的時候，也要多做一次 unarchive。

比方說，如果我們的 App 的某個地方可以設定顏色，我們想把 UIColor 變成設
定值，UIColor 就是一種無法直接存入 NSUserDefaults 的物件。所以我們想把
UIColor 存入 NSUserDefaults，就得這麼寫：

``` objc
UIColor *color = [UIColor colorWithHue:1.0 saturation:0.5 brightness:0.5 alpha:1.0];
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:color];
[[NSUserDefaults standardUserDefaults] setObject:data forKey:@"color"];
```

### Copy and Paste

我們在行動裝置上面會比較少實作 Copy and Paste 剪貼功能，原因大概是我們
比較少在行動裝置上使用與開發比較複雜的編輯工作，而將這些工作留在
desktop 環境。如果我們要開發一套 Mac App，如何實作 Copy and Paste，以
及 Drag and Drop，就會是不可不學的知識了。

無論是實作 Copy and Paste 與 Drag and Drop，都是透過 pasteboard 物件，
實作 Drag and Drop 其實只是在開始 Drag 的時候，先把想要拖曳的資料先放
在另外一個專屬的 pasteboard 中，到了要放開的時候再從 pasteboard 中取出
資料。在 Mac 上，pasteboard 物件叫做 NSPasteboard，在 iOS 上叫做
UIPasteboard。

除了像 NSString、NSData 之類的基礎物件之外，許多我們想要可以被複製或拖
拉的資料，如果想要存入到 pasteboard 中，還是得先透過 NSCoding 轉換成
NSData 才有辦法。

像我們之前定義了 KKSongTrack 物件，想要寫入剪貼簿，可以這麼做：

``` objc
NSString *const KKBOXSongTrackPasteboardType = @"song_track";

KKSongTrack *song = [[KKSongTrack alloc] init];
song.songName = @"orz 之歌";
song.albumName = @"orz 專輯";
song.artistName = @"orz";
NSData *data = [NSKeyedArchiver archivedDataWithRootObject:song];

[[UIPasteboard generalPasteboard] setData:data
forPasteboardType:KKBOXSongTrackPasteboardType];
```

讀出來：

``` objc
NSData *pasteData = [[UIPasteboard generalPasteboard] dataForPasteboardType:KKBOXSongTrackPasteboardType];
KKSongTrack *pasteSongTrack = [NSKeyedUnarchiver unarchiveObjectWithData:pasteData];
NSLog(@"pasteSongTrack:%@", pasteSongTrack);
```

### Document-based App

如果我們開發的 App 種類屬於生產力工具，那麼我們很有可能開發的就是一套
Document-based App。

所謂 Document-based App 包括蘋果自己的 iWork 系列，如 Keynote、Numbers、
Pages 等等，主要功能就是讓你瀏覽及編輯特定種類的檔案，像 iWork 系列的
每一個 App，功能就是編輯特定種類的簡報、試算表與文書檔案。

在Cocoa/Cocoa Touch Framework 中，便使用 document－Mac 上叫做
NSDocument、iOS 上面叫做 UIDocument，對前面提到的各種不同類型文件做抽
象描述，包括負責開啟檔案、儲存檔案、自動存檔以及 iCloud 備份同步等工
作，以及描述檔案所在位置與目前狀態等。

在 iOS 上要寫一個 Document-based App，我們會建立一個 UIDocument 的
subclass，而這個 subclass 最重要的就是實作開檔與讀檔兩個 method。比方
說，我們建立了一份叫做 KKPlaylist 的 document，裡頭有一個 array，裡頭
是我們的 KKSongTrack 物件，這個 document 大概會寫成這樣：

KKPlaylist.h

``` objc
@import UIKit;
@interface KKPlaylist : UIDocument
@end
```

KKPlaylist.m

```
#import "KKPlaylist.h"

@interface KKPlaylist()
@property (nonatomic, strong) NSMutableArray *songtracks;
@end

@implementation KKPlaylist

- (instancetype)initWithFileURL:(NSURL *)url
{
	self = [super initWithFileURL:url];
	if (self) {
		self.songtracks = [NSMutableArray array];
	}
	return self;
}

- (id)contentsForType:(NSString *)typeName error:(NSError **)outError
{
	NSData *data = [NSKeyedArchiver archivedDataWithRootObject:self.songtracks];
	return data;
}

- (BOOL)loadFromContents:(id)contents ofType:(NSString *)typeName error:(NSError **)outError
{
	NSArray *songtracks = [NSKeyedUnarchiver unarchiveObjectWithData:contents];
	[self.songtracks setArray:songtracks];
	return YES;
}

@end
```

`contentsForType:error:` 與 `loadFromContents:ofType:error:` 裡頭的
contents 參數是 id 型別，不過其實只接受 NSFileWrapper 與 NSData，如果
在我們的 document 中有不少已經實作了 NSCoding protocol 的物件，我們就
可以輕鬆將物件轉成 NSData 之後存檔，或讀取檔案轉回物件。我們通常對
UIDocument 做三件事情：

1. 開啟檔案，呼叫 `-openWithCompletionHandler:`
2. 關閉檔案，呼叫 `-closeWithCompletionHandler:`
3. 存檔，呼叫 `-saveToURL:forSaveOperation:completionHandler:`

### State Preservation and Restoration

State Preservation and Restoration 是蘋果在 iOS 6 加入的 API，用途是讓
iOS App 可以在開啟的的時候，可以立刻回復到上一次關閉 App 時的狀況，方
便用戶回復到之前的動作，而不受到因為 App 關閉/開啟而打斷。像 Mail 這個
App，當你在寫一封寫到一半的時候關閉 App，下次打開，就會看到之前寫到一
半的那封信，避免用戶找不到上次寫到一半的信在哪裡。

原理是，在應用程式關閉的時候，我們可以先把目前 App 的狀態—像是目前所有
的view controller 物件，統統保存起來，下一次應用程式開啟的時候，如果發
現存在之前所保存的狀態，就讀取出來，重建上次存起來的 view controller。

要實作 State Preservation and Restoration，首先，要能夠被保存的 view
controller，要實作兩個 method：

* `- (void)encodeRestorableStateWithCoder:(NSCoder *)coder:`
* `- (void)decodeRestorableStateWithCoder:(NSCoder *)coder`

在 App Delegate 則要實作：

* `-application:shouldSaveApplicationState:`
* `-application:shouldRestoreApplicationState:`
* `-application:willEncodeRestorableStateWithCoder:`
* `-application:didDecodeRestorableStateWithCoder:`
* `-application:willFinishLaunchingWithOptions:`

流程是：

一、在 App 關閉的時候，首先系統會透過
`-application:shouldSaveApplicationState:` 詢問我們是否要保存狀態，如
果要的話，我們就回傳 YES。

二、前一步回傳 YES 之後，系統就會透過
`-application:shouldRestoreApplicationState:`，提供我們一個 NSCoder，
讓我們把必要的狀態透過這個 NSCoder archive 起來。如果我們的 App 裡頭有
一個 navigation controller，而我們想把整個 navigation controller 保存
起來，可以這麼寫：

``` objc
- (void)application:(UIApplication *)application
willEncodeRestorableStateWithCoder:(NSCoder *)coder
{
	NSMutableArray *viewControllers = [self.navigationControllers.viewControllers copy];
	NSData *data = [NSKeyedArchiver archivedDataWithRootObject:viewControllers];
	[coder encodeObject:data forKey:@"viewControllers"];
}
```

三、在重新開啟 App 的時候，如果系統發現之前我們已經透過 NSCoder 保存狀
態了，那麼，就會向我們透過
`-application:shouldRestoreApplicationState:`，詢問是否要使用上次的狀
態，如果要的話，我們就回傳 YES。

四、接下來 `-application:didDecodeRestorableStateWithCoder:` 就會被呼
叫到，如果我們想還原上次存起來的 navigation controller，可以這麼寫：

``` objc
- (void)application:(UIApplication *)application
didDecodeRestorableStateWithCoder:(NSCoder *)coder
{
	NSData *data = [coder decodeObjectForKey:@"viewControllers"];
	NSArray *viewControllers = [NSKeyedUnarchiver unarchiveObjectWithData:data];
    self.navigationController.viewControllers = viewControllers;
}
```
