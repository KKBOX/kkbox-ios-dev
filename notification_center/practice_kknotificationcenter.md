# 練習：自行實作 NSNotificationCenter

## 練習範圍
- Notification Center

## 練習目標

要了解 Notification Center 是怎麼運作的，一種很好的學習方法就是我們自
己重新實作一次，所以我們要來寫一個自己的 KKNotificationCenter。

Interface 如下：

``` objc
@interface KKNotificationCenter : NSObject

+ (KKNotificationCenter *)defaultCenter;

- (void)addObserver:(id)observer selector:(SEL)aSelector name:(NSString *)aName object:(id)anObject;
- (void)postNotification:(NSNotification *)notification;
- (void)removeObserver:(id)observer;
```

請實作這個 Class，讓這個 Class 擁有和 NSNotificationCenter 相同的功能。
