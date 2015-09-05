Selector
========

我個人在學習新事物的時候，通常會這麼挑戰自己：我有沒有辦法用一句話描述
這件事物是什麼？還有，這件事物可以用在哪裡？我用這種挑戰，確認自己是否
完全理解我想要學習的事物。

我們在接下來的章節中，也會以這樣的方式展開。其實這種作法，也就是蘋果的
敘事風格，比方說，當我們去看 WWDC 2015 影片，蘋果在講什麼是
Localization 的時候[^1]，Localization 的簡短定義就是「讓你的 App 說你
的顧客的語言」（Making your app speak your customer's language），從這
麼簡短的話中，我們就可以得到可以繼續發揮的關鍵字：1. 「你的顧客的語言」，
全世界各國的語言，到底有什麼幽微的不同？2. 「Making」，我們該怎麼做？
有什麼技術的細節？—於是，我們繼續一步一步發展出完整的故事。

在台灣，當我們問一位 iOS 工程師「什麼是 delegate」這樣的問題時，得到的
答案可能是「delegate 就是『代理』」，至於代理了什麼、為什麼要代理，卻
又說不上來。我們在 KKBOX 對自己的要求並不只如此。

以我們現在要討論的 Selector 來說，可以做這樣一個簡短的定義：

> **Selector 就是用字串表示某個物件的某個 method**

用更術語的說法會是：

> **Selector 就是 Objective-C 的 virtual table 中指向實際執行 function
> pointer 的一個 C 字串**

那，Selector 有什麼用途呢？

> **因為 method 可以用字串表示，因此，某個 method
> 就可以變成可以用來傳遞的參數。**

至於要更進一步了解 Selector，我們就要從一些更基本的事情開始講起：
Objective-C 裡頭的物件以及 Class，到底是什麼？

[^1]: WWDC 2015  What's New in Internationalization https://developer.apple.com/videos/wwdc/2015/?id=227 ，2:30 左右
