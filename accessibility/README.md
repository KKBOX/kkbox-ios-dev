Accessibility
=============

蘋果在 iPhoneOS 3.0 中就加入了三項為視力不佳者服務的設計（不過，因為硬體效能的限制，這三項功能只能夠在 iPhone 3Gs 、最新的 iPod Touch，以及 iPad 使用）－如果你沒有辦法清楚看到 iPhone 上預設大小的文字，你可以開啟畫面的局部放大功能，同時用三隻手指點按畫面，就可以放大手指所在的區域；如果覺得白底黑字不夠清楚，可以將畫面反白，切換成黑底白字。

而如果已經完全失去視力，完全無法透過視力操作電腦，iPhone OS 提供一項名為 Voiceover 的語音功能，開啟後會改變一些觸控的操作行為－原本只要按在畫面中某個元件上（例如按鈕等），就會使用這一個元件的功能，在開啟 Voiceover 之後，單點一下，系統會先告訴你目前點到的地方是怎樣的東西，之後再連續點兩下，才是使用這個元件的功能。

Voiceover 可以念出多種語文，除了我們關心的中文之外，英德法義日自然也不會少；相對的，Mac OS X 上面目前還是只有英文而已，希望蘋果之後也會把這些語言的語音支援，加入 Mac OS X 10.7 當中。


※

在 iPad 推出之後，Voiceover 說起來可以發揮的地方更多。比較 iPhone 與 iPad，iPad 會是一台更適合用於閱讀的裝置，搭配 Voiceover，當你用 iBook 翻閱的書籍時，可以幫你把書中的內容念出來（當然，你也可以做一些無聊的事情－例如拿日文語音模組念英文，就會念出日本腔的英文，或是拿日文語音念中文，結果全都當成日文漢字念）。

說到這裡就讓人難免納悶－到目前為止，已經有許許多多媒體機構推出各種 iPad 上的閱讀軟體，或是報紙或是雜誌，從紐約時報到 USA Today，但是沒有幾套把 voiceover 支援做進去。

我不太懂這些媒體在設計軟體的時候到底是怎麼想的，似乎心中念茲在茲的，就是怎樣在螢幕上呈現出紙張的質感，讓你在 iPad 閱讀時盡可能體驗與紙本相同的感受，而這種追求簡直是種偏執；我實在不懂，為什麼一定要在新平台上做原本平台在做的事情，而不是發揮新平台的優勢，善用在原本平台上做不到的功能？

在數位媒材上，你可以在閱讀內容時加入字典查詢功能，就算蘋果還沒有開放內建的字典 library，但是總可以自己做一個 popover 裡頭是字典內容，就算 popover 裡頭是 webview 開網頁字典也行，紐約時報網站上有即時字典功能，在 iPad 軟體裡頭反倒沒有。另外，透過 TTS，你就可以讓沒時間看報紙的人有空檔的時候聽報紙，或是讓原本沒有辦法閱讀報紙的盲人聽到內容；如果平面媒體的危機是愈來愈多人不想看紙本，那為什麼對於透過 iPad 閱讀的讀者而言，在螢幕上模擬紙本質感會很重要？

因為 Voiceover，iPad 可能是目前最適合盲人使用的電腦－在使用鍵盤滑鼠的操作介面中，不管怎樣，盲人都很難確定目前螢幕上滑鼠指標到底在什麼地方，但是在觸控介面上，便能夠較為清楚的選擇要點選的是畫面的上下左右；而透過 Voiceover，也便能夠讓盲人無需另外安裝軟體，就可以方便地收取郵件、閱讀網頁… 等。

儘管如此，還是有個問題－就算蘋果內建的應用程式都有完整的無障礙支援，但是 AppStore 上眾多的第三方應用程式，目前在這方面的努力，還是不太夠。你隨便問個 iPhone developer，大概沒幾個想過這件事。

而你說蘋果的東西是用行銷打造出來的玩意，但是放眼望去，在軟體方面有些事情呢，除了蘋果之外，你又看不到有誰特別關心。

※

在製作軟體時，我們不太需要關心局部放大與反白兩項功能，不過，在 Voiceover 方面，還是要做一些工作，才能夠完全達成。

如果你的應用程式裡頭，都只有用到 iPhone SDK 原本就設計好的那些 UI 元件，例如 UIButton、UILabel、UISlider … 等等，基本上就已經具備了支援 Voiceover 的一定能力，在點選到這些元件上的時候，系統就會把裡頭的文字念出來。

但，假使你不是用文字代表這些元件的意義，而是放入圖片呢？如果圖片來自 bundle 的 resource 裡頭，你用 [UIImage imageNamed:] 載入圖片，那，Voiceover 會把圖片檔名念出來，恐怕沒有人能聽的懂是什麼意思；而如果圖片是用程式碼產生，就什麼相關資訊都沒有了。

對這些繼承自 UIView 的物件來說，有兩種方法可以處理這種狀況。如果你的 UI 是在 Interface Builder （以下簡稱 IB）裡頭產生的，在 IB 的 Inspector 的第四個分頁，就可以設定這個 UI 元件的相關資訊。也可以透過程式碼設定，iPhone SDK 定義了 UIAccessibility 這個 informal protocol，所有繼承自 NSObject 的物件都具備此一 interface，你可以透過 accessibilityLabel、accessibilityHint 等屬性，標記 UI 元件的標題與詳細說明。

開發過程中，可以使用 iPhone Simulator 檢驗目前程式對 Voiceover 的支援狀態，方法是在模擬器中的偏好設定中，把 Accessibility Inspector 打開，模擬器畫面中就會出現一個小畫面，在選取畫面中某個 UI 元件時，Accessibility Inspector 就會列出各種相關資訊。

※

在 iOS 4.0 上，無障礙支援又有另外一項意義，就是這個功能與新增的 UI 自動測試（UIAutomation）相關。在 iOS 4.0 上，當你用 Instument 執行某個 iPhone 應用程式的時候，可以選擇載入一個你之前撰寫好的 Javascript 腳本，接著就會按照腳本的內容，逐一進行各種 UI 測試項目；腳本的內容大概是，你可以透過 Javascript ，將目前應用程式當前畫面當做物件呼叫，要求回傳畫面上所有可以使用的 UI 物件列表，然後，要求應用程式自己去點選這些物件。

每個 UI 元件的無障礙資源資訊，在 UI 自動測試中，就會被當做是這個 UI 物件的 id 使用。比方說，在畫面中有一個叫做「Edit」的按鈕，你從 Javascript 取得的物件列表放在 buttons 這個變數中（型態是 array），那麼，要取得這個按鈕，可以這麼呼叫： buttons[‘Edit’]。如果要點選這個按鈕，就是 buttons[‘Edit’].tap()。

※

蘋果官方文件 Accessibility Programming Guide for iPhone OS 中，寫得算是詳細，接下來說點文件裡頭沒提到的事。

1.

如果你要加入無障礙資源的物件是一個 UIView，基本上不需要管這個物件的 frame，也就是，使用者在螢幕上點到什麼範圍，這個物件才會處理與無障礙支援相關的工作。

不過，很多時候，在使用者介面中的重要文字與圖片，並不是放在直接放在某個 UIView 裡頭，當你使用 Core Animation 製作 UI 的時候，就會把很多東西放在 CALayer 裡－某個 layer 裡頭的 content 屬性可能是一張重要的圖，或是用了 CATextLayer 呈現文字。

CALayer 物件本身沒辦法處理無障礙資源。想要讓系統知道某個 layer 的存在與其所代表的意義，就必須要用一個 container 物件處理，這個 container 通常是這個 layer 的 super layer 所在的 view，實作 UIAccessibilityContainer protocol。需要注意，自己實作了 UIAccessibilityContainer protocol 之後，會完全改變這個 view 裡頭的無障礙支援的行為，所以，如果原本這個 view 的 subviews 中還有許多其他的 UIView 物件，如果我們自己實作的 UIAccessibilityContainer 並沒有處理到這些 subview，這些 subview 原本具有的無障礙支援也就會隨之失效。

我們可以針對每一個 CALayer 物件，產生一個對應的 UIAccessibilityElement 物件，然後透過 setAccessibilityFrame: 指定對應的 CALayer 在畫面上的範圍。

需要注意的是，這邊所指定的位置，必須是這個 layer 在整個應用程式畫面中的位置，而不是 layer 原本的 frame 屬性，layer 本身的 frame 屬性只是相對於上一層的 layer （super layer）的位置而已。我們在這裡可以透過呼叫 convertRect:fromLayer:，轉換出目前 layer，與應用程式啟動時，我們所產生的那個 UIWindow 物件上的 layer，兩者之間的位置關係，這個 CGRect 才是我們想要的資訊。

當 layer 在畫面上的位置有改變時，最後記得要發一次 UIAccessibilityLayoutChangedNotification。

2.

在 iPhone Simulator 上面，如果是模擬 iPhone，可以順利叫出 Accessibility Inspector，但目前如果是要模擬 iPad，Accessibility Inspector 在出現之後就一閃即逝，所以在 iPad 上無障礙支援，還是要在實機上才能測試。

在模擬器上模擬 iPad，在處理無障礙支援時，還有別的問題。在模擬器上面如果呼叫 UIAccessibilityElement 的 initWithAccessibilityContainer:，所產生的物件會是 nil，而如果你按照蘋果官方文件的範例：

?
1
2
3
_accessibleElements = [[NSMutableArray alloc] init];
UIAccessibilityElement *element1 = [[[UIAccessibilityElement alloc] initWithAccessibilityContainer:self] autorelease];
[_accessibleElements addObject:element1];
直接把 nil 加到 NSMutableArray 會產生 exception，所以，如果你加了無障礙支援，又想要在模擬器上面測試，程式就不能這樣寫。

此外，之前提到 iOS 4.0 的 UI 自動測試，在模擬器上面也無法使用。在做任何一項 UI 測試之前，首先要取得一個代表「目前正在執行的應用程式」的物件，也就是要用 Javascript 呼叫 UIATarget.localTarget().frontMostApp()，在實機上可以順利取得這個物件，不過，在模擬器上，就會回傳 null，拿不到這個物件，之後就什麼事情都不能做了。

而看來不是所有的 iOS 4.0 裝置都可以跑 UI 自動測試，我接上衣台 iPhone 3G，Instument 就直接告訴我，這個裝置不在支援之列。