在 AUGraph 中串接 AudioUnit
---------------------------

前一節當中示範了如何只用 Remote IO 呼叫 Audio Unit Processing Graph
API，在這一節中，整個 player 的架構基本上是差不多的，不過，我們從建立
Remote IO component 換成 AUGraph，並且在 AUGraph 當中串接多個 node。在
這個範例中，我們建立了三個 node：mixer、EQ 等化器，然後輸出到 Remote
IO 上。

所以我們有以下成員變數：

``` c
AUGraph audioGraph; // audio graph
AudioUnit mixerUnit; // mixer
AudioUnit EQUnit; // EQ
AudioUnit outputUnit; // remote IO
```

### 建立 Audio Graph

建立 Audio Graph 的過程很簡單，就是呼叫 `NewAUGraph` 與 `AUGraphOpen`，
等到晚一點，我們把 Audio Graph 的內容設定完畢之後，我們還要呼叫
`AUGraphInitialize`。

``` c
NewAUGraph(&audioGraph);
AUGraphOpen(audioGraph);
```

### 建立與串接 node

我們每次要建立一個 node 的時候，都要傳入指定的
AudioComponentDescription，但是設定 AudioComponentDescription 的 code
非常冗長，而且有大量的重複。因此，不少人在寫這段 code 的時候，會想要把
每個 node 再包裝一層變成 Objective-C 物件，像
[DDCoreAudio](https://bitbucket.org/ddribin/ddcoreaudio) 就是這樣的專
案。

蘋果在 iOS 7 推出的 AVAudioEngine 也像是一個 AUGraph 的 Objective-C
wrapper，AVAudioEngine 這個 class 本身就像是 AUGraph，而我們可以透過呼
叫 `-attachNode:` 增加 AVAudioNode 物件，然後用 `-connect:to:format:`
這類的 method 串接 AVAudioNode。

要在 Audio Graph 中建立新的 node，方法是呼叫 `AUGraphAddNode`，然後可
以用 `AUGraphConnectNodeInput` 串接。這段建立與串接實在看起來很可怕：

``` c
// 建立 mixer node
AudioComponentDescription mixerUnitDescription;
mixerUnitDescription.componentType= kAudioUnitType_Mixer;
mixerUnitDescription.componentSubType = kAudioUnitSubType_MultiChannelMixer;
mixerUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
mixerUnitDescription.componentFlags = 0;
mixerUnitDescription.componentFlagsMask = 0;
AUNode mixerNode;
AUGraphAddNode(audioGraph, &mixerUnitDescription, &mixerNode);

// 建立 EQ node
AudioComponentDescription EQUnitDescription;
EQUnitDescription.componentType= kAudioUnitType_Effect;
EQUnitDescription.componentSubType = kAudioUnitSubType_AUiPodEQ;
EQUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
EQUnitDescription.componentFlags = 0;
EQUnitDescription.componentFlagsMask = 0;
AUNode EQNode;
AUGraphAddNode(audioGraph, &EQUnitDescription, &EQNode);

// 建立 remote IO node
AudioComponentDescription outputUnitDescription;
bzero(&outputUnitDescription, sizeof(AudioComponentDescription));
outputUnitDescription.componentType = kAudioUnitType_Output;
outputUnitDescription.componentSubType = kAudioUnitSubType_RemoteIO;
outputUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
outputUnitDescription.componentFlags = 0;
outputUnitDescription.componentFlagsMask = 0;
AUNode outputNode;
AUGraphAddNode(audioGraph, &outputUnitDescription, &outputNode);

// 將 mixer node 連接到 EQ node
AUGraphConnectNodeInput(audioGraph, mixerNode, 0, EQNode, 0);
// 將 EQ node 連接到 Remote IO
AUGraphConnectNodeInput(audioGraph, EQNode, 0, outputNode, 0);
```

### 拿出 Audio Unit

Audio Unit 是 Audio Node 的操作介面，在建立了 Audio Node 之後，我們便
可以用 `AUGraphNodeInfo` 拿出 Audio Unit。

``` objc
AUGraphNodeInfo(audioGraph, outputNode, &outputUnitDescription, &outputUnit);
AUGraphNodeInfo(audioGraph, EQNode, &EQUnitDescription, &EQUnit);
AUGraphNodeInfo(audioGraph, mixerNode, &mixerUnitDescription, &mixerUnit);
```

### 設定輸入與輸出格式

每個 Audio Unit 在互相串接後，就會將前一個 Audio Unit 的輸出送到下一個
Audio Unit 的輸入端，因此我們要設定每個 Audio Unit 的輸入與輸出格式，
讓音訊資料可以正確通過每一個 Audio Unit。設定輸入與輸出格式的方式是修
改 `kAudioUnitProperty_StreamFormat` 這項屬性，如果要改輸入格式，就將
scope 設定為 `kAudioUnitScope_Input`，反之就是 `kAudioUnitScope_Output`。

我們的第一個 Audio Unit 是 mixer，而接下來會把 render callback
function 綁在 mixer 的 bus 0，後面也全部是在 bus 0 上串接，因此，我們
設定了 mixer 與 EQ 的輸入與輸出格式。由於送到 Remote IO 就會播放，我們
就不用設定 Remote IO 的輸出格式了。

``` c
AudioStreamBasicDescription audioFormat = KKSignedIntLinearPCMStreamDescription();
AudioUnitSetProperty(mixerUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Input, 0, &audioFormat, sizeof(audioFormat));
AudioUnitSetProperty(mixerUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Output, 0, &audioFormat, sizeof(audioFormat));
AudioUnitSetProperty(EQUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Input, 0, &audioFormat, sizeof(audioFormat));
AudioUnitSetProperty(EQUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Output, 0, &audioFormat, sizeof(audioFormat));
AudioUnitSetProperty(outputUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Input, 0, &audioFormat, sizeof(audioFormat));
```

### 設定最大 Frame Per Slice

關於這項設定，要參考蘋果
[Technical Q&A QA1606](https://developer.apple.com/library/ios/qa/qa1606/_index.html)
這份重要的說明。Audio Unit API 是一種 pull model 的設計，不是我們主動
把音訊資料推給 AUGraph，而是讓 AUGraph 透過 render callback function
跟我們索取資料，至於會跟我們要求多少資料，是由系統決定，而不是由我們決
定。

而 AUGraph 會跟我們要求多少資料會變動的，平常的時候，一次會跟我們要求
1024 個 frame，但是當 iOS 裝置在 lock screen 的時候，基於節電的理由，
會變成一次跟我們要比較多的資料，變成 4096 個 frame，但每個 Audio Unit
預設可以通過的 frame 數量是 1156。所以，如果不調整設定，當裝置進入
lock screen 之後，就會因為中間串接的 Audio Unit 無法通過 4096 個 frame
而造成無法播放。

因此我們要將每個 Audio Unit 可以通過的資料量設成 4096。

``` c
UInt32 maxFPS = 4096;
AudioUnitSetProperty(mixerUnit, kAudioUnitProperty_MaximumFramesPerSlice, kAudioUnitScope_Global, 0,&maxFPS, sizeof(maxFPS));
AudioUnitSetProperty(EQUnit, kAudioUnitProperty_MaximumFramesPerSlice, kAudioUnitScope_Global, 0,&maxFPS, sizeof(maxFPS));
AudioUnitSetProperty(outputUnit, kAudioUnitProperty_MaximumFramesPerSlice, kAudioUnitScope_Global, 0,&maxFPS, sizeof(maxFPS));
```

這個數字也決定了我們設定給 converter 使用的 renderBufferSize 的大小，
我們設成 4096 * 4，原因是我們需要提供 4096 個 frame，而每個 frame 裡頭
有左右聲道，所以會是兩個十六位元整數，每個十六位元整數是 2 bytes。

最後是設定 render callback，方式與前一節相同。

### 調整 player 的各項設定

在這個 player 中我們可以透過 Audio Unit 調整各項設定，比方說，我們想要
調整這個 player 的音量，便可以透過 AudioUnitSetParameter，對 mixer 的
bus 0 調整 kMultiChannelMixerParam_Volume 屬性。話說當我們只有 Remote
IO 的時候，想要調整音量，只要直接調整 Remote IO 的音量即可，但假如果我
們有了一個 mixer 在裡頭，對 Remote IO 的調整就會變成無效，只能夠透過調
整 mixer 的音量改變音量。

這個範例中也示範了如何使用 EQ 等化器。iOS 的 EQ 等化器有一些 preset，
存放在一個 CFArray 中，我們可以從中選擇喜愛的 preset 套用。在
iPodEQPresetsArray 與 `-selectEQPreset:` 示範了如何使用 EQ 等化器。

``` objc
- (CFArrayRef)iPodEQPresetsArray
{
	CFArrayRef array;
	UInt32 size = sizeof(array);
	AudioUnitGetProperty(EQUnit, kAudioUnitProperty_FactoryPresets, kAudioUnitScope_Global, 0, &array, &size);
	return array;
}

- (void)selectEQPreset:(NSInteger)value
{
	AUPreset *aPreset = (AUPreset*)CFArrayGetValueAtIndex(self.iPodEQPresetsArray, value);
	AudioUnitSetProperty(EQUnit, kAudioUnitProperty_PresentPreset, kAudioUnitScope_Global, 0, aPreset, sizeof(AUPreset));
}
```

完成的範例 Player 如下：

KKSimpleAUPlayer.h

``` objc
#import <Foundation/Foundation.h>
#import <AudioToolbox/AudioToolbox.h>

@interface KKSimpleAUPlayer : NSObject
- (id)initWithURL:(NSURL *)inURL;
- (void)play;
- (void)pause;

@property (readonly, nonatomic) CFArrayRef iPodEQPresetsArray;
- (void)selectEQPreset:(NSInteger)value;
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

	AUGraph audioGraph;
	AudioUnit mixerUnit;
	AudioUnit EQUnit;
	AudioUnit outputUnit;

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
	AUGraphUninitialize(audioGraph);
	AUGraphClose(audioGraph);
	DisposeAUGraph(audioGraph);

	AudioFileStreamClose(audioFileStreamID);
	AudioConverterDispose(converter);
	free(renderBufferList->mBuffers[0].mData);
	free(renderBufferList);
	renderBufferList = NULL;

	[URLConnection cancel];
}

- (void)buildOutputUnit
{
	// 建立 AudioGraph
	OSStatus status = NewAUGraph(&audioGraph);
	NSAssert(noErr == status, @"We need to create a new audio graph. %d", (int)status);
	status = AUGraphOpen(audioGraph);
	NSAssert(noErr == status, @"We need to open the audio graph. %d", (int)status);

	// 建立 mixer node
	AudioComponentDescription mixerUnitDescription;
	mixerUnitDescription.componentType= kAudioUnitType_Mixer;
	mixerUnitDescription.componentSubType = kAudioUnitSubType_MultiChannelMixer;
	mixerUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
	mixerUnitDescription.componentFlags = 0;
	mixerUnitDescription.componentFlagsMask = 0;
	AUNode mixerNode;
	status = AUGraphAddNode(audioGraph, &mixerUnitDescription, &mixerNode);
	NSAssert(noErr == status, @"We need to add the mixer node. %d", (int)status);

	// 建立 EQ node
	AudioComponentDescription EQUnitDescription;
	EQUnitDescription.componentType= kAudioUnitType_Effect;
	EQUnitDescription.componentSubType = kAudioUnitSubType_AUiPodEQ;
	EQUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
	EQUnitDescription.componentFlags = 0;
	EQUnitDescription.componentFlagsMask = 0;
	AUNode EQNode;
	status = AUGraphAddNode(audioGraph, &EQUnitDescription, &EQNode);
	NSAssert(noErr == status, @"We need to add the EQ effect node. %d", (int)status);

	// 建立 remote IO node
	AudioComponentDescription outputUnitDescription;
	bzero(&outputUnitDescription, sizeof(AudioComponentDescription));
	outputUnitDescription.componentType = kAudioUnitType_Output;
	outputUnitDescription.componentSubType = kAudioUnitSubType_RemoteIO;
	outputUnitDescription.componentManufacturer = kAudioUnitManufacturer_Apple;
	outputUnitDescription.componentFlags = 0;
	outputUnitDescription.componentFlagsMask = 0;
	AUNode outputNode;
	status = AUGraphAddNode(audioGraph, &outputUnitDescription, &outputNode);
	NSAssert(noErr == status, @"We need to add an output node to the audio graph. %d", (int)status);

	// 將 mixer node 連接到 EQ node
	status = AUGraphConnectNodeInput(audioGraph, mixerNode, 0, EQNode, 0);
	NSAssert(noErr == status, @"We need to connect the nodes within the audio graph. %d", (int)status);

	// 將 EQ node 連接到 Remote IO
	status = AUGraphConnectNodeInput(audioGraph, EQNode, 0, outputNode, 0);
	NSAssert(noErr == status, @"We need to connect the nodes within the audio graph. %d", (int)status);


	// 拿出 Remote IO 的 Audio Unit
	status = AUGraphNodeInfo(audioGraph, outputNode, &outputUnitDescription, &outputUnit);
	NSAssert(noErr == status, @"We need to get the audio unit of the output node. %d", (int)status);
	// 拿出 EQ node 的 Audio Unit
	status = AUGraphNodeInfo(audioGraph, EQNode, &EQUnitDescription, &EQUnit);
	NSAssert(noErr == status, @"We need to get the audio unit of the EQ effect node. %d", (int)status);
	// 拿出 mixer node 的 Audio Unit
	status = AUGraphNodeInfo(audioGraph, mixerNode, &mixerUnitDescription, &mixerUnit);
	NSAssert(noErr == status, @"We need to get the audio unit of the mixer node. %d", (int)status);

	// 設定 mixer node 的輸入輸出格式
	AudioStreamBasicDescription audioFormat = KKSignedIntLinearPCMStreamDescription();
	status = AudioUnitSetProperty(mixerUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Input, 0, &audioFormat, sizeof(audioFormat));
	NSAssert(noErr == status, @"We need to set input format of the mixer node. %d", (int)status);
	status = AudioUnitSetProperty(mixerUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Output, 0, &audioFormat, sizeof(audioFormat));
	NSAssert(noErr == status, @"We need to set input format of the mixer effect node. %d", (int)status);

	// 設定 EQ node 的輸入輸出格式
	status = AudioUnitSetProperty(EQUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Input, 0, &audioFormat, sizeof(audioFormat));
	NSAssert(noErr == status, @"We need to set input format of the mixer node. %d", (int)status);
	status = AudioUnitSetProperty(EQUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Output, 0, &audioFormat, sizeof(audioFormat));
	NSAssert(noErr == status, @"We need to set input format of the mixer effect node. %d", (int)status);

	// 設定 Remote IO node 的輸入格式
	status = AudioUnitSetProperty(outputUnit, kAudioUnitProperty_StreamFormat, kAudioUnitScope_Input, 0, &audioFormat, sizeof(audioFormat));
	NSAssert(noErr == status, @"We need to set input format of the mixer node. %d", (int)status);

	// 設定 maxFPS
	UInt32 maxFPS = 4096;
	status = AudioUnitSetProperty(mixerUnit, kAudioUnitProperty_MaximumFramesPerSlice, kAudioUnitScope_Global, 0,&maxFPS, sizeof(maxFPS));
	NSAssert(noErr == status, @"We need to set the maximum FPS to the mixer node. %d", (int)status);
	status = AudioUnitSetProperty(EQUnit, kAudioUnitProperty_MaximumFramesPerSlice, kAudioUnitScope_Global, 0,&maxFPS, sizeof(maxFPS));
	NSAssert(noErr == status, @"We need to set the maximum FPS to the EQ effect node. %d", (int)status);
	status = AudioUnitSetProperty(outputUnit, kAudioUnitProperty_MaximumFramesPerSlice, kAudioUnitScope_Global, 0,&maxFPS, sizeof(maxFPS));
	NSAssert(noErr == status, @"We need to set the maximum FPS to the EQ effect node. %d", (int)status);

	// 設定 render callback
	AURenderCallbackStruct callbackStruct;
	callbackStruct.inputProcRefCon = (__bridge void *)(self);
	callbackStruct.inputProc = KKPlayerAURenderCallback;
	status = AUGraphSetNodeInputCallback(audioGraph, mixerNode, 0, &callbackStruct);
	NSAssert(noErr == status, @"Must be no error.");

	status = AUGraphInitialize(audioGraph);
	NSAssert(noErr == status, @"Must be no error.");

	//  建立 converter 要使用的 buffer list
	UInt32 bufferSize = 4096 * 4;
	renderBufferSize = bufferSize;
	renderBufferList = (AudioBufferList *)calloc(1, sizeof(UInt32) + sizeof(AudioBuffer));
	renderBufferList->mNumberBuffers = 1;
	renderBufferList->mBuffers[0].mNumberChannels = 2;
	renderBufferList->mBuffers[0].mDataByteSize = bufferSize;
	renderBufferList->mBuffers[0].mData = calloc(1, bufferSize);

	CAShow(audioGraph);
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
//		[self play];
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
	if (!playerStatus.stopped) {
		return;
	}
	OSStatus status = AUGraphStart(audioGraph);
	NSAssert(noErr == status, @"AUGraphStart, error: %ld", (signed long)status);
	status = AudioOutputUnitStart(outputUnit);
	NSAssert(noErr == status, @"AudioOutputUnitStart, error: %ld", (signed long)status);
	playerStatus.stopped = NO;
}

- (void)pause
{
	OSStatus status = AUGraphStop(audioGraph);
	NSAssert(noErr == status, @"AUGraphStart, error: %ld", (signed long)status);
	status = AudioOutputUnitStop(outputUnit);
	NSAssert(noErr == status, @"AudioOutputUnitStop, error: %ld", (signed long)status);
	playerStatus.stopped = YES;
}

- (CFArrayRef)iPodEQPresetsArray
{
	CFArrayRef array;
	UInt32 size = sizeof(array);
	AudioUnitGetProperty(EQUnit, kAudioUnitProperty_FactoryPresets, kAudioUnitScope_Global, 0, &array, &size);
	return array;
}

- (void)selectEQPreset:(NSInteger)value
{
	AUPreset *aPreset = (AUPreset*)CFArrayGetValueAtIndex(self.iPodEQPresetsArray, value);
	AudioUnitSetProperty(EQUnit, kAudioUnitProperty_PresentPreset, kAudioUnitScope_Global, 0, aPreset, sizeof(AUPreset));
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
		if (!playerStatus.stopped) {
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
