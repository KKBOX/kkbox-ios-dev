單元測試
========

單元測試（Unit Test）就是以程式測試程式。—當你在開發軟體的時候，你不但
撰寫了原本的功能，同時也寫出每個功能對應的程式碼，測試每個功能是否正常
運作。

每次在講到單元測試的時候，總是會有「軟體開發過程是否應該要寫單元測試」
這樣的疑問，你會看到有不少人說寫單元測試會佔用額外的時間，如果開發軟體
是為了創業，那麼目標應該是要儘快完成、儘快上線，才能夠儘快了解用戶與市
場真正的需求。我們對這個問題，我們先不要用爭辯的方式討論，我們來換個方
式—我們用體驗的。

在上一章結束時，我們要寫一個
[貪食蛇](../delegate/practice_snake_game.md)遊戲當做練習。我們在這個遊
戲中，我們於是寫了相關的程式，像是控制蛇的移動，還有在蛇吃到水果的時候，
尾巴要加長…等等。接下來就是執行、測試，我們把程式跑起來，可能會覺得這
條蛇可能哪裡怪怪的，可能在吃到水果的瞬間，尾巴並沒有立刻變長，而是等蛇
再走了一兩格才變得比較對，或，尾巴長出去的方向，好像不太對？

我們可以選擇用眼睛這類的感官檢查程式是否有問題，但是在貪食蛇這個程式中，
Timer 每隔 0.5 秒就會觸發一次，蛇每隔 0.5 秒就會移動一格，如果剛吃到水
果的時候，因為我們的程式邏輯有問題，尾巴長出去的方向不對，我們只有 0.5
秒的時間可以用肉眼捕捉這個問題，那，我們真的有辦法在程式出錯的時候，有
效發覺問題嗎？假如我們寫的程式，其實並不是貪食蛇這種輕鬆的小遊戲呢？

3A 原則
-------

那，我們來試試看單元測試這條途徑。

在 Xcode 裡頭建立專案的時候，Xcode會幫我們的 App 同時建立一個單元測試
的 Bundle—其實蘋果也鼓勵你寫單元測試—在這個 Bundle 中，會出現一個繼承
自 XCTestCase 的 class，在裡頭撰寫任何用 test 開頭的 method，像
`-testHit`，都是一條test case。也就是，我們寫測試的時候，就是寫出一群
用 test 為開頭的 method。

在撰寫測試的時候，基本原則是一次只測試一項 function 或 method，同時一
個 test case 會包含所謂的 3A：Arrange、Action 與 Assert

- Arrange: 先設定我們在這次測試中，所預期的結果
- Action: 就是我們想要測試的 function 或 method
- Assert: 確認在 Action 發生後，確認在執行了想要測試 function 或
  method 後，的確符合我們在 Arrange 階段設定的目標

舉個例子，我們預期一條長度為 6 、正在往左邊移動的蛇，在先往上走一格、
再往右走一格、再往下走一格之後，這條蛇的頭一定會撞到自己的身體，如果我
們的程式說蛇頭沒有撞到，就一定有 Bug。就可以拆解成：

- Arrange: 頭應該會撞到身體
- Action: 讓蛇執行往上右下移動的動作
- Assert: 確認頭真的撞到身體了

這個 case 或許會像這樣：

``` objc
- (void)testHit
{
	KKSnake *snake = [[KKSnake alloc] initWithWorldSize:KKMakeSnakeWorldSize(10, 10) length:6];
	[snake changeDirection:KKSnakeDirectionUp];[[snake move];
	[snake changeDirection:KKSnakeDirectionRight];[snake move];
	[snake changeDirection:KKSnakeDirectionDown];[snake move];
	XCTAssertEqual([snake isHeadHitBody], YES, @"must hit the body.");
}
```

我們可以在 Xcode 裡頭按下 Product->Test 執行單元測試。如果
XCTAssertEqual 這行 assert 出現問題，Xcode 就會立刻出現警告。

覆蓋率（coverage）
-----------------
