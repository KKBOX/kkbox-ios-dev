Delegate 與 Protocol
====================

在閱讀這一章之前，相信你應該已經寫過一些簡單的 iOS程式，知道如果我們想
要一個像是系統設定（Setting.app）那樣的表格介面，我們會建立
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





