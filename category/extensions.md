Extensions
----------

Objective-C 語言中有一項叫做 extensions 的設計，也可以用來拆分一個很大
的class，語法與 category 非常相似，但是不太一樣。在語法上，extensions
像是一個沒有名字的 category，在 class 名稱之後直接加上空的括弧，而
extensions 定義的 method，需要放在原本的class 實作中。

以下是一個使用 extensions 的例子：

``` objc
@interface MyClass : NSObject
@end

@interface MyClass()
- (void)doSomthing;
@end

@implementation MyClass
- (void)doSomthing
{
}
@end
```

在 `@interface MyClass ()`這段宣告中，我們並沒有在括弧中定義任何名稱，
接著，`doSomthing`又是直接在 `MyClass` 中實作。 extensions 可以有幾個
用途：

### 拆分 Header

如果我們就是打算實作一個很大的 class，但是覺得 header裡頭已經列出了太
多的 method，我們可以將一部分 method 搬到 extensions 的定義裡頭。

另外，extension 除了可以放 method 之外，也可以放成員變數，而 一個
class 可以擁有不只一個 extension，所以如果一個 class 真的有非常非常多
的 method 與成員變數，我們可以把這些 method 與成員變數，放在多個
extension 中。

### 管理 Private Methods

這其實是更常見的用途。我們在寫一個 class 的時候，內部有一些 method 不需
要、我們也不想要放在 public header 中，但是如果不將這些 method 放在
header 裡頭，又會出現一個困擾：在 Xcode 4.3 之前，如果這些 private
method 在程式碼中不放在其他 method 前面，其他的 method 在呼叫這些
method 的時候，compiler 會不斷跳出警告，而這種無關緊要的警告一多，我們往
往會忽視真正重要的警告。[^1]

想要避免這些警告，要不就是把 private method都放在最前面，但這樣並不能完
全解決問題，因為 private method之間也會相互呼叫，花時間確認每個method之
間的呼叫順序並不是很經濟的事；要不就是都用 `performSelector:` 呼叫，這
樣問題更大，就像前面提到，在 method 改名、呼叫 refactoring 工具的時候，
這樣非常危險。

蘋果提供的建議是，我們在 .m 或 .mm 檔案開頭的地方宣告一個 extensions，
將private method 都放在這個地方，如此一來，其他 method 就可以找到
private method 的宣告。從 Xcode 4 開始—至少到 Xcode 7 都是如此，在
Xcode 中所提供的 file template 中，如果你選擇建立一個 UIViewController
的 subclass，就可以看到在 .m 檔案的最前面，幫你預留了一塊 extensions
的宣告。

在這邊也順便提一下 Swift 裡頭的 extension。在 Swift 語言中，我們可以直
接使用 `extension` 關鍵字，建立 class 的 extension，擴充一個 class；
Swift 的 extension 與 Objective-C 的 category 的主要差別是，
Objective-C 的 category 要給定一個名字，而 Objective-C 的 extension 是
沒有名字的 category，至於 Swift 的 extension 則統一都沒有名字。

所以，如果你有一個 Swift class 叫做 MyClass：

```
class MyClass {
}
```

只要這樣就可以直接建立 extension：

```
extension MyClass {
}
```

另外，Swift 除了可以用 extension 擴充 class 之外，甚至可以擴充
protocol 與 struct。我們會在後面講 delegate 的時候說明 protocol。

```
protocol MyProtocol {
}

extension MyProtocol {
}

struct MyStruct {
}

extension MyStruct {
}
```

[^1]: 請見 [Objective-C Feature Availability Index](https://developer.apple.com/library/ios/#releasenotes/ObjectiveC/ObjCAvailabilityIndex/)
