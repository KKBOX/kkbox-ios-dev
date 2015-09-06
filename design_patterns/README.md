所謂的設計模式
==============

在這邊對前面的章節做一個簡短的回顧。

我們在前面提到了許多名詞，像是 Delegate、Singleton、Notification
Center、Abstract Factory…等等，這些名詞用來描述我們平常撰寫程式時所使
用的方法，我們通常會稱為「設計模式」（Design Pattern）。

根據 Wikipedia的解釋，設計模式就是「在軟體設計中，針對一般問題、並且可
以重複使用的解決方案」（a design pattern is a general reusable
solution to a commonly occurring problem within a given context in
software design）。

iOS 與 Mac OS X 中大量使用上面提到的這些設計模式，但其實平常在開發的時
候，我們倒不會刻意強調設計模式，因為使用的數量已經多到像是呼吸喝水一樣
自然。像我們開始寫第一行 iOS 程式的時候，可能會寫在 AppDelegate 中，而
AppDelegate 這個名詞本身就帶有兩個設計模式的意味：AppDelegate 就是
UIApplication 的 delegate，而 UIApplication 是 Singleton 物件，所以在
寫第一行 code 的時候，你已經遇到了 Singleton 與 Delegate。

坊間有許多跟設計模式有關的專書，非常建議一讀。但我們要注意一下，學習設
計模式與怎麼使用設計模式又是一回事，某些設計模式可能用來解決某些特定問
題，不見得適合其他場合。並不是用了比較多設計模式就會比較好，使用設計模
式並不是軍備競賽。

某方面來說，我們可以把設計模式，想成是一些物件之間的排列關係。而使用設
計模式最後要做的，就是當我們的 App 或 Library 中有很多不同種類的物件時，
我們怎麼把這些物件排列成合理的關係。
