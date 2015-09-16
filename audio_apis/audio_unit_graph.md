使用 Audio Unit Processing Graph 開發播放軟體
---------------------------------------------

如果我們選擇要使用 Audio Unit Processing Graph API 開發播放網路串流的
Player，前半部的工作跟一個 Audio Queue Player 差不多，還是要先發送網路
連線抓取檔案，建立 parser，並且讓 parser parse 出 pakcet。

但是後半部就變得不一樣，Audio Queue 會幫我們把 packet 從原來的格式轉成
LCPM 格式，所以我們只要建好 Audio Queue Buffer，再 enque 到 Audio
Queue 中；但如果我們使用 Audio Unit Processing Graph API，我們就要自己
透過 converter 將 MP3 等格式轉換成 LPCM 格式播放。

Audio Queue API 與 Audio Unit Processing Graph API 的行為也不太一樣，
使用 Audio Queue API 的時候，我們會主動把 buffer 送到 Audio Queue 中，

在以下這個範例中，我們先不進入如何使用 AUGraph，只使用了 Remote IO—如
果我們不需要一些複雜的播放效果，像 EQ 等化器或混音，只要單獨有 Remote
IO 其實就可以播放。

### 建立 Remote IO 與設定 Render Callback

由於建立 Audio Queue 的時候需要傳入 audio format，所以我們是在 parser
取得了 audio foramt 之後，才建立 Audio Queue。不過，使用 Audio Unit
Processing Graph API 開發播放軟體時，我們是是用 audio format 建立
conveter，所以我們可以在建立 player 的時候，就先建立好 Remote IO 的
Audio Unit，並且對 Remote IO 的 Audio Unit 設定好 render callback。

建立 Remote IO 的 Audio Unit 的時候，我們會先建立一個用來表示
component 條件的 AudioComponentDescription，設定 componentType 為
kAudioUnitType\_Output，代表我們要的是一個輸出用的 node，然後 subtype
設定成 kAudioUnitSubType\_RemoteIO。接著使用 `AudioComponentFindNext`
找到符合的 node，然後從這個 node 中拿出這個 node 的操作介面，也就是
Audio Unit。

``` objc
AudioComponentDescription outputUnitDescription;
bzero(&outputUnitDescription, sizeof(AudioComponentDescription));
outputUnitDescription.componentType = kAudioUnitType_Output;
outputUnitDescription.componentSubType = kAudioUnitSubType_RemoteIO;
outputUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
outputUnitDescription.componentFlags = 0;
outputUnitDescription.componentFlagsMask = 0;

AudioComponent outputComponent = AudioComponentFindNext(NULL, &outputUnitDescription);
OSStatus status = AudioComponentInstanceNew(outputComponent, &audioUnit);
```

接著是從 Audio Unit 設定輸入格式，我們接下來建立 converter 的時候，也
會將 MP3 轉成這種格式的 LPCM，然後送到 Remote IO node。

``` objc
AudioStreamBasicDescription audioFormat = KKSignedIntLinearPCMStreamDescription();
AudioUnitSetProperty(audioUnit,
kAudioUnitProperty_StreamFormat,
	kAudioUnitScope_Input, 0,
	&audioFormat, sizeof(audioFormat));
```

我們來看一下輸入格式的部份。前面提到，即使都叫做 LPCM，裡頭也可能有很
多不同的格式，可能使用整數、也可能使用浮點數表示。

我們在這邊使用的是 16 位元整數，而每個 frame 裡頭有左右兩個聲道，因此
左右兩個聲道分別有兩個 byte，一個 frame 就是兩個 byte 加起來變成四個
bytes，因此 mBytesPerFrame 設定為 4。在 LCPM 格式中，一個 packet 只有
一個 frame，所以每個 packet 也是四個 bytes。

``` objc
AudioStreamBasicDescription destFormat;
bzero(&destFormat, sizeof(AudioStreamBasicDescription));
destFormat.mSampleRate = 44100.0;
destFormat.mFormatID = kAudioFormatLinearPCM;
destFormat.mFormatFlags = kLinearPCMFormatFlagIsSignedInteger;
destFormat.mFramesPerPacket = 1;
destFormat.mBytesPerPacket = 4;
destFormat.mBytesPerFrame = 4;
destFormat.mChannelsPerFrame = 2;
destFormat.mBitsPerChannel = 16;
destFormat.mReserved = 0;
```

接著就是設定 render callback。我們把 render callback 設定成
`KKPlayerAURenderCallback` 這個 function，之後只要呼叫
`AudioOutputUnitStart`，就會在專屬的 audio render thread 中呼叫這個
callback function。

```
AURenderCallbackStruct callbackStruct;
callbackStruct.inputProcRefCon = (__bridge void *)(self);
callbackStruct.inputProc = KKPlayerAURenderCallback;
status = AudioUnitSetProperty(audioUnit,
	kAudioUnitProperty_SetRenderCallback,
	kAudioUnitScope_Global, 0,
	&callbackStruct, sizeof(callbackStruct));
```

### 建立 Converter

等到 Audio File Stream ID parse 出遠端檔案的格式後，我們就可以建立
converter 了。在建立的過程中，要傳入輸入給 converter 的格式，以及
converter 應該要轉出的格式。

``` objc
AudioStreamBasicDescription destFormat = KKSignedIntLinearPCMStreamDescription();
AudioConverterNew(&streamDescription, &destFormat, &converter);
```

### 實作 Render Callback

### 實作 Conveter 的 Fill Callback

KKSimpleAUPlayer.h

``` objc
#import <Foundation/Foundation.h>
#import <AudioToolbox/AudioToolbox.h>

@interface KKSimpleAUPlayer : NSObject
- (id)initWithURL:(NSURL *)inURL;
- (void)play;
- (void)pause;
@property (readonly, getter=isPlaying) BOOL playing;
@end
```

KKSimpleAUPlayer.m


``` objc
#import "KKSimpleAUPlayer.h"

static void KKAudioFileStreamPropertyListener(void* inClientData,
	AudioFileStreamID inAudioFileStream,
	AudioFileStreamPropertyID inPropertyID,
	UInt32* ioFlags);
static void KKAudioFileStreamPacketsCallback(void* inClientData,
	UInt32 inNumberBytes,
	UInt32 inNumberPackets,
	const void* inInputData,
	AudioStreamPacketDescription *inPacketDescriptions);
static OSStatus KKPlayerAURenderCallback(void *userData,
	AudioUnitRenderActionFlags *ioActionFlags,
	const AudioTimeStamp *inTimeStamp,
	UInt32 inBusNumber,
	UInt32 inNumberFrames,
	AudioBufferList *ioData);
static OSStatus KKPlayerConverterFiller(AudioConverterRef inAudioConverter,
	UInt32* ioNumberDataPackets,
	AudioBufferList* ioData,
	AudioStreamPacketDescription** outDataPacketDescription,
	void* inUserData);

static const OSStatus KKAudioConverterCallbackErr_NoData = 'kknd';

static AudioStreamBasicDescription KKSignedIntLinearPCMStreamDescription();

@interface KKSimpleAUPlayer () <NSURLConnectionDelegate>
{
	NSURLConnection *URLConnection;
	struct {
		BOOL stopped;
		BOOL loaded;
	} playerStatus ;

	AudioComponentInstance audioUnit;

	AudioFileStreamID audioFileStreamID;
	AudioStreamBasicDescription streamDescription;
	AudioConverterRef converter;
	AudioBufferList *renderBufferList;
	UInt32 renderBufferSize;

	NSMutableArray *packets;
	size_t readHead;
}
- (double)framePerSecond;
@end

AudioStreamBasicDescription KKSignedIntLinearPCMStreamDescription()
{
	AudioStreamBasicDescription destFormat;
	bzero(&destFormat, sizeof(AudioStreamBasicDescription));
	destFormat.mSampleRate = 44100.0;
	destFormat.mFormatID = kAudioFormatLinearPCM;
	destFormat.mFormatFlags = kLinearPCMFormatFlagIsSignedInteger;
	destFormat.mFramesPerPacket = 1;
	destFormat.mBytesPerPacket = 4;
	destFormat.mBytesPerFrame = 4;
	destFormat.mChannelsPerFrame = 2;
	destFormat.mBitsPerChannel = 16;
	destFormat.mReserved = 0;
	return destFormat;
}

@implementation KKSimpleAUPlayer

- (void)dealloc
{
	AudioFileStreamClose(audioFileStreamID);
	AudioConverterDispose(converter);
	free(renderBufferList->mBuffers[0].mData);
	free(renderBufferList);
	renderBufferList = NULL;

	[URLConnection cancel];
}

- (void)buildOutputUnit
{
	// 建立 remote IO node
	AudioComponentDescription outputUnitDescription;
	bzero(&outputUnitDescription, sizeof(AudioComponentDescription));
	outputUnitDescription.componentType = kAudioUnitType_Output;
	outputUnitDescription.componentSubType = kAudioUnitSubType_RemoteIO;
	outputUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
	outputUnitDescription.componentFlags = 0;
	outputUnitDescription.componentFlagsMask = 0;

	AudioComponent outputComponent = AudioComponentFindNext(NULL, &outputUnitDescription);
	OSStatus status = AudioComponentInstanceNew(outputComponent, &audioUnit);
	NSAssert(noErr == status, @"Must be no error.");

	// 設定 remote IO node 的輸入格式
	AudioStreamBasicDescription audioFormat = KKSignedIntLinearPCMStreamDescription();
	AudioUnitSetProperty(audioUnit,
	kAudioUnitProperty_StreamFormat,
		kAudioUnitScope_Input, 0,
		&audioFormat, sizeof(audioFormat));

	// 設定 render callback
	AURenderCallbackStruct callbackStruct;
	callbackStruct.inputProcRefCon = (__bridge void *)(self);
	callbackStruct.inputProc = KKPlayerAURenderCallback;
	status = AudioUnitSetProperty(audioUnit,
		kAudioUnitProperty_SetRenderCallback,
		kAudioUnitScope_Global, 0,
		&callbackStruct, sizeof(callbackStruct));
	NSAssert(noErr == status, @"Must be no error.");

	//  建立 converter 要使用的 buffer list
	UInt32 bufferSize = 4096;
	renderBufferSize = bufferSize;
	renderBufferList = (AudioBufferList *)calloc(1, sizeof(UInt32) + sizeof(AudioBuffer));
	renderBufferList->mNumberBuffers = 1;
	renderBufferList->mBuffers[0].mNumberChannels = 2;
	renderBufferList->mBuffers[0].mDataByteSize = bufferSize;
	renderBufferList->mBuffers[0].mData = calloc(1, bufferSize);
}

- (id)initWithURL:(NSURL *)inURL
{
	self = [super init];
	if (self) {
		[self buildOutputUnit];

		playerStatus.stopped = NO;
		packets = [[NSMutableArray alloc] init];

		// 第一步：建立 Audio Parser，指定 callback，以及建立 HTTP 連線，
		// 開始下載檔案
		AudioFileStreamOpen((__bridge void * _Nullable)(self),
			KKAudioFileStreamPropertyListener,
			KKAudioFileStreamPacketsCallback,
			kAudioFileMP3Type, &audioFileStreamID);
		URLConnection = [[NSURLConnection alloc] initWithRequest:[NSURLRequest requestWithURL:inURL] delegate:self];
		[self play];
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
	OSStatus status = AudioOutputUnitStart(audioUnit);
	NSAssert(noErr == status, @"AudioOutputUnitStart, error: %ld", (signed long)status);
}
- (void)pause
{
	OSStatus status = AudioOutputUnitStop(audioUnit);
	NSAssert(noErr == status, @"AudioOutputUnitStop, error: %ld", (signed long)status);
}

- (BOOL)isPlaying
{
	UInt32 property = 0;
	UInt32 propertySize = sizeof(property);
	AudioUnitGetProperty(audioUnit,
		kAudioOutputUnitProperty_IsRunning,
		kAudioUnitScope_Global, 0, &property, &propertySize);
	return property != 0;
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

- (void)_createAudioQueueWithAudioStreamDescription:(AudioStreamBasicDescription *)audioStreamBasicDescription
{
	memcpy(&streamDescription, audioStreamBasicDescription, sizeof(AudioStreamBasicDescription));
	AudioStreamBasicDescription destFormat = KKSignedIntLinearPCMStreamDescription();
	AudioConverterNew(&streamDescription, &destFormat, &converter);
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

	if (readHead == 0 & [packets count] > (int)([self framePerSecond] * 3)) {
		if (![self isPlaying]) {
			[self play];
		}
	}
}

#pragma mark -
#pragma mark Properties

- (BOOL)isStopped
{
	return playerStatus.stopped;
}

- (OSStatus)callbackWithNumberOfFrames:(UInt32)inNumberOfFrames
  ioData:(AudioBufferList  *)inIoData busNumber:(UInt32)inBusNumber
{
	@synchronized(self) {
		if (readHead < [packets count]) {
			@autoreleasepool {
				UInt32 packetSize = inNumberOfFrames;
				// 第七步： Remote IO node 的 render callback 中，呼叫 converter 將 packet 轉成 LCPM
				OSStatus status =
				AudioConverterFillComplexBuffer(converter,
					KKPlayerConverterFiller,
					(__bridge void * _Nullable)(self),
					&packetSize, renderBufferList, NULL);
				if (noErr != status && KKAudioConverterCallbackErr_NoData != status) {
					OSStatus status = AudioOutputUnitStop(audioUnit);
					NSAssert(noErr == status, @"AudioOutputUnitStop, error: %ld", (signed long)status);
					return -1;
				}
				else if (!packetSize) {
					inIoData->mNumberBuffers = 0;
				}
				else {
					inIoData->mNumberBuffers = 1;
					inIoData->mBuffers[0].mNumberChannels = 2;
					inIoData->mBuffers[0].mDataByteSize = renderBufferList->mBuffers[0].mDataByteSize;
					inIoData->mBuffers[0].mData = renderBufferList->mBuffers[0].mData;
					renderBufferList->mBuffers[0].mDataByteSize = renderBufferSize;
				}
			}
		}
		else {
			inIoData->mNumberBuffers = 0;
			return -1;
		}
	}

	return noErr;
}


- (OSStatus)_fillConverterBufferWithBufferlist:(AudioBufferList *)ioData
  packetDescription:(AudioStreamPacketDescription** )outDataPacketDescription
{
	static AudioStreamPacketDescription aspdesc;

	if (readHead >= [packets count]) {
		return KKAudioConverterCallbackErr_NoData;
	}

	ioData->mNumberBuffers = 1;
	NSData *packet = packets[readHead];
	void const *data = [packet bytes];
	UInt32 length = (UInt32)[packet length];
	ioData->mBuffers[0].mData = (void *)data;
	ioData->mBuffers[0].mDataByteSize = length;

	*outDataPacketDescription = &aspdesc;
	aspdesc.mDataByteSize = length;
	aspdesc.mStartOffset = 0;
	aspdesc.mVariableFramesInPacket = 1;

	readHead++;
	return 0;
}

@end

void KKAudioFileStreamPropertyListener(void * inClientData,
	AudioFileStreamID inAudioFileStream,
	AudioFileStreamPropertyID inPropertyID,
	UInt32 * ioFlags)
{
	KKSimpleAUPlayer *self = (__bridge KKSimpleAUPlayer *)inClientData;
	if (inPropertyID == kAudioFileStreamProperty_DataFormat) {
		UInt32 dataSize	 = 0;
		OSStatus status = 0;
		AudioStreamBasicDescription audioStreamDescription;
		Boolean writable = false;
		status = AudioFileStreamGetPropertyInfo(inAudioFileStream,
		  kAudioFileStreamProperty_DataFormat, &dataSize, &writable);
		status = AudioFileStreamGetProperty(inAudioFileStream,
		    kAudioFileStreamProperty_DataFormat, &dataSize, &audioStreamDescription);

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
		// 檔案格式資訊，建立 converter

		[self _createAudioQueueWithAudioStreamDescription:&audioStreamDescription];
	}
}

void KKAudioFileStreamPacketsCallback(void* inClientData,
	UInt32 inNumberBytes,
	UInt32 inNumberPackets,
	const void* inInputData,
	AudioStreamPacketDescription* inPacketDescriptions)
{
	// 第四步： Audio Parser 成功 parse 出 packets，我們將這些資料儲存
	// 起來

	KKSimpleAUPlayer *self = (__bridge KKSimpleAUPlayer *)inClientData;
	[self _storePacketsWithNumberOfBytes:inNumberBytes
		numberOfPackets:inNumberPackets
		inputData:inInputData
		packetDescriptions:inPacketDescriptions];
}

static OSStatus KKPlayerAURenderCallback(void *userData,
	AudioUnitRenderActionFlags *ioActionFlags,
	const AudioTimeStamp *inTimeStamp,
	UInt32 inBusNumber,
	UInt32 inNumberFrames,
	AudioBufferList *ioData)
{
	// 第六步： Remote IO node 的 render callback
	KKSimpleAUPlayer *self = (__bridge KKSimpleAUPlayer *)userData;
	OSStatus status = [self callbackWithNumberOfFrames:inNumberFrames
		ioData:ioData busNumber:inBusNumber];
	if (status != noErr) {
		ioData->mNumberBuffers = 0;
		*ioActionFlags |= kAudioUnitRenderAction_OutputIsSilence;
	}
	return status;
}

OSStatus KKPlayerConverterFiller (AudioConverterRef inAudioConverter,
	UInt32* ioNumberDataPackets,
	AudioBufferList* ioData,
	AudioStreamPacketDescription** outDataPacketDescription,
	void* inUserData)
{
	// 第八步： AudioConverterFillComplexBuffer 的 callback
	KKSimpleAUPlayer *self = (__bridge KKSimpleAUPlayer *)inUserData;
	*ioNumberDataPackets = 0;
	OSStatus result = [self _fillConverterBufferWithBufferlist:ioData
		packetDescription:outDataPacketDescription];
	if (result == noErr) {
		*ioNumberDataPackets = 1;
	}
	return result;
}
```
