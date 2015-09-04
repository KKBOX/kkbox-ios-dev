```
Incident Identifier: AB2474CA-D530-4CAF-A539-741C7AC07387
CrashReporter Key:   F39417F0-0875-4CD7-A31D-F303D5C74AF4
Hardware Model:      iPhone5,2
Process:         KKBOX [3261]
Path:            /Users/USER/KKBOX.app/KKBOX
Identifier:      tw.com.skysoft.iPhone
Version:         6.0.26
Code Type:       ARM
Parent Process:  launchd [1]

Date/Time:       2014-10-25T23:22:15Z
OS Version:      iPhone OS 7.0.2 (11A501)
Report Version:  104

Exception Type:  SIGABRT
Exception Codes: #0 at 0x38ef41fc
Crashed Thread:  0

Application Specific Information:
*** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '*** -[__NSPlaceholderDictionary initWithObjects:forKeys:count:]: attempt to insert nil object from objects[2]'

Last Exception Backtrace:
0   CoreFoundation                       0x2e649e8b __exceptionPreprocess + 131
1   libobjc.A.dylib                      0x389446c7 objc_exception_throw + 36
2   CoreFoundation                       0x2e587aef -[__NSPlaceholderDictionary initWithObjects:forKeys:count:] + 532
3   CoreFoundation                       0x2e5878b3 +[NSDictionary dictionaryWithObjects:forKeys:count:] + 48
4   KKBOX                                0x002989a9 -[KKExploreCardWithImageAndCaptionCell drawRect:] (KKExploreCardCollectionItemCell.m:159)
5   UIKit                                0x30e4f749 -[UIView(CALayerDelegate) drawLayer:inContext:] + 370
6   QuartzCore                           0x30a86049 -[CALayer drawInContext:] + 98
7   QuartzCore                           0x30a6f813 CABackingStoreUpdate_ + 1856
8   QuartzCore                           0x30b49735 ___ZN2CA5Layer8display_Ev_block_invoke + 50
9   QuartzCore                           0x30a6f0c3 x_blame_allocations + 80
10  QuartzCore                           0x30a6ed77 CA::Layer::display_() + 1116
11  QuartzCore                           0x30a52969 CA::Layer::display_if_needed(CA::Transaction*) + 206
12  QuartzCore                           0x30a52601 CA::Layer::layout_and_display_if_needed(CA::Transaction*) + 22
13  QuartzCore                           0x30a5200d CA::Context::commit_transaction(CA::Transaction*) + 226
14  QuartzCore                           0x30a51e1f CA::Transaction::commit() + 312
15  UIKit                                0x30dc786b _afterCACommitHandler + 124
16  CoreFoundation                       0x2e614f71 __CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__ + 18
17  CoreFoundation                       0x2e6128ff __CFRunLoopDoObservers + 284
18  CoreFoundation                       0x2e612c4b __CFRunLoopRun + 736
19  CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
20  CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
21  GraphicsServices                     0x332b42eb GSEventRunModal + 136
22  UIKit                                0x30e341e5 UIApplicationMain + 1134
23  KKBOX                                0x000e012d main (main.m:24)
24  libdyld.dylib                        0x38e3dab7 start + 0

Thread 0 Crashed:
0   libsystem_kernel.dylib               0x38ef41fc __pthread_kill + 8
1   libsystem_pthread.dylib              0x38f5ba53 pthread_kill + 56
2   libsystem_c.dylib                    0x38ea502d abort + 74
3   KKBOX                                0x00514937 uncaught_exception_handler + 24
4   CoreFoundation                       0x2e64a18d __handleUncaughtException + 578
5   libobjc.A.dylib                      0x38944927 _objc_terminate() + 172
6   libc++abi.dylib                      0x3830a1b3 std::__terminate(void (*)()) + 76
7   libc++abi.dylib                      0x38309d17 __cxa_rethrow + 100
8   libobjc.A.dylib                      0x3894480f objc_exception_rethrow + 40
9   CoreFoundation                       0x2e57d5b7 CFRunLoopRunSpecific + 640
10  CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
11  GraphicsServices                     0x332b42eb GSEventRunModal + 136
12  UIKit                                0x30e341e5 UIApplicationMain + 1134
13  KKBOX                                0x000e012d main (main.m:24)
14  libdyld.dylib                        0x38e3dab7 start + 0

Thread 1:
0   libsystem_kernel.dylib               0x38ee1838 kevent64 + 24
1   libdispatch.dylib                    0x38e2a643 _dispatch_mgr_thread + 36

Thread 2:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2ef6b827 -[NSRunLoop(NSRunLoop) runMode:beforeDate:] + 252
6   Foundation                           0x2efbc669 -[NSRunLoop(NSRunLoop) run] + 78
7   KKBOX                                0x0052b099 +[GAI threadMain:] + 62
8   Foundation                           0x2f02ddc7 __NSThread__main__ + 1060
9   libsystem_pthread.dylib              0x38f5ac5d _pthread_body + 138
10  libsystem_pthread.dylib              0x38f5abcf _pthread_start + 100
11  libsystem_pthread.dylib              0x38f58cd0 thread_start + 6

Thread 3:
0   libsystem_kernel.dylib               0x38ef4c7c __workq_kernreturn + 8
1   libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 4:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2ef6b827 -[NSRunLoop(NSRunLoop) runMode:beforeDate:] + 252
6   Foundation                           0x2ef8b3bb -[NSRunLoop(NSRunLoop) runUntilDate:] + 84
7   KKBOX                                0x002827f1 -[KKCloudPlaylistPullFullOperation main] (KKCloudPlaylistPullFullOperation.m:98)
8   Foundation                           0x2ef78c35 -[__NSOperationInternal _start:] + 770
9   Foundation                           0x2f01cafd __NSOQSchedule_f + 58
10  libdispatch.dylib                    0x38e2de77 _dispatch_queue_drain + 372
11  libdispatch.dylib                    0x38e2af9b _dispatch_queue_invoke + 40
12  libdispatch.dylib                    0x38e2e751 _dispatch_root_queue_drain + 74
13  libdispatch.dylib                    0x38e2e9d1 _dispatch_worker_thread2 + 54
14  libsystem_pthread.dylib              0x38f58dff _pthread_wqthread + 296
15  libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 5:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2efb8651 +[NSURLConnection(Loader) _resourceLoadLoop:] + 318
6   Foundation                           0x2f02ddc7 __NSThread__main__ + 1060
7   libsystem_pthread.dylib              0x38f5ac5d _pthread_body + 138
8   libsystem_pthread.dylib              0x38f5abcf _pthread_start + 100
9   libsystem_pthread.dylib              0x38f58cd0 thread_start + 6

Thread 6:
0   libsystem_kernel.dylib               0x38ef4440 __select + 20
1   libsystem_pthread.dylib              0x38f5ac5d _pthread_body + 138
2   libsystem_pthread.dylib              0x38f5abcf _pthread_start + 100
3   libsystem_pthread.dylib              0x38f58cd0 thread_start + 6

Thread 7:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e5c11ab CFRunLoopRun + 96
5   CoreMotion                           0x2ec35399 CLSF_thorntonUpdate_6x6 + 57222
6   libsystem_pthread.dylib              0x38f5ac5d _pthread_body + 138
7   libsystem_pthread.dylib              0x38f5abcf _pthread_start + 100
8   libsystem_pthread.dylib              0x38f58cd0 thread_start + 6

Thread 8:
0   libsystem_kernel.dylib               0x38ee1ad4 semaphore_wait_trap + 8
1   MediaToolbox                         0x2fa6ed0f fpa_AsyncMovieControlThread + 1752
2   CoreMedia                            0x2eba923f figThreadMain + 192
3   libsystem_pthread.dylib              0x38f5ac5d _pthread_body + 138
4   libsystem_pthread.dylib              0x38f5abcf _pthread_start + 100
5   libsystem_pthread.dylib              0x38f58cd0 thread_start + 6

Thread 9:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2ef6b827 -[NSRunLoop(NSRunLoop) runMode:beforeDate:] + 252
6   KKBOX                                0x0025bbed -[KKBOXAPICallOperation runloop] (KKBOXAPICallOperation.m:57)
7   KKBOX                                0x0025bd2b -[KKBOXAPICallOperation main] (KKBOXAPICallOperation.m:74)
8   Foundation                           0x2ef78c35 -[__NSOperationInternal _start:] + 770
9   Foundation                           0x2f01cafd __NSOQSchedule_f + 58
10  libdispatch.dylib                    0x38e2d4bf _dispatch_async_redirect_invoke + 108
11  libdispatch.dylib                    0x38e2e7e5 _dispatch_root_queue_drain + 222
12  libdispatch.dylib                    0x38e2e9d1 _dispatch_worker_thread2 + 54
13  libsystem_pthread.dylib              0x38f58dff _pthread_wqthread + 296
14  libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 10:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2ef6b827 -[NSRunLoop(NSRunLoop) runMode:beforeDate:] + 252
6   KKBOX                                0x0025bbed -[KKBOXAPICallOperation runloop] (KKBOXAPICallOperation.m:57)
7   KKBOX                                0x0025c295 -[KKBOXAPICallOperation main] (KKBOXAPICallOperation.m:145)
8   Foundation                           0x2ef78c35 -[__NSOperationInternal _start:] + 770
9   Foundation                           0x2f01cafd __NSOQSchedule_f + 58
10  libdispatch.dylib                    0x38e2d4bf _dispatch_async_redirect_invoke + 108
11  libdispatch.dylib                    0x38e2e7e5 _dispatch_root_queue_drain + 222
12  libdispatch.dylib                    0x38e2e9d1 _dispatch_worker_thread2 + 54
13  libsystem_pthread.dylib              0x38f58dff _pthread_wqthread + 296
14  libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 11:
0   libsystem_kernel.dylib               0x38ef4c7c __workq_kernreturn + 8
1   libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 12:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2ef6b827 -[NSRunLoop(NSRunLoop) runMode:beforeDate:] + 252
6   KKBOX                                0x0025bbed -[KKBOXAPICallOperation runloop] (KKBOXAPICallOperation.m:57)
7   KKBOX                                0x0025bd2b -[KKBOXAPICallOperation main] (KKBOXAPICallOperation.m:74)
8   Foundation                           0x2ef78c35 -[__NSOperationInternal _start:] + 770
9   Foundation                           0x2f01cafd __NSOQSchedule_f + 58
10  libdispatch.dylib                    0x38e2d4bf _dispatch_async_redirect_invoke + 108
11  libdispatch.dylib                    0x38e2e7e5 _dispatch_root_queue_drain + 222
12  libdispatch.dylib                    0x38e2e9d1 _dispatch_worker_thread2 + 54
13  libsystem_pthread.dylib              0x38f58dff _pthread_wqthread + 296
14  libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 13:
0   libsystem_kernel.dylib               0x38ef4c7c __workq_kernreturn + 8
1   libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 14:
0   libsystem_kernel.dylib               0x38ef4c7c __workq_kernreturn + 8
1   libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 15:
0   libsystem_kernel.dylib               0x38ef4c7c __workq_kernreturn + 8
1   libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 16:
0   libsystem_kernel.dylib               0x38ee1a84 mach_msg_trap + 20
1   CoreFoundation                       0x2e614561 __CFRunLoopServiceMachPort + 154
2   CoreFoundation                       0x2e612c81 __CFRunLoopRun + 790
3   CoreFoundation                       0x2e57d541 CFRunLoopRunSpecific + 522
4   CoreFoundation                       0x2e57d323 CFRunLoopRunInMode + 104
5   Foundation                           0x2ef6b827 -[NSRunLoop(NSRunLoop) runMode:beforeDate:] + 252
6   KKBOX                                0x001b12bd -[KKAlbumCoverImageFetchOperation main] (KKAlbumCoverManager.m:464)
7   Foundation                           0x2ef78c35 -[__NSOperationInternal _start:] + 770
8   Foundation                           0x2f01cafd __NSOQSchedule_f + 58
9   libdispatch.dylib                    0x38e2d4bf _dispatch_async_redirect_invoke + 108
10  libdispatch.dylib                    0x38e2e7e5 _dispatch_root_queue_drain + 222
11  libdispatch.dylib                    0x38e2e9d1 _dispatch_worker_thread2 + 54
12  libsystem_pthread.dylib              0x38f58dff _pthread_wqthread + 296
13  libsystem_pthread.dylib              0x38f58cc4 start_wqthread + 6

Thread 0 crashed with ARM Thread State:
    pc: 0x38ef41fc     r7: 0x27d868f4     sp: 0x27d868e8     r0: 0x00000000
    r1: 0x00000000     r2: 0x00000000     r3: 0xffffffff     r4: 0x00000006
    r5: 0x3ad2118c     r6: 0x1802de30     r8: 0x313cf0da     r9: 0x3ad21e30
   r10: 0x313b4457    r11: 0x00000019     ip: 0x00000148     lr: 0x38f5ba53
  cpsr: 0x00000010

Link Register Analysis:
  Symbol: pthread_kill + 56
  Description: We have determined that the link register (lr) is very likely to contain the return address of frame #0's calling function, and have inserted it into the crashing thread's backtrace as frame #1 to aid in analysis. This determination was made by applying a heuristic to determine whether the crashing function was likely to have created a new stack frame at the time of the crash.
  Type: 1

Binary Images:
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
0x2d173000 - 0x2d176fff  AccessibilitySettingsLoader armv7s  <2c36f6b91cc63b6892f3aaf7a4d4255a> /System/Library/AccessibilityBundles/AccessibilitySettingsLoader.bundle/AccessibilitySettingsLoader
0x2d233000 - 0x2d31cfff  RawCamera armv7s  <ae3d5bd17362338584b6022dc4427d10> /System/Library/CoreServices/RawCamera.bundle/RawCamera
0x2d448000 - 0x2d549fff  AVFoundation armv7s  <3ea3fd2b98d1360292bcb40c93e444e3> /System/Library/Frameworks/AVFoundation.framework/AVFoundation
0x2d54a000 - 0x2d572fff  libAVFAudio.dylib armv7s  <6f10a606028b3d2a862b2b1b8bf81eb3> /System/Library/Frameworks/AVFoundation.framework/libAVFAudio.dylib
0x2d573000 - 0x2d573fff  Accelerate armv7s  <9340338f3cdf347abe4a88c2f59b5b12> /System/Library/Frameworks/Accelerate.framework/Accelerate
0x2d57d000 - 0x2d74afff  vImage armv7s  <479b5c4701833284ab587a1d2fdb5627> /System/Library/Frameworks/Accelerate.framework/Frameworks/vImage.framework/vImage
0x2d74b000 - 0x2d82dfff  libBLAS.dylib armv7s  <da4fa367557d3028b02458e2cdf6d84d> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libBLAS.dylib
0x2d82e000 - 0x2dae9fff  libLAPACK.dylib armv7s  <066ea8372dd23f6d89011f9a4a872d6f> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libLAPACK.dylib
0x2daea000 - 0x2db58fff  libvDSP.dylib armv7s  <a5dcfe68199839b989c7be120c14ccb4> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libvDSP.dylib
0x2db59000 - 0x2db6bfff  libvMisc.dylib armv7s  <ea636bbda5ee33119a4e731aed02fa31> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libvMisc.dylib
0x2db6c000 - 0x2db6cfff  vecLib armv7s  <663aefa25bc5367baa72ca144ac26d18> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/vecLib
0x2db6d000 - 0x2db8cfff  Accounts armv7s  <23bffaebc2f23390817a66949208b36c> /System/Library/Frameworks/Accounts.framework/Accounts
0x2db8d000 - 0x2db8dfff  AdSupport armv7s  <e27e7aa767c7332f833fc8449af286b9> /System/Library/Frameworks/AdSupport.framework/AdSupport
0x2db8e000 - 0x2dbf3fff  AddressBook armv7s  <27d4ff7aebd4386e908e1b70b76a5516> /System/Library/Frameworks/AddressBook.framework/AddressBook
0x2dbf4000 - 0x2dd05fff  AddressBookUI armv7s  <51bf673528f235e5991e2b00b59518b8> /System/Library/Frameworks/AddressBookUI.framework/AddressBookUI
0x2dd06000 - 0x2dd17fff  AssetsLibrary armv7s  <24f602d680f13ca5b00cdc6a0d157bb1> /System/Library/Frameworks/AssetsLibrary.framework/AssetsLibrary
0x2dd18000 - 0x2de5bfff  AudioCodecs armv7s  <555c8ebca0c9327396c2c4838cc79455> /System/Library/Frameworks/AudioToolbox.framework/AudioCodecs
0x2de5c000 - 0x2e20afff  AudioToolbox armv7s  <e3d0e69f3ad632a4b1a29ecbc20ea8d3> /System/Library/Frameworks/AudioToolbox.framework/AudioToolbox
0x2e20b000 - 0x2e310fff  CFNetwork armv7s  <aa9d0dfc3c8337e5bd095f7b6d4b53e8> /System/Library/Frameworks/CFNetwork.framework/CFNetwork
0x2e311000 - 0x2e36cfff  CoreAudio armv7s  <73de1dcc06233a7c983aaab354f19eec> /System/Library/Frameworks/CoreAudio.framework/CoreAudio
0x2e36d000 - 0x2e383fff  CoreBluetooth armv7s  <3268403ee3fc33ca935038630018bd12> /System/Library/Frameworks/CoreBluetooth.framework/CoreBluetooth
0x2e384000 - 0x2e574fff  CoreData armv7s  <f58b2fa876db34d9babcc5e17893ae16> /System/Library/Frameworks/CoreData.framework/CoreData
0x2e575000 - 0x2e6b8fff  CoreFoundation armv7s  <29f79e2eeff633d1acdf695ad41f916c> /System/Library/Frameworks/CoreFoundation.framework/CoreFoundation
0x2e6b9000 - 0x2e7ddfff  CoreGraphics armv7s  <b355402aa362364c838b34e9aa91acba> /System/Library/Frameworks/CoreGraphics.framework/CoreGraphics
0x2e7df000 - 0x2e81afff  libCGFreetype.A.dylib armv7s  <13b1f99f838c3440941004d726d7cf64> /System/Library/Frameworks/CoreGraphics.framework/Resources/libCGFreetype.A.dylib
0x2e81c000 - 0x2e826fff  libCMSBuiltin.A.dylib armv7s  <95e72800562936fd9ff7b60a38c498a6> /System/Library/Frameworks/CoreGraphics.framework/Resources/libCMSBuiltin.A.dylib
0x2ea0b000 - 0x2ea25fff  libRIP.A.dylib armv7s  <b443405791f93791bc6140c5c50f86bf> /System/Library/Frameworks/CoreGraphics.framework/Resources/libRIP.A.dylib
0x2ea26000 - 0x2eafefff  CoreImage armv7s  <c23091e38fc7316b8c3b5b94efa2689d> /System/Library/Frameworks/CoreImage.framework/CoreImage
0x2eaff000 - 0x2eb4cfff  CoreLocation armv7s  <87d3830bbe123e45a611eac7e4bab134> /System/Library/Frameworks/CoreLocation.framework/CoreLocation
0x2eb84000 - 0x2ebfbfff  CoreMedia armv7s  <80585a9c30bb39679cd285697075c668> /System/Library/Frameworks/CoreMedia.framework/CoreMedia
0x2ebfc000 - 0x2eca4fff  CoreMotion armv7s  <07a70dfe457f31eeafccdc453d5fc09f> /System/Library/Frameworks/CoreMotion.framework/CoreMotion
0x2eca5000 - 0x2ecfdfff  CoreTelephony armv7s  <d1abcb27328a3d32b50e43f7d8723f88> /System/Library/Frameworks/CoreTelephony.framework/CoreTelephony
0x2ecfe000 - 0x2ed8dfff  CoreText armv7s  <640ae778897530d089098f55db9ffb58> /System/Library/Frameworks/CoreText.framework/CoreText
0x2ed8e000 - 0x2ed9dfff  CoreVideo armv7s  <09adc070759b39ad9a0cc38513339d1c> /System/Library/Frameworks/CoreVideo.framework/CoreVideo
0x2ed9e000 - 0x2ee5cfff  EventKit armv7s  <6a59cfcd59cd3c9391a60bdfa9b19fbf> /System/Library/Frameworks/EventKit.framework/EventKit
0x2ee5d000 - 0x2ef4ffff  EventKitUI armv7s  <86cb5a91468d365495d9107c46b9b946> /System/Library/Frameworks/EventKitUI.framework/EventKitUI
0x2ef60000 - 0x2f14afff  Foundation armv7s  <3baf454a0faf3f9ea8d08538fb233dfd> /System/Library/Frameworks/Foundation.framework/Foundation
0x2f14b000 - 0x2f175fff  GLKit armv7s  <a01140a43577352b9846e8ecd9a0f662> /System/Library/Frameworks/GLKit.framework/GLKit
0x2f328000 - 0x2f37efff  IOKit armv7s  <c504536b28b43297a5b874d0a4d958e8> /System/Library/Frameworks/IOKit.framework/Versions/A/IOKit
0x2f37f000 - 0x2f58ffff  ImageIO armv7s  <ff5ce78d675c375883a2b78ad7bc726e> /System/Library/Frameworks/ImageIO.framework/ImageIO
0x2f590000 - 0x2f7d8fff  JavaScriptCore armv7s  <daace5d68dfd3f299b1755a5f7b8cb6c> /System/Library/Frameworks/JavaScriptCore.framework/JavaScriptCore
0x2f7d9000 - 0x2f87bfff  MapKit armv7s  <7a13d31e0bd93ea1a42f5ce5435c71d9> /System/Library/Frameworks/MapKit.framework/MapKit
0x2f87c000 - 0x2f880fff  MediaAccessibility armv7s  <385b8559d9bd3721b466d4d074b4cc26> /System/Library/Frameworks/MediaAccessibility.framework/MediaAccessibility
0x2f881000 - 0x2fa69fff  MediaPlayer armv7s  <32b2c665039b30fe9a78ee59a9e5143b> /System/Library/Frameworks/MediaPlayer.framework/MediaPlayer
0x2fa6a000 - 0x2fd23fff  MediaToolbox armv7s  <9309ab10b2d3316d8ee1a47a9138cb3e> /System/Library/Frameworks/MediaToolbox.framework/MediaToolbox
0x2fd24000 - 0x2fdbffff  MessageUI armv7s  <8c878c2cf67535cd9c88f99d1c6b81e4> /System/Library/Frameworks/MessageUI.framework/MessageUI
0x2fdc0000 - 0x2fe23fff  MobileCoreServices armv7s  <7641d47fcebb30019f1b2d572b6184d3> /System/Library/Frameworks/MobileCoreServices.framework/MobileCoreServices
0x2fe7f000 - 0x2feb2fff  OpenAL armv7s  <ad6db8d403d7374abb78a06d40ddd6b1> /System/Library/Frameworks/OpenAL.framework/OpenAL
0x30857000 - 0x3085ffff  OpenGLES armv7s  <f66e6ce6393c316784f0a6cf2c3cfa24> /System/Library/Frameworks/OpenGLES.framework/OpenGLES
0x30861000 - 0x30861fff  libCVMSPluginSupport.dylib armv7s  <d5abff68493b3a8b88099c122d49246c> /System/Library/Frameworks/OpenGLES.framework/libCVMSPluginSupport.dylib
0x30865000 - 0x30868fff  libCoreVMClient.dylib armv7s  <aa8b008095d2312c8aa226e3d10b027b> /System/Library/Frameworks/OpenGLES.framework/libCoreVMClient.dylib
0x30869000 - 0x30870fff  libGFXShared.dylib armv7s  <667e409cfe5b39c58e50240c841cf7ef> /System/Library/Frameworks/OpenGLES.framework/libGFXShared.dylib
0x30871000 - 0x308b1fff  libGLImage.dylib armv7s  <73b69adcf3b434548d424857d93e3505> /System/Library/Frameworks/OpenGLES.framework/libGLImage.dylib
0x30a4a000 - 0x30b8ffff  QuartzCore armv7s  <38ffb26d65b3385ab45faa5a8fc5d963> /System/Library/Frameworks/QuartzCore.framework/QuartzCore
0x30b90000 - 0x30be6fff  QuickLook armv7s  <ec4d1971e0253414948bed34e195decb> /System/Library/Frameworks/QuickLook.framework/QuickLook
0x30be9000 - 0x30c29fff  Security armv7s  <3bdd534aeba03353acf5eef99d47c2e0> /System/Library/Frameworks/Security.framework/Security
0x30c2a000 - 0x30c9efff  Social armv7s  <42a050c5a8e73bcb8039f5b34b351fa8> /System/Library/Frameworks/Social.framework/Social
0x30d5e000 - 0x30d71fff  StoreKit armv7s  <52e20e657f903fe3b743943579edd41e> /System/Library/Frameworks/StoreKit.framework/StoreKit
0x30d72000 - 0x30dc1fff  SystemConfiguration armv7s  <ec3065bf336830cb85e9072fe4480a4f> /System/Library/Frameworks/SystemConfiguration.framework/SystemConfiguration
0x30dc4000 - 0x314e3fff  UIKit armv7s  <5f5397297fa73ab0889777dd49ebd8e1> /System/Library/Frameworks/UIKit.framework/UIKit
0x314e4000 - 0x31532fff  VideoToolbox armv7s  <dee33b6c37d13f13b81a0160e7efa423> /System/Library/Frameworks/VideoToolbox.framework/VideoToolbox
0x317a5000 - 0x317aefff  AOSNotification armv7s  <d52fe916a3ea3cb88174198668a70cf4> /System/Library/PrivateFrameworks/AOSNotification.framework/AOSNotification
0x31812000 - 0x3185bfff  AccessibilityUtilities armv7s  <b61fc80d33663b34a1f4807bc07ed120> /System/Library/PrivateFrameworks/AccessibilityUtilities.framework/AccessibilityUtilities
0x318af000 - 0x318cffff  AccountsUI armv7s  <1a36a808008733a3b0dcba14017c67a7> /System/Library/PrivateFrameworks/AccountsUI.framework/AccountsUI
0x318d0000 - 0x318d4fff  AggregateDictionary armv7s  <b5a1d0802c003f24b347242a5293daa6> /System/Library/PrivateFrameworks/AggregateDictionary.framework/AggregateDictionary
0x31ad2000 - 0x31ae6fff  AirTraffic armv7s  <8c0533b7adca3efeace4ea1628a80b19> /System/Library/PrivateFrameworks/AirTraffic.framework/AirTraffic
0x31ae7000 - 0x31e22fff  Altitude armv7s  <d3231c90d897359a93a945cf19e76563> /System/Library/PrivateFrameworks/Altitude.framework/Altitude
0x31e57000 - 0x31e94fff  AppSupport armv7s  <eaacdfb4974531409eda0f399b99fcb5> /System/Library/PrivateFrameworks/AppSupport.framework/AppSupport
0x31e95000 - 0x31eccfff  AppleAccount armv7s  <e3860cad86cb3f4ba87909e890f25e4f> /System/Library/PrivateFrameworks/AppleAccount.framework/AppleAccount
0x31f6c000 - 0x31f7cfff  ApplePushService armv7s  <c757d1d3414c3274ad34f1e86d1a7bd3> /System/Library/PrivateFrameworks/ApplePushService.framework/ApplePushService
0x31fb4000 - 0x31fc0fff  AssetsLibraryServices armv7s  <3ff8908f255132c688a2e8ae1e6bc221> /System/Library/PrivateFrameworks/AssetsLibraryServices.framework/AssetsLibraryServices
0x31fc1000 - 0x31fdcfff  AssistantServices armv7s  <311a52aacd91310293715414a922820b> /System/Library/PrivateFrameworks/AssistantServices.framework/AssistantServices
0x31ffd000 - 0x32000fff  BTLEAudioController armv7s  <631049fd95ae3dad8ab065b43840404d> /System/Library/PrivateFrameworks/BTLEAudioController.framework/BTLEAudioController
0x32001000 - 0x32024fff  BackBoardServices armv7s  <0238e0b01eb03e9d92409f7dc0d27c85> /System/Library/PrivateFrameworks/BackBoardServices.framework/BackBoardServices
0x32027000 - 0x3202cfff  BluetoothManager armv7s  <d1fb9112ad3f340d8051315445eff177> /System/Library/PrivateFrameworks/BluetoothManager.framework/BluetoothManager
0x3202d000 - 0x32051fff  Bom armv7s  <c7f17a5e9bf43b1291052841d478ae95> /System/Library/PrivateFrameworks/Bom.framework/Bom
0x32064000 - 0x320acfff  BulletinBoard armv7s  <3011c66c543531a382346a2a12f60a6f> /System/Library/PrivateFrameworks/BulletinBoard.framework/BulletinBoard
0x320f0000 - 0x320f8fff  CaptiveNetwork armv7s  <0863d2a2c3ce3ababd236d49bc41e26b> /System/Library/PrivateFrameworks/CaptiveNetwork.framework/CaptiveNetwork
0x320f9000 - 0x321d3fff  Celestial armv7s  <e54ab561b9383e31bf10f3e0611de842> /System/Library/PrivateFrameworks/Celestial.framework/Celestial
0x321d4000 - 0x321dffff  CertInfo armv7s  <78f93f2defb7306ca8f70cdcdf95cf54> /System/Library/PrivateFrameworks/CertInfo.framework/CertInfo
0x321e0000 - 0x321e5fff  CertUI armv7s  <3366ec6a08c93a9c98488e9696ebf1d2> /System/Library/PrivateFrameworks/CertUI.framework/CertUI
0x322ae000 - 0x322cefff  ChunkingLibrary armv7s  <719f2a07f62f3245b8b7d814ddd6c4f3> /System/Library/PrivateFrameworks/ChunkingLibrary.framework/ChunkingLibrary
0x3231f000 - 0x3232afff  CommonUtilities armv7s  <9c015db83e55300dbd48f4f28db35978> /System/Library/PrivateFrameworks/CommonUtilities.framework/CommonUtilities
0x3232b000 - 0x3232ffff  CommunicationsFilter armv7s  <9336295ddf07377681a39ed84af77aaa> /System/Library/PrivateFrameworks/CommunicationsFilter.framework/CommunicationsFilter
0x323c7000 - 0x323f7fff  ContentIndex armv7s  <ea1ac0c33c273e48be7b4f1a8f2ce6e6> /System/Library/PrivateFrameworks/ContentIndex.framework/ContentIndex
0x323f8000 - 0x323fafff  CoreAUC armv7s  <835f573269fa3329891ee5477c0c48c0> /System/Library/PrivateFrameworks/CoreAUC.framework/CoreAUC
0x32407000 - 0x3245afff  CoreDAV armv7s  <15ef8f74676d36a58b25c8e4ae42df7f> /System/Library/PrivateFrameworks/CoreDAV.framework/CoreDAV
0x3249b000 - 0x32599fff  CoreMediaStream armv7s  <83c99a6c3e0f3023abb025162eb3ba58> /System/Library/PrivateFrameworks/CoreMediaStream.framework/CoreMediaStream
0x32633000 - 0x3263dfff  CoreRecents armv7s  <0351246c23a63badab2047173e7bf080> /System/Library/PrivateFrameworks/CoreRecents.framework/CoreRecents
0x3268b000 - 0x326a9fff  CoreServicesInternal armv7s  <81f48b4e902a36f68f38df1514fd02a0> /System/Library/PrivateFrameworks/CoreServicesInternal.framework/CoreServicesInternal
0x326aa000 - 0x326abfff  CoreSurface armv7s  <b841c412278b31e8ab3fdcc9ef3f4952> /System/Library/PrivateFrameworks/CoreSurface.framework/CoreSurface
0x326ac000 - 0x32712fff  CoreSymbolication armv7s  <bc8f947cca233d0f846455ab2bd75b29> /System/Library/PrivateFrameworks/CoreSymbolication.framework/CoreSymbolication
0x3274d000 - 0x32751fff  CoreTime armv7s  <57a1ef07717a37b1acbb219dee6c4305> /System/Library/PrivateFrameworks/CoreTime.framework/CoreTime
0x32752000 - 0x327acfff  CoreUI armv7s  <b8d30ee2ace23424b1894da1221896c8> /System/Library/PrivateFrameworks/CoreUI.framework/CoreUI
0x327ad000 - 0x327fafff  CoreUtils armv7s  <a0ed77908628345face830cd19fdaa0d> /System/Library/PrivateFrameworks/CoreUtils.framework/CoreUtils
0x327fb000 - 0x32800fff  CrashReporterSupport armv7s  <048637e8b7bc3cd491ac9a13c67f48c6> /System/Library/PrivateFrameworks/CrashReporterSupport.framework/CrashReporterSupport
0x32801000 - 0x32837fff  DataAccess armv7s  <d45ade7190993faaba5495911502ca77> /System/Library/PrivateFrameworks/DataAccess.framework/DataAccess
0x329c9000 - 0x329defff  DataAccessExpress armv7s  <065dee6e6811309189e682901dc2188e> /System/Library/PrivateFrameworks/DataAccessExpress.framework/DataAccessExpress
0x32a19000 - 0x32a1cfff  DataMigration armv7s  <9db6af4e660f36b289b5498202828768> /System/Library/PrivateFrameworks/DataMigration.framework/DataMigration
0x32a21000 - 0x32a22fff  DiagnosticLogCollection armv7s  <c9dc18474a693be8beb77a32e9bff97b> /System/Library/PrivateFrameworks/DiagnosticLogCollection.framework/DiagnosticLogCollection
0x32a23000 - 0x32a3dfff  DictionaryServices armv7s  <b11ee1719c90367a8173643f39dcae46> /System/Library/PrivateFrameworks/DictionaryServices.framework/DictionaryServices
0x32a59000 - 0x32a76fff  EAP8021X armv7s  <b0770c75118e3d8fb9eb8d1397409bdb> /System/Library/PrivateFrameworks/EAP8021X.framework/EAP8021X
0x32a7f000 - 0x32a8afff  ExFAT armv7s  <8e0b177c08bf34edabb3937b0139c002> /System/Library/PrivateFrameworks/ExFAT.framework/ExFAT
0x32a8b000 - 0x32a9bfff  FTAWD armv7s  <03a5778e9c2037e3802c2d9406c1d7f4> /System/Library/PrivateFrameworks/FTAWD.framework/FTAWD
0x32a9c000 - 0x32a9efff  FTClientServices armv7s  <6e90e2d568893ca6b196286629aeecb1> /System/Library/PrivateFrameworks/FTClientServices.framework/FTClientServices
0x32a9f000 - 0x32ac8fff  FTServices armv7s  <aa7e3982cd2e30e3897bfbda6ed07802> /System/Library/PrivateFrameworks/FTServices.framework/FTServices
0x32ac9000 - 0x32ee4fff  FaceCore armv7s  <96a335f8e5ad382283eedb4b7f627c8a> /System/Library/PrivateFrameworks/FaceCore.framework/FaceCore
0x33107000 - 0x33113fff  GenerationalStorage armv7s  <8ca94191735b38c296ceb0a1b7d6f9ef> /System/Library/PrivateFrameworks/GenerationalStorage.framework/GenerationalStorage
0x33114000 - 0x332acfff  GeoServices armv7s  <619d7e9213f631ba8198549e0ed89e4e> /System/Library/PrivateFrameworks/GeoServices.framework/GeoServices
0x332ad000 - 0x332bbfff  GraphicsServices armv7s  <d887316a59d5342ebd0a6a5e8ea4d7cf> /System/Library/PrivateFrameworks/GraphicsServices.framework/GraphicsServices
0x3334a000 - 0x333cffff  HomeSharing armv7s  <7fac964790c9320099a82d580ae9432b> /System/Library/PrivateFrameworks/HomeSharing.framework/HomeSharing
0x333d0000 - 0x333dcfff  IAP armv7s  <8a3fc4fb6ce23adf941905a3744ab8de> /System/Library/PrivateFrameworks/IAP.framework/IAP
0x33442000 - 0x33476fff  IDS armv7s  <fc20f09d1fcb3ec5bde9e7d7b8718271> /System/Library/PrivateFrameworks/IDS.framework/IDS
0x334e2000 - 0x334f3fff  IDSFoundation armv7s  <3936bc676b103a0d9bab55dbe0f33865> /System/Library/PrivateFrameworks/IDSFoundation.framework/IDSFoundation
0x334f4000 - 0x33558fff  IMAVCore armv7s  <facc1ed04f69340eab5ba9648d642d6d> /System/Library/PrivateFrameworks/IMAVCore.framework/IMAVCore
0x33559000 - 0x335e5fff  IMCore armv7s  <238b1756f1b23477b032eb94116c7f9f> /System/Library/PrivateFrameworks/IMCore.framework/IMCore
0x33665000 - 0x336bffff  IMFoundation armv7s  <48fb553357293073b55db1c37790df53> /System/Library/PrivateFrameworks/IMFoundation.framework/IMFoundation
0x336c9000 - 0x336d0fff  IOMobileFramebuffer armv7s  <452bfbda283c39cf95b400da3a553a12> /System/Library/PrivateFrameworks/IOMobileFramebuffer.framework/IOMobileFramebuffer
0x336d1000 - 0x336d6fff  IOSurface armv7s  <d8adf4a312fc34bf915b6f779450d6fa> /System/Library/PrivateFrameworks/IOSurface.framework/IOSurface
0x33723000 - 0x33728fff  IncomingCallFilter armv7s  <4afe562532003eee95efccc03968d10e> /System/Library/PrivateFrameworks/IncomingCallFilter.framework/IncomingCallFilter
0x33748000 - 0x33754fff  Librarian armv7s  <2e4847d5f83a3609bbe955ebe4982ac9> /System/Library/PrivateFrameworks/Librarian.framework/Librarian
0x33755000 - 0x3378efff  MIME armv7s  <9be7262f994d36198059ce25d8881c65> /System/Library/PrivateFrameworks/MIME.framework/MIME
0x3378f000 - 0x337ccfff  MMCS armv7s  <1298ffc6057d34dc87e65d369bf3c66c> /System/Library/PrivateFrameworks/MMCS.framework/MMCS
0x337d5000 - 0x337e0fff  MailServices armv7s  <00abdcca62743fd58fe43ea2405ef497> /System/Library/PrivateFrameworks/MailServices.framework/MailServices
0x33814000 - 0x3388cfff  ManagedConfiguration armv7s  <eaf716901eea3605afffc783242fcd55> /System/Library/PrivateFrameworks/ManagedConfiguration.framework/ManagedConfiguration
0x3388d000 - 0x3388efff  Marco armv7s  <d2c12a9cb9833ad9ab47edec46b8de2c> /System/Library/PrivateFrameworks/Marco.framework/Marco
0x3388f000 - 0x33907fff  MediaControlSender armv7s  <4c0017a9ceea3191af04244f25aab767> /System/Library/PrivateFrameworks/MediaControlSender.framework/MediaControlSender
0x3393f000 - 0x33949fff  MediaRemote armv7s  <2bee47a1f52033379806849605e0cf61> /System/Library/PrivateFrameworks/MediaRemote.framework/MediaRemote
0x3394a000 - 0x33962fff  MediaStream armv7s  <dcb940462fe03d71970aca1cedf25596> /System/Library/PrivateFrameworks/MediaStream.framework/MediaStream
0x339c6000 - 0x33a98fff  Message armv7s  <f3abf378cab337e88547ffbd08f9cfaa> /System/Library/PrivateFrameworks/Message.framework/Message
0x33a9d000 - 0x33a9ffff  MessageSupport armv7s  <1008f2f58ca5317fa56eb49acf4da577> /System/Library/PrivateFrameworks/MessageSupport.framework/MessageSupport
0x33aaa000 - 0x33ab5fff  MobileAsset armv7s  <89df8176733130b7a22b93f0f6990b26> /System/Library/PrivateFrameworks/MobileAsset.framework/MobileAsset
0x33ad9000 - 0x33ae1fff  MobileBluetooth armv7s  <d9631876c4fa360193baca1a1369e003> /System/Library/PrivateFrameworks/MobileBluetooth.framework/MobileBluetooth
0x33af4000 - 0x33afbfff  MobileIcons armv7s  <47b04e3e3d1531729207515ffa1f0a54> /System/Library/PrivateFrameworks/MobileIcons.framework/MobileIcons
0x33afc000 - 0x33afffff  MobileInstallation armv7s  <5f4deac3041631a9bc310d32e0eba04f> /System/Library/PrivateFrameworks/MobileInstallation.framework/MobileInstallation
0x33b00000 - 0x33b08fff  MobileKeyBag armv7s  <e86226dd92633562ae880eb619ae2fb2> /System/Library/PrivateFrameworks/MobileKeyBag.framework/MobileKeyBag
0x33b30000 - 0x33b33fff  MobileSystemServices armv7s  <7ccabe1b59f635bfb866ea672139080a> /System/Library/PrivateFrameworks/MobileSystemServices.framework/MobileSystemServices
0x33b52000 - 0x33b5dfff  MobileWiFi armv7s  <3709673f12163bbb8ead912f146094db> /System/Library/PrivateFrameworks/MobileWiFi.framework/MobileWiFi
0x33b94000 - 0x33d1bfff  MusicLibrary armv7s  <4e3605a1df833c55816f272507c8da8c> /System/Library/PrivateFrameworks/MusicLibrary.framework/MusicLibrary
0x33dd0000 - 0x33dd5fff  Netrb armv7s  <73176bc012be3a89a8f41005b6fcdbce> /System/Library/PrivateFrameworks/Netrb.framework/Netrb
0x33dd6000 - 0x33ddbfff  NetworkStatistics armv7s  <57f5831c2cc43b8281157882df7e6447> /System/Library/PrivateFrameworks/NetworkStatistics.framework/NetworkStatistics
0x33ddc000 - 0x33df9fff  Notes armv7s  <86824a1374103929855cfab813d34f90> /System/Library/PrivateFrameworks/Notes.framework/Notes
0x33dfa000 - 0x33dfcfff  OAuth armv7s  <f8769c1821ad3b1f9bfa004f8d9c7506> /System/Library/PrivateFrameworks/OAuth.framework/OAuth
0x34554000 - 0x3458ffff  OpenCL armv7s  <1e7f9319adb7309c9c50f2a4b860b67c> /System/Library/PrivateFrameworks/OpenCL.framework/OpenCL
0x34b36000 - 0x34b5cfff  PersistentConnection armv7s  <9dd93dcc3b5133f2b92cb8ce2c854a11> /System/Library/PrivateFrameworks/PersistentConnection.framework/PersistentConnection
0x34cc0000 - 0x34e35fff  PhotoLibraryServices armv7s  <2ce8ae3b632e3b3793b81524c50bd402> /System/Library/PrivateFrameworks/PhotoLibraryServices.framework/PhotoLibraryServices
0x34f73000 - 0x34fa0fff  PhysicsKit armv7s  <aa8b66ecce7d3d4aba28370333f5b7c5> /System/Library/PrivateFrameworks/PhysicsKit.framework/PhysicsKit
0x34fa1000 - 0x34fa4fff  PowerLog armv7s  <b39b62177cf538f4be20f518402ba9a9> /System/Library/PrivateFrameworks/PowerLog.framework/PowerLog
0x35020000 - 0x3508cfff  Preferences armv7s  <e0be490e64303934a5b8bbcf30691a42> /System/Library/PrivateFrameworks/Preferences.framework/Preferences
0x3508d000 - 0x350c4fff  PrintKit armv7s  <826a71b346d23f05a3c6c20019d2cd48> /System/Library/PrivateFrameworks/PrintKit.framework/PrintKit
0x350c8000 - 0x3514ffff  ProofReader armv7s  <c4d261a63f0237ec9419a46abf0c89ec> /System/Library/PrivateFrameworks/ProofReader.framework/ProofReader
0x35150000 - 0x3515afff  ProtocolBuffer armv7s  <502c8769742f354d97a3273260e8584f> /System/Library/PrivateFrameworks/ProtocolBuffer.framework/ProtocolBuffer
0x3515b000 - 0x3518bfff  PrototypeTools armv7s  <107f7cf35c263171b10f4adfb81ae35b> /System/Library/PrivateFrameworks/PrototypeTools.framework/PrototypeTools
0x3518c000 - 0x35200fff  Quagga armv7s  <19af82b757173d0db60429300235c4e7> /System/Library/PrivateFrameworks/Quagga.framework/Quagga
0x35201000 - 0x35250fff  Radio armv7s  <2dcae1a18ff53de0a6114d93237a1933> /System/Library/PrivateFrameworks/Radio.framework/Radio
0x352d9000 - 0x35359fff  SAObjects armv7s  <87143b37d6d837b3917ddf065991efc8> /System/Library/PrivateFrameworks/SAObjects.framework/SAObjects
0x35366000 - 0x35385fff  ScreenReaderCore armv7s  <fb14cddcab1f3c1497acd638d41d931b> /System/Library/PrivateFrameworks/ScreenReaderCore.framework/ScreenReaderCore
0x35459000 - 0x3547bfff  SpringBoardFoundation armv7s  <9fdc36b95e983f32b86b98515040ea4a> /System/Library/PrivateFrameworks/SpringBoardFoundation.framework/SpringBoardFoundation
0x3547c000 - 0x35490fff  SpringBoardServices armv7s  <8e31d0a20f6e311e814c9441ee066040> /System/Library/PrivateFrameworks/SpringBoardServices.framework/SpringBoardServices
0x35491000 - 0x354aafff  SpringBoardUI armv7s  <3115f76db448369ca2ea23d665c621ab> /System/Library/PrivateFrameworks/SpringBoardUI.framework/SpringBoardUI
0x354ab000 - 0x354c1fff  SpringBoardUIServices armv7s  <3104518cb2dd36e687e106f767b00768> /System/Library/PrivateFrameworks/SpringBoardUIServices.framework/SpringBoardUIServices
0x356a0000 - 0x357b7fff  StoreServices armv7s  <49b5dd271ebb3ad28c3ab0bd7025eab0> /System/Library/PrivateFrameworks/StoreServices.framework/StoreServices
0x357b8000 - 0x357c7fff  StreamingZip armv7s  <fb0b01aaeaee360094bace6460c91549> /System/Library/PrivateFrameworks/StreamingZip.framework/StreamingZip
0x35866000 - 0x35868fff  TCC armv7s  <a5f1bc68799e3089b97fdbc682608bb7> /System/Library/PrivateFrameworks/TCC.framework/TCC
0x35869000 - 0x358b1fff  TelephonyUI armv7s  <cebe06cf1adb3a5397da55f562f86575> /System/Library/PrivateFrameworks/TelephonyUI.framework/TelephonyUI
0x358b2000 - 0x358d3fff  TelephonyUtilities armv7s  <89161379ee943028814036c307c89b5a> /System/Library/PrivateFrameworks/TelephonyUtilities.framework/TelephonyUtilities
0x358d4000 - 0x35c45fff  KBLayouts_iPhone.dylib armv7s  <1aecca20df35304e8b984eeb590a0b6d> /System/Library/PrivateFrameworks/TextInput.framework/KBLayouts_iPhone.dylib
0x35c46000 - 0x35c6afff  TextInput armv7s  <cdaeeb7ed09033ef91e91e71587be2e1> /System/Library/PrivateFrameworks/TextInput.framework/TextInput
0x35c6b000 - 0x35e3afff  TextToSpeech armv7s  <053f080a453330fb8217d02a6142509e> /System/Library/PrivateFrameworks/TextToSpeech.framework/TextToSpeech
0x35e3b000 - 0x35e61fff  ToneKit armv7s  <f5d1b20366d035e2905b50da3e8776ee> /System/Library/PrivateFrameworks/ToneKit.framework/ToneKit
0x35e62000 - 0x35e77fff  ToneLibrary armv7s  <314e3d36e4493fbaa3026b7f92f52be2> /System/Library/PrivateFrameworks/ToneLibrary.framework/ToneLibrary
0x35ec5000 - 0x35f85fff  UIFoundation armv7s  <00ae39c404023c32b393035110a9a4f9> /System/Library/PrivateFrameworks/UIFoundation.framework/UIFoundation
0x35f86000 - 0x35f9cfff  Ubiquity armv7s  <bf747b36030e3dadb0d25c67289f9b65> /System/Library/PrivateFrameworks/Ubiquity.framework/Ubiquity
0x35f9d000 - 0x35fa0fff  UserFS armv7s  <77579190a1c23bf7b2d596a9ee667e66> /System/Library/PrivateFrameworks/UserFS.framework/UserFS
0x35fb6000 - 0x36205fff  VectorKit armv7s  <08567db9172935a3a0296280e4897e93> /System/Library/PrivateFrameworks/VectorKit.framework/VectorKit
0x363a5000 - 0x363c2fff  VoiceServices armv7s  <45df838dc2d030b08bbfeb205c6b9305> /System/Library/PrivateFrameworks/VoiceServices.framework/VoiceServices
0x363e5000 - 0x3640afff  WebBookmarks armv7s  <846e49f755b43ab7b101c8a56a2d8f33> /System/Library/PrivateFrameworks/WebBookmarks.framework/WebBookmarks
0x36420000 - 0x36edafff  WebCore armv7s  <8aa6d34110cd3bde93ccfa8e6b56f4ae> /System/Library/PrivateFrameworks/WebCore.framework/WebCore
0x36edb000 - 0x36f9bfff  WebKit armv7s  <fbfcca71232a3e8190a32ba05f1f1700> /System/Library/PrivateFrameworks/WebKit.framework/WebKit
0x370d9000 - 0x370dffff  XPCKit armv7s  <94cec8e834283668b2d78bf51917b6cb> /System/Library/PrivateFrameworks/XPCKit.framework/XPCKit
0x370e0000 - 0x370e8fff  XPCObjects armv7s  <94c1f17fd60333a39fecf47051e98256> /System/Library/PrivateFrameworks/XPCObjects.framework/XPCObjects
0x3728c000 - 0x372affff  iCalendar armv7s  <1d51d0a6ba91303db54714a58eee4570> /System/Library/PrivateFrameworks/iCalendar.framework/iCalendar
0x372b4000 - 0x372f5fff  iTunesStore armv7s  <18ccf7eb84ea3fb1a19a6d48481f78dd> /System/Library/PrivateFrameworks/iTunesStore.framework/iTunesStore
0x37e47000 - 0x37e48fff  libAXSafeCategoryBundle.dylib armv7s  <b8b6203bf8493d9d8178fddbbc67e124> /usr/lib/libAXSafeCategoryBundle.dylib
0x37e49000 - 0x37e4efff  libAXSpeechManager.dylib armv7s  <d3ce1f3fc3623d02a94471d73e4ac8f9> /usr/lib/libAXSpeechManager.dylib
0x37e4f000 - 0x37e56fff  libAccessibility.dylib armv7s  <13499cacfbf13a128c6624edcd609983> /usr/lib/libAccessibility.dylib
0x3804f000 - 0x38065fff  libCRFSuite.dylib armv7s  <88d76daeba8234e789924c051a426607> /usr/lib/libCRFSuite.dylib
0x38079000 - 0x3807afff  libMobileCheckpoint.dylib armv7s  <a64358a01d513c61ada96bf6af828820> /usr/lib/libMobileCheckpoint.dylib
0x3807b000 - 0x38090fff  libMobileGestalt.dylib armv7s  <01cef4af33d43389b8a52005fe468291> /usr/lib/libMobileGestalt.dylib
0x38091000 - 0x38097fff  libMobileGestaltExtensions.dylib armv7s  <dfc8865f817a3858b4d1a9542eaaf578> /usr/lib/libMobileGestaltExtensions.dylib
0x380ae000 - 0x380affff  libSystem.B.dylib armv7s  <c612702fe67f319894598600bad61560> /usr/lib/libSystem.B.dylib
0x3811a000 - 0x38146fff  libTelephonyUtilDynamic.dylib armv7s  <4e2a592c1bcd3527b3b7b960d621f817> /usr/lib/libTelephonyUtilDynamic.dylib
0x3828f000 - 0x3829bfff  libbsm.0.dylib armv7s  <34a4b8ea80e4390ab8a146e0de95b6b1> /usr/lib/libbsm.0.dylib
0x3829c000 - 0x382a6fff  libbz2.1.0.dylib armv7s  <b246a3f7a5243be189afe4f7582042fa> /usr/lib/libbz2.1.0.dylib
0x382a7000 - 0x382f2fff  libc++.1.dylib armv7s  <18b3a243f7923c39951c97ab416ed3e6> /usr/lib/libc++.1.dylib
0x382f3000 - 0x3830dfff  libc++abi.dylib armv7s  <2e20d75c97d339a297a21de19c6a6d4b> /usr/lib/libc++abi.dylib
0x3831d000 - 0x38324fff  libcupolicy.dylib armv7s  <4540ecc8ecdb3c95ad6d0b5175d3b120> /usr/lib/libcupolicy.dylib
0x38350000 - 0x38350fff  libgcc_s.1.dylib armv7s  <3b247b24b05b31ba824fd703217be8aa> /usr/lib/libgcc_s.1.dylib
0x3836b000 - 0x38458fff  libiconv.2.dylib armv7s  <ff50709f8e04318da55e13c9096bba03> /usr/lib/libiconv.2.dylib
0x38459000 - 0x385aafff  libicucore.A.dylib armv7s  <4584cde425f43a2686bd362c62ff8d9f> /usr/lib/libicucore.A.dylib
0x385b2000 - 0x385b2fff  liblangid.dylib armv7s  <9babf315c8b739ff98c078fb894ba3c4> /usr/lib/liblangid.dylib
0x385b3000 - 0x385bdfff  liblockdown.dylib armv7s  <41f0f2cd691f3b79b7bd6a9131be7b0d> /usr/lib/liblockdown.dylib
0x385be000 - 0x385d3fff  liblzma.5.dylib armv7s  <cd6105f66a033d06afb45cf2ca3c1644> /usr/lib/liblzma.5.dylib
0x388ff000 - 0x38913fff  libmis.dylib armv7s  <03aad1b678a43337bb007fa32accfa75> /usr/lib/libmis.dylib
0x3893c000 - 0x38adbfff  libobjc.A.dylib armv7s  <0cc1bf8b5caa39fd90ca9cfc94e03fcb> /usr/lib/libobjc.A.dylib
0x38ba3000 - 0x38bb8fff  libresolv.9.dylib armv7s  <763ddffb38af3444b74501dde37a5949> /usr/lib/libresolv.9.dylib
0x38be1000 - 0x38c78fff  libsqlite3.dylib armv7s  <0cd7d6e04761365480a2078daee86959> /usr/lib/libsqlite3.dylib
0x38c79000 - 0x38cc6fff  libstdc++.6.dylib armv7s  <894bc61807683540a1d475ae8b117140> /usr/lib/libstdc++.6.dylib
0x38cc7000 - 0x38cedfff  libtidy.A.dylib armv7s  <9ac4925f9e803e48a846ae28aba6d355> /usr/lib/libtidy.A.dylib
0x38cf1000 - 0x38da4fff  libxml2.2.dylib armv7s  <810acee8bebe317492118d752643bde3> /usr/lib/libxml2.2.dylib
0x38da5000 - 0x38dc6fff  libxslt.1.dylib armv7s  <e3269cf2460835588f0f9b8f5bed13b2> /usr/lib/libxslt.1.dylib
0x38dc7000 - 0x38dd3fff  libz.1.dylib armv7s  <d14399220e74365cbf13a57859a31782> /usr/lib/libz.1.dylib
0x38dd4000 - 0x38dd8fff  libcache.dylib armv7s  <371dad0c805634ac9ad03150a7bb227d> /usr/lib/system/libcache.dylib
0x38dd9000 - 0x38de1fff  libcommonCrypto.dylib armv7s  <d4150f408ea232dc87cd4501bb5214d5> /usr/lib/system/libcommonCrypto.dylib
0x38de2000 - 0x38de6fff  libcompiler_rt.dylib armv7s  <880e0424cc4a399db4b46bda30abe7a8> /usr/lib/system/libcompiler_rt.dylib
0x38de7000 - 0x38dedfff  libcopyfile.dylib armv7s  <0f3a2d037bfc36b083aeffe1eac9d297> /usr/lib/system/libcopyfile.dylib
0x38dee000 - 0x38e27fff  libcorecrypto.dylib armv7s  <8af5878efc1a3eeb8e3ca9ec454855a8> /usr/lib/system/libcorecrypto.dylib
0x38e28000 - 0x38e3bfff  libdispatch.dylib armv7s  <4967565fe4ab3dfea12e7fdd5f858359> /usr/lib/system/libdispatch.dylib
0x38e3c000 - 0x38e3dfff  libdyld.dylib armv7s  <4df4c7a401b338fe82a2ccf79ccffc3b> /usr/lib/system/libdyld.dylib
0x38e3e000 - 0x38e3efff  libkeymgr.dylib armv7s  <2c710df49c6034d780bfef6565fbec53> /usr/lib/system/libkeymgr.dylib
0x38e3f000 - 0x38e45fff  liblaunch.dylib armv7s  <4cbb558bb18c369e8114546f8209bf4c> /usr/lib/system/liblaunch.dylib
0x38e46000 - 0x38e49fff  libmacho.dylib armv7s  <91172db864f93dcc8948bd943021fc41> /usr/lib/system/libmacho.dylib
0x38e4a000 - 0x38e4bfff  libremovefile.dylib armv7s  <f30347755ba53479bc6f7b5d0e7e2903> /usr/lib/system/libremovefile.dylib
0x38e4c000 - 0x38e59fff  libsystem_asl.dylib armv7s  <f9ac0494e76831868b220e2725ecad47> /usr/lib/system/libsystem_asl.dylib
0x38e5a000 - 0x38e5afff  libsystem_blocks.dylib armv7s  <cd492afd4dae33e08cc18b6d4bfebe1a> /usr/lib/system/libsystem_blocks.dylib
0x38e5b000 - 0x38ebdfff  libsystem_c.dylib armv7s  <1a06c713ca5a3e63ba8dc34cf453940c> /usr/lib/system/libsystem_c.dylib
0x38ebe000 - 0x38ec0fff  libsystem_configuration.dylib armv7s  <1a1bc9be539831269eb572fa4f29bb4d> /usr/lib/system/libsystem_configuration.dylib
0x38ec1000 - 0x38ec7fff  libsystem_dnssd.dylib armv7s  <7cb95a7379c4382da1332aaaae312177> /usr/lib/system/libsystem_dnssd.dylib
0x38ec8000 - 0x38ee0fff  libsystem_info.dylib armv7s  <4d68069bec1d3c6481663e452a0f126b> /usr/lib/system/libsystem_info.dylib
0x38ee1000 - 0x38ef9fff  libsystem_kernel.dylib armv7s  <71ddc20d2095384b84d5f42d20cc2474> /usr/lib/system/libsystem_kernel.dylib
0x38efa000 - 0x38f18fff  libsystem_m.dylib armv7s  <f1cc6d9397ad3e5eb351f4074e23dbe6> /usr/lib/system/libsystem_m.dylib
0x38f19000 - 0x38f2afff  libsystem_malloc.dylib armv7s  <f9f95a0430403e0b8a5fc0025d59798b> /usr/lib/system/libsystem_malloc.dylib
0x38f2b000 - 0x38f4afff  libsystem_network.dylib armv7s  <e590650d0ec332d49eeb4b2f8d7c5b39> /usr/lib/system/libsystem_network.dylib
0x38f4b000 - 0x38f52fff  libsystem_notify.dylib armv7s  <0e409b117a5535d9b293f931e2e88d8d> /usr/lib/system/libsystem_notify.dylib
0x38f53000 - 0x38f57fff  libsystem_platform.dylib armv7s  <2a845c549ab93dfc92b008de8cb1a5a8> /usr/lib/system/libsystem_platform.dylib
0x38f58000 - 0x38f5dfff  libsystem_pthread.dylib armv7s  <9d185d6eca7b373a95bd2c90294a8fc1> /usr/lib/system/libsystem_pthread.dylib
0x38f5e000 - 0x38f5ffff  libsystem_sandbox.dylib armv7s  <2cda4e839a863765a8efcf1831b5b7d3> /usr/lib/system/libsystem_sandbox.dylib
0x38f60000 - 0x38f62fff  libsystem_stats.dylib armv7s  <5650116a20cc32b693f1d314566a8fc8> /usr/lib/system/libsystem_stats.dylib
0x38f63000 - 0x38f63fff  libunwind.dylib armv7s  <6fdd98b80180359199c8f01ae5272f2a> /usr/lib/system/libunwind.dylib
0x38f64000 - 0x38f7efff  libxpc.dylib armv7s  <9590e2d66ee03ba6869abfda05ae5dc5> /usr/lib/system/libxpc.dylib
```
