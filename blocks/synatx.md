Block 的語法
------------

一直以來還是有不少人不滿 block 語法，甚至有人成立了一個叫做
[fuckingblocksyntax.com](http://fuckingblocksyntax.com) 的網站。這個網
站的網域名稱不怎麼優雅，不過裡頭倒是清楚整理了我們應該如何宣告 block。

將 block 宣告成變數（local variable）的語法是：

``` objc
returnType (^blockName)(parameterTypes) = ^returnType(parameters) {...};
```

宣告成 Objective-C property 的語法是：

``` objc
@property (nonatomic, copy) returnType (^blockName)(parameterTypes);
```

宣告成 method 的參數（method parameter）的語法是：

``` objc
- (void)someMethodThatTakesABlock:(returnType (^)(parameterTypes))blockName;
```

在執行某個需要傳入 block 當做參數的 method 的時候，則是用這以下方式呼
叫。這也是絕大多數用 block 當做 callback 的處理方式：

``` objc
[someObject someMethodThatTakesABlock:^returnType (parameters) {...}];
```

把一種 block 宣告成 typedef：

``` objc
typedef returnType (^TypeName)(parameterTypes);
TypeName blockName = ^returnType(parameters) {...};
```

Block 也可以當成 C function 的參數或是回傳結果的型別，但是，在這種狀況
下，我們不能夠直接使用 returnType (^)(parameterTypes) 這種語法，必須要
先宣告成 typedef 才行。也就是說，這樣會被當成不合法：

``` objc
(void (^)(void)) test((void (^)(void)) block) {
	return block;
}
```

但可以寫成這樣：

``` objc
typedef void (^TestBlock)(void);
TestBlock test(TestBlock block) {
	return block;
}
```

雖然 C function 的參數不能夠使用 returnType (^)(parameterTypes) 語法，
但是一個 block 倒是可以使用這種語法撰寫輸入與回傳值的型別，但其實在這
種狀況下，還是比較建議使用 typedef 宣告。比方說，我們現在要宣告一個
block，這個 block 會回傳另外一個型別為 int(^)(void) 的 block，就會寫
成這樣：

``` objc
int (^(^counter_maker)(void))(void) = ^ {
	__block int x = 0;
	return ^ {
		return ++x;
	};
};
```

可讀性實在非常差。不如寫成這樣：

``` objc
typedef int (^CounterMakerBlock)(void);
CounterMakerBlock (^counter_maker)(void) = ^ {
	__block int x = 0;
	return ^ {
		return ++x;
	};
};
```
