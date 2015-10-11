

對這些繼承自 UIView 的物件來說，有兩種方法可以處理這種狀況。如果你的
UI 是在 Interface Builder （以下簡稱 IB）裡頭產生的，在 IB 的
Inspector 的第四個分頁，就可以設定這個 UI 元件的相關資訊。也可以透過程
式碼設定，iPhone SDK 定義了 UIAccessibility 這個 informal protocol，所
有繼承自 NSObject 的物件都具備此一 interface，你可以透過
accessibilityLabel、accessibilityHint 等屬性，標記 UI 元件的標題與詳細
說明。

開發過程中，可以使用 iOS Simulator 檢驗目前程式對 Voiceover 的支援狀態，
方法是在模擬器中的偏好設定中，把 Accessibility Inspector 打開，模擬器
畫面中就會出現一個小畫面，在選取畫面中某個 UI 元件時，Accessibility
Inspector 就會列出各種相關資訊。