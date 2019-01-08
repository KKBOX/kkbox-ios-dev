用 Objective-C 操作 DOM
-----------------------

來簡單講講 Objective-C 怎麼操作 DOM。假如我們現在有一個簡單的 HTML 檔案：

``` html
<div id="main">
</div>
```

那麼，在 JavaScript 裡頭，我們會這樣取得 main 這個 div 的 DOM 物件：

``` js
var main = document.getElementById('main');
```

我們便可以把這段程式翻譯成 ObjC：

``` objc
DOMDocument *document = [[webView mainFrame] DOMDocument];
DOMHTMLElement *main = (DOMHTMLElement *)[document getElementById:@"main"];
```

我們也可以用一些比較新的 API，例如 querySelector。

``` js
var main = document.querySelector('#main');
main.style.backgroundColor = '#AAA';
```

``` objc
DOMDocument *document = [[webView mainFrame] DOMDocument];
DOMHTMLElement *main = (DOMHTMLElement *)[document querySelector:@"#main"];
```

假如我們想要在 main 這個 div 中，產生、並加入一個新的 child node，會這樣寫：

``` js
var main = document.getElementById('main');
var child = document.createElement('div');
child.innerHTML = 'child:' + (main.childElementCount + 1);
main.appendChild(child);
```

話說 childElementCount 也是一個比較新的 API，WebKit 要 4.0 之後才支援，
Firefox 則是在 3.5 之後才支援。把上面那段程式翻譯成 ObjC 的話：

``` objc
DOMDocument *document = [[webView mainFrame] DOMDocument];
DOMHTMLElement *main = (DOMHTMLElement *)[document
    getElementById:@"main"];
DOMHTMLElement *newElement = (DOMHTMLElement *)[document
    createElement:@"div"];
newElement.innerHTML = [NSString stringWithFormat:@"child:%d",
    main.childElementCount + 1];
[main appendChild:newElement];
```

如果要修改某個 HTML element 的 CSS 樣式，在 JavaScript 是這樣：

``` js
var main = document.getElementById('main');
main.style.backgroundColor = '#AAA';
```

用 Objective-C 寫是這樣：

``` objc
DOMDocument *document = [[webView mainFrame] DOMDocument];
DOMHTMLElement *main = (DOMHTMLElement *)[document
    getElementById:@"main"];
DOMCSSStyleDeclaration *style = main.style;
[style setProperty:@"background-color" value:@"#AAA" priority:@""];
// 也可以寫成 [style setBackgroundColor:@"#AAA"];
```
