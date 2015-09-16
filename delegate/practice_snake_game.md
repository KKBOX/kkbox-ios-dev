練習：貪食蛇
------------

<script async class="speakerdeck-embed"
data-id="beb706e0e3fc01304bd50ad3499c6df2"
data-ratio="1.2994923857868"
src="//speakerdeck.com/assets/embed.js"></script>

### 練習範圍

- MVC
- DTO (Data Transfer Objects)
- Quartz 2D
- Delegate
- Queue 資料結構
- NSTimer
- UIGestureRecognizer

### 練習目標

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

### 練習內容

#### MVC

我們使用 MVC 架構寫這個小遊戲

- Model:
  	- 蛇的 Model
		- property 包括：
		  - 蛇目前的身體每個座標點的 array，座標點是我們自己定義的
            Class，只有兩個屬性：x 與 y，都是 NSInteger
		  - 蛇目前的行進方向
		- method 包括：
		  - 要求蛇移動一格
		  - 要求蛇增加長度
		  - 詢問蛇現在頭是否碰到自己的身體
		  - 詢問蛇的頭是否剛好碰到某個點
  - 水果：就只是一個座標點
- View:
  - 負責在畫面中使用 Quartz 2D 繪製蛇與水果
  - 上面被加上了 UIGestureRecognier 的動作，指向 Controller，當發生
	Swipe 事件時，會通知 Controller 要改變蛇的方向
- Controller:
  - 擁有蛇與水果的 Mode
  - 擁有 View
  - View 與 Controller 是 delegate 的關係，當 View 要重繪時，會跟
	Controller 索取一次蛇與水果的 Model
  - 執行一個 Timer，每執行一次，會要求蛇移動一格，並要求 View 重繪
	一次。並且檢查蛇是否撞到了水果或自己的身體，決定是否要延長蛇的身體，
	或是宣布遊戲結束

#### Queue 資料結構

蛇的身體其實是一個 Queue，Queue 裡頭是一堆座標，當蛇在移動的時候，事實
上是位在尾巴的座標物件被 pop 掉，而我們從蛇的頭的位置 push 進一個座標。

#### 單元測試

我們要對蛇的 Model 寫單元測試。寫單元測試時，要注意 AAA 原則：Arrange、
Act、Assert。像是：

- Arrange：先定義蛇如果要移動一次，移動後身體應該會出現在什麼位置
- Act：要求蛇移動一次
- Assert：確定蛇在移動之後，身體位置跟我們在 Arrange 時設定的位置一致

我們應該先寫蛇的 Model，然後寫單元測試，最後才去寫 View 與 Controller。
