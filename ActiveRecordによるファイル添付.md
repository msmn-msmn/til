# ★ActiveRecordによるファイル添付
## ❇️概要
ActiveRecordを使ったファイル添付のやり方の備忘録<br>
環境は<br>
```
Rails7.0
gem 'activestorage-validator'
bootstrap
```
です。<br>
<br>
🧩参考<br>
gem 'activestorage-validator'：[https://github.com/igorkasyanchuk/activestorage-validator](https://github.com/aki77/activestorage-validator)<br>
bootstrap：https://getbootstrap.jp/docs/5.3/getting-started/introduction/
<br>
## ❇️やり方
- モデルファイルに_attachedを追加
- コントローラーファイルに削除メソッドとストロングパラメーターを追加
- ルートファイルに削除メソッドのパスを追加
- ビューファイルに添付フォームと削除ボタンを追加
<br>

### 1️⃣モデルファイルに_attachedを追加
app\models\site.rbを編集<br>
```
# == Schema Information
#
# Table name: sites
#
#  id          :bigint           not null, primary key
#  name        :string(255)
#  subtitle    :string(255)
#  description :text(65535)
#  created_at  :datetime         not null
#  updated_at  :datetime         not null
#

class Site < ApplicationRecord
  + has_one_attached :og_image
  + has_one_attached :favicon
  + has_many_attached :main_images

  validates :name, presence: true, length: { maximum: 100 }
  validates :subtitle, length: { maximum: 100 }
  validates :description, length: { maximum: 400 }
  + validates :og_image, attachment: { purge: true, content_type: %r{\Aimage/(png|jpeg)\Z}, maximum: 524_288_000 }
  + validates :favicon, attachment: { purge: true, content_type: %r{\Aimage/png\Z}, maximum: 524_288_000 }
  + validates :main_images, attachment: { purge: true, content_type: %r{\Aimage/(png|jpeg)\Z}, maximum: 524_288_000 }
end
```

添付ファイルをモデルに紐づけて保存できるように<br>
```
   has_one_attached :og_image
   has_one_attached :favicon
   has_many_attached :main_images
```
を追加する。<br>
has_one_attachedは添付ファイルが一つの時に使用。<br>
has_many_attached :main_imagesはファイルを複数添付したい時に使用。<br>
purge: trueはバリデーションエラーが発生した場合、その添付ファイルを自動で削除する。<br>
content_type:は添付できるファイル形式の設定。<br>
maximum:は添付できる最大のファイルサイズの設定。<br>


### 2️⃣コントローラーファイルに削除メソッドとストロングパラメーターを追加
app\controllers\admin\sites_controller.rbを編集
```
def og_image_clear
    authorize(@site)

    @site.og_image.purge
    redirect_to edit_admin_site_path(@site)
  end

  def favicon_clear
    authorize(@site)

    @site.favicon.purge
    redirect_to edit_admin_site_path(@site)
  end

  def main_images_clear
    authorize(@site)

    attachment = @site.main_images.find(params[:blob_id])
    attachment.purge
    redirect_to edit_admin_site_path(@site)
  end
```
