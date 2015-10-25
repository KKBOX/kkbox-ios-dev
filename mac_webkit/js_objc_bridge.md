Javascript 與 Objective-C 的溝通
--------------------------------

在寫 JavaScript 的時候，可以使用一個叫做 window的物件，像是我們想要從
現在的網頁跳到另外一個網頁的時候，就會去修改`window.location.href` 的
位置；在我們的 Objective-C程式碼中，如果我們可以取得指定的 WebView 的
指標，也就可以拿到這個出現在JavaScript 中的 window 物件，也就是
`[webView windowScriptObject]`。

這個物件就是 WebView 裡頭的 JavaScript 與我們的 Objective-C程式之間的
橋樑－window 物件可以取得網頁裡頭所有的 JavaScript函數與物件，而如果我
們把一個 Objective-C 物件設定成 `windowScriptObject`的 value，
JavaScript 也便可以呼叫 Objective-C 物件的method。於是，我們可以在
Objective-C 程式裡頭要求 WebView 執行一段JavaScript，也可以反過來讓
JavaScript 呼叫一段用 Objective-C實作的功能。

### 用 Objective-C 取得與設定 JavaScript 物件

要從 Objective-C 取得網頁中的 JavaScript 物件，也就是對
`windowScriptObject` 做一些 KVC 呼叫，像是 `valueForKey:` 與
`valueForKeyPath:` 。如果我們在 JavaScript裡頭，想要知道目前的網頁位置，
會這麼寫：

``` js
var location = window.location.href;
```

用 ObjC 就可以這麼呼叫：

``` objc
NSString *location = [[webView windowScriptObject]
    valueForKeyPath:@"location.href"];
```

如果我們要設定 window.location.href，要求開啟另外一個網頁，在 Javascript
裡頭：

``` js
window.location.href = 'http://zonble.net';
```

Objective-C：

``` objc
[[webView windowScriptObject] setValue:@"https://kkbox.com" forKeyPath:@"location.href"];
```

由於 Objective-C 與 Javascript 本身的語言特性不同，在兩種語言之間相互
傳遞東西之間，就可以看到兩者的差別－

-   Javascript 雖然是物件導向語言，但是並沒有 class，所以將 JS 物件傳到
    Objective-C 程式裡頭，除了基本字串會轉換成
    `NSString`、基本數字會轉成 `NSNumber`，像是 Array 等其他物件，在
    Objective-C 中，都是 `WebScriptObject` 這個
    Class。也就是，Javascript 的 Array 不會幫你轉換成 `NSArray`。
-   從 Javascript 裡頭傳一個空物件給 Objective-C 程式，用的不是
    Objective-C 裡頭原本表示「沒有東西」的方式，像是
    `NULL`、`nil`、`NSNull` 等，而是專屬 WebKit 使用的 `WebUndefined`。

所以，如果我們想要看一個 Javascript Array裡頭有什麼東西，就要先取得這
個物件裡頭叫做 length 的 value，然後用 `webScriptValueAtIndex:` 去看在
該 index 位置的內容。假如我們在 JS 裡頭這樣寫：

``` js
var JSArray = {'zonble', 'dot', 'net'};
for (var i = 0; i < JSArray.length; i++) {
    console.log(JSArray[i]);
}
```

Objective-C 裡頭就會變成這樣：

``` objc
WebScriptObject *obj = (WebScriptObject *)JSArray;

NSUInteger count = [[obj valueForKey:@"length"] integerValue];
NSMutableArray *a = [NSMutableArray array];
for (NSUInteger i = 0; i < count; i++) {
    NSString *item = [obj webScriptValueAtIndex:i];
    NSLog(@"item:%@", item);
}
```

### 用 Objective-C 呼叫 JavaScript Function

要用 Objective-C 呼叫網頁中的 JavaScript function，大概有幾種方法。第
一種是直接寫一段跟你在網頁中會撰寫的JavaScript 一模一樣的程式，叫
`windowScriptObject` 用`evaluateWebScript:` 執行。例如，我們想要在網頁
中產生一個新的 JavaScript function，內容是：

``` js
function x(x) {
    return x + 1;
}
```

所以在 Objective-C 中可以這樣寫；

``` objc
[[webView windowScriptObject] evaluateWebScript:@"function x(x) {return x + 1;}"];
```

接下來我們就可以呼叫 `window.x()`：

``` objc
NSNumber *result = [[webView windowScriptObject] evaluateWebScript:@"x(1)"];
NSLog(@"result:%d", [result integerValue]); // Returns 2
```

由於在 Javascript 中，每個 funciton其實都是物件，所以我們還可以直接取
得 `window.x` 叫這個物件執行自己。在 Javascript 裡頭如果這樣寫：

``` js
window.x.call(window.x, 1);
```

Objective-C 中便是這樣：

``` objc
WebScriptObject *x = [[webView windowScriptObject] valueForKey:@"x"];
NSNumber *result = [x callWebScriptMethod:@"call"
                      withArguments:[NSArray arrayWithObjects:x,
                                    [NSNumber numberWithInt:1], nil]];
```

這種讓某個 `WebScriptObject` 自己執行自己的寫法，其實比較不會用於從
Objective-C 呼叫 Javascript 這一端，而是接下來會提到的，由 Javascript
呼叫 Objective-C，因為這樣 Javascript 就可以把一個 callback function
送到 Objective-C，因為這樣 程式裡頭。

如果我們在做網頁，我們只想要更新網頁中的一個區塊，就會利用 AJAX
的技巧，只對這個區塊需要的資料，對 server 發出 request，並且在 request
完成的時候，要求執行一段 callback function，更新這一個區塊的顯示內容。從
Javascript 呼叫 Objective-C 也可以做類似的事情，如果 Objective-C
程式裡頭需要一定時間的運算，或是我們可能是在 Obj C
裡頭抓取網路資料，我們便可以把一個 callback function 送到 Objective-C
程式裡，要求 Objective-C 程式在做完工作後，執行這段 callback function。

### DOM

WebKit 裡頭，所有的 DOM 物件都繼承自 `DOMObject`，`DOMObject` 又繼承自
`WebScriptObject`，所以我們在取得了某個 DOM 物件之後，也可以從
Objective-C 程式中，要求這個 DOM 物件執行 Javascript 程式。

假如我們的網頁中，有一個 id 叫做 “\#s” 的文字輸入框（text
input），而我們希望現在鍵盤輸入的焦點放在這個輸入框上，在 JS
裡頭會這樣寫：

``` js
document.querySelector('#s').focus();
```

``` objc
DOMDocument *document = [[webView mainFrame] DOMDocument];
[[document querySelector:@"#s"] callWebScriptMethod:@"focus
                                withArguments:nil];
```

### 用 JavaScript 存取 Objective-C 的 Value

要讓網頁中的 JavaScript 程式可以呼叫 Objective-C 物件，方法是把某個
Objective-C 物件註冊成 JavaScript 中 window 物件的屬性。之後，
JavaScript便也可以呼叫這個物件的 method，也可以取得這個物件的各種
Value，只要是KVC 可以取得的 Value，像是`NSString`、`NSNumber`、
`NSDate`、`NSArray`、`NSDictionary`、`NSValue`…等。`JavaScript`傳
Array 到 Objective-C 時，還需要特別做些處理才能變成 `NSArray`，從
Objective-C 傳一個 `NSArray` 到 JavaScript 時，會自動變成 JavaScript
Array。

首先我們要注意的是將 Objective-C 物件註冊給 window物件的時機，由於每次
重新載入網頁，window物件的內容都會有所變動－畢竟每個網頁都會有不同的
JavaScript程式，所以，我們需要在適當的時機做這件事情。我們首先要指定
WebView 的frame loading delegate（用 setFrameLoadDelegate:），並且實作
`webView:didClearWindowObject:forFrame:`，`WebView` 只要更新了
`windowScriptObject`，就會呼叫這一段程式。假如我們現在要讓網頁中的
`JavaScript` 可以使用目前的 controller 物件，會這樣寫：

``` objc
- (void)webView:(WebView *)sender
       didClearWindowObject:(WebScriptObject *)windowObject
       forFrame:(WebFrame *)frame
{
    [windowObject setValue:self forKey:@"controller"];
}
```

如此一來，只要呼叫 `window.controller`，就可以呼叫我們的 Objective-C物
件。假如我們的 Objective-C Class 裡頭有這些成員變數：

``` objc
@interface MyController : NSObject
{
    IBOutlet WebView *webView;
    IBOUtlet  NSWindow *window;

    NSString *stringValue;
    NSInteger numberValue;
    NSArray *arrayValue;
    NSDate *dateValue;
    NSDictionary *dictValue;
    NSRect frameValue;
}
@end
```

指定一下 Value：

``` objc
stringValue = @"string";
numberValue = 24;
arrayValue = [[NSArray arrayWithObjects:@"text",
                       [NSNumber numberWithInt:30], nil] retain];
dateValue = [[NSDate date] retain];
dictValue = [[NSDictionary dictionaryWithObjectsAndKeys:@"value1", @"key1",
              @"value2", @"key2",
              @"value3", @"key3", nil] retain];
frameValue = [window frame];
```

用 JavaScript 讀讀看：

``` js
var c = window.controller;
var main = document.getElementById('main');
var HTML = '';
if (c) {
    HTML += '<p>' + c.stringValue + '<p>';
    HTML += '<p>' + c.numberValue + '<p>';
    HTML += '<p>' + c.arrayValue + '<p>';
    HTML += '<p>' + c.dateValue + '<p>';
    HTML += '<p>' + c.dictValue + '<p>';
    HTML += '<p>' + c.frameValue + '<p>';
    main.innerHTML = HTML;
}
```

結果如下： ..

> string 24 text,30 2010-09-09 00:01:04 +0800 { key1 = value1; key2 =
> value2; key3 = value3; } NSRect: {{275, 72}, {570, 657}}

不過，如果你看完上面的範例，就直接照做，應該不會直接成功出現正確的結果，
而是會拿到一堆`undefined`，原因是，Objective-C 物件的 Value 預設被保護
起來，不會讓Javascript 直接存取。要讓 Javascript 可以存取 Objective-C
物件的Value，需要實作 `+isKeyExcludedFromWebScript:` 針對傳入的 Key一
一處理，如果我們希望 Javascript 可以存取這個 key，就回傳 `NO`：

``` objc
+ (BOOL)isKeyExcludedFromWebScript:(const char *)name
{
    if (!strcmp(name, "stringValue")) {
        return NO;
    }
    return YES;
}
```

除了可以讀取 Objective-C 物件的 Value 外，也可以設定 Value，相當於在
Objective-C 中使用 `setValue:forKey:`，如果在上面的 Javascript程式中，
我們想要修改 `stringValue`，直接呼叫`c.stringValue = 'new value'` 即可。
像前面提到，在這裡傳給 Objective-C的 Javascript 物件，除了字串與數字外，
class 都是`WebScriptObject`，空物件是 `WebUndefined`。

### 用 JavaScript 呼叫 Objective-C method

Objective-C 的語法沿襲自 Small Talk，Objective-C 的 selector，與
JavaScript 的 function 語法有相當的差異。WebKit
預設的實作是，如果我們要在 JavaScript 呼叫 Objective-C
selector，就是把所有的參數往後面擺，並且把所有的冒號改成底線，而原來
selector 如果有底線的話，又要另外處理。假使我們的 controller 物件有個
method，在 Objective-C 中寫成這樣：

``` objc
- (void)setA:(id)a b:(id)b c:(id)c;
```

在 JS 中就這麼呼叫：

``` js
controller.setA_b_c_('a', 'b', 'c');
```

實在有點醜。所以 WebKit 提供一個方法，可以讓我們把某個 Objective-C
selector 變成好看一點的 Javascript function。我們要實作
`webScriptNameForSelector`:

``` objc
+ (NSString *)webScriptNameForSelector:(SEL)selector
{
    if (selector == @selector(setA:b:c:)) {
        return @"setABC";
    }
    return nil;
}
```

以後就可以這麼呼叫：

``` js
controller.setABC('a', 'b', 'c');
```

我們同樣可以決定哪些 selector 可以給 Javascript
使用，哪些要保護起來，方法是實作
`isSelectorExcludedFromWebScript:`。而我們可以改變某個 Objective-C
selector 在 Javascript 中的名稱，我們也可以改變某個 value 的
key，方法是實作 `webScriptNameForKey:`。

有幾件事情需要注意一下：

#### 用 JavaScript 呼叫 Objective-C 的 property

在上面，我們用 JavaScript 呼叫`window.controller.stringValue`，與設定
裡頭的 value 時，這邊很像我們使用Objective-C 2.0 的語法，但其實做的是
不一樣的事情。用 JavaScript 呼叫`controller.stringValue`，對應到的
Objective 語法是`[controller valueForKey:@"stringValue"]`，而不是呼叫
Objective-C 物件的property。

如果我們的 Objective-C 物件有個 property 叫做`stringValue`，我們知道，
Objective-C property 其實會在編譯時，變成getter/setter method，在
JavaScript 裡頭，我們便應該要呼叫`controller.stringValue()` 與
`controller.setStringValue_()`。

#### Javascript 中，Function 即物件的特性

Javascript 的 function 是物件，當一個 Objective-C 物件的 method 出現在
Javascript 中時，這個 method 在 Javascript 中，也可以或多或少當做物件
處理。我們在上面產生了setABC，也可以試試看把它倒出來瞧瞧：

``` js
console.log(controller.setABC);
```

我們可以從結果看到：

``` js
function setABC() { [native code] }
```

這個 function 是 native code。因為是 native code，所以我們無法對這個
function 呼叫 call 或是 apply。

另外，在把我們的 Objective-C 物件註冊成 `window.controller`後，我們會
許也會想要讓 controller 變成一個 function 來執行，像是呼叫
`window.controller()`；或是，我們就只想要產生一個可以讓 JS 呼叫的
function，而不是整個物件都放進 Javascript 裡頭。我們只要在 Objective-C
物件中，實作 `invokeDefaultMethodWithArguments:`，就可以回傳在呼叫
`window.controller()` 時想要的結果。

現在我們可以綜合練習一下。前面提到，由於我們可以把 Javascript 物件以
`WebScriptObject` 這個 class 傳入 Objective-C 程式，Objective-C程式中
也可以要求執行 `WebScriptObject` 的各項 function。我們假如想把 A與 B
兩個數字丟進 Objective-C程式裡頭做個加法，加完之後出現在網頁上，於是我
們寫了一個 Objective-C method：

``` objc
- (void)numberWithA:(id)a plusB:(id)b callback:(id)callback
{
    NSInteger result = [a integerValue] + [b integerValue];

    [callback callWebScriptMethod:@"call" withArguments:
        [NSArray arrayWithObjects:callback, [NSNumber
            numberWithInteger:result], nil]];
 }
```

Javascript 裡頭就可以這樣呼叫：

``` js
window.controller.numberWithA_plusB_callback_(1, 2, function(result) {
    var main = document.getElementById('main');
    main.innerText = result;
});
```

### 其他平台上 WebKit 的實作

除了 Mac OS X，WebKit 這幾年也慢慢移植到其他的作業系統與 framework中，
也或多或少都有 Native API 要求 WebView 執行 Js，以及從 JS 呼叫Native
API 的機制。

跟 Mac OS X 比較起來，iOS 上 `UIWebView` 的公開 API 實在少上許多。想要
讓`UIWebView` 執行一段 JavaScript ，可以透過呼叫
`stringByEvaluatingJavaScriptFromString:`，只會回傳字串結果，所以能夠
做到的事情也就變得有限，通常大概就拿來取得像window.title 這些資訊。在
iOS 上我們沒辦法將某個 Objective-C 物件變成 JS物件，所以，在網頁中觸發
了某些事件，想要通知 Objective-C這一端，往往會選擇使用像「zonble://」
這類 Customized URL scheme。

Android 的 `WebView` 物件提供一個叫做 `addJavascriptInterface()` 的
method，可以將某個 Java 物件註冊成 JavaScript 的 `window`物件的某個屬
性，就可以讓 JavaScript 呼叫 Java 物件。不過，在呼叫 Java物件時，只能
夠傳遞簡單的文字、數字，複雜的 JavaScript物件就沒辦法了。而在 Android
上想要 `WebView` 執行一段JavaScript，在文件中沒看到相關資料，網路上面
找到的說法是，可以透過`loadUrl()`，把某段 JavaScript， 用 bookmarklet
的形式傳進去。

在 `QtWebKit` 裡頭，可以對 `QWebFrame` 呼叫
`addToJavaScriptWindowObject`，把某個 `QObject` 暴露在 JavaScript環境
中，我不清楚 JavaScript 可以傳遞哪些東西到 `QObject` 裡頭就是了。在
`QtWebKit` 中也可以取得網頁裡頭的 DOM物件（`QWebElement`、
`QWebElementCollection`），我們可以對 `QWebFrame`還有這些 DOM 物件呼叫
`evaluateJavaScript`，執行 Javascript。

GTK 方面，因為是 C API，所以在應用程式與 Javascript。之間，就不是透過
操作包裝好的物件，而是呼叫 WebKit 裡頭 JavaScript Engine的 C API。
