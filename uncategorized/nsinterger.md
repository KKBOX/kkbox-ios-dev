NSInteger 與 NSUInteger
-----------------------

我們在使用 Objcective-C 語言寫程式的時候，往往使用 NSInteger 與
NSUInteger 代表帶號與非帶號的整數。嚴格來說，NSInteger 並不能算是
「Objcective-C 的整數」，因為 NSInteger 其實是 C 語言的形態，而不是
Objcective-C 物件，用來代表數字的 Objcective-C 物件，是像 NSNumber，以
及我們在前面小計算機練習中用到的 NSDecimalNumber。

NSInteger 就是 C 的整數。我們來看一下 <objc/NSObjcRuntime.h> 裡頭的定
義：

``` objc
#if __LP64__ || (TARGET_OS_EMBEDDED && !TARGET_OS_IPHONE) || TARGET_OS_WIN32 || NS_BUILD_32_LIKE_64
typedef long NSInteger;
typedef unsigned long NSUInteger;
#else
typedef int NSInteger;
typedef unsigned int NSUInteger;
#endif
```

在 <objc/NSObjcRuntime.h> 裡頭宣告了一些 Macro，在 64 位元環境下，
NSInteger 被宣告成 long，也就是 64 位元的整數，在 32 位元環境下則是
int。同時，NSUInteger 在 64 與 32 位元環境下分別是 unsigned long 與
unsigned int。

我們寫的程式往往需要同時在 64 位元與 32 位元環境上執行，像 iPhone 5 與
之前的機種使用 armv7 CPU、是 32 位元環境，iPhone 5S 之後則在 arm64 上。
因此，當我們在使用整數的時候，即使我們也可以直接使用 int 或 long，但我
們會盡量使用 NSInteger 與 NSUInteger，讓 compiler 幫我們決定應該是 32
或 64 位元整數。

另外，在使用浮點數的時候，也盡量使用 CGFloat。CGFloat 一樣會在不同環境
下，被當成 32 或 64 位元浮點數。
