Auto-Release
------------

如果我們今天有一個 method，會回傳一個 Objective-C 物件，假使寫成這樣：

``` objc
- (NSNumber *)one
{
    return [[NSNumber alloc] initWithInt:1];
}
```

那麼，每次用到了由 one 這個 method產生出來的物件，用完之後，都需要記住
要 release這個物件，很容易造成疏忽。慣例上，我們會讓這個 method 回傳
auto-release的物件。像是寫成這樣：

``` objc
- (NSNumber *)one
{
    return [[[NSNumber alloc] initWithInt:1] autorelease];
}
```

所謂的 auto-release 其實也沒有多麼自動，而是說，在這一輪 run loop中我
們先不釋放這個物件，讓這個物件可以在這一輪 run loop 中都可以使用，但是
先打上一個標籤，到了下一輪 run loop 開始時，讓 runtime 判斷有哪些前一
輪runloop 中被標成是 auto-release 的物件，這個時候才減少retain count
決定是否要釋放物件。

我們在這邊遇到了一個陌生的名詞： run loop，我們會在
[Responder](../responder/README.md) 這一章當中說明。

在建立 Foundation 物件的時候，除了可以呼叫 `alloc` 、 `init` 以及
`new`之外（ `new` 這個 method 其實就相當於呼叫了 `alloc` 與 `init`；比
方說，我們呼叫 `[NSObject new]` ，就等同於呼叫了`[[NSObject alloc]
init]` 。），還可以呼叫另外一組與物件名稱相同的method。

以 `NSString` 為例，有一個叫做 `initWithString` 的 instance method，就
有一個對應的 class method 叫做 `stringWithFormat` ，使用這一組method，
就會產生 auto-release 的物件。也就是說，呼叫了
`[NSString stringWithFormat:...]` ，相當於呼叫了`[[[NSString alloc]
initWithFormat:...] autorelease]` 。使用這一組method，可以讓程式碼較為
精簡。
