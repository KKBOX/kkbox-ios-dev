實作 Category
-------------

Category 的語法很簡單，一樣是用 @interface 關鍵字宣告 header，在
@implementation 與 @end 關鍵字當中的範圍是實作，然後在原本的 class名稱
後面，用中括弧表示新增的 category 名稱。

舉例來說，我們今天雖然寫的是 Objective-C 語言，但是想要變得更像 Small
Talk 一點，所以我們不想用 `NSLog`印出某個物件的資料，而是每個物件都有
個把自己印出來的 method，所以我們對NSObject 建立了一個叫做
SmallTalkish 的 category。

``` objc
@interface NSObject (SmallTalish)
- (void)printNl;
@end

@implementation NSObject (SmallTalish)
- (void)printNl
{
    NSLog(@"%@", self);
}
@end
```

如此一來，每個物件都增加了 `printNl` 這個 method。可以這麼呼叫：

``` objc
[myObject printNl];
```

前一章提到，我們在排序一個裡頭都是字串的 Array 的時候，可以呼叫
`localizedCompare:`，但，假如我們希望所有的字串都一定要用中文筆劃順序
排序，我們可以寫一個自己的method，例如 `strokeCompare:`。

``` objc
@interface NSString (CustomCompare)
- (NSComparisonResult)strokeCompare:(NSString *)anotherString;
@end

@implementation NSString (CustomCompare)
- (NSComparisonResult)strokeCompare:(NSString *)anotherString
{
    NSLocale *strokeSortingLocale = [[[NSLocale alloc]
              initWithLocaleIdentifier:@"zh@collation=stroke"]
              autorelease];
    return [self compare:anotherString
                 options:0
                   range:NSMakeRange(0, [self length])
                  locale:strokeSortingLocale];
}
@end
```

在存檔的時候，檔名的慣例是原本的 class 名稱加上 category的名稱，中間用
加號連接，以我們剛剛建立的 CustomCompare為例，存檔時就要存成
NSString+CustomCompare.h 及 NSString+CustomCompare.m。
