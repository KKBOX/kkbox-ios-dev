使用 AVAudioEngine 開發播放軟體
-----

AVAudioEngine 是蘋果在 iOS 8 時推出的音訊處理相關 API。

相較於 Core Audio/Audio Graph/Audio Unit 這些 C API，AVAudioEngine 相對比較高
階：我們可以把 AVAudioEngine 想像成是把 Core Audio API 再用 Objective-C 物件包裝
一層，對一些麻煩、重複的工作做一些抽象化，不過，基本上還是要有一些跟音訊處理相關
的背景知識，才有辦法操作 AVAudioEngine。

由於只是包裝一層，所以你還是可以從這些介面中，接觸到原本的 C API，像是
`AVAudioUnit` 這個 class 有個屬性，叫做 `audioUnit`，就可以拿到 `AudioUnit` 型態
的資料；在 `AVAudioBuffer` 中，也可以拿到屬於 C API 的 `AudioBufferList` 資料。

在蘋果剛推出 AVAudioEngine 的時候，個人感覺是，蘋果主要希望讓像是遊戲或混音軟體
的廠商用比較高階的 API，輕鬆製作有多的不同 input 時的混音應用，所以一開始也只有
提供 `AVAudioFile` 與 `AVAudioPCMBuffer` 這些 API，讓你透過 `AVAudioPCMBuffer`從
各種不同的音檔中讀出 PCM buffer，然後丟進 `AVAudioEngine` 混音。在這個時候，還缺
少了處理串流的資料的編碼/解碼的相關 API，像是 `AVAudioCompressedBuffer` 與
`AVAudioConverter`，都是後面幾代 iOS 才陸續出現。在我們的經驗中，如果你想要盡可
能使用 AVAudioEngine API 實作一個串流播放器的話，你會需要用到 iOS 11 以上的
API。

## AVAudioEngine 與 Audio Unit Processing Graph 的關係

我們前面講過 Audio Unit Processing Graph API，所以我們可以了解 AVAudioEngine API
與 Core Audio之間的一些對應關係：

AVAudioEngine 包裝了 AUGraph：在使用 AVAudioEngine API 時，AVAudioEngine 這個
class 本身扮演了 AUGraph 的角色。我們在使用 AUGraph 的時候，當中所有的 AUNode都
要自己手動建立，使用 AVAudioEngine 則比較輕鬆：AVAudioEngine 本身就提供幾個基本
的節點，像是 `inputNode`、`outputNode`、`mainMixerNode` 等，只要直接呼叫
AVAudioEngine的這幾個 property，就會用 lazy 的方式產生出幾個基本的節點。

但，如果你要手動增加其他的節點，還是可以自己建立想要的 AUAudioNode 的 subclass，
像是 EQ 等化器、Reverb 殘響效果…等等，然後透過 AVAudioEngine 的 `-attachNode:`加
入節點，以及用 `-connect:to:fromBus:toBus:format:` 將節點串連起來。AVAudioEngine
的 `-attachNode:`對應到 AUGraph 的 `AUGraphAddNode`，
`-connect:to:fromBus:toBus:format:` 則對應到`AUGraphConnectNodeInput`。

`AUAudioNode` 包裝了 `AUNode` ，像是輸入、輸出、混音，以及播放過程當中的我們想要
加入的各種效果，都是各自的 AUAudioNode。以播放來說，我們會特別注意
`AVAudioPlayerNode`：`AVAudioPlayerNode` 是 `AVAudioEngine` 的播放資料來源，我們
要播放音訊，首先要建立自己的 `AVAudioPlayerNode`，然後連接到 main mixer node 之
類的節點上。

在使用 AUGraph API 的時候，我們的作法是準備好 render callback function，當
AUGraph 需要資料的時候，從 render callback function 中提供資料。至於在使用
AVAudioEngine 時，我們則是反過來，主動透過 schedule… 開頭的一系列 method，或是提
供一個 Audio File，或是把 PCM 資料包在 `AVAudioPCMBuffer` 物件裡頭，餵入
AVAudioPlayerNode。

如果我們想要播放的是直接從網路抓取的音樂資料，而且是壓縮音檔，那麼就必須把 MP3、
AAC 之類的檔案，手動轉換成 PCM 資料。在 AVAudioEngine API 中，有兩種不同的 buffer
物件：`AVAudioPCMBuffer` 與 `AVAudioCompressedBuffer`，兩者都繼承自
`AVAudioBuffer`。

`AVAudioPCMBuffer` 用來包裝 PCM 格式的資料，而如果是壓縮音檔，就得先裝在
`AVAudioCompressedBuffer` 這個容器中，然後，你可以用`AVAudioConverter` 轉換格
式，以播放的情境來說，你就往往需要把`AVAudioCompressedBuffer` 轉換成
``AVAudioCompressedBuffer`，在錄音的情境下則反之。

我們知道，並不是只有一種格式的 PCM，而是可以是 16 位元整數、32 位元整數或 32 位
元浮點數…等等不同格式的 PCM 資料。所以我們可以看到 `AVAudioPCMBuffer` 有三個屬
性：`floatChannelData`、`int16ChannelData` 與 `int32ChannelData`，裡頭分別是不同
型態的 PCM 資料。請注意，在三個屬性當中，一次只會有一個屬性中會有資料，例如，如
果你用的是 16 位元整數的 PCM 資料，那麼就只有 `int16ChannelData` 會拿到資料，其
他兩個屬性當中都會是空的。

## 用 AVAudioEngine 實作播放器

用 AVAudioEngine 實作一個播放器的流程大概是：

* 建立 `AVAudioEngine` 的 instance
* 將各種音效處理的 AVAudioUnit 物件加入到 `AVAudioEngine` 中，這時候會呼叫到
  `-attachNode:`、`-connect:to:fromBus:toBus:format:`…等等
* 建立 `AVAudioPlayerNode` 的 instance，連接到 main mixer node 上
* 發送網路連線抓取音檔
* 將音檔的 packet 儲存起來
* 要求 `AVAudioEngine` 與 `AVAudioPlayerNode` 開始播放
* 將壓縮音檔的 packet 轉換成 `AVAudioPCMBuffer`，餵入 `AVAudioPlayerNode`。

寫起來會像是這樣：

KKSimpleAudioEnginePlayer.h

``` objc
@import Foundation;

@interface KKSimpleAudioEnginePlayer : NSObject
- (instancetype)initWithURL:(NSURL *)inURL;
- (void)play;
- (void)pause;
@property (readonly, getter=isPlaying) BOOL playing;
@end
```

KKSimpleAudioEnginePlayer.m

```objc
#import "KKSimpleAudioEnginePlayer.h"
@import AVFoundation;
@import AudioToolbox;

static void KKAudioFileStreamPropertyListener(void* inClientData,
    AudioFileStreamID inAudioFileStream,
    AudioFileStreamPropertyID inPropertyID,
    UInt32* ioFlags);
static void KKAudioFileStreamPacketsCallback(void* inClientData,
    UInt32 inNumberBytes,
    UInt32 inNumberPackets,
    const void* inInputData,
    AudioStreamPacketDescription *inPacketDescriptions);

@interface KKSimpleAudioEnginePlayer() <NSURLConnectionDelegate>
{
    struct {
        BOOL stopped;
        BOOL loaded;
    } playerStatus ;

	AudioFileStreamID audioFileStreamID;
}

@property (strong, nonatomic) AVAudioEngine *audioEngine;
@property (strong, nonatomic) AVAudioPlayerNode *player;
@property (strong, nonatomic) NSURLConnection *URLConnection;
@property (strong, nonatomic) NSMutableArray<NSData *> *packets;
@property (assign, nonatomic) size_t readHead;
@property (strong, nonatomic) AVAudioConverter *converter;
@property (strong, nonatomic) AVAudioFormat *format;
@property (strong, nonatomic) AVAudioFormat *destinationPCMFormat;
@end

@implementation KKSimpleAudioEnginePlayer

- (instancetype)initWithURL:(NSURL *)inURL
{
	self = [super init];
	if (self) {
		self.audioEngine = [[AVAudioEngine alloc] init];
		self.player = [[AVAudioPlayerNode alloc] init];
		[self.audioEngine attachNode:self.player];
		[self.audioEngine connect:self.player to:self.audioEngine.mainMixerNode fromBus:0 toBus:0 format:nil];
		self.packets = [[NSMutableArray alloc] init];
		self.destinationPCMFormat = [[AVAudioFormat alloc] initWithCommonFormat:AVAudioPCMFormatFloat32 sampleRate:44100 channels:2 interleaved:NO];

		// 第一步：建立 Audio Parser，指定 callback，以及建立 HTTP 連線，
        // 開始下載檔案
		AudioFileStreamOpen((__bridge void *)(self),
            KKAudioFileStreamPropertyListener,
            KKAudioFileStreamPacketsCallback,
            kAudioFileMP3Type, &audioFileStreamID);
        self.URLConnection = [[NSURLConnection alloc] initWithRequest:[NSURLRequest requestWithURL:inURL] delegate:self];
        playerStatus.stopped = YES;

	}
	return self;
}

- (AVAudioPCMBuffer *)read
{
	const NSInteger packetPerSlice = 8;

	AVAudioPCMBuffer *pcmBuffer = [[AVAudioPCMBuffer alloc] initWithPCMFormat:self.destinationPCMFormat frameCapacity:self.format.streamDescription->mFramesPerPacket * packetPerSlice];
	NSError *error = nil;
	AVAudioConverterOutputStatus status = [self.converter convertToBuffer:pcmBuffer error:&error withInputFromBlock:^AVAudioBuffer * _Nullable(AVAudioPacketCount inNumberOfPackets, AVAudioConverterInputStatus * _Nonnull outStatus) {

		if (self.readHead >= self.packets.count) {
			*outStatus = AVAudioConverterInputStatus_EndOfStream;
			return nil;
		}

		NSMutableData *packetData = [NSMutableData data];
		AudioStreamPacketDescription *packetDescriptions = calloc(sizeof(AudioStreamPacketDescription), packetPerSlice);
		NSInteger i = 0;
		for (i = 0; i < packetPerSlice; i++) {
			NSData *data = self.packets[self.readHead];
			AudioStreamPacketDescription packetDescription;
			packetDescription.mVariableFramesInPacket = 1;
			packetDescription.mDataByteSize = (UInt32)data.length;
			packetDescription.mStartOffset = (UInt32)packetData.length;
			memcpy(&packetDescriptions[i], &packetDescription, sizeof(AudioStreamPacketDescription));
			[packetData appendData:data];
			self.readHead++;
			if (self.readHead >= self.packets.count) {
				break;
			}
		}
		AVAudioCompressedBuffer *compressedBuffer = [[AVAudioCompressedBuffer alloc] initWithFormat:self.format packetCapacity:packetPerSlice maximumPacketSize:self.format.streamDescription->mFramesPerPacket];

		memcpy(compressedBuffer.data, packetData.bytes, packetData.length);
		memcpy(compressedBuffer.packetDescriptions, packetDescriptions, sizeof(AudioStreamPacketDescription) * i);
		free(packetDescriptions);

		compressedBuffer.packetCount = (AVAudioPacketCount)i;
		compressedBuffer.byteLength = (uint32_t)packetData.length;
		*outStatus = AVAudioConverterInputStatus_HaveData;
		return compressedBuffer;

	}];
	if (status != AVAudioConverterOutputStatus_HaveData) {
		return nil;
	}
	return pcmBuffer;
}

- (void)enqueueBuffer
{
	// 第六步：讀出 PCM Buffer，加到 AVAudioPlayerNode 中
	AVAudioPCMBuffer *buffer = [self read];
	if (buffer) {
		[self.player scheduleBuffer:buffer completionHandler:^{
			[self enqueueBuffer];
		}];
	}
}

- (void)play
{
	NSError *error = nil;
	[self.audioEngine startAndReturnError:&error];
	if (error) {
		return;
	}
	[self.player play];
	[self enqueueBuffer];
	playerStatus.stopped = NO;
}

- (void)pause
{
	NSError *error = nil;
	[self.audioEngine startAndReturnError:&error];
	[self.player stop];
	playerStatus.stopped = YES;
}

- (double)packetsPerSecond
{
    if (self.format) {
        return self.format.sampleRate / self.format.streamDescription->mFramesPerPacket;
    }
    return 44100.0/1152.0;
}

- (void)_createAudioConverterWithAudioStreamDescription:(AudioStreamBasicDescription *)audioStreamBasicDescription
{
	self.format = [[AVAudioFormat alloc] initWithStreamDescription:audioStreamBasicDescription];
	self.converter = [[AVAudioConverter alloc] initFromFormat:self.format toFormat:self.destinationPCMFormat];
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
        [self.packets addObject:packet];
    }

    //  第五步，因為 parse 出來的 packets 夠多，緩衝內容夠大，因此開始
    //  播放

    if (self.readHead == 0 && [self.packets count] > (int)([self packetsPerSecond] * 3)) {
        if (playerStatus.stopped) {
            [self play];
        }
    }
}

- (BOOL)playing
{
	return playerStatus.stopped == NO;
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

@end

void KKAudioFileStreamPropertyListener(void * inClientData,
    AudioFileStreamID inAudioFileStream,
    AudioFileStreamPropertyID inPropertyID,
    UInt32 * ioFlags)
{
    KKSimpleAudioEnginePlayer *self = (__bridge KKSimpleAudioEnginePlayer *)inClientData;
    if (inPropertyID == kAudioFileStreamProperty_DataFormat) {
        UInt32 dataSize  = 0;
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

        [self _createAudioConverterWithAudioStreamDescription:&audioStreamDescription];
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

    KKSimpleAudioEnginePlayer *self = (__bridge KKSimpleAudioEnginePlayer *)inClientData;
    [self _storePacketsWithNumberOfBytes:inNumberBytes
        numberOfPackets:inNumberPackets
        inputData:inInputData
        packetDescriptions:inPacketDescriptions];
}
```
