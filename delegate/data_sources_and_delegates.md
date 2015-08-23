Data Source 與 Delegate 的差別？
--------------------------------

我們現在可以來看看 `UITableView` 與 `UITableViewController`是怎麼運作
的。 `UITableViewController` 在 `loadView` 中建立了一個`UITableView`
的 instance，指定成是自己的 view，同時將這個 view 的delegate 與 data
source 設定成自己。一個 class 可以根據需要，將 delegate拆成好幾個，以
`UITableView` 來說，跟表格中有什麼資料有關的，就放在 data source 中，
其餘的 method 放在 delegate 中。

我們在 Mac OS X 會用到的最龐大的 UI 元件，莫過於 `WebView`，雖然在 iOS
上 `UIWebView` 被閹割到只剩下四個 delegate method，但是 Mac OS X上足足
有五大類 delegate method，網頁頁框的載入進度、個別圖片檔案的載入進度、
下載檔案的 UI呈現、該不該開新視窗或是新分頁、沒有安裝 Java 或是 Flash
要怎麼呈現、用JavaScript 跳出 alert 該怎麼呈現…都是一堆 delegate
method。

假如先不管 `UITableView` 怎麼重複使用 `UITableViewCell` 的機制（這個機
制還頂複雜），我們要更新 `UITableView` 的資料時，先指定data source 物
件後，要呼叫一次 `reloadData` 。 `reloadData`可能是這樣寫的：

``` objc
- (void)reloadData
{
    NSInteger sections = 1;
    if ([dataSource respondsToSelector:
        @selector(numberOfSectionsInTableView:)]) {
        sections = [dataSource numberOfSectionsInTableView:self];
    }
    for (NSInteger section = 0; section < sections; section++) {
        NSInteger rows = [dataSource tableView:self
                         numberOfRowsInSection:section];
        for (NSInteger row = 0; row < rows; row++) {
            NSIndexPath *indexPath = [NSIndexPath indexPathForRow:row
                                                        inSection:section];
            UITableViewCell *cell = [dataSource tableView:self
                                    cellForRowAtIndexPath:indexPath];
            // Do something with the cell...
        }
    }
}
```

我們注意到幾件事情：首先，因為 `numberOfSectionsInTableView:` 被定義成
是optional 的 delegate method，delegate 不見得要實作，所以我們會用
`respondsToSelector:` 檢查是否有實作。我們可以在 protocol的宣告中，指
定某個 delegate method 是 required 或是optional，如果不特別指定的話，
預設都是 required。我們簡單看一下`UITableViewDataSource` 就知道如何定
義 required 與 optional 的 delegate method。

``` objc
@protocol UITableViewDataSource<NSObject>
@required
- (NSInteger)tableView:(UITableView *)tableView
  numberOfRowsInSection:(NSInteger)section;
- (UITableViewCell *)tableView:(UITableView *)tableView
  cellForRowAtIndexPath:(NSIndexPath *)indexPath;
@optional
- (NSInteger)numberOfSectionsInTableView:(UITableView *)tableView;
// ....
@end
```

另外，就是定義在 data source 的 method，是在 `reloadData` 中被呼叫，因
此我們可以知道 `UITableView` 的 data source 與 delegate的最大差別：我
們絕對不可以在 data source 定義的 method 中呼叫`reloadData`，不然就會
進入無窮迴圈！
