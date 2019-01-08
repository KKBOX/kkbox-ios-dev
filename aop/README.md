管好你的 Tracking Code
======================

[SourceDNA](https://sourcedna.com) 是一家提供 App 安全性報告的公司，這
家公司會定時掃描 AppStore 以及 Google Play 上的 App，檢查各個 App 裡頭
用了哪些 SDK 與 library，而哪些 App 用了有問題、有安全疑慮的 SDK。我們
可以瀏覽這家公司所提供的統計，看看哪些是大家最常使用的 SDK/library，也
可以查看某個特定 App 用了哪些東西。

我們可以看到，其實大家用的東西好像都差不多，最常用的 SDK 大概分成幾類，
一種是社交類，像為了可以使用 Facebook 帳號登入，所以大家都用了Facebook
SDK；第二則是廣告投放類，像 KKBOX 這種一開始就以收費為主要商業的模式的
App，想要導入廣告商業模式，就自己投資一個廣告部門…在市場上其實是少數，
許多 App 都是倚靠 Google AdMob、台灣的 Vpon 等廣告平台爭取營收，自然也
導入了廣告平台的 SDK。再來是用來蒐集 crash report 的 SDK，我們在前面的
[如何收集 Crash Reports](../crash_reports/collection.md) 討論了不少。

另外一種則是數據分析（Analytics）的 SDK，包括 Google Analytics、Flurry、
Mixpanel 等等，而 Facebook SDK 等並非專屬數據分析用，但也具備這部份功
能。這種 SDK 的主要用途是得到 App 中的各種用量數字，像是在 App 中，每
個畫面被看了多少次（App Views），哪些事情用戶做了多少次，像某個按鈕被
按了幾次（Events），用來判斷我們的用戶喜歡以及不喜歡什麼，進一步決定後
續的產品開發方向。

我們也常用這些數字計算成其他數字，像是回訪率：昨天使用我們 App 的用戶，
有多少今天還在繼續使用？然後我們可以知道我們有多少每日活躍用戶（Daily
Active Users，DAU）以及每月活躍用戶（Monthly Active Users，MAU）。或是
計算轉換率—廣告投放的次數如何轉換成安裝與使用的用戶數，以及 App 的功能，
如何把用戶轉換成付費用戶的數字。

各種 SDK 中，就屬數據分析類用起來最麻煩。我們要做 Facebook 帳號登入，
只要改動登入頁面就好了，廣告投放的 SDK 只需要放在打算放置廣告的 view
controller 上，收集 crash report 類型的 SDK 呢，只需要在 App 開啟的時
候（App Delegate 的 `-applicationDidFinishLaunching:withOptions:`），
設好 HockeyApp 或 Crashilytics 的 ID，就會攔截發生 exception 的 signal
handler，取得 crash report 回報。但使用數據分析的 SDK，要改動哪裡呢？

—到處都得改。

在使用數據分析的 SDK 的時候，只要是想要知道數據的地方，就要加一段
tracking code，所以，如果我們想要知道 App 中所有畫面的使用量，就得改動
所有的 view controller，有一百個畫面，就可能要有一百段 tracking code。

Tracking code 往往寫了一次之後又經常改動，而改動的範圍又往往不是簡單的
增加或刪除，而是整批改動。像是，我們可能一開始用 Flurry，用了一陣子之
後覺得不好用，想要換成 Google Analytics，之後又覺得不好用，打算改用
MixPanel 統計數據。結果就是，你一開始寫了一百段 tracking code 之後，又
先把這一百個散亂在專案中一堆地方的 tracking code 拔掉，再重新在專案中
一百個不同的地方，寫上一百段 tracking code。

當我們遇到重複的程式碼的時候，第一個想到的，大概就是使用繼承。比方說，
當我們的所有的 view controller 在 `viewWillAppear:` 的地方，都想要加上
一段 tracking code，我們大概就會想要讓所有的 view controller 都繼承自
某個最上層的 view controller，在最上層這邊實作一次數據統計的 tracking
code。

但這樣做實際上很麻煩，假使我們希望我們在這個專案的 view controller 可
以在其他專案中重複使用，就會發現，這樣我們還得把實作 tracking code 的
這個最上層的 view controller class 一起搬到另外一個 class 裡頭；另一方
面，我們可能在專案中使用來自第三方的各種套件，像是 Cocoapods 提供的元
件，或是來自某個 git submodule，也沒什麼空間讓我們改動這些 class。

所以我們就想問：

- 有沒有可能，我們可以把這些 tracking code 拆出來，不要寫在原本的
  class 裡頭？在不改動原本的 code 的狀況下，就可以產生數據分析所需要的
  log？
- 有沒有可能，我們可以把這些 tracking code 集中在一起，集中在同一個檔
  案中。當我們不想使用原本所提供的數據分析服務的時候，只要把這個檔案砍
  掉就可以了？

當我們遇到這樣的問題，眼光自然就會投向 Aspect-oriented programming，
在台灣通常翻譯成切面導向或剖面導向的程式設計，對岸通常翻譯成面向側面的
編程，不過大家會更常直接稱呼簡稱—AOP。
