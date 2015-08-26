# Practice: Snake Game

<script async class="speakerdeck-embed"
data-id="beb706e0e3fc01304bd50ad3499c6df2"
data-ratio="1.2994923857868"
src="//speakerdeck.com/assets/embed.js"></script>

## 練習範圍

- MVC
- DTO (Data Transfer Objects)
- Quartz 2D
- Delegate
- Queue 資料結構
- NSTimer
- UIGestureRecognizer

## 練習目標

我們要寫一個 iOS 版本的貪食蛇遊戲：

- 這個遊戲一開始畫面中央只有一個開始按鈕，按下開始之後才會開始遊戲
- 遊戲開始時，畫面中央有一條長度為 2 單位的蛇，另外有一個水果，蛇的初
  始方向是往左方走
- 每格 0.5 秒蛇會移動一格
- 我們可以用 swipe 手勢改變蛇的方向，但蛇只能夠往左右轉。也就是，當蛇
  在往左方走的時候，只有往上或往下的 swipe 手勢有用
- 如果蛇的頭碰到了水果，蛇的身體長度就會加 2 單位
- 如果蛇的頭碰到了畫面邊界，會從畫面的另外一邊冒出來
- 如果蛇的頭碰到了自己的身體，遊戲結束，重新出現開始按鈕。按下開始按鈕
  會重新開始遊戲

## 練習內容
