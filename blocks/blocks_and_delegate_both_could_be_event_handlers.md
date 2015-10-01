Block 與 Delegate 都可以想成是 Event Handler
--------------------------------------------

我們在前一章當中提到，Objective-C 裡頭的 delegate 其實相當於 C\# 裡頭
event handler 的角色，但 C\# 裡頭的 event handler 在語法上會更接近
Objective-C 的 block，都是匿名函式。

我們可以來看一個
[來自 MSDN 的例子](https://msdn.microsoft.com/en-us/library/aa645739(v=vs.71).aspx)
。以下的 C\# 程式中，有一個叫做 `Changed` 的 event handler。

``` c#
using System;
namespace MyCollections
{
   using System.Collections;

   // A delegate type for hooking up change notifications.
   public delegate void ChangedEventHandler(object sender, EventArgs e);

   // A class that works just like ArrayList, but sends event
   // notifications whenever the list changes.
   public class ListWithChangedEvent: ArrayList
   {
      // An event that clients can use to be notified whenever the
      // elements of the list change.
      public event ChangedEventHandler Changed;

      // Invoke the Changed event; called whenever list changes
      protected virtual void OnChanged(EventArgs e)
      {
         if (Changed != null)
            Changed(this, e);
      }

      // Override some of the methods that can change the list;
      // invoke event after each
      public override int Add(object value)
      {
         int i = base.Add(value);
         OnChanged(EventArgs.Empty);
         return i;
      }

      public override void Clear()
      {
         base.Clear();
         OnChanged(EventArgs.Empty);
      }
   }
}
```

如果寫成 Objective-C 並且使用 delegate，會像這樣：

``` objc
@class ListWithChangedEvent
@protocol ListWithChangedEventDelegate <NSObject>
- (void)listDidChange:(ListWithChangedEvent *)list args:(EventArgs *)e;
@end

@interface ListWithChangedEvent : NSObject
@property (weak, nonatomic) id <ListWithChangedEventDelegate> delegate;
@end

@implementation ListWithChangedEvent

- (void)onChanged:(EventArgs *)e
{
	[self.delegate listDidChange:self args:e];
}

- (void)add:(id)value
{
	[super add:value];
	[self onChanged:nil];
}

- (void)clear:(id)value
{
	[super clear:value];
	[self onChanged:nil];
}

@end
```

那如果是 block 呢？可以發現，會更接近 C\# 裡頭的寫法了。

``` objc
typedef void (^changedEventHandler)(id, EventArgs *);

@interface ListWithChangedEvent : NSObject
@property (copy, nonatomic) changedEventHandler changed;
@end

@implementation ListWithChangedEvent

- (void)onChanged:(EventArgs *)e
{
	if (self.changed) {
		self.changed(self, e);
	}
}

- (void)add:(id)value
{
	[super add:value];
	[self onChanged:nil];
}

- (void)clear:(id)value
{
	[super clear:value];
	[self onChanged:nil];
}

@end
```

還是順道一提，在 C\# 裡頭，我們可能會把所有的 callback 都叫做 event，
但是在 Cocoa 與 Cocoa Touch 的世界裡頭，我們只會把來自硬體的輸入叫做
event。
