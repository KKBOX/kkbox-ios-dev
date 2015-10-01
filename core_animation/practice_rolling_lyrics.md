練習：KKBOX 動態歌詞
--------------------

### 練習範圍

- Core Animation
- Timer

### 練習內容

KKBOX 的播放畫面中提供了動態歌詞功能，在畫面中有許多的 text layer，而
當歌曲剛好到了某行歌詞的時候，這行歌詞會 highlight 起來。

請使用 Core Animation 實作這樣的效果。我們可以不必按照真正的歌曲進度實
作這個練習，可以找一首歌曲，把每一行歌詞都變成一個 layer，然後建立
timer，每當 timer 跑一次，就把下一行歌詞 highlight 起來，原本
highlight 的那一行就取消 highlight。
