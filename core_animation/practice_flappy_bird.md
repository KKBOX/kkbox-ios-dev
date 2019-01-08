練習：Flappy Bird
-----------------

### 練習範圍

- Core Animation
- CADisplayLink

### 練習內容

這個遊戲也可以使用 SpriteKit 等 framework 實作，不過，我們在這邊使用
Core Animation，也一樣可以完成這個遊戲。

在 Flappy Bird 這個遊戲中，看起來像是鳥在不斷往前飛行，但其實鳥並沒有
水平移動，而是只有垂直移動而已：當我們觸控螢幕的時候，鳥會往上移動，接
著隨著時間就會一直往下掉落。真的在水平移動的，其實是後面的柱子。

無論是鳥或是柱子，都是 CALayer，接著，我們會安排一個 CADisplayLink，定
時更新鳥與柱子的位置。

- 先放置鳥的 layer，放在畫面的正中央。鳥的大小為 40x40 pixel。
- 然後我們設定好一系列柱子的 layer。柱子的上下分別為兩個 layer，我們先
  用迴圈放置三十對柱子（先假設這麼難玩的遊戲應該不會有人可以突破三十關…）
  總共就有 60 個 layer
  - 每對柱子中，上方的柱子的高度為 130 到 130 + (螢幕高度 - 320) 之間
  - 上方柱子與下方柱子的距離為 130 pixel
  - 下方柱子的高度就是從上方柱子的 y + 130 開始，繼續填滿螢幕高度
  - 每根柱子的寬度是 60 pixel
  - 每對柱子與柱子的水平距離是 140 pixel
  - 第一根柱子的 x 軸剛好在畫面寬度外
- 用 UIGestureRecognizer 寫一個 tap 的 action，叫做 fly。裡頭的實作是
  鳥的 position 的 y 會減少 100 pixel
- 寫一個 CADisplayLink，CADisplayLink 的 action 中：
  - 每執行一次，鳥的 position 的 y 軸都往下掉 4 pixel
  - 每執行一次，每根柱子的 position 的 x 軸都減 1 pixle
  - 檢查鳥的 y 是否超過螢幕範圍，如果是，判定鳥落地，遊戲結束
  - 檢查每根柱子的 frame 是否與鳥的 frame 交疊（CGRectIntersectsRect），
    如果是，遊戲結束
  - 先讓這個 CADisplayLink 保持 pause 狀態（paused 屬性設成 YES）
- 在畫面正中央放置一個「開始遊戲按鈕」
  - 按鈕按下去後，按鈕會消失
  - 把 CADisplayLink 的 paused 設成 NO，進行遊戲
