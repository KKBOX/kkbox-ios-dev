# KKBOX iOS/Mac OS X 基礎開發教材

這份教材是為了 KKBOX iOS/Mac OS X 開發部門新人訓練期間所設計，目的是培
養出可以開發、維護 KKBOX 的 iOS 與 Mac OS X 版本，以及我們其他軟體產品
的工程師。

我們所重視的不是如何快速上手，而是偏重由下而上的學習：先了解整個開發框
架的底層，以及整個框架的基本觀念，然後才去一個個去看在這個框架中有哪些
API、以及有哪些第三方 library 可以使用。

這和我們的工作型態有關，在一些以專案為主的公司裡頭，可能注重的是如何快
速完成專案、如何以最快的速度完成新 App，交付 App 之後就不再維護。但
KKBOX 算是一個有一些年紀的產品，我們在 2008 年九月推出第一版 Mac OS X
版本，在 2009 年一月推出第一版 iOS 版本，一路從 Mac OS X 10.4、
iPhoneOS 2 的時代寫到現在，我們會花上許多時間解決、並且避免軟體中出現
的問題。而要對系統到底怎麼運作要有一定的認識，才能夠知道怎樣閱讀 crash
report，知道怎麼辨識問題並修正。

不同於坊間大多數的 iOS 開發書籍，因為 KKBOX 同時有 iOS 與 Mac OS X 的
版本，在這份教材中，我們會同時講到 iOS 與 Mac OS X，但如果同一個重要觀
念同時出現在 iOS 與 Mac OS X 中，會以 iOS 為主。

而無論 iOS 或 Mac OS X，都有非常龐大的 SDK 與第三方的生態圈，在各種
API 與 library 的介紹上，也以 KKBOX 產品中會用到的為主，像我們在這份教
材中，會打算講 Audio 相關的部份，絕大多數的 iOS 開發者可能並不需要知道
如何在 iOS 上處理 Audio，但 KKBOX 是一家做音樂服務的公司。至於像遊戲開
發等，雖然我們之前也用過像 Cocos2D 這些功能做過一些小專案，但不會在這
邊佔上篇幅。

在開始之前，我們假設你已經知道怎樣下載以及安裝 Xcode，怎樣在 Xcode中建
立新的專案，也搞定了可以在實機上執行自己的 App 的憑證問題，也會使用
Interface Builder 連結 IBOutlet。

你也應該已經懂了一些 Objective-C 基本語法，像是：所有的Objective-C 物
件，都是一個指標，而想要呼叫 Objective-C 物件的method，語法就是使用中
括弧夾起來，像是`[myObject myMethod]`。

我們也假設你已經知道怎麼宣告一個 Objective-C物件，也知道怎麼使用一些基
本的 Foundation 物件，像是字串要使用`NSString`、array 可以使用
`NSArray`、要使用 hash table 的時候，可以呼叫`NSDictionary`，還有
`NSSet`、`NSIndexPath`、`NSIndexSet`…。你大概也已經知道了 property語法，
怎樣使用 for…in 寫一段 Fast Enumerating 等等。

如果這些都還不熟西，可以先閱讀一些入門書籍，如
[Objective-C Programming: The Big Nerd Ranch Guide](https://www.bignerdranch.com/we-write/objective-c-programming/)
就相當不錯。

事先聲明：雖然在敘述上，我們只能夠按照章節順序排列，但是在整個 Cocoa
Framework中，許多觀念其實互為因果或是互相糾纏，所以在某個章節中，可能
會事先講到跟後面章節有關的事情，但是先不要介意，如果遇到這樣的狀況，我
們會在後面繼續說清楚。
