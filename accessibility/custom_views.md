進階的 Accessibility 設定
-----------------------

如果你要加入無障礙資源的物件是一個 UIView，基本上不需要管這個物件的
frame，也就是，使用者在螢幕上點到什麼範圍，這個物件才會處理與無障礙支
援相關的工作。

不過，很多時候，在使用者介面中的重要文字與圖片，並不是放在直接放在某個
UIView 裡頭，當你使用 Core Animation 製作 UI 的時候，就會把很多東西放
在 CALayer 裡－某個 layer 裡頭的 content 屬性可能是一張重要的圖，或是
用了 CATextLayer 呈現文字。

CALayer 物件本身沒辦法處理無障礙資源。想要讓系統知道某個 layer 的存在
與其所代表的意義，就必須要用一個 container 物件處理，這個 container 通
常是這個 layer 的 super layer 所在的 view，實作
UIAccessibilityContainer protocol。需要注意，自己實作了
UIAccessibilityContainer protocol 之後，會完全改變這個 view 裡頭的無障
礙支援的行為，所以，如果原本這個 view 的 subviews 中還有許多其他的
UIView 物件，如果我們自己實作的 UIAccessibilityContainer 並沒有處理到
這些 subview，這些 subview 原本具有的無障礙支援也就會隨之失效。

我們可以針對每一個 CALayer 物件，產生一個對應的 UIAccessibilityElement
物件，然後透過 setAccessibilityFrame: 指定對應的 CALayer 在畫面上的範
圍。

需要注意的是，這邊所指定的位置，必須是這個 layer 在整個應用程式畫面中
的位置，而不是 layer 原本的 frame 屬性，layer 本身的 frame 屬性只是相
對於上一層的 layer （super layer）的位置而已。我們在這裡可以透過呼叫
convertRect:fromLayer:，轉換出目前 layer，與應用程式啟動時，我們所產生
的那個 UIWindow 物件上的 layer，兩者之間的位置關係，這個 CGRect 才是我
們想要的資訊。

當 layer 在畫面上的位置有改變時，最後記得要發一次
UIAccessibilityLayoutChangedNotification。