# ★CSSエラーの対処
## ❇️概要
CSS関係で引っかかったことの備忘録
<br>
<br>
## ❓エラー事例
### ✅CSSの表示が崩れる
#### ・状況
| 状況                                 | 説明                              |
| ---------------------------------- | ------------------------------- |
| **DOMツリーの崩れ**                      | 実際にブラウザが解釈した構造が壊れること            |
| **ネスト構造の乱れ**                       | タグが正しく入れ子になっていない状態              |
| **後続のスタイル影響**                      | CSSが期待通り効かなくなったり、隣接要素に影響したりする   |
| **HTML構文エラー / validation warning** | Linterやvalidatorで検出される可能性があるエラー |


#### ・原因
| 原因           | 説明                               |
| ------------ | -------------------------------------- |
| **閉じタグを忘れた**                       | HTMLの構文ミス（パーサーが誤解釈する）           |
| HTMLが壊れていた   | 「構文エラーでDOM構造が崩れてた」                     |
| ボタンのスタイルがずれた | 「`<i>`の未閉じが原因で、親要素のCSS指定が効いてなかった」      |
| レイアウト崩れ      | 「タグ未閉じでレンダリングが崩れて、意図しないフロート・マージンになってた」 |

<br>
<br>

### ✅背景画像よりも上のレイヤーに文字が出てこない
#### ・状況
| 状況                 | 説明                                                       |
| ------------------ | -------------------------------------------------------- |
| ヘッダー画像の上に文字が表示されない | Swiperによるスライドショーの `.main-title`（見出し）が、背景の画像に隠れてしまい表示されない or 背景画像の下段に表示される |
<br>
<br>

#### ・原因
| 原因                          | 説明                                                                         |
| --------------------------- | -------------------------------------------------------------------------- |
| 画像要素（`.main_image`）の優先描画    | 画像がDOM内で `.main-title` より後にレンダリングされたり、z-indexが文字よりも低い値になっていた               |
| 要素配置（position/z-index）の設定不備 | `.swiper` に position: relative はあるものの、.main-title の z-index や要素の並び順に問題があった |

<br>
<br>

#### 📝 考察
親要素に position: relative、子要素に position: absolute; z-index: 10; を付与するのが基本。<br>
ただし、画像側の表示設定（クラス名やDOMの順序、デフォルトのz-index等）によっては、<br>
後から追加した文字要素が背後に回ってしまうことがある。<br>
<br>
今回の対策としては、文字要素を画像要素よりも後に（あるいは上に）配置するようにDOM順を見直すか、<br>
.main-title の z-index を十分高い値に設定することで解決できる。<br>
<br>
<br>
#### 🧩コード例<br>
```
#app\assets\stylesheets\application.scss

.swiper {
  width: 100%; // ページ幅いっぱいに広げる
  height: 400px; // お好みの高さに調整（例：メインビジュアル）
  position: relative; // ページネーションやナビボタンを絶対配置する場合
  overflow: hidden; // スライドがはみ出さないように
}

.swiper-wrapper {
  display: flex; // スライドを横に並べる
  transition-property: transform; // スライド時の動きに必要
}

.swiper-slide {
  flex-shrink: 0;
  width: 100%;
}

.main-title {
  position: absolute; // 親要素に対して絶対配置
  z-index: 10;       // 画像より上に表示
  // ここに位置調整のプロパティを追加
  top: 70%;
  left: 5%;
}


#app\views\layouts\_header.html.slim

header.swiper
  .main-title
      h1 = link_to current_site.name, root_path
      p.lead = current_site.subtitle
  .swiper-wrapper
    - current_site.main_images.each do |img|
      .swiper-slide
        = image_tag img.variant(resize: "1200x400"), class: "main_image"
```
<br>

このコードでは.swiper-wrapperの画像の上に.main-titleの文字が表示される。<br>
しかし、app\views\layouts\_header.html.slimを次のように書き換えると文字は画像の枠の下に表示される。<br>
```
header.swiper
  .swiper-wrapper
    - current_site.main_images.each do |img|
      .swiper-slide
        = image_tag img.variant(resize: "1200x400"), class: "main_image"
  .main-title
      h1 = link_to current_site.name, root_path
      p.lead = current_site.subtitle
```
<br>

原因として、.main-titleは.swiperに対して絶対位置をとるように設定されている。しかし、間に配置された<br>
.swiper-wrapperと.swiper-slideの表示設定の影響により上手く機能しなかったと考えられる。<br>
