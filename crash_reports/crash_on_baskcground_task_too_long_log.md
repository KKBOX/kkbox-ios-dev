```
Incident Identifier: B1D7316A-FD33-4DF0-916B-DB9ACD1D4D90
CrashReporter Key:   5014cd6a9eed070777d980dc79669e705b635bad
Hardware Model:      iPhone6,2
Process:             Facebook [13362]
Path:                /private/var/mobile/Containers/Bundle/Application/364EA2D0-A60E-4398-8706-D1BFDC02EA47/Facebook.app/Facebook
Identifier:          com.facebook.Facebook
Version:             8291884 (27.0)
Code Type:           ARM (Native)
Parent Process:      launchd [1]

Date/Time:           2015-04-03 14:02:13.306 +0800
Launch Time:         2015-04-03 14:01:30.127 +0800
OS Version:          iOS 8.2 (12D508)
Report Version:      105

Exception Type:  00000020
Exception Codes: 0x000000008badf00d
Highlighted Thread:  4

Application Specific Information:
<BKNewProcess: 0x12de5ce60; com.facebook.Facebook; pid: 13362; hostpid: -1> has active assertions beyond permitted time:
{(
    <BKProcessAssertion: 0x12dd36310> id: 7899-78F88300-DBEB-4A82-A9A4-C34702F64B09 name: Background Content Fetching (1065) process: <BKNewProcess: 0x12de5ce60; com.facebook.Facebook; pid: 13362; hostpid: -1> permittedBackgroundDuration: 30.000000 reason: backgroundContentFetching owner pid:7899 preventSuspend  preventThrottleDownUI  preventIdleSleep  preventSuspendOnSleep
)}

Elapsed total CPU time (seconds): 15.290 (user 15.290, system 0.000), 30% CPU
Elapsed application CPU time (seconds): 0.262, 1% CPU

Thread 0 name:  Dispatch queue: com.apple.main-thread
Thread 0:
0   libsystem_kernel.dylib        	0x36d7c474 mach_msg_trap + 20
1   libsystem_kernel.dylib        	0x36d7c269 mach_msg + 37
2   CoreFoundation                	0x2883c57f __CFRunLoopServiceMachPort + 143
3   CoreFoundation                	0x2883ab25 __CFRunLoopRun + 1013
4   CoreFoundation                	0x287883ad CFRunLoopRunSpecific + 473
5   CoreFoundation                	0x287881bf CFRunLoopRunInMode + 103
6   GraphicsServices              	0x2fd751fd GSEventRunModal + 133
7   UIKit                         	0x2bdf2439 UIApplicationMain + 1437
8   Facebook                      	0x0007181f 0x63000 + 59423
9   libdyld.dylib                 	0x36cc5aad start + 1

Thread 1 name:  Dispatch queue: com.apple.libdispatch-manager
Thread 1:
0   libsystem_kernel.dylib        	0x36d7c224 kevent64 + 24
1   libdispatch.dylib             	0x36cb10ed _dispatch_mgr_invoke + 277
2   libdispatch.dylib             	0x36ca5d37 _dispatch_mgr_thread + 35

Thread 2:
0   libsystem_kernel.dylib        	0x36d909c0 __workq_kernreturn + 8
1   libsystem_pthread.dylib       	0x36e0ae39 _pthread_wqthread + 789
2   libsystem_pthread.dylib       	0x36e0ab10 start_wqthread + 4

Thread 3:
0   libsystem_kernel.dylib        	0x36d7c474 mach_msg_trap + 20
1   libsystem_kernel.dylib        	0x36d7c269 mach_msg + 37
2   Facebook                      	0x000725f3 0x63000 + 62963
3   libsystem_pthread.dylib       	0x36e0ce21 _pthread_body + 137
4   libsystem_pthread.dylib       	0x36e0cd93 _pthread_start + 115
5   libsystem_pthread.dylib       	0x36e0ab1c thread_start + 4

Thread 4 name:  fbtelephonycache
Thread 4:
0   libsystem_kernel.dylib        	0x36d7c474 mach_msg_trap + 20
1   libsystem_kernel.dylib        	0x36d7c269 mach_msg + 37
2   CoreFoundation                	0x2883c57f __CFRunLoopServiceMachPort + 143
3   CoreFoundation                	0x2883ab25 __CFRunLoopRun + 1013
4   CoreFoundation                	0x287883ad CFRunLoopRunSpecific + 473
5   CoreFoundation                	0x287881bf CFRunLoopRunInMode + 103
6   Facebook                      	0x0009a6fb 0x63000 + 227067
7   Foundation                    	0x295ab687 __NSThread__main__ + 1115
8   libsystem_pthread.dylib       	0x36e0ce21 _pthread_body + 137
9   libsystem_pthread.dylib       	0x36e0cd93 _pthread_start + 115
10  libsystem_pthread.dylib       	0x36e0ab1c thread_start + 4

Thread 5 name:  AVAudioSession Notify Thread
Thread 5:
0   libsystem_kernel.dylib        	0x36d7c474 mach_msg_trap + 20
1   libsystem_kernel.dylib        	0x36d7c269 mach_msg + 37
2   CoreFoundation                	0x2883c57f __CFRunLoopServiceMachPort + 143
3   CoreFoundation                	0x2883ab25 __CFRunLoopRun + 1013
4   CoreFoundation                	0x287883ad CFRunLoopRunSpecific + 473
5   CoreFoundation                	0x287881bf CFRunLoopRunInMode + 103
6   libAVFAudio.dylib             	0x2748c3eb GenericRunLoopThread::Entry(void*) + 131
7   libAVFAudio.dylib             	0x2747e909 CAPThread::Entry(CAPThread*) + 193
8   libsystem_pthread.dylib       	0x36e0ce21 _pthread_body + 137
9   libsystem_pthread.dylib       	0x36e0cd93 _pthread_start + 115
10  libsystem_pthread.dylib       	0x36e0ab1c thread_start + 4

Thread 6 name:  com.apple.NSURLConnectionLoader
Thread 6:
0   libsystem_kernel.dylib        	0x36d7c474 mach_msg_trap + 20
1   libsystem_kernel.dylib        	0x36d7c269 mach_msg + 37
2   CoreFoundation                	0x2883c57f __CFRunLoopServiceMachPort + 143
3   CoreFoundation                	0x2883ab25 __CFRunLoopRun + 1013
4   CoreFoundation                	0x287883ad CFRunLoopRunSpecific + 473
5   CoreFoundation                	0x287881bf CFRunLoopRunInMode + 103
6   CFNetwork                     	0x2833c8f3 +[NSURLConnection(Loader) _resourceLoadLoop:] + 483
7   Foundation                    	0x295ab687 __NSThread__main__ + 1115
8   libsystem_pthread.dylib       	0x36e0ce21 _pthread_body + 137
9   libsystem_pthread.dylib       	0x36e0cd93 _pthread_start + 115
10  libsystem_pthread.dylib       	0x36e0ab1c thread_start + 4

Thread 7:
0   libsystem_kernel.dylib        	0x36d909c0 __workq_kernreturn + 8
1   libsystem_pthread.dylib       	0x36e0ae39 _pthread_wqthread + 789
2   libsystem_pthread.dylib       	0x36e0ab10 start_wqthread + 4

Thread 8 name:  com.apple.coremedia.player.async
Thread 8:
0   libsystem_kernel.dylib        	0x36d7c4c4 semaphore_wait_trap + 8
1   libdispatch.dylib             	0x36caf5db _dispatch_semaphore_wait_slow + 187
2   MediaToolbox                  	0x2a290b67 fpa_AsyncMovieControlThread + 1963
3   CoreMedia                     	0x28fe0c79 figThreadMain + 185
4   libsystem_pthread.dylib       	0x36e0ce21 _pthread_body + 137
5   libsystem_pthread.dylib       	0x36e0cd93 _pthread_start + 115
6   libsystem_pthread.dylib       	0x36e0ab1c thread_start + 4

No thread state (register information) available
Binary Images:
0x63000 - 0x1caafff Facebook armv7  <50d489f108813a0e9bd2e093de509fe6> /var/mobile/Containers/Bundle/Application/364EA2D0-A60E-4398-8706-D1BFDC02EA47/Facebook.app/Facebook
0x1fed5000 - 0x1fef8fff dyld armv7s  <d959cf6ea9b23eebac21b656a5551dab> /usr/lib/dyld
0x26f3f000 - 0x26f47fff AccessibilitySettingsLoader armv7s  <b7ae00ad3cd73534ba0365cec43b1a32> /System/Library/AccessibilityBundles/AccessibilitySettingsLoader.bundle/AccessibilitySettingsLoader
0x26fb6000 - 0x26fbffff QuickSpeak armv7s  <45d0907fcc743e76829c47375da10dcb> /System/Library/AccessibilityBundles/QuickSpeak.bundle/QuickSpeak
0x27306000 - 0x27472fff AVFoundation armv7s  <339e734775eb39b982ec0c089a752320> /System/Library/Frameworks/AVFoundation.framework/AVFoundation
0x27473000 - 0x274d1fff libAVFAudio.dylib armv7s  <ef35407264a93f56a53a36dc11b72544> /System/Library/Frameworks/AVFoundation.framework/libAVFAudio.dylib
0x274d2000 - 0x2750afff AVKit armv7s  <b32eb1e922d732f581534c647f9f1894> /System/Library/Frameworks/AVKit.framework/AVKit
0x2750b000 - 0x2750bfff Accelerate armv7s  <b170327a82973885aba35a81ee82882b> /System/Library/Frameworks/Accelerate.framework/Accelerate
0x2751c000 - 0x27737fff vImage armv7s  <e6895345bbd03617a31c13774e775579> /System/Library/Frameworks/Accelerate.framework/Frameworks/vImage.framework/vImage
0x27738000 - 0x2781efff libBLAS.dylib armv7s  <153f3233991f3c47b762ef1743c32e0a> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libBLAS.dylib
0x2781f000 - 0x27ae3fff libLAPACK.dylib armv7s  <aa5471640b8b3bb3b4dd3fad5ed697db> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libLAPACK.dylib
0x27ae4000 - 0x27af5fff libLinearAlgebra.dylib armv7s  <1ea6ed99c4863d4085eb884e9a616903> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libLinearAlgebra.dylib
0x27af6000 - 0x27b72fff libvDSP.dylib armv7s  <708711e55e7c3d67a44ca33803d225af> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libvDSP.dylib
0x27b73000 - 0x27b85fff libvMisc.dylib armv7s  <3c7e8723a7233076a6a0ff239e4c58eb> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/libvMisc.dylib
0x27b86000 - 0x27b86fff vecLib armv7s  <0a8061e9131332f28e903a478d0b6e36> /System/Library/Frameworks/Accelerate.framework/Frameworks/vecLib.framework/vecLib
0x27b87000 - 0x27baefff Accounts armv7s  <09f1e91f2f903f0d9e91ea22c584cfba> /System/Library/Frameworks/Accounts.framework/Accounts
0x27baf000 - 0x27baffff AdSupport armv7s  <76c5dc22151d3689b300971ef780e2a8> /System/Library/Frameworks/AdSupport.framework/AdSupport
0x27bb0000 - 0x27c1efff AddressBook armv7s  <fdcde0e632433fccbf0f758694bb5e40> /System/Library/Frameworks/AddressBook.framework/AddressBook
0x27c1f000 - 0x27d48fff AddressBookUI armv7s  <4a01e230f037317d82a10b210d2a292a> /System/Library/Frameworks/AddressBookUI.framework/AddressBookUI
0x27d49000 - 0x27d5bfff AssetsLibrary armv7s  <3fcc2aed41f7349089ac37874db2215a> /System/Library/Frameworks/AssetsLibrary.framework/AssetsLibrary
0x27ee1000 - 0x28155fff AudioToolbox armv7s  <c9e12d7ce99833ce969b53099cd194ba> /System/Library/Frameworks/AudioToolbox.framework/AudioToolbox
0x282c0000 - 0x28449fff CFNetwork armv7s  <d52bddca8cf330959bc40eb43e26d30d> /System/Library/Frameworks/CFNetwork.framework/CFNetwork
0x2844a000 - 0x284cbfff CloudKit armv7s  <aefe5795b29833608d0fa6ca7ffc2218> /System/Library/Frameworks/CloudKit.framework/CloudKit
0x284cc000 - 0x2852bfff CoreAudio armv7s  <bb6bc21edca73eb49d2631266ba46d82> /System/Library/Frameworks/CoreAudio.framework/CoreAudio
0x2853e000 - 0x28545fff CoreAuthentication armv7s  <15f15f4a928d306db3dad9920dfb4d64> /System/Library/Frameworks/CoreAuthentication.framework/CoreAuthentication
0x28546000 - 0x28563fff CoreBluetooth armv7s  <6ab741ad5acc333786f134f936878578> /System/Library/Frameworks/CoreBluetooth.framework/CoreBluetooth
0x28564000 - 0x2876ffff CoreData armv7s  <7415681a633b37b293ad2fbe776b3396> /System/Library/Frameworks/CoreData.framework/CoreData
0x28770000 - 0x28a9dfff CoreFoundation armv7s  <9924d9f1b68836cfa0dc45ad776a10ba> /System/Library/Frameworks/CoreFoundation.framework/CoreFoundation
0x28a9e000 - 0x28bc8fff CoreGraphics armv7s  <e50b5639d14a39d0b83449cd48707fcc> /System/Library/Frameworks/CoreGraphics.framework/CoreGraphics
0x28c08000 - 0x28c0afff libCGXType.A.dylib armv7s  <c816f78ae8113d599d194d04d501c6e1> /System/Library/Frameworks/CoreGraphics.framework/Resources/libCGXType.A.dylib
0x28c0b000 - 0x28c15fff libCMSBuiltin.A.dylib armv7s  <a5011cd35faa344d983be6e9224983c5> /System/Library/Frameworks/CoreGraphics.framework/Resources/libCMSBuiltin.A.dylib
0x28dfd000 - 0x28e18fff libRIP.A.dylib armv7s  <c2266f3ec31c38798f113d59c5102db7> /System/Library/Frameworks/CoreGraphics.framework/Resources/libRIP.A.dylib
0x28e19000 - 0x28f27fff CoreImage armv7s  <44b768c0b39c3197b6ecc5bed08cd0af> /System/Library/Frameworks/CoreImage.framework/CoreImage
0x28f28000 - 0x28f80fff CoreLocation armv7s  <27f6fb3e7d053990989c057d0876bd8b> /System/Library/Frameworks/CoreLocation.framework/CoreLocation
0x28fb2000 - 0x2904dfff CoreMedia armv7s  <3afe18bd92493ccb94501e9ffb07eb89> /System/Library/Frameworks/CoreMedia.framework/CoreMedia
0x2904e000 - 0x2912bfff CoreMotion armv7s  <c6e4968c002738f18dbf7169c232b994> /System/Library/Frameworks/CoreMotion.framework/CoreMotion
0x2912c000 - 0x2918afff CoreTelephony armv7s  <580b75cd06563449931429e0fc6dd16b> /System/Library/Frameworks/CoreTelephony.framework/CoreTelephony
0x2918b000 - 0x29257fff CoreText armv7s  <b8f896f010923eaebd88f04d9fa92db5> /System/Library/Frameworks/CoreText.framework/CoreText
0x29258000 - 0x2926dfff CoreVideo armv7s  <982f9388857d38dd98d7f4b793cefd74> /System/Library/Frameworks/CoreVideo.framework/CoreVideo
0x2926e000 - 0x29363fff EventKit armv7s  <5b8e871f5568346099b77f9e6974ef9a> /System/Library/Frameworks/EventKit.framework/EventKit
0x294db000 - 0x296defff Foundation armv7s  <c5d6421377e13c2e8bf4cbd917d874b1> /System/Library/Frameworks/Foundation.framework/Foundation
0x296df000 - 0x2970afff GLKit armv7s  <a48c06955f193a34a468beb245689cba> /System/Library/Frameworks/GLKit.framework/GLKit
0x2970b000 - 0x2972afff GSS armv7s  <3b3cf67b07403c02a0002d795ea46fbf> /System/Library/Frameworks/GSS.framework/GSS
0x297de000 - 0x29833fff IOKit armv7s  <c23ce2b864ec3c9ca424fab1145e6241> /System/Library/Frameworks/IOKit.framework/Versions/A/IOKit
0x29834000 - 0x29a76fff ImageIO armv7s  <7c57f7fe7e1c3604a8d86053c716cdf2> /System/Library/Frameworks/ImageIO.framework/ImageIO
0x29a77000 - 0x29dc5fff JavaScriptCore armv7s  <7adced020bfc37a595c0811423f9fde8> /System/Library/Frameworks/JavaScriptCore.framework/JavaScriptCore
0x29f8c000 - 0x29f92fff LocalAuthentication armv7s  <89a7fb34c0643f559c8a3999e3320be8> /System/Library/Frameworks/LocalAuthentication.framework/LocalAuthentication
0x29f93000 - 0x29fa2fff SharedUtils armv7s  <290a076038053d4dbeb715ae41c3b98a> /System/Library/Frameworks/LocalAuthentication.framework/Support/SharedUtils.framework/SharedUtils
0x29fa3000 - 0x2a0a2fff MapKit armv7s  <41e70af583b13c05bbe7d7cbe5ed3295> /System/Library/Frameworks/MapKit.framework/MapKit
0x2a0a3000 - 0x2a0abfff MediaAccessibility armv7s  <4cea4ac5cfd4303a86b331c10e7ce8a6> /System/Library/Frameworks/MediaAccessibility.framework/MediaAccessibility
0x2a0ac000 - 0x2a28bfff MediaPlayer armv7s  <3a4bc2cfd8653621ad5944b42cad8fdc> /System/Library/Frameworks/MediaPlayer.framework/MediaPlayer
0x2a28c000 - 0x2a606fff MediaToolbox armv7s  <b3cab8f05a4e32e193e5e1da502727a3> /System/Library/Frameworks/MediaToolbox.framework/MediaToolbox
0x2a607000 - 0x2a6c5fff MessageUI armv7s  <f41ef6b613fc32f8b1291ca26327e403> /System/Library/Frameworks/MessageUI.framework/MessageUI
0x2a6c6000 - 0x2a732fff Metal armv7s  <00a89ca37d0834898a8e81b76357fcd1> /System/Library/Frameworks/Metal.framework/Metal
0x2a733000 - 0x2a7c6fff MobileCoreServices armv7s  <01ea079439393391821634683ccb412a> /System/Library/Frameworks/MobileCoreServices.framework/MobileCoreServices
0x2a8bd000 - 0x2a8eefff OpenAL armv7s  <6e48355ff76d31c480f0ad0caed1daf7> /System/Library/Frameworks/OpenAL.framework/OpenAL
0x2b2a4000 - 0x2b2acfff OpenGLES armv7s  <02aa6ebd148a3449a5fb73365d45ea87> /System/Library/Frameworks/OpenGLES.framework/OpenGLES
0x2b2ae000 - 0x2b2aefff libCVMSPluginSupport.dylib armv7s  <18deef6946913a25969285502316b674> /System/Library/Frameworks/OpenGLES.framework/libCVMSPluginSupport.dylib
0x2b2af000 - 0x2b2b1fff libCoreFSCache.dylib armv7s  <d252e7981a543f6580ba951a037d2590> /System/Library/Frameworks/OpenGLES.framework/libCoreFSCache.dylib
0x2b2b2000 - 0x2b2b5fff libCoreVMClient.dylib armv7s  <405d9912249e3a3a9e60bfb9edcb19d8> /System/Library/Frameworks/OpenGLES.framework/libCoreVMClient.dylib
0x2b2b6000 - 0x2b2befff libGFXShared.dylib armv7s  <248be3146cab319292bd30806d11f792> /System/Library/Frameworks/OpenGLES.framework/libGFXShared.dylib
0x2b2bf000 - 0x2b301fff libGLImage.dylib armv7s  <1e9a22a10aa2364ebdcb68d7f8a153e6> /System/Library/Frameworks/OpenGLES.framework/libGLImage.dylib
0x2b45c000 - 0x2b50dfff PassKit armv7s  <04b0fadfdf4232ab8260b9424c148efa> /System/Library/Frameworks/PassKit.framework/PassKit
0x2b50e000 - 0x2b582fff Photos armv7s  <af5cfd98f67230bd95fac97fcf18b391> /System/Library/Frameworks/Photos.framework/Photos
0x2b7aa000 - 0x2b8fcfff QuartzCore armv7s  <87f4d4be177f3a9c8c251da670dcf45b> /System/Library/Frameworks/QuartzCore.framework/QuartzCore
0x2b8fd000 - 0x2b950fff QuickLook armv7s  <ab076503a86c325f80e2c50d05ffe636> /System/Library/Frameworks/QuickLook.framework/QuickLook
0x2bb3f000 - 0x2bb80fff Security armv7s  <7434b508472d355298f93d5e8a5e725e> /System/Library/Frameworks/Security.framework/Security
0x2bb81000 - 0x2bbf9fff Social armv7s  <50be0cb4d06334999dbe7a505708bb84> /System/Library/Frameworks/Social.framework/Social
0x2bd0e000 - 0x2bd23fff StoreKit armv7s  <7b8c6b8baee035b18e2d0b09ac219d1d> /System/Library/Frameworks/StoreKit.framework/StoreKit
0x2bd24000 - 0x2bd81fff SystemConfiguration armv7s  <53b886e1d32f32a2b6c0cb2e0d93ccf2> /System/Library/Frameworks/SystemConfiguration.framework/SystemConfiguration
0x2bd84000 - 0x2c62afff UIKit armv7s  <81da17cbaad83ba5acaa630781446352> /System/Library/Frameworks/UIKit.framework/UIKit
0x2c62b000 - 0x2c692fff VideoToolbox armv7s  <867315c8522039d3aeb629502059f6d6> /System/Library/Frameworks/VideoToolbox.framework/VideoToolbox
0x2c6c2000 - 0x2c8a8fff WebKit armv7s  <8fd2d7a7389c3efa89700e371946577f> /System/Library/Frameworks/WebKit.framework/WebKit
0x2cb99000 - 0x2cba4fff AOSNotification armv7s  <d3e35dcc6918381aa0745eedc97f80a4> /System/Library/PrivateFrameworks/AOSNotification.framework/AOSNotification
0x2cc1c000 - 0x2cc41fff AXRuntime armv7s  <58248fce427e3c21aef462b1d2d02956> /System/Library/PrivateFrameworks/AXRuntime.framework/AXRuntime
0x2cc6d000 - 0x2cc74fff AccessibilityUI armv7s  <3db469f259e63bb2ad1ed65678ee3127> /System/Library/PrivateFrameworks/Accessibility.framework/Frameworks/AccessibilityUI.framework/AccessibilityUI
0x2cc8c000 - 0x2cc8ffff AccessibilityUIUtilities armv7s  <ce7636c546db3e278945bc13b84bbaaa> /System/Library/PrivateFrameworks/Accessibility.framework/Frameworks/AccessibilityUIUtilities.framework/AccessibilityUIUtilities
0x2cc92000 - 0x2cc95fff ZoomServices armv7s  <57a9fb8122bd381ebdbd23cb59087649> /System/Library/PrivateFrameworks/Accessibility.framework/Frameworks/ZoomServices.framework/ZoomServices
0x2cc96000 - 0x2ccf0fff AccessibilityUtilities armv7s  <0baabefefc0d38d182e271f978422b6b> /System/Library/PrivateFrameworks/AccessibilityUtilities.framework/AccessibilityUtilities
0x2ccfe000 - 0x2cd4efff AccountsDaemon armv7s  <65fd994bdc603127acb44c690381e141> /System/Library/PrivateFrameworks/AccountsDaemon.framework/AccountsDaemon
0x2cd4f000 - 0x2cd70fff AccountsUI armv7s  <88a4f07ebb583241b94abaa4db1856a4> /System/Library/PrivateFrameworks/AccountsUI.framework/AccountsUI
0x2cd71000 - 0x2cd75fff AggregateDictionary armv7s  <1f4aae1e74f230488125530b315d01f6> /System/Library/PrivateFrameworks/AggregateDictionary.framework/AggregateDictionary
0x2cf3e000 - 0x2cf69fff AirPlaySupport armv7s  <c727539915aa32bc80dd90711c9d309c> /System/Library/PrivateFrameworks/AirPlaySupport.framework/AirPlaySupport
0x2d16f000 - 0x2d1adfff AppSupport armv7s  <baba5ec9cb6c3b5e8097412705a57bd7> /System/Library/PrivateFrameworks/AppSupport.framework/AppSupport
0x2d1ae000 - 0x2d1f6fff AppleAccount armv7s  <018fcce59c923c9ab9e2ea148fe1f2a4> /System/Library/PrivateFrameworks/AppleAccount.framework/AppleAccount
0x2d2ce000 - 0x2d2defff AppleIDSSOAuthentication armv7s  <9c9def717ceb373ea121b3e49d116078> /System/Library/PrivateFrameworks/AppleIDSSOAuthentication.framework/AppleIDSSOAuthentication
0x2d2df000 - 0x2d31cfff AppleJPEG armv7s  <7ccf7d8822283911a26b7500b3b437b2> /System/Library/PrivateFrameworks/AppleJPEG.framework/AppleJPEG
0x2d327000 - 0x2d339fff ApplePushService armv7s  <32ae1ae398cc3f0e9ec0426ca0870d34> /System/Library/PrivateFrameworks/ApplePushService.framework/ApplePushService
0x2d33a000 - 0x2d340fff AppleSRP armv7s  <8b69881099043bf2bd8b3ce229ad1b32> /System/Library/PrivateFrameworks/AppleSRP.framework/AppleSRP
0x2d375000 - 0x2d37efff AssertionServices armv7s  <f55c714a5a013c528dfbe7c0d1689b91> /System/Library/PrivateFrameworks/AssertionServices.framework/AssertionServices
0x2d37f000 - 0x2d397fff AssetsLibraryServices armv7s  <2eeb5c26a5243db08a8fc5ae68d154e7> /System/Library/PrivateFrameworks/AssetsLibraryServices.framework/AssetsLibraryServices
0x2d398000 - 0x2d3cefff AssistantServices armv7s  <4231dc07a0b33cebaec3320a6ac7cf88> /System/Library/PrivateFrameworks/AssistantServices.framework/AssistantServices
0x2d3fe000 - 0x2d402fff BTLEAudioController armv7s  <09bd58dd309a35fcbd0797970b740e6b> /System/Library/PrivateFrameworks/BTLEAudioController.framework/BTLEAudioController
0x2d403000 - 0x2d41afff BackBoardServices armv7s  <205d23f721fd35e584853e2d487af9b3> /System/Library/PrivateFrameworks/BackBoardServices.framework/BackBoardServices
0x2d41d000 - 0x2d452fff BaseBoard armv7s  <d780982e45423486ab6b6b6ad21abd23> /System/Library/PrivateFrameworks/BaseBoard.framework/BaseBoard
0x2d453000 - 0x2d459fff BluetoothManager armv7s  <753fc00efc263cd9a98eb33ac6898fc5> /System/Library/PrivateFrameworks/BluetoothManager.framework/BluetoothManager
0x2d45a000 - 0x2d480fff Bom armv7s  <e4cf22bce7a53599bcccdaf89140fdc0> /System/Library/PrivateFrameworks/Bom.framework/Bom
0x2d552000 - 0x2d559fff CacheDelete armv7s  <796e062ad6f13a1db26efcc99360cefc> /System/Library/PrivateFrameworks/CacheDelete.framework/CacheDelete
0x2d59e000 - 0x2d5c7fff CalendarFoundation armv7s  <98f804e98b9a3cd8b8db61afb60fa766> /System/Library/PrivateFrameworks/CalendarFoundation.framework/CalendarFoundation
0x2d6de000 - 0x2d6e6fff CaptiveNetwork armv7s  <10fccd2e42fe35919ec80d881c735054> /System/Library/PrivateFrameworks/CaptiveNetwork.framework/CaptiveNetwork
0x2d6e7000 - 0x2d809fff Celestial armv7s  <9738f5492d733592a8c0e70100d4fe8b> /System/Library/PrivateFrameworks/Celestial.framework/Celestial
0x2d817000 - 0x2d82ffff CertInfo armv7s  <81941a4b91e435e987daf432e43faef3> /System/Library/PrivateFrameworks/CertInfo.framework/CertInfo
0x2d830000 - 0x2d835fff CertUI armv7s  <8f7c4dfd3aca3ef789dbb4551f65a122> /System/Library/PrivateFrameworks/CertUI.framework/CertUI
0x2d972000 - 0x2d993fff ChunkingLibrary armv7s  <4b029225b4033493aa979657e5503a16> /System/Library/PrivateFrameworks/ChunkingLibrary.framework/ChunkingLibrary
0x2dd42000 - 0x2ddedfff CloudPhotoLibrary armv7s  <d0d018f2c56130dc8ed830e5acd55c42> /System/Library/PrivateFrameworks/CloudPhotoLibrary.framework/CloudPhotoLibrary
0x2de3e000 - 0x2de40fff CommonAuth armv7s  <592c71c8a6fe3c1a880313f6f345137a> /System/Library/PrivateFrameworks/CommonAuth.framework/CommonAuth
0x2de41000 - 0x2de51fff CommonUtilities armv7s  <f3e078abb75f3788b1d2b2a564eb786e> /System/Library/PrivateFrameworks/CommonUtilities.framework/CommonUtilities
0x2de52000 - 0x2de56fff CommunicationsFilter armv7s  <e7342616fbc730888bf4c6db4ccaa4e2> /System/Library/PrivateFrameworks/CommunicationsFilter.framework/CommunicationsFilter
0x2df17000 - 0x2df1bfff ConstantClasses armv7s  <44624a0f6f633645a00a3cf9a30b4c23> /System/Library/PrivateFrameworks/ConstantClasses.framework/ConstantClasses
0x2df1c000 - 0x2df54fff ContentIndex armv7s  <c76c9897423c3a4cb583f0c28ff83f3a> /System/Library/PrivateFrameworks/ContentIndex.framework/ContentIndex
0x2df55000 - 0x2df58fff CoreAUC armv7s  <a9b4259de0743f539097e0a709ae357b> /System/Library/PrivateFrameworks/CoreAUC.framework/CoreAUC
0x2df85000 - 0x2dfd9fff CoreDAV armv7s  <fe5b9ce965bd3b2093c337b03be22d1d> /System/Library/PrivateFrameworks/CoreDAV.framework/CoreDAV
0x2dfda000 - 0x2dffafff CoreDuet armv7s  <b44ee1884aa53ad4be7a2ad93a7ca163> /System/Library/PrivateFrameworks/CoreDuet.framework/CoreDuet
0x2e000000 - 0x2e010fff CoreDuetDaemonProtocol armv7s  <589ab858f78e3372bbfc55036a1f372a> /System/Library/PrivateFrameworks/CoreDuetDaemonProtocol.framework/CoreDuetDaemonProtocol
0x2e017000 - 0x2e019fff CoreDuetDebugLogging armv7s  <a54577d565b83022ad7da2e3c5a5a8f1> /System/Library/PrivateFrameworks/CoreDuetDebugLogging.framework/CoreDuetDebugLogging
0x2e168000 - 0x2e268fff CoreMediaStream armv7s  <ffce49555072331aa985ad018b0b79ef> /System/Library/PrivateFrameworks/CoreMediaStream.framework/CoreMediaStream
0x2e269000 - 0x2e304fff CorePDF armv7s  <93e4925313663cbf970beb535d829f1b> /System/Library/PrivateFrameworks/CorePDF.framework/CorePDF
0x2e365000 - 0x2e36ffff CoreRecents armv7s  <7f91e59aaab938f8a9a7677f40b5a1de> /System/Library/PrivateFrameworks/CoreRecents.framework/CoreRecents
0x2e370000 - 0x2e3c8fff CoreRecognition armv7s  <d31dcee34a803c7f8e1983e75294eed3> /System/Library/PrivateFrameworks/CoreRecognition.framework/CoreRecognition
0x2e3e7000 - 0x2e405fff CoreServicesInternal armv7s  <ecd55295b62d386c90ae564978272244> /System/Library/PrivateFrameworks/CoreServicesInternal.framework/CoreServicesInternal
0x2e54e000 - 0x2e5bdfff CoreSymbolication armv7s  <35cfbe3c7beb36438ddbe04a32176b37> /System/Library/PrivateFrameworks/CoreSymbolication.framework/CoreSymbolication
0x2e605000 - 0x2e685fff CoreUI armv7s  <9c6bc8f4fa783388b8c09596402bd051> /System/Library/PrivateFrameworks/CoreUI.framework/CoreUI
0x2e686000 - 0x2e6f0fff CoreUtils armv7s  <8054b84ba8a33c148b9689dd3f88aac6> /System/Library/PrivateFrameworks/CoreUtils.framework/CoreUtils
0x2e6f1000 - 0x2e6f6fff CrashReporterSupport armv7s  <95aaae72f9fc3756b5bca68275245fab> /System/Library/PrivateFrameworks/CrashReporterSupport.framework/CrashReporterSupport
0x2e6f7000 - 0x2e6fcfff DAAPKit armv7s  <e682b52c60cc310f98d26a84c01832e3> /System/Library/PrivateFrameworks/DAAPKit.framework/DAAPKit
0x2e6fd000 - 0x2e707fff DCIMServices armv7s  <6da01edd76dc3d448f02f74114c16de6> /System/Library/PrivateFrameworks/DCIMServices.framework/DCIMServices
0x2e708000 - 0x2e74dfff DataAccess armv7s  <fb7cb30047e43d408f3d35837346d1e4> /System/Library/PrivateFrameworks/DataAccess.framework/DataAccess
0x2e93f000 - 0x2e960fff DataAccessExpress armv7s  <d0e876ba262039c5ab0f92acb607eb12> /System/Library/PrivateFrameworks/DataAccessExpress.framework/DataAccessExpress
0x2e96b000 - 0x2e982fff DataDetectorsCore armv7s  <ea197848c06839fe99069b46f98de1f7> /System/Library/PrivateFrameworks/DataDetectorsCore.framework/DataDetectorsCore
0x2e99e000 - 0x2e9a4fff DataMigration armv7s  <94908a4ed7b034ac8e30c1cae7e4feab> /System/Library/PrivateFrameworks/DataMigration.framework/DataMigration
0x2e9ae000 - 0x2e9affff DiagnosticLogCollection armv7s  <9d84890eefe132548cfeaf1f411dcb82> /System/Library/PrivateFrameworks/DiagnosticLogCollection.framework/DiagnosticLogCollection
0x2e9b0000 - 0x2e9cafff DictionaryServices armv7s  <e13a04ece193371f8dc1bea3773fb476> /System/Library/PrivateFrameworks/DictionaryServices.framework/DictionaryServices
0x2e9e9000 - 0x2ea08fff EAP8021X armv7s  <15c71e8b120b3b6c9c232de6dff4055c> /System/Library/PrivateFrameworks/EAP8021X.framework/EAP8021X
0x2eb12000 - 0x2eb14fff FTClientServices armv7s  <44c007f094e23516ad440dccc4b8448e> /System/Library/PrivateFrameworks/FTClientServices.framework/FTClientServices
0x2eb15000 - 0x2eb44fff FTServices armv7s  <5a7e144ef33837359ed59ab12f752df2> /System/Library/PrivateFrameworks/FTServices.framework/FTServices
0x2eb45000 - 0x2ef68fff FaceCore armv7s  <9731ccedf231369ba7da37d22b807286> /System/Library/PrivateFrameworks/FaceCore.framework/FaceCore
0x2ef7b000 - 0x2ef86fff FindMyDevice armv7s  <67de2557174d3b629f56aad569607cb9> /System/Library/PrivateFrameworks/FindMyDevice.framework/FindMyDevice
0x2efd1000 - 0x2efd1fff FontServices armv7s  <68f4057f3bc0315088973b27f3183287> /System/Library/PrivateFrameworks/FontServices.framework/FontServices
0x2efd2000 - 0x2f0a7fff libFontParser.dylib armv7s  <63d94b0574793ee9998ce725153a6f54> /System/Library/PrivateFrameworks/FontServices.framework/libFontParser.dylib
0x2f0c6000 - 0x2f0d5fff libGSFontCache.dylib armv7s  <d9ec77b2334a3eacba956a7343890a8a> /System/Library/PrivateFrameworks/FontServices.framework/libGSFontCache.dylib
0x2f1b9000 - 0x2f1d4fff FrontBoardServices armv7s  <6a0538ed555e396bbecb9183314da66a> /System/Library/PrivateFrameworks/FrontBoardServices.framework/FrontBoardServices
0x2fabd000 - 0x2fad3fff GenerationalStorage armv7s  <39f3748c838438a3842b9a5ec5b6c62b> /System/Library/PrivateFrameworks/GenerationalStorage.framework/GenerationalStorage
0x2fad4000 - 0x2fd6bfff GeoServices armv7s  <d5489b4c67a93e3680f902bc1955cfc0> /System/Library/PrivateFrameworks/GeoServices.framework/GeoServices
0x2fd6c000 - 0x2fd7cfff GraphicsServices armv7s  <c552c842c10431dfba8141dffd9beb52> /System/Library/PrivateFrameworks/GraphicsServices.framework/GraphicsServices
0x2fe9e000 - 0x2feedfff Heimdal armv7s  <89658112834d3b0b99406319de0ddc69> /System/Library/PrivateFrameworks/Heimdal.framework/Heimdal
0x2ffb6000 - 0x3003afff HomeSharing armv7s  <3c8e2e302d0339558b66726ed22436cf> /System/Library/PrivateFrameworks/HomeSharing.framework/HomeSharing
0x30099000 - 0x300f8fff IDS armv7s  <c48a45fd39883c57b85320bd7a70668e> /System/Library/PrivateFrameworks/IDS.framework/IDS
0x300f9000 - 0x30124fff IDSFoundation armv7s  <ed2d37f9d0bf39818eb7d200b0244dd2> /System/Library/PrivateFrameworks/IDSFoundation.framework/IDSFoundation
0x302d8000 - 0x3033cfff IMFoundation armv7s  <a4ac416845d838d4ad7505a4e4c9dca4> /System/Library/PrivateFrameworks/IMFoundation.framework/IMFoundation
0x30344000 - 0x30347fff IOAccelerator armv7s  <a92bf62bbbc837f5959f8d2d0eee3afd> /System/Library/PrivateFrameworks/IOAccelerator.framework/IOAccelerator
0x3034a000 - 0x30350fff IOMobileFramebuffer armv7s  <3f879ba3ba5633a1b111433ba49bc775> /System/Library/PrivateFrameworks/IOMobileFramebuffer.framework/IOMobileFramebuffer
0x30351000 - 0x30356fff IOSurface armv7s  <4bf18307118c3bc3a9917d3f773988d3> /System/Library/PrivateFrameworks/IOSurface.framework/IOSurface
0x30357000 - 0x30358fff IOSurfaceAccelerator armv7s  <f11999f87f65300a92f60c505dac2986> /System/Library/PrivateFrameworks/IOSurfaceAccelerator.framework/IOSurfaceAccelerator
0x30359000 - 0x303a0fff ITMLKit armv7s  <4f22c3313c6b336aa562abd42c4a03b1> /System/Library/PrivateFrameworks/ITMLKit.framework/ITMLKit
0x303f4000 - 0x303fafff IntlPreferences armv7s  <accfbce79e30359180248894c678cc6d> /System/Library/PrivateFrameworks/IntlPreferences.framework/IntlPreferences
0x303fb000 - 0x30431fff LanguageModeling armv7s  <65b32d31324e3a2daf3fd629a907a389> /System/Library/PrivateFrameworks/LanguageModeling.framework/LanguageModeling
0x304ac000 - 0x304e8fff MIME armv7s  <98740de075f53965a75c73b437df39e5> /System/Library/PrivateFrameworks/MIME.framework/MIME
0x304e9000 - 0x30543fff MMCS armv7s  <bbe274f8019036c2bcfd1422eb2470ba> /System/Library/PrivateFrameworks/MMCS.framework/MMCS
0x30592000 - 0x3059efff MailServices armv7s  <910aeb57c97631f6a6b2caea7ce975a6> /System/Library/PrivateFrameworks/MailServices.framework/MailServices
0x305d1000 - 0x30670fff ManagedConfiguration armv7s  <67a0f6f150623b4e88b16fc847338994> /System/Library/PrivateFrameworks/ManagedConfiguration.framework/ManagedConfiguration
0x3067f000 - 0x30680fff Marco armv7s  <183a524e828a38a1967db1cebf9641c7> /System/Library/PrivateFrameworks/Marco.framework/Marco
0x30681000 - 0x306f9fff MediaControlSender armv7s  <309c78d824963df3a7de696b6232e781> /System/Library/PrivateFrameworks/MediaControlSender.framework/MediaControlSender
0x30794000 - 0x307a8fff MediaRemote armv7s  <bc65fb89e4a73ff0934f0e482147f4ee> /System/Library/PrivateFrameworks/MediaRemote.framework/MediaRemote
0x307a9000 - 0x307bbfff MediaServices armv7s  <1b8727bbee1b3a4285f0bfefa8de9995> /System/Library/PrivateFrameworks/MediaServices.framework/MediaServices
0x307bc000 - 0x307d4fff MediaStream armv7s  <27d170e32e813a3ab2efbb9fccc63d21> /System/Library/PrivateFrameworks/MediaStream.framework/MediaStream
0x30839000 - 0x30916fff Message armv7s  <57ac6626694e34e691ff3cec4692f104> /System/Library/PrivateFrameworks/Message.framework/Message
0x3091c000 - 0x3091efff MessageSupport armv7s  <b4d71df6d17f355293c2a1d9ecc45a6b> /System/Library/PrivateFrameworks/MessageSupport.framework/MessageSupport
0x30930000 - 0x3093dfff MobileAsset armv7s  <c0713b14b8b4330cb3bc8485a3b4b422> /System/Library/PrivateFrameworks/MobileAsset.framework/MobileAsset
0x30964000 - 0x3096dfff MobileBluetooth armv7s  <154a9ed4a113346ebf210ace642cffd3> /System/Library/PrivateFrameworks/MobileBluetooth.framework/MobileBluetooth
0x30989000 - 0x30991fff MobileIcons armv7s  <004226de78993bd6b86fdcdb46623677> /System/Library/PrivateFrameworks/MobileIcons.framework/MobileIcons
0x30992000 - 0x30999fff MobileInstallation armv7s  <4df1bd55e76c3145afcd9b3abbec22ac> /System/Library/PrivateFrameworks/MobileInstallation.framework/MobileInstallation
0x3099a000 - 0x309a6fff MobileKeyBag armv7s  <08adbb5fd2ef39f1bfbfa41f1eb54c1e> /System/Library/PrivateFrameworks/MobileKeyBag.framework/MobileKeyBag
0x309d3000 - 0x309d6fff MobileSystemServices armv7s  <ca2adcf15ce834c989c4af2bc7c4ae79> /System/Library/PrivateFrameworks/MobileSystemServices.framework/MobileSystemServices
0x309f8000 - 0x30a05fff MobileWiFi armv7s  <36db4443a9913096a55123474eaaa027> /System/Library/PrivateFrameworks/MobileWiFi.framework/MobileWiFi
0x30a44000 - 0x30c13fff MusicLibrary armv7s  <074c7ae7112d3dd3a18b4adf8cb1d0f1> /System/Library/PrivateFrameworks/MusicLibrary.framework/MusicLibrary
0x30d91000 - 0x30d9ffff NanoPreferencesSync armv7s  <3e3baa16a90c3da0b208c47748c36100> /System/Library/PrivateFrameworks/NanoPreferencesSync.framework/NanoPreferencesSync
0x30da0000 - 0x30db3fff NanoRegistry armv7s  <74f4c04e9cc033639828b558f3b1b505> /System/Library/PrivateFrameworks/NanoRegistry.framework/NanoRegistry
0x30dc8000 - 0x30dcdfff Netrb armv7s  <c7112f327abf3d48aaedc024a5ca2c33> /System/Library/PrivateFrameworks/Netrb.framework/Netrb
0x30dce000 - 0x30dd4fff NetworkStatistics armv7s  <457ccadd5a1b3dada1f9236340e33c61> /System/Library/PrivateFrameworks/NetworkStatistics.framework/NetworkStatistics
0x30dd5000 - 0x30df2fff Notes armv7s  <14da6174e1113217a97451d5881adcd7> /System/Library/PrivateFrameworks/Notes.framework/Notes
0x30df3000 - 0x30df7fff NotificationsUI armv7s  <aedee5a1658b332793115d09573fbccd> /System/Library/PrivateFrameworks/NotificationsUI.framework/NotificationsUI
0x30df8000 - 0x30dfafff OAuth armv7s  <31e2239fe22c393d81b0a9453fe5fcb3> /System/Library/PrivateFrameworks/OAuth.framework/OAuth
0x31555000 - 0x31591fff OpenCL armv7s  <880c2f8b8b3e3aa5bb847a5bf5feedcc> /System/Library/PrivateFrameworks/OpenCL.framework/OpenCL
0x3164d000 - 0x316c5fff PassKitCore armv7s  <fc863e1eab963588b18cc1c4a8938e24> /System/Library/PrivateFrameworks/PassKitCore.framework/PassKitCore
0x316c6000 - 0x316edfff PersistentConnection armv7s  <be24a077478c369fb6a60c13387e6504> /System/Library/PrivateFrameworks/PersistentConnection.framework/PersistentConnection
0x31858000 - 0x31ac4fff PhotoLibraryServices armv7s  <3ea4f407a5483844a835853ef7c6e122> /System/Library/PrivateFrameworks/PhotoLibraryServices.framework/PhotoLibraryServices
0x31ac5000 - 0x31acefff PhotosFormats armv7s  <3f11a79e1c41381ca136377572994d38> /System/Library/PrivateFrameworks/PhotosFormats.framework/PhotosFormats
0x31acf000 - 0x31b19fff PhysicsKit armv7s  <04636f225d533b12ada3ec7c8091af69> /System/Library/PrivateFrameworks/PhysicsKit.framework/PhysicsKit
0x31b1a000 - 0x31b30fff PlugInKit armv7s  <2d06163c8f53393ea165c280538e23d7> /System/Library/PrivateFrameworks/PlugInKit.framework/PlugInKit
0x31b31000 - 0x31b38fff PowerLog armv7s  <5c785e14cb1e33ed8a77ab49962bbd86> /System/Library/PrivateFrameworks/PowerLog.framework/PowerLog
0x31d3f000 - 0x31deefff Preferences armv7s  <5a06c9b79248347f8e2685c3a7039852> /System/Library/PrivateFrameworks/Preferences.framework/Preferences
0x31def000 - 0x31e2cfff PrintKit armv7s  <939d13136b4b3189bbfd9b4cecdbf4a8> /System/Library/PrivateFrameworks/PrintKit.framework/PrintKit
0x31e2d000 - 0x31e30fff ProgressUI armv7s  <2fbdf50f0abe3b499f7c0a1a536a4824> /System/Library/PrivateFrameworks/ProgressUI.framework/ProgressUI
0x31e31000 - 0x31ec5fff ProofReader armv7s  <325b83e61c11377fb73dc906d1290cc2> /System/Library/PrivateFrameworks/ProofReader.framework/ProofReader
0x31ec6000 - 0x31ed5fff ProtectedCloudStorage armv7s  <df6216ff5f543d67ac8f750db7ff2ea2> /System/Library/PrivateFrameworks/ProtectedCloudStorage.framework/ProtectedCloudStorage
0x31ed6000 - 0x31ee2fff ProtocolBuffer armv7s  <07b1f39a87a4328998ca0c0e45a99128> /System/Library/PrivateFrameworks/ProtocolBuffer.framework/ProtocolBuffer
0x31ee3000 - 0x31f14fff PrototypeTools armv7s  <190a865020a930bd9091a5992fbfb710> /System/Library/PrivateFrameworks/PrototypeTools.framework/PrototypeTools
0x31f17000 - 0x31f86fff Quagga armv7s  <243d25491bf73018aa0aa5238fbb6541> /System/Library/PrivateFrameworks/Quagga.framework/Quagga
0x3219c000 - 0x321d8fff RemoteUI armv7s  <01cce3fc80e13ed7a88ae57e7d7dd485> /System/Library/PrivateFrameworks/RemoteUI.framework/RemoteUI
0x32274000 - 0x32309fff SAObjects armv7s  <45378b9b1ac2318b96c95ea98f8a5399> /System/Library/PrivateFrameworks/SAObjects.framework/SAObjects
0x32316000 - 0x32336fff ScreenReaderCore armv7s  <4155dd811eab33e8bd1408c6b6554be8> /System/Library/PrivateFrameworks/ScreenReaderCore.framework/ScreenReaderCore
0x324c9000 - 0x324fbfff SpringBoardFoundation armv7s  <a78b5c0ab81331da84dacd9445662cee> /System/Library/PrivateFrameworks/SpringBoardFoundation.framework/SpringBoardFoundation
0x324fc000 - 0x32516fff SpringBoardServices armv7s  <67583143a2da3e69885b4f987366ab5a> /System/Library/PrivateFrameworks/SpringBoardServices.framework/SpringBoardServices
0x3252a000 - 0x3254dfff SpringBoardUIServices armv7s  <b49f8accf605333d937712e42e4e710d> /System/Library/PrivateFrameworks/SpringBoardUIServices.framework/SpringBoardUIServices
0x325bb000 - 0x32887fff StoreKitUI armv7s  <b6920a27e76c3b0a9f9318bfaf6452d2> /System/Library/PrivateFrameworks/StoreKitUI.framework/StoreKitUI
0x32888000 - 0x329b4fff StoreServices armv7s  <8a5a058ba2003680b4bfc8f34a061adf> /System/Library/PrivateFrameworks/StoreServices.framework/StoreServices
0x32a85000 - 0x32a87fff TCC armv7s  <80a05e480ea53d26a4f31123beed8afb> /System/Library/PrivateFrameworks/TCC.framework/TCC
0x32a95000 - 0x32adafff TelephonyUI armv7s  <ec6a1019e9ae36b297c9bbabce4bb459> /System/Library/PrivateFrameworks/TelephonyUI.framework/TelephonyUI
0x32adb000 - 0x32b20fff TelephonyUtilities armv7s  <997799acc591303b94e3802ed95a1343> /System/Library/PrivateFrameworks/TelephonyUtilities.framework/TelephonyUtilities
0x336e4000 - 0x3370cfff TextInput armv7s  <c1ca800641f335a19e817ed327e38246> /System/Library/PrivateFrameworks/TextInput.framework/TextInput
0x3370d000 - 0x3371ffff TextToSpeech armv7s  <8208dd231f8e3e8cb75d812b944f6173> /System/Library/PrivateFrameworks/TextToSpeech.framework/TextToSpeech
0x33795000 - 0x337c8fff UIAccessibility armv7s  <a5b7dfecdff535eeb2e5c1cfb4193536> /System/Library/PrivateFrameworks/UIAccessibility.framework/UIAccessibility
0x337c9000 - 0x3388bfff UIFoundation armv7s  <f26ac9e2df5b3b0ca86df5b79f3bd7f8> /System/Library/PrivateFrameworks/UIFoundation.framework/UIFoundation
0x338b2000 - 0x338b5fff UserFS armv7s  <23943f5513333bdfafe65297f2f08e55> /System/Library/PrivateFrameworks/UserFS.framework/UserFS
0x338ce000 - 0x33e08fff VectorKit armv7s  <f1f2d6a1634e36909227aedea517c798> /System/Library/PrivateFrameworks/VectorKit.framework/VectorKit
0x3402e000 - 0x3404cfff VoiceServices armv7s  <1fb46eee95e33905868312efa8070869> /System/Library/PrivateFrameworks/VoiceServices.framework/VoiceServices
0x340e5000 - 0x3410bfff WebBookmarks armv7s  <e5f0f8d22b673f448c28cc04817e21d6> /System/Library/PrivateFrameworks/WebBookmarks.framework/WebBookmarks
0x34121000 - 0x34ca1fff WebCore armv7s  <e9f5aaac26533d58845011802e906140> /System/Library/PrivateFrameworks/WebCore.framework/WebCore
0x34ca2000 - 0x34d60fff WebKitLegacy armv7s  <6d126fab646433b5b70e53fca3bc3909> /System/Library/PrivateFrameworks/WebKitLegacy.framework/WebKitLegacy
0x34f0a000 - 0x34f10fff XPCKit armv7s  <43d8bffab44734feacc683811255e619> /System/Library/PrivateFrameworks/XPCKit.framework/XPCKit
0x34f11000 - 0x34f19fff XPCObjects armv7s  <3db1acbcfc593ad982187ae0af01d993> /System/Library/PrivateFrameworks/XPCObjects.framework/XPCObjects
0x35104000 - 0x35128fff iCalendar armv7s  <126f92193371363ea4ca89295c5465ae> /System/Library/PrivateFrameworks/iCalendar.framework/iCalendar
0x35148000 - 0x35183fff iTunesStore armv7s  <e37d5437eb8c3ee8ba8fe110e5df39cf> /System/Library/PrivateFrameworks/iTunesStore.framework/iTunesStore
0x35184000 - 0x35305fff iTunesStoreUI armv7s  <081c3e6e4b9d3356a9dad2cfe8a501ad> /System/Library/PrivateFrameworks/iTunesStoreUI.framework/iTunesStoreUI
0x359d8000 - 0x359d9fff libAXSafeCategoryBundle.dylib armv7s  <db0e24b4d9ad3847b99b1165d3697c1a> /usr/lib/libAXSafeCategoryBundle.dylib
0x359da000 - 0x359e0fff libAXSpeechManager.dylib armv7s  <d705ce6b23263295bd2e3680effe923a> /usr/lib/libAXSpeechManager.dylib
0x359e1000 - 0x359e9fff libAccessibility.dylib armv7s  <6e34291faad43781a17464523142c7a1> /usr/lib/libAccessibility.dylib
0x35c3d000 - 0x35c53fff libCRFSuite.dylib armv7s  <81bb6391c1643d038ea597f57762d71f> /usr/lib/libCRFSuite.dylib
0x35c86000 - 0x35d8afff libFosl_dynamic.dylib armv7s  <2fc617cd0aee379988c3863964aaf1ad> /usr/lib/libFosl_dynamic.dylib
0x35da4000 - 0x35dbbfff libMobileGestalt.dylib armv7s  <9ba6174d5d1b31698d70c4855b6078da> /usr/lib/libMobileGestalt.dylib
0x35dbc000 - 0x35dc4fff libMobileGestaltExtensions.dylib armv7s  <30877c65f1f43ffd895c7dd86dd929e4> /usr/lib/libMobileGestaltExtensions.dylib
0x35de1000 - 0x35de2fff libSystem.B.dylib armv7s  <02104bfb10f13d22a1cd5bf04ba0c10e> /usr/lib/libSystem.B.dylib
0x35e53000 - 0x35e97fff libTelephonyUtilDynamic.dylib armv7s  <7788f77375dd3756a934364b512c4137> /usr/lib/libTelephonyUtilDynamic.dylib
0x35fa7000 - 0x35fc9fff libarchive.2.dylib armv7s  <3e3a22a4d4603e20967a04898efb99ba> /usr/lib/libarchive.2.dylib
0x35fca000 - 0x35fcafff libassertion_extension.dylib armv7s  <362ff6610704394bb58ab4c1492fb307> /usr/lib/libassertion_extension.dylib
0x35ff9000 - 0x36005fff libbsm.0.dylib armv7s  <ab7814c202aa3a978d5fc4066d507ded> /usr/lib/libbsm.0.dylib
0x36006000 - 0x3600ffff libbz2.1.0.dylib armv7s  <b174e42e490732959f042109e83b4068> /usr/lib/libbz2.1.0.dylib
0x36010000 - 0x3605afff libc++.1.dylib armv7s  <85cef3e4e8453efda7a44d09a528ca73> /usr/lib/libc++.1.dylib
0x3605b000 - 0x36076fff libc++abi.dylib armv7s  <f854f7b9943835089d64cdb166fb7944> /usr/lib/libc++abi.dylib
0x36078000 - 0x36085fff libcmph.dylib armv7s  <006a8f6104073fd59f65f0ee76b7e8f7> /usr/lib/libcmph.dylib
0x36086000 - 0x3608efff libcupolicy.dylib armv7s  <1536179fc38339e3a413d4ed3e835285> /usr/lib/libcupolicy.dylib
0x360b5000 - 0x360cefff libextension.dylib armv7s  <db293f05a78935cb9b9d576d4acef7c1> /usr/lib/libextension.dylib
0x360ed000 - 0x360f0fff libheimdal-asn1.dylib armv7s  <70ce5b437d51333e831f09b6d70996db> /usr/lib/libheimdal-asn1.dylib
0x360f1000 - 0x361defff libiconv.2.dylib armv7s  <579ad521cf163395bf9b360b8700b569> /usr/lib/libiconv.2.dylib
0x361df000 - 0x3634dfff libicucore.A.dylib armv7s  <b57df05026d73ec0aac4a815217f6185> /usr/lib/libicucore.A.dylib
0x3635a000 - 0x3635afff liblangid.dylib armv7s  <1c7a4628d4f93f57b0e9122aad9a41bc> /usr/lib/liblangid.dylib
0x3635b000 - 0x36365fff liblockdown.dylib armv7s  <6bcacfdd347535f8aca89e45d19be50e> /usr/lib/liblockdown.dylib
0x36366000 - 0x3637bfff liblzma.5.dylib armv7s  <d503bc52f6ab336a951c7498fb2818c6> /usr/lib/liblzma.5.dylib
0x366f5000 - 0x3670afff libmis.dylib armv7s  <d76f316f54883fb0a44fe4178069d126> /usr/lib/libmis.dylib
0x36733000 - 0x3692dfff libobjc.A.dylib armv7s  <8529d172d0d4376295751bc1657c184d> /usr/lib/libobjc.A.dylib
0x369e2000 - 0x369f8fff libresolv.9.dylib armv7s  <db3fa6e2c18b38b38a25828c2eb2eb94> /usr/lib/libresolv.9.dylib
0x36a23000 - 0x36ad4fff libsqlite3.dylib armv7s  <c71c02e14d9e3f20b1e54eef5c0bfeec> /usr/lib/libsqlite3.dylib
0x36b22000 - 0x36b48fff libtidy.A.dylib armv7s  <97fd6bd77752374fb766bbbc31d83b6f> /usr/lib/libtidy.A.dylib
0x36b49000 - 0x36b51fff libtzupdate.dylib armv7s  <e2906e95adfe33b48c7a9314c73fc515> /usr/lib/libtzupdate.dylib
0x36b55000 - 0x36c0bfff libxml2.2.dylib armv7s  <fad7ce69d3b03c2f97f39dc70fd1b35e> /usr/lib/libxml2.2.dylib
0x36c0c000 - 0x36c2dfff libxslt.1.dylib armv7s  <eed3c48ecd8c3f688357c7e9634c28a5> /usr/lib/libxslt.1.dylib
0x36c2e000 - 0x36c3afff libz.1.dylib armv7s  <9e6cb733a633391eaf5eeccd9b9ab4d4> /usr/lib/libz.1.dylib
0x36c3b000 - 0x36c3ffff libcache.dylib armv7s  <464e89321abb31a29f42b476cd75786d> /usr/lib/system/libcache.dylib
0x36c40000 - 0x36c49fff libcommonCrypto.dylib armv7s  <0c2911b8dc653852913d4402f021a6c6> /usr/lib/system/libcommonCrypto.dylib
0x36c4a000 - 0x36c4efff libcompiler_rt.dylib armv7s  <2e4a01eff3dd333b99ab14b54580aecd> /usr/lib/system/libcompiler_rt.dylib
0x36c4f000 - 0x36c55fff libcopyfile.dylib armv7s  <d365247190fe3bafbadb0e3a35b988fd> /usr/lib/system/libcopyfile.dylib
0x36c56000 - 0x36ca2fff libcorecrypto.dylib armv7s  <e93edd093a633c40af2fcf9de68a813b> /usr/lib/system/libcorecrypto.dylib
0x36ca3000 - 0x36cc3fff libdispatch.dylib armv7s  <d047f787bfc33eb08eaf3a34f82a1eb5> /usr/lib/system/libdispatch.dylib
0x36cc4000 - 0x36cc5fff libdyld.dylib armv7s  <d76438e6086a38eb8d9e38b7d3a4fedd> /usr/lib/system/libdyld.dylib
0x36cc6000 - 0x36cc6fff libkeymgr.dylib armv7s  <d961518f774a38ce80d87302f5f2d270> /usr/lib/system/libkeymgr.dylib
0x36cc7000 - 0x36cc7fff liblaunch.dylib armv7s  <94848ea6b094336d893c38753795f5ac> /usr/lib/system/liblaunch.dylib
0x36cc8000 - 0x36ccbfff libmacho.dylib armv7s  <a7e04a25a2143498be67845bc23fff4a> /usr/lib/system/libmacho.dylib
0x36ccc000 - 0x36ccdfff libremovefile.dylib armv7s  <99984750ecaf3483976db606e4a6d763> /usr/lib/system/libremovefile.dylib
0x36cce000 - 0x36cdffff libsystem_asl.dylib armv7s  <983391f3b4763ae59c7730f0b80dbb0b> /usr/lib/system/libsystem_asl.dylib
0x36ce0000 - 0x36ce0fff libsystem_blocks.dylib armv7s  <9024447e02ef3270b46771adc93133c9> /usr/lib/system/libsystem_blocks.dylib
0x36ce1000 - 0x36d44fff libsystem_c.dylib armv7s  <a84f366484fd3d21b3b0ee60ac01ac59> /usr/lib/system/libsystem_c.dylib
0x36d45000 - 0x36d47fff libsystem_configuration.dylib armv7s  <80e2c84c178837c5b91e591f6a4d7201> /usr/lib/system/libsystem_configuration.dylib
0x36d48000 - 0x36d49fff libsystem_coreservices.dylib armv7s  <9290fc392177307faa8403f9aa89553f> /usr/lib/system/libsystem_coreservices.dylib
0x36d4a000 - 0x36d56fff libsystem_coretls.dylib armv7s  <6a0f500cccaf3e848179ce8bae2a0880> /usr/lib/system/libsystem_coretls.dylib
0x36d57000 - 0x36d5dfff libsystem_dnssd.dylib armv7s  <12b8450299ab3104bf59e0e4852c63c4> /usr/lib/system/libsystem_dnssd.dylib
0x36d5e000 - 0x36d7afff libsystem_info.dylib armv7s  <f37a73a834bb355ca035e82f81de2d26> /usr/lib/system/libsystem_info.dylib
0x36d7b000 - 0x36d95fff libsystem_kernel.dylib armv7s  <31ae14feacbd345cb0a6f3041834e3ea> /usr/lib/system/libsystem_kernel.dylib
0x36d96000 - 0x36db5fff libsystem_m.dylib armv7s  <0c284ff2f4d130299a6cf38fe5ff8303> /usr/lib/system/libsystem_m.dylib
0x36db6000 - 0x36dc8fff libsystem_malloc.dylib armv7s  <5c198ca70fa7378da137cd816decbbf8> /usr/lib/system/libsystem_malloc.dylib
0x36dc9000 - 0x36df6fff libsystem_network.dylib armv7s  <c1e9979a8c973feeb4acbce97e80508a> /usr/lib/system/libsystem_network.dylib
0x36df7000 - 0x36dfcfff libsystem_networkextension.dylib armv7s  <6d884e2846e83abf8208027e115dddea> /usr/lib/system/libsystem_networkextension.dylib
0x36dfd000 - 0x36e04fff libsystem_notify.dylib armv7s  <787d39efb46b3b4d94fb3cc95b618779> /usr/lib/system/libsystem_notify.dylib
0x36e05000 - 0x36e09fff libsystem_platform.dylib armv7s  <badd9ef3da483107966fde4120b12fe2> /usr/lib/system/libsystem_platform.dylib
0x36e0a000 - 0x36e10fff libsystem_pthread.dylib armv7s  <2d3aa96e56a73fcdb25f3e0a536e6eeb> /usr/lib/system/libsystem_pthread.dylib
0x36e11000 - 0x36e13fff libsystem_sandbox.dylib armv7s  <8cb953de567c3fbcad59887f5afbfb41> /usr/lib/system/libsystem_sandbox.dylib
0x36e14000 - 0x36e17fff libsystem_stats.dylib armv7s  <d882f76e1481348aaeae6fb274046cee> /usr/lib/system/libsystem_stats.dylib
0x36e18000 - 0x36e1dfff libsystem_trace.dylib armv7s  <4bc82d65e90b3ecbb8317222946ff47f> /usr/lib/system/libsystem_trace.dylib
0x36e1e000 - 0x36e1efff libunwind.dylib armv7s  <b95c6662d06a3a9988fe6265a26ddc42> /usr/lib/system/libunwind.dylib
0x36e1f000 - 0x36e3afff libxpc.dylib armv7s <0db6e5db4d633912804338bee3b1ea27> /usr/lib/system/libxpc.dylib
```
