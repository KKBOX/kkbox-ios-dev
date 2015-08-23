Category
========

Category：**不用繼承物件，就直接增加新的 method，或替換原本的 method**。

前一章提到，Objective-C 語言中，每個 class 有哪些 method，都是在
runtime時加入的，我們可以透過 runtime 提供的一個叫做 `class_addMethod`
的function，加入對應某個 selector 的實作。而在 runtime 時想要加入新的
method，使用 category 會是更容易理解與寫作的方法，因為可以使用與宣告
class 時差不多的語法，同時也以一般實作 method 的方式，實作我們要加入的
method。









```


[^1]: 請見 [Objective-C Feature Availability Index](https://developer.apple.com/library/ios/#releasenotes/ObjectiveC/ObjCAvailabilityIndex/_index.html#//apple_ref/doc/uid/TP40012243)
