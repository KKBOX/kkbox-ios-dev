JavaScriptCore Framework
------------------------

我們在 Mac OS X 上面，也可以透過 C API，要求 WebView 執行Javascript。
我們可以使用 JavaScriptCore Framework。

先簡單示範一下。如果我們想要簡單改一下 `window.location.href`：

``` objc
JSGlobalContextRef globalContext = [[webView mainFrame] globalContext];
JSValueRef exception = NULL;
JSStringRef script = JSStringCreateWithUTF8CString(
    "window.location.href='http://zonble.net'");
JSEvaluateScript(globalContext, script, NULL, NULL, 0, &exception);
JSStringRelease(script);
```

### JavaScriptCore 簡介

JavaScriptCore 是 WebKit 的 JavaScript 引擎，一般來說，在 Mac OS X上，
我們想要製作各種網頁與 Native API 程式互動的功能，大概不會選擇使用
JavaScriptCore，因為現在寫 Mac OS X 的桌面應用程式，多半會直接選擇使用
Objective-C 語言與 Cocoa API，各種需要的功能，都有像在前面提到的
Objective-C 方案－使用 WebKit Framework 中的 `WebScriptObject` 與各種
DOM 物件。

在 Objective-C 程式中當然可以使用 JavaScriptCore 所提供的 C API，但是
在實際撰寫程式時，遠比使用 Objective-C 呼叫麻煩－你的主要controller 物
件還是用 Objective-C 撰寫的，如果直接用 JavaScriptCore 產生
JavaScriptCore 可以呼叫的 callback，這些 callback C function還是會需要
向 Objective-C 物件要資料，等於是多繞了一圈。

然而如果真的有需要的話，我們的確可以混用 JavaScriptCore 與 WebKit 的
Objective-C API，每個 WebScriptObject 中，都可以用 `JSObject` 這個
method 取得對應的 JSObjectRef。就跟許多 Mac OS X 或 iOS 上的 framework
一樣，Objective-C 物件中往往包含 C 的 API，我們於是可以從 `UIColor` 得
到`CGColor`，從 `UIImage` 得到 `CGImage`，在 `WebScriptObject` 中，就
是`JSObjectRef`。

不過，在 WebKitGTK+ 上，由於 GTK 本身是 C API，所以透過JavaScriptCore，
看來就變成 GTK 應用程式與網頁內的 JS程式相互呼叫、傳遞資料最重要的方法
（不太熟悉GTK，所以不太確定還有哪些其他的方法）。

在 GTK 網站上有一個簡單的 [範例程式](http://webkitgtk.org/gcds.c)，除
了示範怎樣產生一個新 Window，裡頭放一個 WebView開始執行外，還包括怎樣
產生一個網頁中 JavaScript 可以呼叫的 C function，並且將這個 C function
註冊到成 JavaScript 的 `window`物件的某個成員function。看這個程式還頂
有趣的－絕大部分程式的命名規則與風格，都是屬於GTK 的風格，但是在呼叫到
JavaScriptCore 的時候，卻又是蘋果的CoreFoundation 的風格。

JavaScriptCore 雖然是 WebKit 的 JS 引擎，但是其實並不一定需要WebKit。
JavaScriptCore 中所有的 API 都是對著一個 context操作，我們要操控
WebView，就是對著 WebView 中某個 frame 的 global context 操作
（ `[webFrame globalContext]` ），而這個 context可以不需要有一個
WebFrame物件就可以自己產生，我們可以在完全沒有圖形介面、不用到 Web排版
引擎的狀況下，產生、執行 JavaScript 程式碼。

所以可以看到，不少專案將這個引擎從 WebKit 中拆分出來，另外橋接其他的
library，在 GTK 上就有 SEED，讓你可以用 JavaScript 呼叫 `GObject`，用
JavaScript 寫 GTK 應用程式。在 Mac OS X 上則有橋接 JavaScript 與各種
Cocoa 物件的專案，像是 JSCocoa，以及以 JSCocoa 為基礎，作為代替
AppleScript 的 JSTalk，後來則出現了 CocoaScript，像如果你要寫 Sketch的
plug-in，就會用到 CocoaScript 語言。

不過，在這便只大概講一下用 WebView 開啟的網頁，怎麼透過 JavaScriptCore
呼叫 C Function。

### JavaScriptCore 裡頭的物件

我對 JavaScriptCore 這個東西不算熟，就像前面說的，真的在寫 Cocoa
程式往往直接呼叫 `WebScriptObject`。JavaScriptCore
裡頭有幾種基本的資料：

-   `JSGlobalContextRef` ：執行 JavaScript 的 context
-   `JSValueRef` ：在 JavaScript 中所使用的各種資料，包括字串、數字以及
    function，都會包裝成 Value，我們可以從數字、JSStringRef 或 JSObject
    產生 JSValueRef，也可以轉換回來。需要特別注意的是，JS 裡頭的 null
    也是一個 JSValueRef（JSValueMakeUndefined 與 JSValueMakeNull）。
-   `JSStringRef` ：JavaScriptCore 使用的字串。用完記得要 release。
-   `JSObjectRef` ：JavaScript Array、Function 等。

來寫點程式：

``` objc
- (void)webView:(WebView *)sender
  didClearWindowObject:(WebScriptObject *)windowObject
  forFrame:(WebFrame *)frame
{
    JSGlobalContextRef globalContext = [frame globalContext];
    JSStringRef name = JSStringCreateWithUTF8CString("myFunc");
    JSObjectRef obj =
        JSObjectMakeFunctionWithCallback(globalContext, name,
            (JSObjectCallAsFunctionCallback)myFunc);
    JSObjectSetProperty (globalContext, [windowObject JSObject],
        name, obj, 0, NULL);
    JSStringRelease(name);
}
```

因為每次重新載入網頁，JavaScript 裡頭的 `window`這個物件的內容就會更新
一次，所以我們要等待 `WebView` 告訴我們應該要更新`windowObject` 的時候，
我們才做我們要做的事情－在 `window`中加入一個可以讓 JavaScript 呼叫的
C function。我們首先要做兩件事情，第一是取得 `WebFrame` 裡頭的
`globalContext` （ `[frame globalContext]` ），還有 `windowObject` 這
個Objective-C 物件裡頭的 `JSObject` （ `[windowObject JSObject]` ）。

接著，我們要用 C 產生一個 JavaScript function 物件，在這邊用的是
`JSObjectMakeFunctionWithCallback`，代表我們想要產生一個可以用來呼叫 C
Function 的 JavaScript function，我們要提供這個 JavaScript function的
名稱，還有對應到哪個 C function。如果我們想產生的 JavaScript function
不需要呼叫 C function，可以改用 `JSObjectMakeFunction`；最後，我們把這
個function 物件，註冊給 `windowObject` 的 JSObject上，於是我們現在便可
以在 JS 中呼叫 `window.myFunc()` 了。

來個綜合練習－我們現在在 JS 中傳入兩個數字，透過 C function加完之後，
執行一段 JS callback。我們的 JS 程式這麼寫－

``` js
window.myFunc(1, 1, function(result) {
    var main = document.getElementById('main');
    main.innerText = result;
});
```

在 myFunc 中，我們來練習一下 JavaScriptCore 裡頭的一些東西：

``` objc
JSValueRef myFunc(JSContextRef ctx, JSObjectRef function,
    JSObjectRef thisObject, size_t argumentCount,
    const JSValueRef arguments[], JSValueRef* exception)
{
    if (argumentCount < 3) {
        JSStringRef string = JSStringCreateWithUTF8CString("UTF8String");
        JSValueRef result = JSValueMakeString(ctx, string);
        JSStringRelease(string);
        return result;
    }
    if (!JSValueIsNumber(ctx, arguments[0])) {
        JSStringRef string =
        JSStringCreateWithCFString((CFStringRef)@"NSString");
        JSValueRef result = JSValueMakeString(ctx, string);
        JSStringRelease(string);
        return result;
    }
    if (!JSValueIsNumber(ctx, arguments[1])) {
        return JSValueMakeNumber(ctx, 42.0);
    }
    if (!JSValueIsObject(ctx, arguments[2])) {
        return JSValueMakeNull(ctx);
    }

    double leftOperand = JSValueToNumber(ctx, arguments[0], exception);
    double rightOperand = JSValueToNumber(ctx, arguments[1], exception);
    JSObjectRef callback = JSValueToObject(ctx, arguments[2], exception);
    JSValueRef result = JSValueMakeNumber(ctx, leftOperand + rightOperand);
    JSValueRef myArguments[1] = {result};
    JSObjectCallAsFunction(ctx, callback, thisObject, 1,
        myArguments, exception); return JSValueMakeNull(ctx);
}
```

我們希望至少要有三個參數傳進來，前兩個參數是數字，最後一個參數是
`JSObject`，如果不是的話，就簡單回傳一點東西－在這邊可以看到，
`JSStringRef`除了可以用 UTF8 字串產生，也可以從 `CFString` 產生。

我們接下來把 `JSValue` 轉成 double，簡單做個加法，最後用
`JSObjectCallAsFunction`，執行 JavaScript callback－其實這邊還應該要用
`JSObjectIsFunction`，來檢查一下這個 `JSObject` 到底是不是 function才
是。
