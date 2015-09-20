Core Animation
==============

Core Animation 是在 iOS 上負責一般 UI 繪圖與動畫的 framework。當你想要
讓你的 app 有更豐富的動畫效果，但是你並不打算做套 3D 遊戲、還沒打算使
用 OpenGL 或 Metal 這些 3D 繪圖的 framework 的話，你就應該先看一下
Core Animation。掌握了 Core Animation 之後，絕大多數 app 裡頭可能用到
的動畫，你都應該可以做出來。

Core Animation 大概分成三種主要的 class：

- CALayer
- CAAnimation
- CATransaction

我們不妨把 CALayer 想像成是演員，而 CAAnimation 則是劇本，各種動畫效果，
就是讓演員按照劇本演出。至於 CATransaction 則是整體的劇場設定，比方說，
我們今天要演一齣戲，這場戲原本要演兩個小時，但我們希望這齣戲可以一個小
時演完，那麼，這種「整齣戲演出的速度」這樣的設定，就是 CATransaction的
功能。
