Callback Hell
-------------

Block 通常用在處理 callback，而我們往往會在 callback 裡頭又做另外一件
事情，而這件事情又可能又有另外一個 callback，而這個 callback 裡頭要做
的事情又有另外一個 callback…於是，我們可能會寫出這種深度非常深的程式碼：

``` objc
[someObject doSomethingWithCallback:^{
	[someObject doSomethingWithCallback:^{
		[someObject doSomethingWithCallback:^{
			[someObject doSomethingWithCallback:^{
				[someObject doSomethingWithCallback:^{
					[someObject doSomethingWithCallback:^{
					}];
				}];
			}];
		}];
	}];
}];
```

這種狀況我們稱之為 Callback Hell—無限延長的 Callback 地獄，這種現象除
了會出現在 Objective-C 的 block 之外，也出現在各式各樣的程式語言中，尤
其是在 JavaScript 開發中，特別常討論 Callback Hell。為了要解決Callback
Hell，從 ECMAScript 6 開始，就改成使用 Promise 的寫法處理Callback，而
像是 Parse 的
[Bolts Framework](https://github.com/BoltsFramework/Bolts-iOS)，便是將
Promise 從JavaScript port 到 Objective-C 中。

使用了 Botls Framework 之後，我們可以將各種要非同步執行的工作，包裝成
BFTask 物件，我們便可以將上面那段 code，改寫成這個樣子：

``` objc
[[[[someObject doSomething] continueWithBlock:^id(BFTask *task) {
	return [someObject doSomething];
}] continueWithBlock:^id(BFTask *task) {
	return [someObject doSomething];
}] continueWithBlock:^id(BFTask *task) {
	return nil;
}];
```

雖然一樣是 callback 之間不斷串連，但是在程式碼中，我們把 callback 之間
從不斷加深的關係，變成了不斷往下的關係，也因此變得比較好讀。我們在這邊
不討論如何包裝 BFTask 物件，詳情請看 Bolts Framework 本身的文件。
