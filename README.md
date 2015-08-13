# KKBOX iOS 開發教材


在開始之前，我假設你已經知道怎樣下載以及安裝 Xcode，怎造在 Xcode
中建立新的專案，使用 Interface Builder 連結某個
IBOutlet；你也應該已經懂了一些 Objective-C 基本語法，像是：所有的
Objective-C 物件，都是一個指標，而想要呼叫 Objective-C 物件的
method，語法就是使用中括弧夾起來，像是 `[myObject myMethod]`。

我也假設你已經知道怎麼宣告一個 Objective-C
物件，也知道怎麼使用一些基本的 Foundation 物件，像是字串要使用
`NSString`、array 可以使用 `NSArray`、要使用 hash table 的時候，可以呼叫
`NSDictionary`，還有
`NSSet`、`NSIndexPath`、`NSIndexSet`…。你大概也已經知道了 property
語法，怎樣使用 for…in 寫一段 Fast Enumerating
等等。如果這邊講到的事情你還不清楚的話，建議先找一些其他的坊間教材，做一些練習。當你做完一定程度的練習之後，你應該就會遇到
selector 這個名詞。

事先聲明：雖然在敘述上，我們只能夠按照章節順序排列，但是在整個 Cocoa
Framework
中，許多觀念其實互為因果或是互相糾纏，所以在某個章節中，可能會事先講到跟後面章節有關的事情，但是先不要介意，如果遇到這樣的狀況，我們會在後面繼續說清楚。

