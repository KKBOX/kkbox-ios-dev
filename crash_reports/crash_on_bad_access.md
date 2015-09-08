實戰：Bad Access
----------------

因為在 ARC 問世後，Bad Access 造成的 crash 的確少很多，所以在這邊放的
是一份在 2012 年時為了解釋 Bad Access 是如何形成而留下來的 crash
report，但內容不是很完整，只保留了發生 Bad Access 時的 thread。

這是在 2012 年時 Pinterest App 發生的 crash，可以注意到當時是在 iPhone
4S 上使用 iOS 5 作業系統。在這份 crash report 中，可以清楚看到 iOS 5
之前如何處理記憶體不足警告的流程。

從 run loop 中，UIApplication 首先收到了記憶體不足警告，UIApplication
接著就透過 Notification Center 通知所有的 view controller 記憶體不足，
我們在
[記憶體管理 Part 3 - Memory Warnings](memory_management_part_3/README.md)
提過，iOS 5 之前，只要發生記憶體警告，就會要求所有不是在最前景的 view
controller 將自己的 view 釋放掉，`purgeMemoryForReason:` 與
`unloadViewForced:` 在做的，就是強迫釋放 view 這件事情。在強迫釋放
view 發生 crash，於是可以推斷，就算我們沒有把 Pinterest 的 crash
report 解開，也可以看出，0x0005995e 這個位置會是某個 view controller
的 `viewDidUnload`。

雖然 iOS 5 就推出了 ARC，但當時很多在 iOS 5 問世之前就寫出來的 App，並
沒有立刻轉換到 ARC 架構上，這份 crash report 呈現了當時這段轉換期。另
外，我們在[Crash 的類型](crash_types.md)也提到，在 `viewDidUnload` 最
容易發生的，就是忘記像 UIButton 這種建立時就是 auto release 的物件給
retain 起來，然後在 `viewDidUnload` 的時候多 release 了一次。

```
Incident Identifier: 3486ADCD-070E-43C8-ADC0-44E254DB92E8
CrashReporter Key:   babb1c6e8923eb91911e323103f4d82fa0bc7fe2
Hardware Model:      iPhone4,1
Process:         Pinterest [12210]
Path:            /var/mobile/Applications/4BFAD77B-FCE9-4EE1-A36D-ADFA55303130/Pinterest.app/Pinterest
Identifier:      Pinterest
Version:         ??? (???)
Code Type:       ARM (Native)
Parent Process:  launchd [1]

Date/Time:       2012-03-20 09:19:27.054 +0800
OS Version:      iPhone OS 5.1 (9B179)
Report Version:  104

Exception Type:  EXC_BAD_ACCESS (SIGSEGV)
Exception Codes: KERN_INVALID_ADDRESS at 0x20aba07b
Crashed Thread:  0

Thread 0 name:  Dispatch queue: com.apple.main-thread
Thread 0 Crashed:
0   libobjc.A.dylib                 0x35839f78 objc_msgSend + 16
1   Pinterest                       0x0005995e 0x1000 + 362846
2   UIKit                           0x33a84bd8 -[UIViewController unloadViewForced:] + 244
3   UIKit                           0x33bcc492 -[UIViewController purgeMemoryForReason:] + 58
4   Pinterest                       0x000a2df4 0x1000 + 663028
5   Pinterest                       0x000594f0 0x1000 + 361712
6   Foundation                      0x37b634f8 __57-[NSNotificationCenter addObserver:selector:name:object:]_block_invoke_0 + 12
7   CoreFoundation                  0x37305540 ___CFXNotificationPost_block_invoke_0 + 64
8   CoreFoundation                  0x37291090 _CFXNotificationPost + 1400
9   Foundation                      0x37ad73e4 -[NSNotificationCenter postNotificationName:object:userInfo:] + 60
10  Foundation                      0x37ad8c14 -[NSNotificationCenter postNotificationName:object:] + 24
11  UIKit                           0x33b9726a -[UIApplication _performMemoryWarning] + 74
12  UIKit                           0x33b97364 -[UIApplication _receivedMemoryNotification] + 168
13  libdispatch.dylib               0x345942da _dispatch_source_invoke + 510
14  libdispatch.dylib               0x34591b7a _dispatch_queue_invoke$VARIANT$mp + 46
15  libdispatch.dylib               0x34591eba _dispatch_main_queue_callback_4CF$VARIANT$mp + 150
16  CoreFoundation                  0x3730c2a6 __CFRunLoopRun + 1262
17  CoreFoundation                  0x3728f49e CFRunLoopRunSpecific + 294
18  CoreFoundation                  0x3728f366 CFRunLoopRunInMode + 98
19  GraphicsServices                0x3219c432 GSEventRunModal + 130
20  UIKit                           0x33a13e76 UIApplicationMain + 1074
21  Pinterest                       0x0000328a 0x1000 + 8842
22  Pinterest                       0x00003248 0x1000 + 8776
 ```
