NSCoding
========

NSCoding 是 Cocoa/Cocoa Touch Framework 中的序列化（Serialization）的
實作，所謂的序列化，就是「物件變檔案、檔案變物件」：我們可以將目前程式
中正在使用的物件轉換成資料格式，因此可以存成檔案，或是在網路上傳輸、交
換，或反之，我們也可以把已經儲存的檔案，再恢復成物件。

對比其他的程式語言，JavaScript 的 serialization 格式就是 JSON，而 PHP
語言裡頭，我們會呼叫 `serialize()` 與 `unserialize()` 轉換 PHP 物件與
字串；幾乎重要的程式語言都有自己的 serialization 機制，開發 Mac OS X與
iOS App 如果不懂 NSCoding，相當於寫 JavaScript 卻不懂什麼是 JSON —不過
最近似乎有種JSON 快要統一天下的趨勢，我們現在大概在所有的語言當中，都
可以將 JSON 與各種物件做雙向的轉換。

在 Cocoa/Cocoa Framework 中，我們也可以將資料序列化成 JSON 格式，不過
在屬於這個開發 Framework 的傳統中，會更常使用 Plist 格式與 NSCoding。
Plist 格式有多種格式，包括文字與 Binary 格式，而 NSCoding 在做的事情則
是把各式各樣的不同物件，轉換成 NSData。
