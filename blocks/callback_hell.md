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

// 講 promise 與 bolts
