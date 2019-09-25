使用 Audio Queue 開發播放軟體
-----------------------------

我們現在來寫一個簡單的 Audio Queue Player，這個 Player 能夠播放位在
server 上的 MP3 檔案。

為了說明如何使用 Audio Queue API，所以這邊程式做了一定程度的簡化，很多
需要做的事情，其實在這邊沒做。在這個 player 中，我們首先需要以下的成員
變數：

* 用來保存 pakcket 的 array，在這邊使用了一個 NSMutableArray，裡頭會是
  一堆 NSData。在真正的產品 code 中其實不該這麼寫，因為這樣等於是把所
  有收到的 audio 資料都丟進記憶體中，如果是 300 mb 的 MP3，就會用到
  300 mb 的記憶體，在產品 code 中應該要把一部分資料寫入暫存檔中。
* AudioQueueRef：就是我們的 Audio Queue。
* AudioStreamBasicDescription：我們在建立 Audio Queue 時所使用的 Audio
  檔案格式。
* AudioFileStreamID：parser。
* readHead：是一個 size_t，用來表示我們現在讀到第幾個 packet。
* 一個用來抓取資料的 NSURLConnection 物件。

這個 Player 播放的步驟包括

1. 建立 Parser 與網路連線
2. 收到部分資料並 parse packet
3. 收到 parser 分析出的檔案格式資料，建立 Audio Queue
4. 收到 parser 分析出的 packet，保存 packet
5. packet 數量夠多的時候，enqueue buffer
6. 收到 Audio Queue 播放完畢的通知，繼續 enqueue

### 第一步：建立 Parser 與網路連線

在建立 Audio Queue 的時候，需要用 AudioStreamBasicDescription 傳入詳細
的資料格式，包括 sample rate、這個檔案有多少 channel 等等，但我們現在
還不知道遠端音檔的格式，所以會稍晚建立。

我們用 `AudioFileStreamOpen` function 建立 AudioFileStreamID，也就是我
們的 parser，在這邊我們傳入了 kAudioFileMP3Type，代表說，我們猜測遠端
的檔案是一個 MP3 檔，而 `AudioFileStreamOpen` 其實只有大概參考這個提示
而已，如果遠端的檔案是 MP3 之外的其他格式，但我們在
`AudioFileStreamOpen` 告訴 AudioFileStreamID 的卻是 MP3，我們的
AudioFileStreamID 還是有能夠辨別出到底是哪種檔案。

不過 `AudioFileStreamOpen` 很容易誤判，如果在建立 AudioFileStreamID 之
後，我們一開始透過呼叫 `AudioFileStreamParseBytes` 給
AudioFileStreamID 的資料不夠多，AudioFileStreamID 就常常回傳誤判的結果，
像明明是 MP3 格式，卻告訴我們是 MP2 或 MP1。

在 `AudioFileStreamParseBytes` 裡頭還要傳入兩個 callback function，在
這邊我們傳入我們定義好的 `KKAudioFileStreamPropertyListener` 與
`KKAudioFileStreamPacketsCallback`。
`KKAudioFileStreamPropertyListener` 是屬性改變的 callback，我們在這個
callback 主要想知道的是取得檔案格式的通知。而
`KKAudioFileStreamPacketsCallback` 則是 packet 的 callback，會在分析出
了 packet 的時候呼叫。

接著我們就可以用 NSURLConnection 抓取檔案了。

### 第二步：收到部分資料與 parse packet

在 NSURLConnection 的 delegate method `connection:didReceiveData:` 中，
我們會收到在這一輪 run loop 中，NSURLConnection 抓到了多少資料，我們便
可以透過 `AudioFileStreamParseBytes`，將收到的資料交給
AudioFileStreamID 分析。

就像前面提到，我們最好一開始呼叫 `AudioFileStreamParseBytes` 的時候，
就先給一包比較大的資料，所以我們在收到資料的時候，可能先找個
NSMutableData 然後 append 進去，等到 bytes 足夠的時候才呼叫
`AudioFileStreamParseBytes`，不過這邊為了簡化所以沒有這麼做。

前面也提到，如果我們用 Core Audio API 播放 MP4 檔案，必須要將 moov
atom 放在 mdat 之前，正是因為 AudioFileStreamID 無法解析這種格式的 MP4。
另外，如果你要播放的是 AAC ADTS 格式的檔案，這個檔案的前方可能會有一些
ID3 檔頭，我們也要自己把 ID3 檔頭濾掉，不然 AudioFileStreamID 會無法解
析。

### 第三步：收到 parser 分析出的檔案格式資料，建立 Audio Queue

當 AudioFileStreamID 從我們提供的資料分析出檔案格式之後，就會呼叫我們
在建立 AudioFileStreamID 時傳入的 `KKAudioFileStreamPropertyListener`。
這個 callback 是在 AudioFileStreamID 的屬性改變的時候被呼叫，因為
AudioFileStreamID 其實有不少屬性（都是 AudioFileStreamPropertyID），所
以在很多狀況下都會呼叫這個 callback，但我們現在只想要知道資料格式而已，
所以只寫了 kAudioFileStreamProperty_DataFormat 的相關判斷。

得到資料格式之後，就可以建立 Audio Queue 了。因為我們要建立的是輸出用
的 Audio Queue，所以呼叫 `AudioQueueNewOutput` 建立，並且傳入
`KKAudioQueueOutputCallback` 這個 callback function，這個 function 會
在 Audio Queue 資料快播完的時候呼叫。

``` objc
OSStatus status = AudioQueueNewOutput(audioStreamBasicDescription,
	KKAudioQueueOutputCallback,
	(__bridge void * _Nullable)(self),
	CFRunLoopGetCurrent(),
	kCFRunLoopCommonModes, 0, &outputQueue);
```

### 第四步：收到 parser 分析出的 packet，保存 packet

AudioFileStreamID 把 packet 分析出來之後，會呼叫
`KKAudioFileStreamPacketsCallback`。在這個 callback function 中，我們
會收到 audio data 所在的記憶體指標，packet 的數量，以及一連串的 packet
description，我們可以從 packet description 中知道每個 packet 相對於傳
入的記憶體指標的 offset 與長度。

在這邊，我們把每個 packet 的資料存入 NSData 物件中，然後放在一開始建立
的 NSMutableArray 中保存。

### 第五步：packet 數量夠多的時候，enqueue buffer

我們等到收到足夠大小的 packet 才開始播放，在這邊定義的時間是要超過三秒。
因為 packet 數量不夠就開始播放，我們不會聽到連續順暢的音樂，而會是斷斷
續續的雜訊，至於播放一秒鐘要多少 packet，可以用一個 packet 有多少frame
以及一秒鐘需要多少 frame 推算。

要開始播放，就是對 Audio Queue 做 enqueue buffer。在蘋果的 sample code
中，會建立三個 Audio Queue buffer 循環使用，我們這邊的寫法比較偷懶，每
次需要 enquueu buffer 的時候，都建立一個新的 Audio Queue Buffer，但是
每次進入 `KKAudioQueueOutputCallback` 的時候，都呼叫一次
`AudioQueueFreeBuffer`，把之前使用的 buffer 釋放掉。

在建立 buffer 的時候，我們就要決定 buffer 的大小，在這邊我們透過要播放
多少 packet 決定 buffer 的大小。我們在這邊寫成每次要 enqueue 五秒的
buffer，相當於大約 190 個 packet，所以我們跑了一個簡單的迴圈把每個
packet 的大小加總，就是 buffer 的大小，然後把放在 packet 裡頭的 bytes
用 `memcpy` 複製到 buffer 的 mAudioData 裡頭。

Enqueue buffer 之後，我們會調整 read head 的位置，記錄已經送出了多少
packet。

### 第六步：收到 Audio Queue 播放完畢的通知，繼續 enqueue

在前一個 buffer 播放快要完畢的時候（經驗中大概是完畢前一秒鐘左右），我
們會收到 `KKAudioQueueOutputCallback`，這時候繼續 enqueue buffer 即可。
當我們發現 read head 已經到了跟 packet 的數量一樣多，代表 packet 用完，
也就是歌曲播放完畢。

寫完這個 player，只要這樣就可以播放歌曲了：

``` objc
NSString *URL = @"http://zonble.net/MIDI/orz.mp3";
KKSimplePlayer *player = [[KKSimplePlayer alloc] initWithURL:[NSURL URLWithString:URL]];
```

### 接下來要做的事情

因為這是一個很簡單的 player，所以很多事情沒有做。

要讓這個 player 功能更加完整，我們首先應該要寫一個 protocol，定義這個
player 的 delegate，讓外部的 UI 知道目前 player 的狀況。再來，我們要想
辦法解決播放大檔的問題，不該把所有資料都放進記憶體裡。

我們這個 player也假設網路速度非常順暢，從網路載入資料的速度比播放速度
快，如果在寫產品 code，我們還要處理「packet 已經用完，但是網路連線並沒
有把資料抓完」這種狀況。

接著需要寫跟播放時間相關的程式。

Audio Queue API 提供 `AudioQueueGetCurrentTime` 與
`AudioQueueDeviceGetCurrentTime` 這兩個function，可以取得 Audio Queue
的播放時間，一個軟體層某個 Audio Queue 開始了多久，另外一個則是某個
Audio Queue 佔用了硬體多久。 其中，`AudioQueueDeviceGetCurrentTime` 會
比 `AudioQueueGetCurrentTime` 來得精確。

我們需要注意，這兩個 function 回傳的時間，都是我們呼叫了
`AudioQueueStart` 之後過了多久，與我們現在要播放的歌曲播到哪裡沒有直接
關係，比方說，我們只呼叫了`AudioQueueStart`，但是並沒有 enquue 任何
buffer，`AudioQueueGetCurrentTime` 與 `AudioQueueDeviceGetCurrentTime`
還是會繼續計算沒有聲音的時間。如果我們遇到網路斷斷續續，聲音時有時無的
狀況，Audio Queue 的開啟時間與歌曲播放時間就會對不起來。

現在這邊的這個 player 一次 enqueue 大約五秒的 buffer，在這五秒當中到底
是播到第兩秒還是第三秒，就沒有比較好的 API 可以知道。我們在使用 Audio
Queue 計算播放時間的時候，大概會用 packet 的位置搭配
`AudioQueueGetCurrentTime` 與 `AudioQueueDeviceGetCurrentTime`，或是搭
配使用 NSDate 物件計算，但這麼做總是充滿 work around 的感覺。

在 iOS 上，還要記得處理 Audio Session，這點我們稍晚說明。

我們的 Audio Queue Player 程式碼如下：

KKSimplePlayer.h

``` objc
#import <Foundation/Foundation.h>
#import <AudioToolbox/AudioToolbox.h>

@interface KKSimplePlayer : NSObject
- (instancetype)initWithURL:(NSURL *)inURL;
- (void)play;
- (void)pause;
@property (readonly, getter=isStopped) BOOL stopped;
@end
```

KKSimplePlayer.m

``` objc
#import "KKSimplePlayer.h"

static void KKAudioFileStreamPropertyListener(void * inClientData,
	AudioFileStreamID inAudioFileStream,
	AudioFileStreamPropertyID inPropertyID,
	UInt32 * ioFlags);
static void KKAudioFileStreamPacketsCallback(void * inClientData,
	UInt32 inNumberBytes,
	UInt32 inNumberPackets,
	const void * inInputData,
	AudioStreamPacketDescription *inPacketDescriptions);
static void KKAudioQueueOutputCallback(void * inUserData,
	AudioQueueRef inAQ,
	AudioQueueBufferRef inBuffer);
static void KKAudioQueueRunningListener(void * inUserData,
	AudioQueueRef inAQ,
	AudioQueuePropertyID inID);

@interface KKSimplePlayer ()
{
	NSURLConnection *URLConnection;
	struct {
		BOOL stopped;
		BOOL loaded;
	} playerStatus ;

	AudioFileStreamID audioFileStreamID;
	AudioQueueRef outputQueue;
	AudioStreamBasicDescription streamDescription;
	NSMutableArray *packets;
	size_t readHead;
}
- (double)packetsPerSecond;
@end

@implementation KKSimplePlayer

- (void)dealloc
{
	AudioQueueReset(outputQueue);
	AudioFileStreamClose(audioFileStreamID);
	[URLConnection cancel];
}

- (instancetype)initWithURL:(NSURL *)inURL
{
	self = [super init];
	if (self) {
		playerStatus.stopped = NO;
		packets = [[NSMutableArray alloc] init];

		// 第一步：建立 Audio Parser，指定 callback，以及建立 HTTP 連線，
		// 開始下載檔案
		AudioFileStreamOpen((__bridge void * _Nullable)(self),
			KKAudioFileStreamPropertyListener,
			KKAudioFileStreamPacketsCallback,
			kAudioFileMP3Type, &audioFileStreamID);
		URLConnection = [[NSURLConnection alloc] initWithRequest:[NSURLRequest requestWithURL:inURL] delegate:self];
	}
	return self;
}

- (double)packetsPerSecond
{
	if (streamDescription.mFramesPerPacket) {
		return streamDescription.mSampleRate / streamDescription.mFramesPerPacket;
	}

	return 44100.0/1152.0;
}

- (void)play
{
	AudioQueueStart(outputQueue, NULL);
}
- (void)pause
{
	AudioQueuePause(outputQueue);
}

#pragma mark -
#pragma mark NSURLConnectionDelegate

- (void)connection:(NSURLConnection *)connection
	didReceiveResponse:(NSURLResponse *)response
{
	if ([response isKindOfClass:[NSHTTPURLResponse class]]) {
		if ([(NSHTTPURLResponse *)response statusCode] != 200) {
			NSLog(@"HTTP code:%ld", [(NSHTTPURLResponse *)response statusCode]);
			[connection cancel];
			playerStatus.stopped = YES;
		}
	}
}

- (void)connection:(NSURLConnection *)connection didReceiveData:(NSData *)data
{
	// 第二步：抓到了部分檔案，就交由 Audio Parser 開始 parse 出 data
	// stream 中的 packet。
	AudioFileStreamParseBytes(audioFileStreamID, (UInt32)[data length], [data bytes], 0);
}

- (void)connectionDidFinishLoading:(NSURLConnection *)connection
{
	NSLog(@"Complete loading data");
	playerStatus.loaded = YES;
}
- (void)connection:(NSURLConnection *)connection didFailWithError:(NSError *)error
{
	NSLog(@"Failed to load data: %@", [error localizedDescription]);
	playerStatus.stopped = YES;
}

#pragma mark -
#pragma mark Audio Parser and Audio Queue callbacks

- (void)_enqueueDataWithPacketsCount:(size_t)inPacketCount
{
	NSLog(@"%s", __PRETTY_FUNCTION__);
	if (!outputQueue) {
		return;
	}

	if (readHead == [packets count]) {
		// 第六步：已經把所有 packet 都播完了，檔案播放結束。
		if (playerStatus.loaded) {
			AudioQueueStop(outputQueue, false);
			playerStatus.stopped = YES;
			return;
		}
	}

	if (readHead + inPacketCount >= [packets count]) {
		inPacketCount = [packets count] - readHead;
	}

	UInt32 totalSize = 0;
	UInt32 index;

	for (index = 0 ; index < inPacketCount ; index++) {
		NSData *packet = packets[index + readHead];
		totalSize += packet.length;
	}

	OSStatus status = 0;
	AudioQueueBufferRef buffer;
	status = AudioQueueAllocateBuffer(outputQueue, totalSize, &buffer);
	assert(status == noErr);
	buffer->mAudioDataByteSize = totalSize;
	buffer->mUserData = (__bridge void * _Nullable)(self);

	AudioStreamPacketDescription *packetDescs = calloc(inPacketCount,
		sizeof(AudioStreamPacketDescription));

	totalSize = 0;
	for (index = 0 ; index < inPacketCount ; index++) {
		size_t readIndex = index + readHead;
		NSData *packet = packets[readIndex];
		memcpy(buffer->mAudioData + totalSize, packet.bytes, packet.length);

		AudioStreamPacketDescription description;
		description.mStartOffset = totalSize;
		description.mDataByteSize = packet.length;
		description.mVariableFramesInPacket = 0;
		totalSize += packet.length;
		memcpy(&(packetDescs[index]), &description, sizeof(AudioStreamPacketDescription));
	}
	status = AudioQueueEnqueueBuffer(outputQueue, buffer, (UInt32)inPacketCount, packetDescs);
	free(packetDescs);
	readHead += inPacketCount;
}

- (void)_createAudioQueueWithAudioStreamDescription:(AudioStreamBasicDescription *)audioStreamBasicDescription
{
	memcpy(&streamDescription, audioStreamBasicDescription, sizeof(AudioStreamBasicDescription));
	OSStatus status = AudioQueueNewOutput(audioStreamBasicDescription,
		KKAudioQueueOutputCallback,
		(__bridge void * _Nullable)(self),
		CFRunLoopGetCurrent(),
		kCFRunLoopCommonModes, 0, &outputQueue);
	assert(status == noErr);
	status = AudioQueueAddPropertyListener(outputQueue,
		kAudioQueueProperty_IsRunning,
		KKAudioQueueRunningListener,
		(__bridge void * _Nullable)(self));
	AudioQueuePrime(outputQueue, 0, NULL);
	AudioQueueStart(outputQueue, NULL);
}

- (void)_storePacketsWithNumberOfBytes:(UInt32)inNumberBytes
	numberOfPackets:(UInt32)inNumberPackets
	inputData:(const void *)inInputData
	packetDescriptions:(AudioStreamPacketDescription *)inPacketDescriptions
{
	for (int i = 0; i < inNumberPackets; ++i) {
		SInt64 packetStart = inPacketDescriptions[i].mStartOffset;
		UInt32 packetSize = inPacketDescriptions[i].mDataByteSize;
		assert(packetSize > 0);
		NSData *packet = [NSData dataWithBytes:inInputData + packetStart length:packetSize];
		[packets addObject:packet];
	}

	//	第五步，因為 parse 出來的 packets 夠多，緩衝內容夠大，因此開始
	//	播放

	if (readHead == 0 && [packets count] > (int)([self packetsPerSecond] * 3)) {
		AudioQueueStart(outputQueue, NULL);
		[self _enqueueDataWithPacketsCount: (int)([self packetsPerSecond] * 3)];
	}
}

- (void)_audioQueueDidStart
{
	NSLog(@"Audio Queue did start");
}

- (void)_audioQueueDidStop
{
	NSLog(@"Audio Queue did stop");
	playerStatus.stopped = YES;
}

#pragma mark -
#pragma mark Properties

- (BOOL)isStopped
{
	return playerStatus.stopped;
}

@end

void KKAudioFileStreamPropertyListener(void * inClientData,
	AudioFileStreamID inAudioFileStream,
	AudioFileStreamPropertyID inPropertyID,
	UInt32 * ioFlags)
{
	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inClientData;
	if (inPropertyID == kAudioFileStreamProperty_DataFormat) {
		UInt32 dataSize	 = 0;
		OSStatus status = 0;
		AudioStreamBasicDescription audioStreamDescription;
		Boolean writable = false;
		status = AudioFileStreamGetPropertyInfo(inAudioFileStream,
			kAudioFileStreamProperty_DataFormat,
			&dataSize, &writable);
		status = AudioFileStreamGetProperty(inAudioFileStream,
			kAudioFileStreamProperty_DataFormat,
			&dataSize, &audioStreamDescription);

		NSLog(@"mSampleRate: %f", audioStreamDescription.mSampleRate);
		NSLog(@"mFormatID: %u", audioStreamDescription.mFormatID);
		NSLog(@"mFormatFlags: %u", audioStreamDescription.mFormatFlags);
		NSLog(@"mBytesPerPacket: %u", audioStreamDescription.mBytesPerPacket);
		NSLog(@"mFramesPerPacket: %u", audioStreamDescription.mFramesPerPacket);
		NSLog(@"mBytesPerFrame: %u", audioStreamDescription.mBytesPerFrame);
		NSLog(@"mChannelsPerFrame: %u", audioStreamDescription.mChannelsPerFrame);
		NSLog(@"mBitsPerChannel: %u", audioStreamDescription.mBitsPerChannel);
		NSLog(@"mReserved: %u", audioStreamDescription.mReserved);

		// 第三步： Audio Parser 成功 parse 出 audio 檔案格式，我們根據
		// 檔案格式資訊，建立 Audio Queue，同時監聽 Audio Queue 是否正
		// 在執行

		[self _createAudioQueueWithAudioStreamDescription:&audioStreamDescription];
	}
}

void KKAudioFileStreamPacketsCallback(void * inClientData,
	UInt32 inNumberBytes,
	UInt32 inNumberPackets,
	const void * inInputData,
	AudioStreamPacketDescription *inPacketDescriptions)
{
	// 第四步： Audio Parser 成功 parse 出 packets，我們將這些資料儲存
	// 起來

	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inClientData;
	[self _storePacketsWithNumberOfBytes:inNumberBytes
		numberOfPackets:inNumberPackets
		inputData:inInputData
		packetDescriptions:inPacketDescriptions];
}

static void KKAudioQueueOutputCallback(void * inUserData,
	AudioQueueRef inAQ,AudioQueueBufferRef inBuffer)
{
	AudioQueueFreeBuffer(inAQ, inBuffer);
	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inUserData;
	[self _enqueueDataWithPacketsCount:(int)([self packetsPerSecond] * 5)];
}

static void KKAudioQueueRunningListener(void * inUserData,
	AudioQueueRef inAQ, AudioQueuePropertyID inID)
{
	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inUserData;
	UInt32 dataSize;
	OSStatus status = 0;
	status = AudioQueueGetPropertySize(inAQ, inID, &dataSize);
	if (inID == kAudioQueueProperty_IsRunning) {
		UInt32 running;
		status = AudioQueueGetProperty(inAQ, inID, &running, &dataSize);
		running ? [self _audioQueueDidStart] : [self _audioQueueDidStop];
	}
}

```

如果用 Swift 寫，則大概像這樣（需要 Swift 2.0 以上）：

KKSimplePlayer.swift

``` swift
import Foundation
import AudioToolbox

class KKSimplePlayer: NSObject {
	var URL: NSURL
	var URLSession: NSURLSession!
	var packets = [NSData]()
	var audioFileStreamID: AudioFileStreamID = nil
	var outputQueue: AudioQueueRef = nil
	var streamDescription: AudioStreamBasicDescription?
	var readHead: Int = 0

	var loaded = false
	var stopped = false

	init(URL: NSURL) {
		self.URL = URL
		super.init()
		let selfPointer = unsafeBitCast(self, UnsafeMutablePointer<Void>.self)
		AudioFileStreamOpen(selfPointer,
		                    KKAudioFileStreamPropertyListener,
		                    KKAudioFileStreamPacketsCallback,
		                    kAudioFileMP3Type, &self.audioFileStreamID)
		let configuration = NSURLSessionConfiguration.defaultSessionConfiguration()
		self.URLSession = NSURLSession(configuration: configuration, delegate: self, delegateQueue: nil)
		let task = self.URLSession.dataTaskWithURL(URL)
		task.resume()
	}

	deinit {
		if self.outputQueue != nil {
			AudioQueueReset(outputQueue)
		}
		AudioFileStreamClose(audioFileStreamID)
	}

	func play() {
		if self.outputQueue != nil {
			AudioQueueStart(outputQueue, nil)
		}
	}

	func pause() {
		if self.outputQueue != nil {
			AudioQueuePause(outputQueue)
		}
	}

	private func parseData(data: NSData) {
		AudioFileStreamParseBytes(self.audioFileStreamID, UInt32(data.length), data.bytes, AudioFileStreamParseFlags(rawValue: 0))
	}

	private func createAudioQueue(audioStreamDescription: AudioStreamBasicDescription) {
		var audioStreamDescription = audioStreamDescription
		self.streamDescription = audioStreamDescription
		var status: OSStatus = 0
		let selfPointer = unsafeBitCast(self, UnsafeMutablePointer<Void>.self)
		status = AudioQueueNewOutput(&audioStreamDescription, KKAudioQueueOutputCallback, selfPointer, CFRunLoopGetCurrent(), kCFRunLoopCommonModes, 0, &self.outputQueue)
		assert(noErr == status)
		status = AudioQueueAddPropertyListener(self.outputQueue, kAudioQueueProperty_IsRunning, KKAudioQueueRunningListener, selfPointer)
		assert(noErr == status)
		AudioQueuePrime(self.outputQueue, 0, nil)
		AudioQueueStart(self.outputQueue, nil)
	}

	private func storePackets(numberOfPackets: UInt32, numberOfBytes: UInt32, data: UnsafePointer<Void>, packetDescription: UnsafeMutablePointer<AudioStreamPacketDescription>) {
		for i in 0 ..< Int(numberOfPackets) {
			let packetStart = packetDescription[i].mStartOffset
			let packetSize = packetDescription[i].mDataByteSize
			let packetData = NSData(bytes: data.advancedBy(Int(packetStart)), length: Int(packetSize))
			self.packets.append(packetData)
		}

		if readHead == 0 && Double(packets.count) > self.packetsPerSecond * 3 {
			AudioQueueStart(self.outputQueue, nil)
			self.enqueueDataWithPacketsCount(Int(self.packetsPerSecond * 3))
		}
	}

	private func enqueueDataWithPacketsCount(packetCount: Int) {
		if self.outputQueue == nil {
			return
		}

		var packetCount = packetCount
		if readHead + packetCount > packets.count {
			packetCount = packets.count - readHead
		}

		let totalSize = packets[readHead ..< readHead + packetCount].reduce(0, combine: { $0 + $1.length })
		var status: OSStatus = 0
		var buffer: AudioQueueBufferRef = nil
		status = AudioQueueAllocateBuffer(outputQueue, UInt32(totalSize), &buffer)
		assert(noErr == status)
		buffer.memory.mAudioDataByteSize = UInt32(totalSize)
		let selfPointer = unsafeBitCast(self, UnsafeMutablePointer<Void>.self)
		buffer.memory.mUserData = selfPointer

		var copiedSize = 0
		var packetDescs = [AudioStreamPacketDescription]()
		for i in 0 ..< packetCount {
			let readIndex = readHead + i
			let packetData = packets[readIndex]
			memcpy(buffer.memory.mAudioData.advancedBy(copiedSize), packetData.bytes, packetData.length)
			let description = AudioStreamPacketDescription(mStartOffset: Int64(copiedSize), mVariableFramesInPacket: 0, mDataByteSize: UInt32(packetData.length))
			packetDescs.append(description)
			copiedSize += packetData.length
		}
		status = AudioQueueEnqueueBuffer(outputQueue, buffer, UInt32(packetCount), packetDescs);
		readHead += packetCount
	}
}

extension KKSimplePlayer: NSURLSessionDelegate {
	func URLSession(session: NSURLSession, dataTask: NSURLSessionDataTask, didReceiveData data: NSData) {
		self.parseData(data)
	}
}

func KKAudioFileStreamPropertyListener(clientData: UnsafeMutablePointer<Void>, audioFileStream: AudioFileStreamID, propertyID: AudioFileStreamPropertyID, ioFlag: UnsafeMutablePointer<AudioFileStreamPropertyFlags>) {
	let this = Unmanaged<KKSimplePlayer>.fromOpaque(COpaquePointer(clientData)).takeUnretainedValue()
	if propertyID == kAudioFileStreamProperty_DataFormat {
		var status: OSStatus = 0
		var dataSize: UInt32 = 0
		var writable: DarwinBoolean = false
		status = AudioFileStreamGetPropertyInfo(audioFileStream, kAudioFileStreamProperty_DataFormat, &dataSize, &writable)
		assert(noErr == status)
		var audioStreamDescription: AudioStreamBasicDescription = AudioStreamBasicDescription()
		status = AudioFileStreamGetProperty(audioFileStream, kAudioFileStreamProperty_DataFormat, &dataSize, &audioStreamDescription)
		assert(noErr == status)
		dispatch_async(dispatch_get_main_queue()) {
			this.createAudioQueue(audioStreamDescription)
		}
	}
}

func KKAudioFileStreamPacketsCallback(clientData: UnsafeMutablePointer<Void>, numberBytes: UInt32, numberPackets: UInt32, ioData: UnsafePointer<Void>, packetDescription: UnsafeMutablePointer<AudioStreamPacketDescription>) {
	let this = Unmanaged<KKSimplePlayer>.fromOpaque(COpaquePointer(clientData)).takeUnretainedValue()
	this.storePackets(numberPackets, numberOfBytes: numberBytes, data: ioData, packetDescription: packetDescription)
}

func KKAudioQueueOutputCallback(clientData: UnsafeMutablePointer<Void>, AQ: AudioQueueRef, buffer: AudioQueueBufferRef) {
	let this = Unmanaged<KKSimplePlayer>.fromOpaque(COpaquePointer(clientData)).takeUnretainedValue()
	AudioQueueFreeBuffer(AQ, buffer)
	this.enqueueDataWithPacketsCount(Int(this.packetsPerSecond * 5))
}

func KKAudioQueueRunningListener(clientData: UnsafeMutablePointer<Void>, AQ: AudioQueueRef, propertyID: AudioQueuePropertyID) {
	let this = Unmanaged<KKSimplePlayer>.fromOpaque(COpaquePointer(clientData)).takeUnretainedValue()
	var status: OSStatus = 0
	var dataSize: UInt32 = 0
	status = AudioQueueGetPropertySize(AQ, propertyID, &dataSize);
	assert(noErr == status)
	if propertyID == kAudioQueueProperty_IsRunning {
		var running: UInt32 = 0
		status = AudioQueueGetProperty(AQ, propertyID, &running, &dataSize)
		this.stopped = running == 0
	}
}
<<<<<<< HEAD

=======
>>>>>>> Updates audio_queue.md.
```
