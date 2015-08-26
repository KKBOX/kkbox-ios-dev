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

不過，不同於 JSON 與 JavaScript 的轉換，Objective-C 物件在序列化之後，
不是轉換成文字格式，而是變成 Binary 格式。也就是說，NSCoding 在做的事
情就是，把各式各樣的不同物件，轉換成 NSData。

實作 NSCoding
-------------

NSCoding 的常見用途
-------------------

### Document-based App

### NSUserDefaults

### Copy and Paste

### XIB/NIB

### State Preservation and Restoration



相關閱讀
--------

- [NSCoding Tutorial for iOS: How To Save Your App Data](http://www.raywenderlich.com/1914/nscoding-tutorial-for-ios-how-to-save-your-app-data)
- [NSHipster: NSCoding / NSKeyedArchiver](http://nshipster.com/nscoding/)
- [Use Your Loaf: State Preservation and Restoration](http://useyourloaf.com/blog/state-preservation-and-restoration.html)
