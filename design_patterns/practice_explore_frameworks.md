練習：探索 Cocoa/Cocoa Touch Framework
--------------------------------------

請做一份書面報告。在這份書面報告中，請從 Cocoa/Cocoa Touch 裡頭的各個
Framework 中（像是 Foundation、UIKit…）等等，找到以下的 Class：

1. 找出五個 singleton class
2. 找出五個有 delegate 或 data source 的 class
3. 找出五個會發送 notification 的 class
4. 找出五個使用 block 的 class

然後：

- 請用一百個中文字以內，說明這個 class 的用途
- 請用一百個中文字以內，說明這個 class 為什麼是 singleton/有 delegate…

範例：

* UIDevice：用來代表目前 App 所在裝置狀態的物件，可以用 currentDevice
  回傳代表目前所在裝置的物件，從物件上我們可以知道裝置名稱、作業系統版
  本等資訊
* 由於我們的 app 一次只會在一台裝置上執行，對我們的 app 而言，就只能夠
  知道一台裝置的存在，因此 UIDevice 被設計成 singleton 物件
