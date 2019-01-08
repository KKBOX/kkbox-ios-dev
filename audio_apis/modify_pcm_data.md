修改 LPCM 資料
--------------

在前面兩節中，我們使用 `AudioConverterFillComplexBuffer` 把 MP3 等格式
的資料，轉換成 LPCM 格式後播放。由於 LPCM 格式已經可以算是最原始的資料
了，因此，我們可以透過直接修改 LPCM 資料，創造各種音效。

我們在這邊示範一種最簡單的效果：用相位取消來做去人聲。假設在錄音的過程
中，人聲是只用一個麥克風來錄音，並平均分佈在兩絛聲道上，那麼，只要左右
聲道的音量互減，然後左右聲道都變成是音量互減之後的結果，就可以拿到去除
人聲的效果。不過，這樣也會同時失去立體聲：因為左右聲道的聲音都變成一樣
的了。

我們先來看前一節把 renderList 的資料填入 inIoData 的這段。

``` objc
inIoData->mNumberBuffers = 1;
inIoData->mBuffers[0].mNumberChannels = 2;
inIoData->mBuffers[0].mDataByteSize = renderBufferList->mBuffers[0].mDataByteSize;
inIoData->mBuffers[0].mData = renderBufferList->mBuffers[0].mData;
renderBufferList->mBuffers[0].mDataByteSize = renderBufferSize;
```

我們在這邊的前方，寫一段程式，讓左右聲道互減：

``` objc
size_t sampleCount = renderBufferList->mBuffers[0].mDataByteSize / 4;
for(size_t i = 0; i < sampleCount * 4; i += 4) {
	short *frame = (short *)(renderBufferList->mBuffers[0].mData + i);
	short left = *frame;
	short right = *(frame + 1);
	short new = left - right;
	*frame = new;
	*(frame + 1) = new;
}

inIoData->mNumberBuffers = 1;
inIoData->mBuffers[0].mNumberChannels = 2;
inIoData->mBuffers[0].mDataByteSize = renderBufferList->mBuffers[0].mDataByteSize;
inIoData->mBuffers[0].mData = renderBufferList->mBuffers[0].mData;
renderBufferList->mBuffers[0].mDataByteSize = renderBufferSize;
```

這邊的寫法跟我們設定的 LPCM 格式有關：我們設定的格式是左右聲道分別是兩
個 16 位元帶號整數，因此一個 sample 共有 4 個 bytes，前兩個是左聲道，
後兩個是右聲道。所以 bytes 的大小除四之後，就是 sample 的數量。拿到
left與 right 互減之後，只要這麼一小段程式碼，就可以達到去人聲的效果。—
不過，假如人聲並不是平均分配在左右聲道上，這個作法也就沒什麼作用，也很
有可能會產生預期之外的噪音。

技術上，我們要讓 KKBOX 的每首歌曲都去除人聲，甚至改變音高，做升 key 降
key 的功能，都是做得到的事情。不過在版權議題上，我們不能這麼做。
