MFI 助聽器
----

在 iOS 裝置上，除了可以使用內建喇叭、線接與藍芽耳機、AirPlay、CarPlay 與其他車用
音響等方式播放音樂之外，還有一種比較少人注意到的裝置，就是 MFI（Made for
iPhone）助聽器。

從 iPhone 5 之後的機種開始，用戶在購買 MFI 助聽器之後，可以在「設定」>「輔助使
用」>「聽力」，然後選取「助聽裝置」，與專屬助聽器連接，之後，用戶就可以直接從助
聽器當中撥打電話，也可以從助聽器收聽到 iPhone 正在播放的音樂或影片的聲音。

在台灣，我們從 2018 年初開始，陸續收到用戶在使用 MFI 助聽器收聽音樂的客訴與需
求，也就是從這個時候開始，MFI 助聽器逐漸在台灣普及，包括像是 Signia Pure X 等機
種。相關資料可以參考蘋果官網的說明：[使用 Made for iPhone 助聽裝置](https://support.apple.com/zh-tw/HT201466) 。

在開發聲音相關的 App 的時候，我們需要注意，當 iPhone 連接到 MFI 助聽器的時候，
iOS 會開始跟你要求更大的 Audio Buffer。在一般的狀況下不太會有問題，但假如你改動
了 AVAudioSession 的 Preferred IO Duration（參考
[setPreferredIOBufferDuration:error:](https://developer.apple.com/documentation/avfoundation/avaudiosession/1616589-setpreferrediobufferduration?language=objc)），
你又使用 Audio Graph 播放的話，就可能得注意資料量太大而無法順利通過 Audio Graph
中的 Audio Unit 的問題。

我們在前面的章節〈[在 AUGraph 中串接 AudioUnit](connecting_audiounit.md) 〉就提
到，AUGraph 會跟我們要求多少資料會變動的，平常的時候，一次會跟我們要求 1024 個
frame，但是當 iOS 裝置在 lock screen 的時候，基於節電的理由，會變成一次跟我們要
比較多的資料，變成 4096 個 frame。但如果改動了 Preferred IO Duration，又接上助聽
器，就很有可能會出現比 4096 更大的 frame per slice。

要解決這個問題，我們的方法是，當我們發現 frame per slice 太大，就用
`setPreferredIOBufferDuration:error:`，再把 Preferred IO Duration 改回來。

我們可以拿前面在〈[在 AUGraph 中串接 AudioUnit](connecting_audiounit.md)〉當中的
範例，解釋如何修正這個問題。我們可以在 Remote IO 的 Audio Unit 上
（outputUnit），多加一個 render callback，攔截可能出現的錯誤，我們建立一個叫做
`_registerOutputRenderCallback` 的 method，以及一個叫做
`KKResetPreferredIOBufferDuration` 的 function。

``` obj

static void KKResetPreferredIOBufferDuration() {
	NSError *error = nil;
	if (![[AVAudioSession sharedInstance] setPreferredIOBufferDuration:4096.0 / 44100.0 error:&error]) {
		NSLog(@"------ setPreferredIOBufferDuration failed: %@", error);
	}
}

- (void)_registerOutputRenderCallback
{
	AURenderCallbackStruct callbackStruct;
	callbackStruct.inputProcRefCon = (__bridge void *)(self);

	callbackStruct.inputProc = KKAudioOutputNodeRenderCallback;
	OSStatus status = AudioUnitSetProperty(outputUnit, kAudioUnitProperty_SetRenderCallback, kAudioUnitScope_Input, 0, &callbackStruct, sizeof(callbackStruct));
}
```

在 buildOutputUnit 的最後的地方呼叫：

``` objc
[self _registerOutputRenderCallback];
KKResetPreferredIOBufferDuration()
```

至於 KKAudioOutputNodeRenderCallback 則是寫成


``` objc
static OSStatus KKAudioOutputNodeRenderCallback(void *userData, AudioUnitRenderActionFlags *ioActionFlags, const AudioTimeStamp *inTimestamp, UInt32 inBusNumber, UInt32 inNumberFrames, AudioBufferList *ioData) {
    KKAudioGraph *self = (__bridge KKAudioGraph *)userData;

    OSStatus status = AudioUnitRender(EQUnit, ioActionFlags, inTimestamp, inBusNumber, inNumberFrames, ioData);

    if (status != noErr) {
        // 在 Frame per slice 太大的時候，就呼叫 KKResetPreferredIOBufferDuration() 重設。
        if (status == kAudioUnitErr_TooManyFramesToProcess) {
            KKResetPreferredIOBufferDuration();
        }
        _fillAudioBufferListWithSilence(ioData);
        *ioActionFlags |= kAudioUnitRenderAction_OutputIsSilence;
        return status;
    }

    return noErr;
}
```

