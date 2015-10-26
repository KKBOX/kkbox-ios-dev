NULL、nil、Nil…
---------------

在 Objective-C 語言中，有很多個代表「沒有東西」的東西，一開始也很容易
搞混。包括：

- NULL
- nil
- Nil
- NSNull
- NSNotFound

### NULL

NULL 其實並不算是 Objective-C 的東西，而是屬於 C 語言。NULL 就是 C 語
言當中的空指標，是指向 0 的指標。絕大多數狀況下，nil、Nil 與 NULL 可以
代替使用，但是語意上，當某個 API 想要你傳入某個指標（void *）時，而不
是 id 型別時，雖然你可以在這種狀況下傳入 Objective-C 物件指標，也就是
可以傳入 nil，但是傳入 NULL 意義會比較清楚。

像建立 NSTimer 時，API 在 userInfo 這邊要求的是 id，我們傳入 nil 會比
較好：

``` objc
+ (NSTimer *)timerWithTimeInterval:(NSTimeInterval)seconds
	target:(id)target
	selector:(SEL)aSelector
	userInfo:(id)userInfo
	repeats:(BOOL)repeats
```

而像 UIView 的 `beginAnimations:context` 的定義是：

``` objc
+ (void)beginAnimations:(NSString *)animationID context:(void *)context;
```

在這邊，傳入 NULL 就會比傳入 nil 好。

### nil

nil 是空的 Objective-C 物件指標，也一樣是指向 0。如果我們建立了一個
Objective-C 物件的變數，當我們不想要使用這個物件的時候，便可以將這個變
數指向 nil；我們可以對 nil 呼叫任何的 Objective-C method，都不會產生問
題。

我們需要注意在 Array 與 Dictionary 中使用 nil 的狀況。在使用 NSArray的
`-arrayWithObjects:` 或 NSDictionary 的
`-dictionaryWithObjectsAndKeys:` 這些被標為
`NS_REQUIRES_NIL_TERMINATION` 的 method 的時候，nil 會被當成是最後一個
參數，出現在 nil 之後的參數都會被忽略，而且我們在傳入參數的時候，最後
一個參數也一定要是 nil。

比方說，我們寫一段這樣的程式：

``` objc
NSArray *a = [NSArray arrayWithObjects:@1, @2, nil, @3];
NSLog(@"a:%@", a);
```

這個 array 就只會有 @1 與 @2，@3 就會被截掉。主要原因是，這類 method
大概都是這樣實作的：首先使用一個 va_list 讀取所有傳入的參數，然後用迴
圈呼叫 `va_arg`，只要遇到 nil 就停止迴圈，像下面這段程式：

``` objc
void test(id arg, ...){
	va_list list;
	va_start(list, arg);
	do {
		if (arg == nil) {
			break;
		}
		NSLog(@"arg:%@", arg);
	} while ((arg = va_arg(list, id)));
	va_end(list);
}
```

另外，當我們對 NSMutableArray 插入 nil，或使用 Xcode 4.4 之後的
literal 來寫 NSArray 或 NSDictionary的時候，如果傳入 nil，也會發生
exception 而造成 crash。下面這段 code一定會 crash。

``` objc
NSMutableArray *a = [[NSMutableArray alloc] init];
[a addObject:(id)nil];
```

``` objc
NSArray *a = @[(id)nil];
```

### Nil

nil 是空的 instance，而開頭大寫的 Nil 則是指空的 class。比方說，當我們
想要判斷某個 Class 是不是空的，語意上應該用 Nil 而不是 nil。

我們其實不常判斷一個 Class 是不是 Nil。比較有可能的場合，是為了處理向
下相容，像某個 Class 只在某一版的新 OS 上存在，但我們還需要支援舊的 OS，
所以我們會在確定某個 Class 不是 Nil 的狀況下，才執行某段程式碼：

``` objc
Class cls = NSClassFromString(@"Abcdefg");
if (cls != Nil) {
	// Do something.
}
```

但如果我們去看 <objc/objc.h>，nil 與 Nil 其實是一樣的。

``` objc
#ifndef Nil
# if __has_feature(cxx_nullptr)
#   define Nil nullptr
# else
#   define Nil __DARWIN_NULL
# endif
#endif

#ifndef nil
# if __has_feature(cxx_nullptr)
#   define nil nullptr
# else
#   define nil __DARWIN_NULL
# endif
#endif
```

### NSNull


### NSNotFound
