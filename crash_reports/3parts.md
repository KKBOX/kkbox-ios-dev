Crash Report 的三部分
--------------------

一份 crash report 大概分成三個部分。我們會用一份 crash report 說明，這
份 crash report 來自於 KKBOX 的 QA，而且裡頭還沒有解開。

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
時在做些什麼事情。

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
