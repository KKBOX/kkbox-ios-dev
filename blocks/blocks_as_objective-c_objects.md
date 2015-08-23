Block 作為 Objective-C 物件
---------------------------

前面提到，Block 其實可以當成是一種物件，我們接下來就要來看 block 的作
為物件的特性。

### 記憶體管理

由於 Objective-C 物件需要做記憶體管理，因此，如果你還在手動管理記憶體，
在建立 block 的時候，也必須手動管理 Block 的記憶體。我們可以使用
`Block_Copy()` 和 `Block_Release()` 這兩個 C Function 對 block 做 copy
或 release；或，我們也可以把 block 給 cast 成 id 型別，便可以對 block
呼叫 copy 與 release。不過在啟用 ARC 之後，我們便不需要，Compiler 也會
阻止我們手動管理記憶體。

### Block 的型別

Objective-C 當中每個物件都具有 class，而每個 class 都繼承自 NSObject，
由於 block 具有物件的性質，因此 block 本身也有 class—不過，一個 block
是屬於哪一種 class 平時對我們來說並不會有太大的意義，畢竟我們在建立
block 的時候，並不會指定要建立哪一種 class 的 block，我們也不會去
subclass 某種 block 的 class。基本上，當我們寫好一個 block 之後，這個
block 最後會變成哪個 class，全部都是由 compiler 決定。

在 C 語言當中，記憶體分成三塊：global、stack 與 heap，compiler 在編譯
程式碼的時候，會根據我們所寫出來的 block 到底使用到哪一塊記憶體，將這
個 block 變成不同的 class，包括 `__NSGlobalBlock__`、
`__NSStackBlock__` 與 `__NSMallocBlock__`。知道這件事情通常對我們不會
有什麼幫助，但是可以讓我們了解蘋果的 compiler 曾經發生過的 bug：在某個
狀況下，有個block 應該要使用 global 的記憶體，但是 compiler 卻誤判成只
使用 stack 的記憶體。

如果沒有開啟 ARC，以下這段程式碼會在執行到 `block()` 這一行的時候，發
生 Bad Access 錯誤：

``` objc
- (NSArray *)blocks
{
	int i = 1;
	return @[^{return i;}];
}

- (void)callBlock
{
	int (^block)(void) = [self blocks][0];
	block();
}
```

原因是：在 `-blocks` 所回傳的 NSArray 中所包含的 block 物件中，使用到
了i 這個只出現在 blocks 這個 method 內部的 int 變數，因為這個變數只在
這個 method 中使用，compiler 便認為 i 應該使用 stack 的記憶體，因此也
把回傳的 block 建立成 `__NSMallocBlock__`；於是，當我們在
`-callBlock`裡頭呼叫 `block()` 的時候，原本的記憶體已經被釋放，於是產
生記憶體管理錯誤。
