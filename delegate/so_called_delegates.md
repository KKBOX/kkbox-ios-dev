其他平台上所謂的 Delegate
-------------------------

在其他平台中，也用到了 delegate 這個詞，但是意義不太一樣。

### Design Pattern 中所講的 Delegate

就我的理解，Design Pattern 中所講的 Delegate Pattern，比較像是做一個
Wrapper，有一個 class 在實作 method 時，其實是直接把這個 method的實作
傳遞到自己的成員變數物件的實作上。以 Objective-C語言實作會像這樣：

首先產生一個內部的物件，叫做 `MyInnerClass`：

``` objc
@interface MyInnerClass : NSObject
- (void)doSomething;
@end
@implementation MyInnerClass
- (void)doSomething
{
    NSLog(@"Do something");
}
@end
```

然後， `MyClass` 會把該做的事情，都交給 `MyInnerClass`：

``` objc
@interface MyClass : NSObject
{
    MyInnerClass *innerObject;
}
- (void)doSomething;
@end
@implementation MyClass
- (void)dealloc
{
    [innerObject release];
    [super dealloc];
}
- (id)init
{
    self = [super init];
    if (self) {
        innerObject = [[MyInnerClass alloc] init];
    }
    return self;
}
- (void)doSomething
{
    [innerObject doSomething];
}
@end
```

在 Cocoa Framework 中，會比較像是 `NSButton` 與 `NSButtonCell`的關係。
你或許會問，為什麼 Objective-C 裡頭的 delegate 與 Design Pattern裡頭講
的 Delegate Pattern 意義不一樣？為什麼 Objective-C不按照這套用法？但其
實是， Objective-C 使用 delegate 這個觀念，早於Design Pattern 成書。

### C\# 中所謂的 Delegate

C\# 語言中也有 delegate 這個關鍵字，不過用途卻是處理 anonymous
function，以我們上面的例子，我們打算用 C\# 增加按鈕被點選的 event
handler，原本是這麼寫：

``` c#
private void InitializeComponent()
{
    this.button1 = new System.Windows.Forms.Button();
    this.button1.Click += Button1_Click;
}

private void Button1_Click(object sender, System.EventArgs e)
{
}
```

在 C\# 2.0 可以寫成這樣： [^1]

``` c#
private void InitializeComponent()
{
    this.button1 = new System.Windows.Forms.Button();
    this.button1.Click += delegate(object sender, System.EventArgs e) {
    // Do something here.
    };
}
```

關於在 Objective-C 語言中怎麼使用 anonymous function，我們會在接下來的
章節，講 block 的時候討論。

[^1]: 參見 <http://msdn.microsoft.com/en-us/library/bb882516.aspx>
