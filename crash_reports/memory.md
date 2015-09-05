記憶體不足時產生的 Crash Report
-------------------------------

因為系統整體記憶體不足所產生的 crash report，與其他的 crash report 長
得不太一樣—這種 crash report 中只會顯示當時每個 process 使用了多少系
統資源。此外，就像前一章講到的，你可能從 Xcode 或 iTunes Connect 上蒐
集到這種 crash report，在 Crashlytics 或 HockeyApp 上找不到。

當我們在閱讀這類型 crash report 的時候，需要知道 rpage 的單位，一個
rpage 相當於 4k 的記憶體。所以我們可以看到，在這台 iPad Mini 上，KKBOX
用了22810 個 rpage，所以 KKBOX 大概使用了 87 MB 的記憶體（22810 * 4 /
1024）。

要怎樣修正這種問題呢？首先，87 MB 在 iPad 上，其實不算太多，由於記憶體
不足不見得是單一 App 的問題，而是所有的 App 共同把記憶體用完了，我們看
到幾個系統服務也把記憶體用得很兇，像 mediaserverd（系統底層一個用來播
放影音的 service）用了 100 MB、SpringBoard（就是 iOS 裝置按下 Home 按
鈕可以看到的 App 列表畫面）也用了將近 40 MB。所以，我們可以先試試看重
新開機，把系統服務佔用的資源是放掉，再來看看執行 KKBOX 會不會出問題。

一個 App 到底可以用多少記憶體？蘋果一直沒有說得很清楚，我們最早在
iPhone 3G 上開發 KKBOX 時，KKBOX 可以使用的記憶體極限大概在 40 MB 左右，
由於我們還要把一定比例的歌曲資料載入到記憶體中，所以在畫面上能使用的記
憶體就必須盡量節省。在比較後來的機種上，像 iPhone 5S 或 iPad 3 之後，
大概用到 200MB 左右都沒問題，由於每個機種能用的資源都不太一樣，需要的
時候，我們可以寫個一直 alloc 記憶體的迴圈做實驗，不過在 KKBOX 這邊，我
們手上的裝置也沒有很齊全。

如果我們真的發現 App 中記憶體用太多，就只能夠想辦法節省記憶體。想要節
省記憶體用量，最重要的原則就是—哪邊用了比較多的記憶體，不要透過閱讀程
式碼猜測，而是直接用 Instrument 跑 profiling 觀察，確實找到記憶體的瓶
頸。

至於要節省記憶體用量的作法，大概就是，如果一份資料在當下沒有必要放在記
憶體中，可以考慮先放入檔案中，至於要避免產生太多 view 佔用記憶體，可以
參考
[記憶體管理 Part 3 - Memory Warnings](../memory_management_part_3/README.md)
這一章。

範例 crash report 如下：

```
Incident Identifier: 7BC7B59F-A13D-44C1-8D57-62508830C0C2
CrashReporter Key:   fe242c8a27a16318ac7bbe16c6fbaaff6ce8a3a0
Hardware Model:      iPad2,5
OS Version:          iPhone OS 7.0.4 (11B554a)
Kernel Version:      Darwin Kernel Version 14.0.0: Fri Sep 27 23:00:49 PDT 2013; root:xnu-2423.3.12~1/RELEASE_ARM_S5L8942X
Date:                2015-08-07 11:20:54 +0800
Time since snapshot: 56 ms

Free pages:                              922
Active pages:                            3697
Inactive pages:                          2197
Speculative pages:                       7
Throttled pages:                         74156
Purgeable pages:                         0
Wired pages:                             47617
File-backed pages:                       5424
Anonymous pages:                         477
Compressions:                            0
Decompressions:                          0
Compressor Size:                         0
Uncompressed Pages in Compressor:        0
Largest process:   mediaserverd

Processes
     Name                    <UUID>                       rpages       recent_max   fds      [reason]          (state)

      MobileMail <b3574f4bded1315cb2e50e5de205be48>         1113             1113  200   [vm-pageshortage]  (resume) (continuous)
            tccd <1fea8c5a71943151b5cd304c7eb0fd8c>          170              170  200   [vm-pageshortage]  (daemon)
             kbd <be2d64e41bf43e48a09a23fb129eb0b4>          381              381  200   [vm-pageshortage]  (daemon)
           KKBOX <8367b00614e735f5861044b98c96cd1c>        22810            22810  200   [vm-pageshortage]  (frontmost) (resume)
            ptpd <db9048c36f6c3c18a7330fc96d93a0cf>          657              657  200                      (daemon)
identityservices <18cc20db2e4739a782cc8e38e03eff52>          349              349  200                      (daemon)
         syslogd <6539f4cf4dcf34daadf1d99991926680>          157              157   50                      (daemon)
          powerd <0a253ac2a99236809422214be1700bc0>          121              121  100                      (daemon)
         imagent <bef102e1faef39209926fb25f428a71e>          298              298  100                      (daemon)
   iaptransportd <42faa147f61a314bb735e239f445efaf>          219              219   50                      (daemon)
   mDNSResponder <8922e9954d893eb9a1ab27ca4723bbab>          228              228  100                      (daemon)
            apsd <0dd1fd7c2edc3cf9899a4830541c1bac>          468              468  100                      (daemon)
     dataaccessd <5da732b6ce6935f2928461a022101aba>         1228             1228  200                      (daemon)
    mediaremoted <476eb521b8423428b4e6df20d3fe4091>          327              327   50                      (daemon)
           wifid <a5cf99e5a0f032a69bc2f65050b44291>         1859             1859  200                      (daemon)
    mediaserverd <71101024312538ccae5799b88681e38d>        25807            25807  200                      (daemon)
        sharingd <a95c2cea41b43cc69b0bbe9a03730d45>          460              460  200                      (daemon)
      calaccessd <77a5672f2f653f64acf7367126a8d173>          420              420  200                      (daemon)
    itunesstored <c52ea3e4aceb398e9a98ff595c17669f>         1576             1576  200                      (daemon)
       locationd <c31643022d833911b8b7461fd3964bd5>         2497             2497  200                      (daemon)
    syslog_relay <c4c1e88d92a537888b15d6118c5fa1d1>          107              107  200                      (daemon)
     SpringBoard <3c0e305139b331c6b37d2e9516f5804f>        10207            10207  200
      backboardd <d61df126c4673b25bbfe5d9024be1d48>        13495            13495   50                      (daemon)
      aggregated <a5dda46586ba3a3cbb298bd8aa545e50>          666              666   50                      (daemon)
       lockdownd <6f28a28a0025348aa5361078e41914e3>          340              340  100                      (daemon)
    fairplayd.A2 <6cae0c124e1830598a43f6b4d790917f>          141              141  100                      (daemon)
         configd <c57db43e53a73f8a9360f4d0d9001704>          686              686  100                      (daemon)
       fseventsd <5c909a70b62f33c8856e3158834ba071>          336              336  100                      (daemon)
        BTServer <3933a8148924316b9f19dd3d10a23f00>         3891             3891  100                      (daemon)
       distnoted <38616bd8864034e7bc741f8bd7313349>         1187             1187  100                      (daemon)
  UserEventAgent <a3c7e56924ec3690a994a75a0ea79ee8>          850              850  100                      (daemon)
        networkd <84dfdb49c24132fa8dd10520deb16645>          523              523  100                      (daemon)
filecoordination <4fa03f2b93363668a1159715de4b0270>          186              186  200                      (daemon)
EscrowSecurityAl <65547599d6d331f2aec702509cbb1079>          175              175  200                      (daemon)
    itunescloudd <9a38b56ee4fd3c308766da99af8eeaed>          932              932  200                      (daemon)
     touchsetupd <02780826b4263a7498bda167721b5f8c>          163              163  200                      (daemon)
            afcd <5c18557c26f73b88a54ad94f3cd06d0c>          113              113  200                      (daemon)
notification_pro <852af3fe832e3cc3a3f31d05511a5482>          124              124  200                      (daemon)
          cplogd <148e9e2ff86130ecb63423c183f68da5>          137              137  200                      (daemon)
     pasteboardd <6bb2d8a2beb530b095c82dc2c1cda0f7>          119              119  200                      (daemon)
   wirelessproxd <46066fc432663d45ab7e055082fe0bd6>           11               11  200                      (daemon)
CommCenterClassi <b836b786e0cb3785a18a94e9b13c9991>          363              363   50                      (daemon)
         notifyd <35afacabfed73771889e72a017479709>          251              251  100                      (daemon)
```
