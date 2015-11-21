Animation with Auto Layout
---------------------------

要讓 view 依照 autolayout 去動畫只要在變更 constaint 後在 `UIView:animationWithDuration:`中加入 `[self.view layoutIfNeeded]`即可。

```
self.userNameLabelWidth.constant = 20; // from 100 to 20
[UIView animateWithDuration:0.25 animations:^{
	[self.view layoutIfNeeded];
}];
```