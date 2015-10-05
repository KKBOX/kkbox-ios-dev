CAAnimation 的細部設定
----------------------

### 避免動畫結束時 Layer 回到初始值

我們對一個 CALayer 設定了 CABasicAnimation 或 CAKeyframeAnimation 的動
畫之後，往往會遇到一個問題：我們期待在動畫結束之後，CALayer 會停在動畫
結束時的屬性設定，但是 Core Animation 的預設行為是動畫結束的時候，
CALayer 反而會跳回初始的狀態。這個時候我們就要修改 CAAnimation 的
`fillMode` 屬性。

每個 CAAnimation 的 `fillMode` 屬性預設是設定成 `kCAFillModeRemoved`，
代表的就是，在動畫完成的時候，就把 Layer 上的這個動畫移除掉。如果我們
不想要跳回初始值，就應該把 `fillMode` 設定成 `kCAFillModeForwards`。

### 一個動畫結束之後，再繼續另外一個動畫

如果我們同時對一個 CALayer 加了多個 animation 物件，這些動畫都會同時進
行，而不會一個動畫做完、才接續下一個動畫。如果我們想要一個動畫結束之後，
繼續下一個動畫，你可能會使用 timer 延遲，不過，我們建議使用使用
CAAnimation 的 delegate，讓 delegate 告訴我們動畫結束，才繼續下一步。

CAAnimation 的 delegate mehtod 定義在 NSObject (CAAnimationDelegate)中，
這種把 delegate 定義成 NSObject 的 category 的寫法，叫做 informal
protocol。現在 informal protocol 不是很常見，大概就只有 Core Animation
還在用這種用法。關於 formal protocol 與 informal protocol，請參見
[Formal Protocol 與 Informal Protocol](../delegate/formal_and_informal_protocols.md)
這一節。

CAAnimation 的 delegate method 包括：

- `-(void)animationDidStart:(CAAnimation *)anim;`
- `-(void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag;`

當動畫結束的時候，就會呼叫 `-animationDidStop:finished:`。不過，由於我
們在這邊只會拿到 CAAnimation 物件，如果想要判斷這是哪個 Layer 上面發生
的動畫，然後對這個 Layer 繼續下一步的動畫，大概得要用這種方法來判斷：

``` objc
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag
{
	if (anim == [someLayer animationForKey:@"someKey"]) {
		// 繼續其他的動畫
	}
}
```

在這邊需要特別注意：如果我們想用這樣的判斷式，判斷這個動畫發生在哪個
layer 上，那麼，在建立 animation 物件的時候，就得要把 animation 物件的
`removedOnCompletion` （代表動畫在完成的時候移除）屬性設成 NO。不然等
到動畫結束的時候，呼叫 `[someLayer animationForKey:@"someKey"]` 就會拿
到 nil—因為當 removedOnCompletion 是 YES 的狀況下，動畫完成的時候貝移
除，對 someLayer 來說，在 someKey 這邊的動畫就已經被移走了。

不過，如果我們還是要繼續對某個 layer 增加動畫，一直不把原本已經完成的
動畫拿掉，也會是個問題。所以，我們可以考慮在加入新的動畫之前，先對
layer 呼叫一次 `removeAllAnimations`，把原本的動畫先拿掉。
