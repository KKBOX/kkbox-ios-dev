我們曾經犯過的低級錯誤
----------------------

最後來提一個我們之前花了半個月才找出問題在哪的 bug。問題出在 protocol
不該這麼設計。我們寫了一個 class，這個 class 有兩個 method： `begin`與
`stop`，按下 `begin` 的時候要開始做一件事情，之後想要停止，就呼叫
`stop`，要開始或要結束的時候，都會通知 delegate。程式大概是這樣：

``` objc
@class MyClass;
@protocol MyClassDelegate <NSObject>
- (void)myClassWillBegin:(MyClass *)myClasss;
- (void)myClassDidBegin:(MyClass *)myClasss;
- (void)myClassWillStop:(MyClass *)myClasss;
- (void)myClassDidStop:(MyClass *)myClasss;
@end

@interface MyClass : NSObject
{
    id <MyClassDelegate> delegate;
}
- (void)begin;
- (void)stop;
@property (assign, nonatomic) id <MyClassDelegate> delegate;
@end

@implementation MyClass

- (void)begin
{
    [delegate myClassWillBegin:self];
    // Do something
    [delegate myClassDidBegin:self];
}
- (void)stop
{
    [delegate myClassWillStop:self];
    // Do something
    [delegate myClassDidStop:self];
}
@synthesize delegate;
@end
```

這個 protocol 有什麼問題呢？前面提到，在 `UITableView` 的 data source
的 method 裡頭不該呼叫 `reloadData` 一樣，這邊的幾個 delegate method
的實作裡頭，也都不該隨意的呼叫 `begin` 與 `stop` ，而我在
`myClassWillBegin:`裡頭想要做一些檢查，如果在某些條件下，這件事情不該
跑起來，而應該停止，所以我在`myClassWillBegin:` 裡頭呼叫了`stop`。但這
麼做，並不會讓這件事情結束，因為 `begin` 這個 method 在對delegate 呼叫
完 `myClassWillBegin:`之後，程式還是會繼續往下走，所以還是把 begin 整
個做完了。

這個 protocol 應該這麼設計：

``` objc
@class MyClass;
@protocol MyClassDelegate <NSObject>
- (BOOL)myClassShouldBegin:(MyClass *)myClasss;
- (void)myClassDidBegin:(MyClass *)myClasss;
- (BOOL)myClassShouldStop:(MyClass *)myClasss;
- (void)myClassDidStop:(MyClass *)myClasss;
@end

@interface MyClass : NSObject
{
    id <MyClassDelegate> delegate;
}
- (void)begin;
- (void)stop;
@property (assign, nonatomic) id <MyClassDelegate> delegate;
@end
@implementation MyClass

- (void)begin
{
    if (![delegate myClassShouldBegin:self]) {
        return;
    }
    // Do something
    [delegate myClassDidBegin:self];
}
- (void)stop
{
    if (![delegate myClassShouldStop:self]) {
        return;
    }
    // Do something
    [delegate myClassDidStop:self];
}
@synthesize delegate;
@end
```
