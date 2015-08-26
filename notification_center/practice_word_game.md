# 練習：填字遊戲

<video width="332" height="480" controls>
  <source src="game.m4v" type="video/mp4">
Your browser does not support the video tag.
</video>

## 練習範圍

- MVC
- Notification Center
- Quartz2D
- Delegate

## 練習目標

我們要寫一個填字遊戲的操作介面

1. 在畫面中有一個 8x8 的表格，每個格子裡頭只有一個字
2. 當我們按到其中一個格子的時候，這個格子會被反白起來，並且允許我們可
   以修改格子裡頭的文字
3. 在修改格子裡頭的文字的時候，螢幕鍵盤會升起，我們要調整整個 view 的
   位置，避免螢幕鍵盤蓋到我們正在輸入的格子
4. 移動 view 的速度與鍵盤上升的速度要一致
5. 按下 Enter 之後，鍵盤會收回去，格子裡頭會變成修改後的內容
6. 格子裡頭只能夠允許剛好是一個字（character），不可以不輸入，也不可以
   超過一個字。不是剛好一個字，就無法在鍵盤上按下 Enter

## 練習內容

這個練習也是使用 MVC 架構

- View: 使用 Quartz2D 繪製格線與文字
- Controller: 保有一個 8x8 的二維 array 作為 Model

Controller 是 View 的 delegate。當 View 需要重繪內容的時候，會跟
Controller 要求 Model，然後按照 Model 繪製。

我們使用 UIGestureRecognizer 在 View 上面增加了 tap 事件，在 tap 到的
時候，會計算 tap 所發生的位置，決定是哪一格被 highlight。在 highlight
的時候，我們的 View 會把一個 UITextField 疊在剛好與 highlight 的格子範
圍的上方，同時這個 text field 會變成 first responder。整個 view 雖然有
8x8 個文字，但是只共用同一個 text field，其實也是剛好練習 Mac 上的
"field editor" 的觀念。

我們透過 Notification 知道螢幕鍵盤上升與收起，同時透過 user info 知道
螢幕鍵盤升起需要多少時間，以及最後的位置，再根據這些資訊決定怎麼移動我
們的 view。

讓 text field 只能輸入一個字，以及在按下 enter 之後會改變 Controller中
的 8x8 二維 array，並要求 view 重繪，是透過 text field 的 delegate 完成。
