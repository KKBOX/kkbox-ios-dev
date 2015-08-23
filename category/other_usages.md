Category 還可以有什麼用途？
---------------------------

除了幫原有的 class 增加新 method，我們也會在幾種狀況下使用 category。

### 將一個很大的 Class 切成幾個部分

由於我們可以在建立 class 之後，繼續透過 category 增加method，所以，假
如一個 class 很大，裡頭有數十個method，實作上千行，我們就可以考慮將這
個 class 的 method 拆分成若干個category，讓整個 class 的實作分開在不同
的檔案中，方便知道某一群的 method屬於什麼用途，也方便日後維護。

切開一個很大的 class 可以收到的好處包括：

#### 跨專案

如果你手上同時有好幾個專案，我們在進行專案的時候，由於之前寫的程式碼可
以重複使用，造成每個專案可能共用同一個class，但是每個專案又不見得都會
用到這個 class裡頭全部的實作，我們就可以考慮將只屬於某個專案的實作，拆
分到一個category 中。

#### 跨平台

做為寫 Objective-C語言的工程師，我們非常有可能會遇到跨平台開發的需求，
如果我們某段程式碼只有用到Mac OS X 與 iOS 都有的 library 與 framework
的話，我們的程式就可以同時在Mac OS X 與 iOS 使用。當我們打算在 Mac OS
X 與 iOS 共用同一個class，我們就可以考慮將跨平台的部份與平台相依的部份
拆開，將只屬於某個平台的部份拆成另外一個category，以蘋果自己的例子來說，
在 Mac OS X 與 iOS 上都有`NSString`，但由於兩個平台在繪圖方面的實作有
所不同，所以在繪製字串的部份，就被拆分到`NSStringDrawing` 與
`UIStringDrawing` 這些 category 中。

### 替換原本的實作

由於一個 class 有哪些 method，是在 runtime 時加入的，所以除了可以加入新
的 method 之外，假如我們嘗試再加入一個selector 與已經存在的 method 名
稱相同的實作，我們可以把已經存在的 method的實作，換成我們要加入的實作。
這麼做在 Objective-C語言中是完全合法的，如果 category 裡頭出現了名稱相
同的 method，compiler還是容許編譯成功，只會跳出簡單的警告訊息。

在實務上，這麼做卻非常危險，假如我們自己寫了一個class，我們又另外寫了
一個 category 置換其中的 method，當我們日後想要修改這個 method 的內容，
很容易忽略在 category中的同名 method，結果就是不管我們怎麼改動原本
method裡頭的程式，結果卻是什麼改變都沒有。

我自己曾經犯過一個低級錯誤：在開發時我建立了另外一個 git分支，在新分支
中，我覺得某個 class 太大，於是將部分 method拆到了另外一個 category 中，
但是開發主線卻又在修改這個class，結果造成合併分支的時候，就變成原本的
class 與 category 中出現了相同的 method，花了半天的時間才找到問題出在哪
裡。

除了某一個 category 中可以出現與原本 class 中名稱相同的 method，我們甚
至可以在好幾個 category 中，都出現名稱相同的 method，哪一個 category 在
執行時被最後載入，就會變成是這個 category中的實作。那麼，如果有多個
category，我們怎麼知道哪一個 category會被最後載入呢？Objective-C
runtime 並不保證 category 的載入順序，所以我們必須嚴格避免寫出這種程式。
