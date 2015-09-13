使用 Audio Queue 開發播放軟體
-----------------------------


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

```
#import "KKSimplePlayer.h"

static void ZBAudioFileStreamPropertyListener(void * inClientData, AudioFileStreamID inAudioFileStream, AudioFileStreamPropertyID inPropertyID, UInt32 * ioFlags);
static void ZBAudioFileStreamPacketsCallback(void * inClientData, UInt32 inNumberBytes, UInt32 inNumberPackets, const void * inInputData, AudioStreamPacketDescription *inPacketDescriptions);
static void ZBAudioQueueOutputCallback(void * inUserData, AudioQueueRef inAQ,AudioQueueBufferRef inBuffer);
static void ZBAudioQueueRunningListener(void * inUserData, AudioQueueRef inAQ, AudioQueuePropertyID inID);

@interface KKPacket : NSObject
@property (assign, nonatomic) UInt32 length;
@property (strong, nonatomic) NSData *data;
@end

@implementation KKPacket
@end

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
		AudioFileStreamOpen((__bridge void * _Nullable)(self), ZBAudioFileStreamPropertyListener, ZBAudioFileStreamPacketsCallback, kAudioFileMP3Type, &audioFileStreamID);
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
	// 如果資料需要解密，也是在這一步解密。
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
		KKPacket *packet = packets[index + readHead];
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
		KKPacket *packet = packets[readIndex];
		memcpy(buffer->mAudioData + totalSize, [packet.data bytes], packet.length);

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
	OSStatus status = AudioQueueNewOutput(audioStreamBasicDescription, ZBAudioQueueOutputCallback, (__bridge void * _Nullable)(self), CFRunLoopGetCurrent(), kCFRunLoopCommonModes, 0, &outputQueue);
	assert(status == noErr);
	status = AudioQueueAddPropertyListener(outputQueue, kAudioQueueProperty_IsRunning, ZBAudioQueueRunningListener, (__bridge void * _Nullable)(self));
	AudioQueuePrime(outputQueue, 0, NULL);
	AudioQueueStart(outputQueue, NULL);
}

- (void)_storePacketsWithNumberOfBytes:(UInt32)inNumberBytes numberOfPackets:(UInt32)inNumberPackets inputData:(const void *)inInputData packetDescriptions:(AudioStreamPacketDescription *)inPacketDescriptions
{
	for (int i = 0; i < inNumberPackets; ++i) {
		SInt64 packetStart = inPacketDescriptions[i].mStartOffset;
		UInt32 packetSize = inPacketDescriptions[i].mDataByteSize;
		assert(packetSize > 0);
		KKPacket *packet = [[KKPacket alloc] init];
		packet.length = packetSize;
		packet.data = [NSData dataWithBytes:inInputData + packetStart length:packetSize];
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

void ZBAudioFileStreamPropertyListener(void * inClientData, AudioFileStreamID inAudioFileStream, AudioFileStreamPropertyID inPropertyID, UInt32 * ioFlags)
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

void ZBAudioFileStreamPacketsCallback(void * inClientData, UInt32 inNumberBytes, UInt32 inNumberPackets, const void * inInputData, AudioStreamPacketDescription *inPacketDescriptions)
{
	// 第四步： Audio Parser 成功 parse 出 packets，我們將這些資料儲存
	// 起來

	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inClientData;
	[self _storePacketsWithNumberOfBytes:inNumberBytes numberOfPackets:inNumberPackets inputData:inInputData packetDescriptions:inPacketDescriptions];
}

static void ZBAudioQueueOutputCallback(void * inUserData, AudioQueueRef inAQ,AudioQueueBufferRef inBuffer)
{
	AudioQueueFreeBuffer(inAQ, inBuffer);
	KKSimplePlayer *self = (__bridge KKSimplePlayer *)inUserData;
	[self _enqueueDataWithPacketsCount:(int)([self framePerSecond] * 5)];
}

static void ZBAudioQueueRunningListener(void * inUserData, AudioQueueRef inAQ, AudioQueuePropertyID inID)
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
