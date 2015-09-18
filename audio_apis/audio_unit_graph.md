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

``` objc
AURenderCallbackStruct callbackStruct;
callbackStruct.inputProcRefCon = (__bridge void *)(self);
callbackStruct.inputProc = KKPlayerAURenderCallback;
status = AudioUnitSetProperty(audioUnit,
	kAudioUnitProperty_SetRenderCallback,
	kAudioUnitScope_Global,
	0, // 代表這個 callback function 綁在 bus 0 上
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

在 render callback function 裡頭要做的事情，就是把已經收到的 packet 透
過 converter 轉換成 LCPM 格式之後回傳。

我們的 render callback function 像這樣：

``` objc
OSStatus KKPlayerAURenderCallback(void *userData,
	AudioUnitRenderActionFlags *ioActionFlags,
	const AudioTimeStamp *inTimeStamp,
	UInt32 inBusNumber,
	UInt32 inNumberFrames,
	AudioBufferList *ioData);
```

在這個 function 中會傳入幾個參數，我們最關心的是 inNumberFrames 與
ioData，inNumberFrames 就是 Remote IO 現在要跟我們要求多少 frame 的
audio 資料，我們會根據這個數量，從 MP3 或其他原始格式中轉出多少 frame
的 LCPM，然後將資料填入到 ioData 中。

至於其他幾個參數，像 userData，就是我們在建立 callback function 的時候
透過 AURenderCallbackStruct 的 inputProcRefCon 傳入的物件指標，因為我
們的 packet 是放在我們的 player 物件中，自然要想辦法讓 callback
function 可以有 reference 找到我們的 player。

ioActionFlags 可以讓我們對 Remote IO Unit 做一些額外的操作，假如我們並
沒有 pakcet 可以給 Remote IO 播放，或是發生了其他錯誤，這時候我們就會
在 ` *ioActionFlags |= kAudioUnitRenderAction_OutputIsSilence;` 這行裡
頭告訴告訴 Remote IO 應該靜音。至於 inTimeStamp 則代表這個 callback 是
在什麼時間被呼叫的。

inBusNumber 代表這個 callback function 被連接到 Remote IO 的哪個 bus，
前面提到，Remote IO 的 bus 0 是輸出，用來播放，bus 1 則用來錄音，我們
現在是處理播放，自然會連接到 bus 0。

如果我們不是對 Remote IO，而是對像 mixer 之類的 node 設定 callback，也
可能會設定在 bus 0 之外的 bus，像一個 mixer 在 bus 0 或 bus 1 分別設定
了兩個 callback，就可以對兩個輸入來源做混音處理。

### 實作 Conveter 的 Fill Callback

AudioConverterRef 有三個可以用來轉換格式的 C function：

- AudioConverterConvertBuffer
- AudioConverterConvertComplexBuffer
- AudioConverterFillComplexBuffer

雖然有三個 function，但實際上可以用的只有
`AudioConverterFillComplexBuffer`；`AudioConverterConvertBuffer` 與
`AudioConverterConvertComplexBuffer` 都只能夠處理不同的 LPCM 格式之間
的轉換，像是把整數的 LCPM 轉成浮點的 LCPM。我們想要把 MP3 或 AAC 格式
轉成 LPCM，只能夠呼叫 `AudioConverterFillComplexBuffer`。

呼叫 `AudioConverterFillComplexBuffer` 的時候，必須要傳入一個converter
callback function，我們叫他 filler，我們在這邊傳入了
`KKPlayerConverterFiller`，並且要傳入一個 AudioBufferList。在filler
function 中，我們把 packet 一個一個餵給 converter讓 converter 轉換，轉
出的資料就會填入到傳入的 AudioBufferList；轉換完畢之後，
AudioBufferList 裡頭就會是轉好的 LPCM 檔案。

接著，我們就可以把 AudioBufferList 中的資料，塞入
`KKPlayerAURenderCallback` 傳入的 ioData 中。如果沒有資料，就像上面說
的，透過 ioActionFlags 暫停播放。

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
	UInt32 bufferSize = 4096 * 4;
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
		AudioFileStreamOpen((__bridge void *)(self),
			KKAudioFileStreamPropertyListener,
			KKAudioFileStreamPacketsCallback,
			kAudioFileMP3Type, &audioFileStreamID);
		URLConnection = [[NSURLConnection alloc] initWithRequest:[NSURLRequest requestWithURL:inURL] delegate:self];
		playerStatus.stopped = YES;
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
	[self pause];
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
		if (playerStatus.stopped) {
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
					(__bridge void *)(self),
					&packetSize, renderBufferList, NULL);
				if (noErr != status && KKAudioConverterCallbackErr_NoData != status) {
					[self pause];
					return -1;
				}
				else if (!packetSize) {
					inIoData->mNumberBuffers = 0;
				}
				else {
					// 在這邊改變 renderBufferList->mBuffers[0].mData
				    // 可以產生各種效果
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

OSStatus KKPlayerAURenderCallback(void *userData,
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

### 還有什麼要做的？

這個 player 和我們在前一章的 Audio Queue player 一樣，把 MP3 資料全部
放在記憶體中，一樣會有太佔用記憶體的問題，在遠端檔案太大的時候，我們應
該要把一部分資料放到暫存檔中。不過，如果我們用了暫存檔，也要記得盡量從
暫存檔中一次多拿一些資料，不要每個 packet 一點一點的做 fread，因為其實
iPhone 的 storage 很慢，如果我們一邊在讀取檔案，一邊有任何的 App 在寫
入檔案，就可能影響讀取的速度，而讓聲音聽起來斷斷續續。

另外，我們也沒有實作處理播放時間的部份，這個 player 距離產品 code 還有一
段距離。

總之，在這個版本的 player 中，我們已經可以拿到 LPCM 資料了，我們可以用
這些資料做出許多應用，像是畫出頻譜圖，或是在餵給硬體之前改變LPCM 資料，
直接改變要輸出的聲音。
