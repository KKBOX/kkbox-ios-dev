無所不在的 Delegate
-------------------

由於在 Objective-C 語言中，delegate 相當於 event handler的用途，所以，
當你在其他平台中看到 event handler 用得多頻繁，就等於delegate 用得多頻
繁。舉例來說：

-   在使用 `NSURLConnection` 抓取網路上的資料的時候，無論收到了 HTTP
    response code、是否連線失敗、是否連線結束…都是透過 delegate 回傳。
-   在使用 Core Location 的時候，如果 `CLLocationManager`
    找到了我們的所在位置，或是發現我們正在移動，也都會透過 delegate
    通知。
-   當我們要使用手機拍照、傳送簡訊或是電子郵件等等，當照片拍完，會用
    delegate 回傳 image 物件，簡訊或是電子郵件傳送成功，也會用 delegate
    告訴我們執行完畢。

甚至，當我們在寫一個 iOS 程式的第一步，其實都是在實作一個 delegate
method。我們在 Xcode 裡頭開了一個新專案之後，下一步往往是實作
`application:didFinishLaunchingWithOptions:` 這個method，但是要了解整
個程式的進入點，我們要從 `main.m`來看。裡頭通常只有簡短的幾行：

``` objc
int main(int argc, char *argv[])
{
    @autoreleasepool {
        return UIApplicationMain(argc, argv, nil, nil));
    }
}
```

一個 iOS 程式是從 `main` 這個 function 開始，接著透過呼叫
`UIApplicationMain` 建立 `UIApplication` 這個 Singleton 物件。
`UIApplication` 用來代表一個應用程式的基本狀態，包括 icon上面該顯示多
少則 push notification的數量、支援水平還是垂直畫面、是否顯示狀態列等，
當 `UIApplication`物件被建立起來後，就要通知它的delegate—程式已經開啟
了，請進行下一步，這個 delegate method 就是
`application:didFinishLaunchingWithOptions:`，我們在這邊建立基本的
view controller 與 window，顯示出來。

也就是說，當我們在開始寫第一行 iOS 程式的時候，我們就起碼需要了解什麼
是Singleton 和 delegate，但是在了解之後，想要知道 Mac OS X 與 iOS中眾
多的元件該如何使用，以及怎樣用比較好的方式設計自己的元件，就不是問題了
