設計 Protocol 與實作 Delegate 的方式
------------------------------------

我們來用 delegate 的想法來實作前面提到的狀況。

### 宣告 Protocol 與 Delegate 的方式

我們先來建立一個 `NSButton` 的 subclass，叫做 `MyButton`， `MyButton`
的delegate 必須實作 `MyButtonDelegate` 這個 protocol。.h 檔案中宣告如
下：

``` objc
#import <Cocoa/Cocoa.h>

@class MyButton;

@protocol MyButtonDelegate
- (void)myButtonDidBecomeClicked:(MyButton *)button;
- (void)myButtonDidBecomeDoubleClicked:(MyButton *)button;
- (void)myButtonDidBecomeTripleClicked:(MyButton *)button;
- (void)myButtonDidBecomeQuadrupleClicked:(MyButton *)button;
@end

@interface MyButton : NSButton
{
    id <MyButtonDelegate> delegate;
}
@property (weak, nonatomic) id <MyButtonDelegate> delegate;
@end
```

逐行解釋這個 header 裡頭的內容：

1.  `#import <Cocoa/Cocoa.h>` ：因為 NSButton 是在 AppKit
    裡頭，所以我們必須呼叫對應的 header。
2.  `@class MyButton;` ：因為在我們接下來的 protocol 宣告中，會用到
    `MyButton` 這個 class，但是 `MyButton` 其實是宣告在
    `MyButtonDelegate` 的下面，所以我們需要先預先宣告 `MyButton` 這個
    class 的存在。
3.  從 `@protocol MyButtonDelegate` 開始，就是在宣告 `MyButtonDelegate`
    這個 protocol 裡頭的四個 method。
4.  接下來宣告 `MyButton` 這個 class，裡頭有一個叫做 delegate 的變數。

### 實作 Delegate Methods

假如我們有一個叫做 `MyController` 的 controller 物件，要成為
`MyButton`的 delegate，我們會這麼做。首先是 .h 部分：

``` objc
@interface MyController : NSObject <MyButtonDelegate>
{
    IBOutlet MyButton *myButton;
}
@end
```

我們要先宣告 `MyController` 有實作 `MyButtonDelegate` 這個
protocol，如此一來，假如 `MyController` 漏了實作哪些定義在
`MyButtonDelegate` 裡頭的
method，在編譯的時候就會跳出警告，要求我們修正，如果我們還是不實作的話，執行時，就會發生找不到
selector 對應的實作的錯誤而 crash。

如果 `MyController` 又是其他物件的 delegate的話，我們可以在這段用大於
與小於間包起來的宣告，繼續加入其他的 protocol的名稱，例如
`<MyButtonDelegate, AnotherProtocol>`。至於一個物件是否有實作某個
protocol，我們可以用 `conformsToProtocol:` 檢查。

在 `MyController` 的實作中，我們就只要將 `myButton` 的 delegate設成自
己，然後實作該實作的 method。我個人不太喜歡在 protocol的宣告中出現大量
註解，因為在實作 protocol 的時候，最方便的方式就是直接把protocol 宣告
的 method 複製貼上，接著逐一把肉放進 protocol定義的骨幹裡；如果當中出
現註解，複製貼上之後，還要把這些註解刪掉，其實還頂煩人。

``` objc
@implementation MyController

- (void)awakeFromNib
{
    [myButton setDelegate:self];
}

#pragma mark - MyButtonDelegate

- (void)myButtonDidBecomeClicked:(MyButton *)button
{
}
- (void)myButtonDidBecomeDoubleClicked:(MyButton *)button
{
}
- (void)myButtonDidBecomeTripleClicked:(MyButton *)button
{
}
- (void)myButtonDidBecomeQuadrupleClicked:(MyButton *)button
{
}

@end
```

我們在這邊透過 `setDelegate:` 指定 delegate，如果我們把 delegate宣告成
是一個 IBOutlet 的話，也可以直接在 Interface Builder 中連結。

### Delegate Methods 是怎麼被呼叫的？

`MyButton` 是怎樣呼叫 delegate 的呢？其實很簡單。

``` objc
@implementation MyButton

- (void)mouseDown:(NSEvent *)theEvent
{
    switch ([theEvent clickCount]) {
        case 1:
            [delegate myButtonDidBecomeClicked:self];
            break;
        case 2:
            [delegate myButtonDidBecomeDoubleClicked:self];
            break;
        case 3:
            [delegate myButtonDidBecomeTripleClicked:self];
            break;
        case 4:
            [delegate myButtonDidBecomeQuadrupleClicked:self];
            break;
        default:
            break;
    }
}

@synthesize delegate;
@end
```
