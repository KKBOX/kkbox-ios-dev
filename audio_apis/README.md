Audio API
=========

KKBOX 是一套音樂服務，總是要來講一些跟 Audio 相關的開發。

如果只是想要在 iOS 裝置上播放聲音，其實有不少高階而且簡易的 API。在
AVFoudation Framework 就有 AVPlayer 與 AVAudioPlayer ，AVAudioPlayer可
以輕鬆播放 local 的音訊檔案，而 AVPlayer 可以播放網路上的影音串流。如
果我們開發的 App 是一款遊戲，想要在遊戲過程中觸發音效，像開槍的時候可
以發出槍響，在 SpriteKit 裡頭，SKAction 的
`+laySoundFileNamed:waitForCompletion:` 也非常好用。

然而，因為 KKBOX 的商業需求，以及用戶期待的功能，我們沒辦法使用這些高
階 API 開發 KKBOX，必須使用更底層的 audio API，而就整個 Mac OS X與 iOS
開發中會用到的 API 來論，跟 audio 相關的 API 都是出了名的難用。我們在
這一章中會講比較少的 code，而會花比較多力氣講解寫一套 Audio Player 需
要用到哪些東西。
