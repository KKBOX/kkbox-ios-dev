從其他平台來看 Objective-C 的 Delegate
--------------------------------------

Delegate 算是入門 Objective-C的另外一個障礙，但了解之後就會知道其實很
簡單，一開始可能覺得並不好懂的主要原因是，delegate的這套作法，跟其他平
台在做同樣的事情的時候，作法比較不一樣。

如果你曾經在其他的平台上開發過應用程式，像是微軟的 .Net平台的話，可以
發現，在 C\# 語言中在講一個 class 有哪些成員的時候，會包括properties、
methods 以及 events；但是我們在講 Objective-C 的 class時，並不會提到
events，C\# 中使用 events 在做的事情，在 Objective-C中，我們往往使用
target/action 與 delegate 實作。 [^1]

雖然在 Mac OS X 與 iOS 上分別有 `NSEvent` 與 `UIEvent`，但是這邊的
events 與 .Net 裡頭講的 events 又是兩件事情：`NSEvent`是用來描述鍵盤按
了哪個按鍵，滑鼠移到了什麼位置，而 `UIEvent`則用來描述觸控事件以及耳機
上的播放控制按鈕等等，單純用來描述從硬體輸入了什麼事件，透過作業系統傳
到我們的應用程式中。而C\# 裡頭所稱的 events，則是用來處理 callback。

所謂 callback 是指，當我們呼叫了一個 function 或 method之後，可能會花
上許多時間，或是計算的是大量資料，或是需要透過網路連線，所以我們並不馬
上要求得到回傳的結果，而是等到一段時間之後，計算結果才會透過另外一個
function/method 傳回來。

現在絕大多數的應用程式開發環境都採用 MVC架構，我們將物件分成三類：
model、view、controller，雖然在不同的平台上往往實作出來的結果不太一樣，
像是在微軟的平台上，往往把window （或是 form）當做是 controller 使用，
由 window物件負責管理放在這個 window 上面所有的 control 元件，但是 Mac
OS X 上面window 並不拿來當做 controller，而是被當成 view，controller
在 window之外，而且一個 controller 也可以控制多個 window…但都大抵如此。

在 MVC 的架構中，我們通常會先建好 controller class，然後加入 model 與
view，變成 controller 的成員變數；因為 model 與 view 是 controller的成
員變數，所以 controller 可以直接呼叫 model 與 view，那麼，當 model與
view 發生了變化，要回來通知 controller，我們也可以稱之為callback，例如，
controller建立了一個按鈕，但是直到按鈕被點選之後，controller 才負責做
事。

在 C\# 中，我們要處理一個 event，就要提供一個 event handler，我們現在
要處理的是 Click：

``` csharp
private void InitializeComponent()
{
    this.button1 = new System.Windows.Forms.Button();
    this.button1.Click += Button1_Click;
}

private void Button1_Click(object sender, System.EventArgs e)
{
}
```

如果只是單純的點選事件的話，我們會用 target/action 實作。但，對照 .Net
framework 裡頭，events 不只是 click 而已，還可能會有 double click、
triple click、quadruple click…，如果是 C\# 裡頭，會這麼寫：

``` csharp
this.button1.Click += Button1_Click;
this.button1.DoubleClick += Button1_DoubleClick;
this.button1.TripleClick += Button1_TripleClick;
this.button1.QuadrupleClick += Button1_QuadrupleClick;
```

變成 Objective-C 的話就可能變成：

``` objc
[button1 setTarget:self];
[button1 setAction:@selector(click:)];
[button1 setDoubleTarget:self];
[button1 setDoubleAction:@selector(doubleClick:)];
[button1 setTripleTarget:self];
[button1 setTripleAction:@selector(tripleClick:)];
[button1 setQuadrupleTarget:self];
[button1 setQuadrupleAction:@selector(QuadrupleClick:)];
```

這樣寫起來實在很讓人煩躁（雖然 `NSTableView` 也的確有
`setDoubleAction:` …），所以，當這樣的東西一多，在 Objective-C語言裡頭，
會直接準備好一個物件，這個物件準備好了所有可以呼叫的method，這個物件就
叫做 delegate，而這些可以呼叫的 method 的集合，叫做protocol。準備好這
個物件之後，我們就不用呼叫這麼多`setDoubleAction:`、`setTripleAction:`，
只要呼叫 `setDelegate:`。


[^1]: 我在寫這章的時候，一直在想拿 C\# 到底是不是好主意，畢竟想要學 Objective-C 語言者，不見得都有 C\# 的基礎。之所以以 C\# 舉例，原因是這份資料其實是來自於當初我個人在公司內部的教材，而公司當時進來的新人之前是寫 C\# 的。
