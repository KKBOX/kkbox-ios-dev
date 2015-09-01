Category 是否可以增加新的成員變數或屬性？
-----------------------------------------

因為 Objective-C 物件會被編譯成 C 的 structure，我們雖然可以在
category中增加新的 method，但是我們卻不能夠增加新的成員變數。

在 Mac OS X 10.6 與 iOS 4 之後，蘋果提出一套叫做 Associated Objects的
辦法，讓我們可以在 category 中增加新的getter/setter，觀念差不多是：既
然我們可以用一張表格記錄一個 class 有哪些method，那，我們不就也可以另
外建一張表格，記錄有哪些物件與這個 class相關？

要使用 Associated Objects，我們需要匯入 `objc/runtime.h`，然後呼叫
`objc_setAssociatedObject` 建立 setter，用 `getAssociatedObject` 建立
getter，呼叫時要傳入：我們要讓哪個物件與哪個物件之間建立關連，關連時使
用的是那一個key（型別為 C 字串）。在以下的範例中，我們在 `MyCategory`
這個 category裡，增加一個叫做 myVar 的 property。

``` objc
#import <objc/runtime.h>

@interface MyClass(MyCategory)
@property (retain, nonatomic) NSString *myVar;
@end

@implementation MyClass
- (void)setMyVar:(NSString *)inMyVar
{
    objc_setAssociatedObject(self, "myVar",
           inMyVar, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (NSString *)myVar
{
    return objc_getAssociatedObject(self, "myVar");
}
@end
```

在 `setMyVar:` 中呼叫 `objc_setAssociatedObject` 時，所最後一個參數
`OBJC_ASSOCIATION_RETAIN_NONATOMIC`，是用來決定要用哪一種記憶體管理策
略，管理我們傳入的參數，在我們的例子中，我們傳入的是`NSString`，是一個
Objective-C 物件，所以我們必須要 retain起來。這邊可以傳入的參數還可以
是`OBJC_ASSOCIATION_ASSIGN`、`OBJC_ASSOCIATION_COPY_NONATOMIC`、
`OBJC_ASSOCIATION_RETAIN`以及 `OBJC_ASSOCIATION_COPY`，與 property語法
使用的記憶體管理方式一致。而當我們的 `MyClass` 物件在 dealloc的時候，
所有透過 `objc_setAssociatedObject` 而 retain起來的物件，也都會被一併
釋放。

雖然我們不能在 category 中增加成員變數，但是卻可以在 extensions中宣告。
在 Xcode 4.2 之後，我們可以這麼寫：

``` objc
@interface MyClass()
{
    NSString *myVar;
}
@end
```

我們甚至可以將成員變數直接放在 `@implementation` 的程式區塊中：

``` objc
@implementation MyClass
{
    NSString *myVar;
}
@end
```
