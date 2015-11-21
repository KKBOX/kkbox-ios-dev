Auto Layout
======

Auto Layout：**透過描述 views 之間相對關係動態計算 views 尺寸與位置的方法**。

回過頭來看蘋果會在 WWDC2012 發表 Auto Layout 不是沒有原因的，
對應同年9/21發表螢幕尺寸較長的 iPhone 5，蘋果已經找到因應多尺寸螢幕UI的解決方案。

Auto Layout 之於 Frame Layout 就好比高階程式語言之於組合語言，所以我們也可以說 Auto Layout 是 UI 界的高階語言，當然它也比較容易理解同時效能也會比直接寫低階語言慢。透過 Constraint 來描述 view 之間的關係(通常是距離)，Runtime 會把所有的 Conastaints 丟到 Auto Layout Engine 去計算，在由 Engine 告訴每一個 view 該擁有的尺寸及位置。

案例：
iOS team 一行人在路上撿到 800 塊，他們正在煩惱著怎麼分這筆錢，`如果以 Frame Layout 來分的話`不管撿到多少錢， Zonble 拿 500，清煩拿300，賣摳拿100，依序每個人有拿固定的數目，但就會有人拿不到。

`如果以 Auto Layout 來分的話`就會變成，Zonble 拿的比清煩多200，至少要拿 200; 清煩拿的比賣摳多100，至少要拿 50; 最多只能拿 100

如果他們撿到 1500 ，會怎麼分？


在有 Auto Layout 之前即便我們有 Interface builder 的輔助，我們還是必須根據**螢幕尺寸**的變化去計算各個 view 相對的 frame。光是面對多尺寸螢幕裝置就已經夠複雜，當你的 app 需要支援多語系 (Localization) 面對**不同語言字串的長度**; 不同語系的閱讀方向(阿拉伯文是由右至左書寫); 不同 OS 版本使用不同字型(iOS 8 Helvetica Neue, iOS 9 San Francisco fonts)，而且 San Francisco font 還會根據字型大小自動調整字元間距 ; 更慘絕人寰的是，WWDC2015 發表的 multitasking on iPad ，一個畫面同時執行 2 個 app，意味著 Layout 不再是固定的幾種變化，而是幾百種變化。種種跡象都顯示著 Frame layout 已經不再適用，是時候擁抱 Auto Layout。


關於 Auto Layout 我們有寫一些 Sample code 可以參考
https://github.com/gliyao/LCAutolayoutExample