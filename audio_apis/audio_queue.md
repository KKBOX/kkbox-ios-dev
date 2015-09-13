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
的音訊格式，包括 sample rate、這個檔案有多少 channel 等等，但我們現在
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
`KKAudioFileStreamPropertyListener` 是檔案格式的 callback，當
AudioFileStreamID 分析出檔案格式的時候，會呼叫這個 function。而
`KKAudioFileStreamPacketsCallback` 則是 packet 的 callback，會在分析出
了 packet 的時候呼叫。

接著我們就可以用 NSURLConnection 抓取檔案了。

### 第二步：收到部分資料與 parse packet

### 第三步：收到 parser 分析出的檔案格式資料，建立 Audio Queue

### 第四步：收到 parser 分析出的 packet，保存 packet

### 第五步：packet 數量夠多的時候，enqueue buffer

### 第六步：收到 Audio Queue 播放完畢的通知，繼續 enqueue



KKSimplePlayer.h

``` objc
#import <Foundation/Foundation.h>
#import <AudioToolbox/AudioToolbox.h>

@interface KKSimplePlayer : NSObject
- (id)initWithURL:(NSURL *)inURL;
- (void)play;
- (void)pause;
@property (readonly, getter=isStopped) BOOL stopped;
@end
```

KKSimplePlayer.m

``` objc
#import "KKSimplePlayer.h"

static void KKAudioFileStreamPropertyListener(void * inClientData, AudioFileStreamID inAudioFileStream, AudioFileStreamPropertyID inPropertyID, UInt32 * ioFlags);
static void KKAudioFileStreamPacketsCallback(void * inClientData, UInt32 inNumberBytes, UInt32 inNumberPackets, const void * inInputData, AudioStreamPacketDescription *inPacketDescriptions);
static void KKAudioQueueOutputCallback(void * inUserData, AudioQueueRef inAQ,AudioQueueBufferRef inBuffer);
static void KKAudioQueueRunningListener(void * inUserData, AudioQueueRef inAQ, AudioQueuePropertyID inID);

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
- (double)framePerSecond;
@end

@implementation KKSimplePlayer

- (void)dealloc
{
	AudioQueueReset(outputQueue);
	AudioFileStreamClose(audioFileStreamID);

	[URLConnection cancel];
}

- (id)initWithURL:(NSURL *)inURL
{
	self = [super init];
	if (self) {
		playerStatus.stopped = NO;
		packets = [[NSMutableArray alloc] init];

		// 第一步：建立 Audio Parser，指定 callback，以及建立 HTTP 連線，
		// 開始下載檔案
		AudioFileStreamOpen((__bridge void * _Nullable)(self), KKAudioFileStreamPropertyListener, KKAudioFileStreamPacketsCallback, kAudioFileMP3Type, &audioFileStreamID);
		URLConnection = [[NSURLConnection alloc] initWithRequest:[NSURLRequest requestWithURL:inURL] delegate:self];
	}
	return self;
}

- (double)framePerSecond
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

- (void)connection:(NSURLConnection *)connection didReceiveResponse:(NSURLResponse *)response
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

	AudioStreamPacketDescription *packetDescs = calloc(inPacketCount, sizeof(AudioStreamPacketDescription));

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
	OSStatus status = AudioQueueNewOutput(audioStreamBasicDescription, KKAudioQueueOutputCallback, (__bridge void * _Nullable)(self), CFRunLoopGetCurrent(), kCFRunLoopCommonModes, 0, &outputQueue);
	assert(status == noErr);
	status = AudioQueueAddPropertyListener(outputQueue, kAudioQueueProperty_IsRunning, KKAudioQueueRunningListener, (__bridge void * _Nullable)(self));
	AudioQueuePrime(outputQueue, 0, NULL);
	AudioQueueStart(outputQueue, NULL);
}

- (void)_storePacketsWithNumberOfBytes:(UInt32)inNumberBytes numberOfPackets:(UInt32)inNumberPackets inputData:(const void *)inInputData packetDescriptions:(AudioStreamPacketDescription *)inPacketDescriptions
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

	if (readHead == 0 & [packets count] > (int)([self framePerSecond] * 3)) {
		AudioQueueStart(outputQueue, NULL);
		[self _enqueueDataWithPacketsCount: (int)([self framePerSecond] * 2)];
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

void KKAudioFileStreamPropertyListener(void * inClientData, AudioFileStreamID inAudioFileStream, AudioFileStreamPropertyID inPropertyID, UInt32 * ioFlags)
{
	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inClientData;
	if (inPropertyID == kAudioFileStreamProperty_DataFormat) {
		UInt32 dataSize	 = 0;
		OSStatus status = 0;
		AudioStreamBasicDescription audioStreamDescription;
		Boolean writable = false;
		status = AudioFileStreamGetPropertyInfo(inAudioFileStream, kAudioFileStreamProperty_DataFormat, &dataSize, &writable);
		status = AudioFileStreamGetProperty(inAudioFileStream, kAudioFileStreamProperty_DataFormat, &dataSize, &audioStreamDescription);

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

void KKAudioFileStreamPacketsCallback(void * inClientData, UInt32 inNumberBytes, UInt32 inNumberPackets, const void * inInputData, AudioStreamPacketDescription *inPacketDescriptions)
{
	// 第四步： Audio Parser 成功 parse 出 packets，我們將這些資料儲存
	// 起來

	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inClientData;
	[self _storePacketsWithNumberOfBytes:inNumberBytes numberOfPackets:inNumberPackets inputData:inInputData packetDescriptions:inPacketDescriptions];
}

static void KKAudioQueueOutputCallback(void * inUserData, AudioQueueRef inAQ,AudioQueueBufferRef inBuffer)
{
	AudioQueueFreeBuffer(inAQ, inBuffer);
	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inUserData;
	[self _enqueueDataWithPacketsCount:(int)([self framePerSecond] * 5)];
}

static void KKAudioQueueRunningListener(void * inUserData, AudioQueueRef inAQ, AudioQueuePropertyID inID)
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
