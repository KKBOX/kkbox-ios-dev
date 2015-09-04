測試驅動開發
------------

軟體業界開始注重單元測試，大概是進入二十一世紀的頭幾年的事情，其中一個
明顯的指標就是 Kent Beck 在 2002 年出版的 *Test-Driven Development: By
Example* 這本書，提出了「測試驅動開發」（Test-Driven Development，通常
簡稱 TDD）的觀念，顛覆了不少人寫程式的習慣與流程：在開發軟體的時候，我
們不是先寫主要功能，而是先寫測試。

坊間跟測試驅動開發相關的書籍或其他閱讀資料非常多，我們不會在這邊花費太
多篇幅。簡單來說，Kent Beck 提出寫程式的過程，應該是 "Red, Green,
Refactor" 三個階段：

- Red: 在還沒有主要功能之前，先寫單元測試。由於主要功能都還沒有撰寫，
  自然無法通過剛剛寫出來的單元測試，所以會亮出紅色的燈號。
- Green: 開始實作主要功能，直到可以通過單元測試，讓測試的燈號變成綠色。
- Refactor: 繼續整理寫出的程式碼。

這個流程後來引起一些批評，比較有名的案例像是 Ruby on Rail 的主要作者
DHH（David Heinemeier Hansson） 在個人 blog 上寫了一篇
*[TDD is dead. Long live testing.](http://david.heinemeierhansson.com/2014/tdd-is-dead-long-live-testing.html)*
，反對 TDD 流程，主要論點是，如果非要讓系統中每個部分都要能夠被執行單
元測試，反而會導致不適當、甚至有害的系統設計。此文一出，後續引發了一連
串的爭論。至於在你的開發流程中，是否適合使用測試驅動開發呢？還是那句話：
先別排斥，先試試看吧！
