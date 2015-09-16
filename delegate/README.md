Delegate 與 Protocol
====================

在閱讀這一章之前，相信你應該已經寫過一些簡單的 iOS 程式，知道如果我們
想要一個像是系統設定（Setting.app）那樣的表格介面，我們會建立
`UITableViewController` 的 subclass，這個 controller 的 view 是
`UITableView`。

當我們想要設定表格中的內容，像是這個表格中有多少個 section、每個
section裡頭有多少 row、每個 row 裡頭又是哪些內容…我們不是直接呼叫
`UITableView`的 method，像是呼叫 `[myTableView setSectionCount:3]` 或
是`[myTableView setRowCount:3 atSection:4]`，而是去實作
`UITableViewController` 裡頭幾個像是 template method 的東西，像
`numberOfSectionsInTableView:`、`tableView:numberOfRowsInSection:`等等。

為什麼不是直接去改變 view，而是 controller 要準備一些不知道會被誰呼叫
的method？理由是：`UITableViewController` 是 `UITableView` 的 data
source與 delegate。那，什麼是 delegate？

用我的話來說，delegate 就是 **將眾多的 callback，集中在一個物件上** 。

如果你還不知道怎麼建立 Table View 或 Collection View，請先做完
*Beginning iOS 7 Development Exploring the iOS SDK* 這本書的第七章到第
十章的相關練習。這本書好像中文只有翻譯出 iOS 5 的版本，叫做《探索 iOS
5 程式開發實戰》，不過章節位置差不多。
