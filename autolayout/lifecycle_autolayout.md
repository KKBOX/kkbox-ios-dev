## Lifecycle 
`小心別中 frame 的陷阱`
當你使用 .xib 從 nib loadView 時，一開始的 frame 會是 nib 裡面的 frame，而它會延遲到`-updateViewConstraints:`的時候才會得到正確的 frame size ，而使用 .storyboard 時就可以一開始拿到正確的 frame size 。

尤其是我們在 `-loadView` or `-viewDidLoad` 加入 CALayer 的時候，因為 self.view.frame 還沒更新導致加入的 layer 算錯。


```
// use xib
frame = {{0, 0}, {600, 600}}, loadView
frame = {{0, 0}, {600, 600}}, viewDidLoad
frame = {{0, 0}, {600, 600}}, viewWillAppear
===
frame = {{0, 0}, {414, 736}}, updateViewConstraints (1)
frame = {{0, 0}, {414, 736}}, viewDidLayoutSubviews (1)
frame = {{0, 0}, {414, 736}}, viewDidAppear
frame = {{0, 0}, {414, 736}}, updateViewConstraints (2)
frame = {{0, 0}, {414, 736}}, viewDidLayoutSubviews (2)

// use storyboard
frame = {{0, 0}, {414, 736}} loadView 
frame = {{0, 0}, {414, 736}} viewDidLoad 
frame = {{0, 0}, {414, 736}} viewWillAppear 
frame = {{0, 0}, {414, 736}} updateViewConstraints (1)
frame = {{0, 0}, {414, 736}} viewDidLayoutSubviews (1)
frame = {{0, 0}, {414, 736}} viewDidAppear 
frame = {{0, 0}, {414, 736}} updateViewConstraints (2)
frame = {{0, 0}, {414, 736}} viewDidLayoutSubviews (2)
frame = {{0, 0}, {414, 736}} updateViewConstraints (3)
frame = {{0, 0}, {414, 736}} viewDidLayoutSubviews (3)
```