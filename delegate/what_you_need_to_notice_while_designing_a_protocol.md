注意事項
--------

在上面的範例中，我們看到了設計 delegate 與 protocol 應該注意的地方：

### Delegate 物件不應該指定 Class

我們將 delegate 物件宣告成 `id <MyButtonDelegate> delegate`，意思就是
不需要管這個物件屬於哪個 class，只要是個 Objective-C物件即可，但是這個
物件必須實作 `MyButtonDelegate` protocol。

我們其實可以將 delegate 物件是那個 class 寫死，例如把 `MyButton` 的
delegate 的 class 指定成`MyController`，但這樣做非常不好，如此一來，就
只有 `MyController`可以使用 `MyButton`，其他 controller 都無法使用，就
大大減少了重複使用`MyButton` 的彈性。

Delegate 這種設計方式，也方便我們在同時開發 Mac OS X 與 iOS跨平台專案
時共用程式碼，我們在撰寫某個 model 物件的時候，只使用Foundation 或是其
他兩個平台都有的framework，至於與平台相依的部份，就放進 delegate 中，
然後在 Mac OS X 與iOS 上各自實作 delegate 物件。

總之， *在實作 delegate 的時候，delegate 屬於哪個 class並不重要，重要
的是 delegate 物件有沒有實作我們想要呼叫的 method。*

### Delegate 屬性應該要用 Weak，而非 Strong

在使用 property 語法的時候，如果這個 property 是 Objective-C物件，我們
照理說應該要設定成 strong 或 retain，但是遇到的是 delegate，我們應該設
成 weak 或 assign。

原因是：需要設計 delegate 物件的這個物件，往往是其 delegate物件的成員變
數。在我們的例子中，`MyButton` 的 instance 是 `myButton`，是
`MyController` 的成員變數，自己可能已經被 `MyController` retain了一份。
如果 `MyButton` 又 retain了一次 `MyController`，就會出現循環 retain 的
問題—我已經被別人 retain，我又把別人 retain 一次。

如此，會造成我們會無法釋放 `MyController`：在該釋放 `MyController`的時
候，`MyController` 還是被自己的成員變數 retain，`MyController`得要走到
`dealloc` 才會釋放 `myButton`，但是自己卻因為被 `myButton` 給retain 起
來，而始終走不到 `dealloc`。

#### Delegate Method 的命名方式

Delegate method 的命名有個鮮明的特色，就是這個 method至少會傳入一個參
數，就是把到底是誰呼叫了這個 delegate method傳遞進來。同時，這個
method 也往往以傳入的 class名稱開頭，讓我們可以辨識這是屬於哪個 class
的 delegate method。以`UITableViewDelegate` 為例，假如我們在 iOS的表格
中，選擇了某一列，就會呼叫

``` objc
- (void)tableView:(UITableView *)tableView
        didSelectRowAtIndexPath:(NSIndexPath *)indexPath
```

Method 的名稱就以「tableView」開頭，讓我們知道這是屬於 Table View 的
delegate method，然後第一個參數把這個 Table View 的 iinstance傳入，接
下來才傳入到底是哪一列被選起來的資訊。

至少把是誰呼叫了這個 delegate method 傳入的理由很簡單。以我們的
`MyController` 為例，這個 controller 可能有好幾個 `MyButton`，而這些
`MyButton` 全都把 delegate 指到同一個 controller 上，那麼，controller
就需要知道到底是被哪個 button 呼叫。判斷方式只要簡單比對指標就好了：

``` objc
- (void)myButtonDidBecomeQuadrupleClicked:(MyButton *)button
{
    if (button == myButton1) {
    }
    else if (button == myButton2) {
    }
}
```
