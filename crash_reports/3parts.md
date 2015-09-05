Crash Report 的三部分
--------------------

一份 crash report 大概分成三個部分。我們會用一份 crash report 說明，這
份 crash report 來自於 KKBOX 的 QA，而且裡頭還沒有解開（symbolicate）
－也就是說，從 crash report 的 call stack 這一段，我們只能夠看到發生問
題的 function/method 的記憶體位置，而看不到錯誤發生在程式的哪一行。

### 環境摘要與錯誤主要原因

在 crash report 的最上方可以看到一些基本資訊，其中比較重要的是

1. 機種：在下面這份 crash report 中，可以看到用戶用的是 iPad，同於
   KKBOX 的 iOS 版本同時支援 iPhone 與 iPad，而在 iPhone 與 iPad 上根
   本有兩套不一樣的 UI code，所以我們知道，現在應該要去看跟 iPad 有關
   的問題。
2. 版本號碼：同一個 App 的不同 build，都會產生不一樣的 debug symbol，
   所以如果要找到正確的 debug symbol，就一定要確認版號。在 KKBOX 的開
   發版本中，我們會對 Info.plist 做一些 pre-process，把 Jenkins build
   號碼（是的，KKBOX 使用 Jenkins 做持續整合，QA 都是直接從 Jenkins 上
   抓取需要測試的 build）與這一版程式的 git revision hash 也加到版號中，
   所以，我們可以知道，應該要去 Jenkins build 5145 裡頭去找 debug
   symbol。
3. 錯誤代碼：我們在這邊可以知道 Exception Type 是 EXC_CRASH，並且在
   main thread（thread 0）發生 crash。錯誤代碼對我們理解 crash 非常重
   要，我們會稍後說明。

```
Incident Identifier: CC5B3783-804F-49A7-AF6B-7C7B382CAE76
CrashReporter Key:   73124d372075eabb72b5625621a4396ffe893a49
Hardware Model:      iPad2,5
Process:             KKBOX [155]
Path:                /private/var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/KKBOX
Identifier:          tw.com.skysoft.iPhone
Version:             6.2.66.5145.073c882 (6.2.66)
Code Type:           ARM (Native)
Parent Process:      launchd [1]

Date/Time:           2015-08-06 10:41:52.859 +0800
Launch Time:         2015-08-06 10:39:28.377 +0800
OS Version:          iOS 8.3 (12F69)
Report Version:      104

Exception Type:  EXC_CRASH (SIGABRT)
Exception Codes: 0x0000000000000000, 0x0000000000000000
Triggered by Thread:  0
```

### Call Stack

第二部分是每個 thread 當時的 call stack，讓我們可以看到每個 thread 當
時在做些什麼事情。這邊會是了解問題的關鍵，因為我們要從這段資訊裡頭確實
知道 crash 發生在程式的哪個地方。

根據取得 crash report 的方式不同，你可能會看到不同格式的資料。比方說，
當你直接從裝置上抓下來，你會看到完全沒有解開的 call stack，像這樣：

```
Last Exception Backtrace:
(0x23e61132 0x321dec72 0x23e665f8 0x23e644d4 0x23d939d4 0x23e40272 0x24aefd96 0x24aefc46 0x755ba6 0x2776be1c 0x2776bede 0x27760a60 0x2756f2b6 0x27498c16 0x26eb7440 0x26eb2c90 0x26eb2b18 0x26eb24ba 0x26eb22aa 0x26f05ab8 0x2bad5bfe 0x24dd9d08 0x23e16550 0x23e26a46 0x23e269e2 0x23e25004 0x23d7099c 0x23d707ae 0x2b5201a4 0x274fb690 0xee070 0x32786aaa)

Thread 0 name:  Dispatch queue: com.apple.main-thread
Thread 0 Crashed:
0   libsystem_kernel.dylib        	0x3284cdf0 0x32838000 + 85488
1   libsystem_pthread.dylib       	0x328cdc92 0x328ca000 + 15506
2   libsystem_c.dylib             	0x327eb934 0x327a2000 + 301364
3   KKBOX                         	0x00a12ee2 0x10000 + 10497762
4   KKBOX                         	0x00a6e620 0x10000 + 10872352
5   CoreFoundation                	0x23e61466 0x23d57000 + 1090662
6   libobjc.A.dylib               	0x321deefc 0x321d8000 + 28412
7   libc++abi.dylib               	0x31a02dec 0x319eb000 + 97772
8   libc++abi.dylib               	0x31a028b4 0x319eb000 + 96436
9   libobjc.A.dylib               	0x321dedba 0x321d8000 + 28090
10  CoreFoundation                	0x23d70a38 0x23d57000 + 105016
11  CoreFoundation                	0x23d707ae 0x23d57000 + 104366
12  GraphicsServices              	0x2b5201a4 0x2b517000 + 37284
13  UIKit                         	0x274fb690 0x2748c000 + 456336
14  KKBOX                         	0x000ee070 0x10000 + 909424
15  libdyld.dylib                 	0x32786aac 0x32785000 + 6828
```

如果你是在你自己電腦上，用 Xcode 按下 Run，執行你的 App，因為 Xcode 幫
你保留了 App 的 debug symbol，所以在裝置上發生 crash 的時候，你用
Xcode 的 Devices 選單把 crash report 抓進來的時候，Xcode 會幫你用
debug symbol，完整解開整份 crash report。

但如果一份 crash report 是來自 QA，在 QA 的電腦上有 Xcode，但是並沒有
對應的 debug symbol 時，Xcode 也會嘗試做 symbolicate，但只能夠解開像是
UIKit、 CoreFoundation 之類的系統 library，但是看不到屬於你的 App 的哪
部分。解開系統 library 的部份往往沒什麼用，以上面那個 log 來說，如果你
熟悉 iOS 的運作，就算不解開記憶體位置，也可以看出 thread 0 的 call
stack 中：

- 15 是 `start`
- 14 是 KKBOX 的 main.m 裡頭的 `main`
- 13 是 UIApplicationMain
- 12-5 是在跑 run loop，而由於這是一個 exception，所以在 thread 0 中跟
  KKBOX 有關的部份，大概是 Google Analytics 或 Hockey App 的 exception
  handler

在這份 crash report 中，其實我們更有興趣的是 Last Exception Backtrace
這一段。這段資料代表的是 exception 發生當時到底發生了什麼事，也就是
exception 被 throw 出來的階段，而雖然 crash report 告訴我們發生 crash
的 thread 0，但這時候 thread 0 只是 catch exception 的地方而已。

### Libraries

第三部分是我們的 App 載入了哪些 library，因為很長，所以這邊只放了部分。
在這份 log 中，最重要的資訊是 KKBOX 被載入到 0x10000 - 0xc1bfff 這一段
記憶體的區間中，這個資訊對我們接下來怎麼解開記憶體位置非常重要。

從一個 App 載入了哪些 library，也可以看出一些特色。比方說，KKBOX 除了
載入自己的主要 App 之外，也載入了在自己 bundle 下的一份 Swift runtime，
像是 libswiftCore.dylib、libswiftCoreAudio.dylib 等等，代表 KKBOX 裡頭
使用了 Swift 語言。

我們在第一章就提到，Swift 與 Objective-C 的 runtime 實作不同，所以
Swift 程式必須要連結到特定版本的 Swift runtime，而用 Xcode 開發出來的
Swift App，會在每個 App bundle 裡頭都放一份 Swift runtime—那不就是裝置
裡頭有多少用 Swift 寫的 App，裝置上就放了多少份 Swift runtime？的確如
此，所以還好 Swift 核心 runtime 的尺寸不大，但我們也很擔心以後 Swift
會變得多大就是了。

```
Binary Images:
0x10000 - 0xc1bfff KKBOX armv7  <d91b937e9d073c21a79f5bbbe1cc4dbd> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/KKBOX
0x14e4000 - 0x164bfff libswiftCore.dylib armv7  <fa5b9494d6403f13ae80664a88301250> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftCore.dylib
0x17c0000 - 0x17c7fff libswiftCoreAudio.dylib armv7  <bd2181365f933ed3b330b0517f75fde0> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftCoreAudio.dylib
0x17d4000 - 0x17dffff libswiftCoreGraphics.dylib armv7  <95229d09c03d3eba9fbb038741503af3> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftCoreGraphics.dylib
0x17f4000 - 0x17fbfff libswiftCoreImage.dylib armv7  <59ca6e9173993aa39882799efdafd355> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftCoreImage.dylib
0x1808000 - 0x180ffff libswiftDarwin.dylib armv7  <764c0e157b49314088a4c9f8e1390a1a> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftDarwin.dylib
0x1820000 - 0x1823fff libswiftDispatch.dylib armv7  <e167a8d29dfd3694b0db2d3fc60adbb0> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftDispatch.dylib
0x1830000 - 0x185ffff libswiftFoundation.dylib armv7  <dde3f834069c3e6a9421572c0d01bbe3> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftFoundation.dylib
0x1898000 - 0x189ffff libswiftObjectiveC.dylib armv7  <786938b80ba63395aa5da6935df0c02e> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftObjectiveC.dylib
0x18ac000 - 0x18affff libswiftSecurity.dylib armv7  <12a8743e1ad636ebaaac523d1d709341> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftSecurity.dylib
0x18b8000 - 0x18c3fff libswiftUIKit.dylib armv7  <a04c41396cb939e6b3368360bfaffe10> /var/mobile/Containers/Bundle/Application/63493C9C-7C1B-47CA-83D3-8CC068537B89/KKBOX.app/Frameworks/libswiftUIKit.dylib
0x1fe5c000 - 0x1fe7ffff dyld armv7  <35ecdca1a767375e95ffa0f2a78d76d0> /usr/lib/dyld
0x22663000 - 0x2267efff libJapaneseConverter.dylib armv7  <1531c07cd9613bba8fc6fe9217f33612> /System/Library/CoreServices/Encodings/libJapaneseConverter.dylib
0x228f0000 - 0x22a5dfff AVFoundation armv7  <d269609e868231debb09b8b9b65a7367> /System/Library/Frameworks/AVFoundation.framework/AVFoundation
...
```

這一段資料還可以提供我們另外一種資訊。來看看這個 crash report，KKBOX
直接放在 /Users/USER 目錄下，並沒有放在 sandbox 環境中，而明明就該載入
armv7 的 library，卻載入了一堆 armv6 的 library，再仔細看看，出現了
CydiaSubstrate 這些東西。顯然用戶做了 JB。

```
   0x7d000 -   0x71cfff +KKBOX armv7  <f8a0ae9e6da833e0a9bd332e3d4b8a4c> /Users/USER/KKBOX.app/KKBOX
  0x8b1000 -   0x8b1fff  MobileSubstrate.dylib armv6  <ad3e6cb9e915360ebc71ccbf27bc4ea7> /Library/MobileSubstrate/MobileSubstrate.dylib
  0x903000 -   0x905fff  SubstrateLoader.dylib armv6  <974e4b1ab6e6397db859d79f37b7ab37> /Library/Frameworks/CydiaSubstrate.framework/Libraries/SubstrateLoader.dylib
  0x929000 -   0x935fff  Activator.dylib armv7s  <24b72822b56d30f4b88c83448f3db656> /Library/MobileSubstrate/DynamicLibraries/Activator.dylib
  0x972000 -   0x974fff  Emphasize.dylib armv7  <4a31e195474039db8a69730adc9b7642> /Library/MobileSubstrate/DynamicLibraries/Emphasize.dylib
  0x977000 -   0x97cfff  CydiaSubstrate armv6  <abeb3e46b03b3abeb9d3feba7fefe2fb> /Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate
  0xa82000 -   0xa89fff  libapplist.dylib armv7s  <3ef6ef0d6b7d350982114b6b4221ef01> /usr/lib/libapplist.dylib
  0xa90000 -   0xa95fff  LocalIAPStore.dylib armv7s  <f3341a7878d9319cb0bef7c2d2cbd896> /Library/MobileSubstrate/DynamicLibraries/LocalIAPStore.dylib
  0xa9f000 -   0xaa5fff  RePower.dylib armv7s  <dce8d29ea843350baad4f700b0443795> /Library/MobileSubstrate/DynamicLibraries/RePower.dylib
  0xaaa000 -   0xaaafff  ToneEnabler.dylib armv7s  <21fc136886163804b54e0524f7d1ca25> /Library/MobileSubstrate/DynamicLibraries/ToneEnabler.dylib
  0xaad000 -   0xaaefff  WeeLoader.dylib armv7  <7ff8f9166f93382eb27aba388d599a0a> /Library/MobileSubstrate/DynamicLibraries/WeeLoader.dylib
  0xab1000 -   0xac2fff  WinterBoard.dylib armv6  <2f7ede1815c93754b5c91a06195b485a> /Library/MobileSubstrate/DynamicLibraries/WinterBoard.dylib
  0xad6000 -   0xadafff  colorY0urBoardFree.dylib armv7s  <2e3552b8fe5c3bb8967e25d652d5a33f> /Library/MobileSubstrate/DynamicLibraries/colorY0urBoardFree.dylib
  0xadf000 -   0xae0fff  zeppelin_uikit.dylib armv7  <1401eeb739bb3a5d88a45926eac68b46> /Library/MobileSubstrate/DynamicLibraries/zeppelin_uikit.dylib
 0x77a5000 -  0x77b0fff  QuickSpeak armv7s  <9932eaf14fb23095b04ab610b05ba02e> /System/Library/AccessibilityBundles/QuickSpeak.bundle/QuickSpeak
```
