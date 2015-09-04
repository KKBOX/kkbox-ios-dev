執行測試
--------

寫了測試程式之後，我們可以在 Xcode 裡頭按下 Product->Test 執行單元測試。
如果XCTAssertEqual 這行 assert 出現問題，Xcode 就會立刻出現警告。

不同於之前，我們只能夠在 0.5 秒的時間內，用肉眼判斷我們的程式是不是有
bug，我們現在可以對我們寫出的貪食蛇更有信心：在 test case 之前，要不就
是能通過，要不就是不能通過。當 test case 愈多，也就代表，我們的程式的
確經得起考驗。

我們在 Xcode 裡頭有幾種不同的方式檢視單元測試的結果。在程式碼的編輯畫
面中，每一個 test case 前方會出現一個菱形的圖示，如果這個圖示是空白的，
代表還沒有執行測試，執行完畢之後，如果成功，就會是綠色，反之就會變成紅
色。我們也可以直接用滑鼠按這個菱形圖示，執行 test case。

![Xcode 中執行單元測試](xcode1.png)

在 Xcode 的左方側邊欄的第四項，叫做 Test Navigator，在這邊我們可以找到
我們目前所在專案的所有 test case，在這邊可以看到每個 test case 是成功
或失敗，也可以透過點擊，直接跳到特定 test case 的程式碼。

![Xcode 中執行單元測試](xcode2.png)

在 Xcode 的左方側邊欄的最後一項，叫做 Report Navigator，在這邊可以看到
最近一次完整執行所有 test case 的結果。

![Xcode 中執行單元測試](xcode3.png)
