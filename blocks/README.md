Blocks
======

蘋果在 Mac OS X 10.6 與 iOS 4 之後導入 Block 語法，之後就大幅改變了撰
寫 Objectice-C 語言的方式。

Block 是 Cocoa/Cocoa Touch Framework 中的匿名函式（Anonymous Functions）
的實作。所謂的匿名函式，就是一段 **具有物件性質的程式碼**，這一段程式
碼可以當做函式執行，另一方面，又可以當做物件傳遞；因為可以當做物件傳遞，
所以可以讓某段程式碼變成是某個物件的某個 property，或是當做 method 或
是 function 的參數傳遞，就是因為這種特性，造成最常使用 block 的時機，
就是拿 block 實作 callback。

在有 block 之前，在 Cocoa/Cocoa Touch Framework 上要處理 callback，最
常見的就是使用 delegate（此外也可以使用比較具有 C 語言風格的方式，傳遞
callback function 的 pointer，或是使用 target/action pattern）。在 iOS
4 有了 block 之後，可以看到蘋果自己便大幅改寫了 UIKit 等 Framework 的
API，把原本使用 delegate 處理 callback 的地方，都大幅換成了 block。
