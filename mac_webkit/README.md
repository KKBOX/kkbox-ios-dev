Mac 上的 WebKit
===============

KKBOX 在推出 iOS 版本之前，在 2008 年九月就推出了 Mac 版本。我們會在這
一章當中，討論一個專屬 Mac OS X 平台的問題： WebKit。

iOS 上面一樣有 Web 瀏覽器元件可以使用，包括 UIWebView、WKWebview 以及
iOS 9 之後的 Safari View Cotroller。不過，因為蘋果政策的關係，iOS 上的
Web 瀏覽元件受限非常大，但相對的，在 Mac OS X 平台上，我們可說可以完整
控制 WebKit framework 中的 WebView。

WebView 可說是 Mac OS X 平台上最複雜的 UI 元件。許多元件上都有delegate
的設計，但 WebView 的 delegate 居然有五種之多，包括 downloadDelegate、
frameLoadDelegate、policyDelegate、resourceLoadDelegate 以及UIDelegate，
從這邊便可以看出 WebView 的複雜；甚至，在蘋果網站中，其實沒有屬於 Web
裡頭的完整文件，像如果我們想要操作 WebView 裡頭的 DOM 物件，DOM 物件有
哪些 Objective-C 介面，其實要去 WebKit 的網站查詢。

不同於 KKBOX 的 iOS、Android 與 WindowsPhone 等 mobile 版本，幾乎所有
的內容都用 Native UI 呈現，KKBOX 的 desktop 版本大量使用來自 Web 的內
容，打開 KKBOX Desktop 版本，迎面而來的線上內容，就是一個專屬 KKBOX 的
網站，我們稱之為 portal site。在 portal site 裡頭的頁面，除了使用各種
網頁技術之外，也經常需要與 client side app 的 native code 溝通。

在這一章中，我們就會討論如何使用 Obejctive-C 語言操作 WebView 裡頭的
DOM、網頁裡頭的 JavaScript 如何與 Objective-C 溝通，以及我們可以如何使
用蘋果的 JavaScript 引擎： JavaScriptCore。
