# ★Railsの画像アップロード機能実装手順<br>
## Ⅰ. 概要<br>
  　``gem 'carrierwave', '2.2.2'``を使用してRailsアプリの画像アップロード機能を実装する。<br>

## Ⅱ. 手順<br>
　　以下の手順で実装をしていく<br>
  
### 　1. Gemfileにcarrierwave追加<br>
### 　2. アップローダークラスの生成<br>
### 　3. アップロード画像のカラムを追加<br>
### 　4. アップローダークラスとカラムの紐づけ<br>
### 　5. アバター画像の登録・更新<br>
### 　6. アバター画像の表示<br>

## Ⅲ. 実装<br>
### 　1. Gemfileにcarrierwave追加<br>
　　Gemfile<br>
  　　　　``gem 'carrierwave', '2.2.2'``<br>
　　Ubuntu<br>
  　　　　``$ docker compose exec web bundle install``を実行後に``$ docker compose restart``を実行<br>
<br>
### 　2. アップローダークラスの生成<br> 
　　Carrierwaveをインストールすると、アップローダークラスを以下のコマンドで生成する事が出来ます。<br>
　　Ubuntu<br>
  　　　　``$ docker compose exec web rails generate uploader アップローダー名``<br>
  　　今回は、ユーザーのアバター画像用のアップローダークラスを作成するので、以下のようにAvatarと指定してコマンドを実行します。<br>
　　　Ubuntu<br>
  　　　　``$ docker compose exec web rails generate uploader Avatar``<br>
  　　コマンドを実行すると、app/uploaders/avatar_uploader.rbファイルが作成されます。<br>
  　　作成したアップローダークラス(AvatarUploader)では、アップロードするファイルの<br>
  　　拡張子やサイズ、保存するパスを指定する事が出来ます。<br>
　　　また、デフォルトでstorage :fileが指定されているので、アップロードしたファイルはpublic/配下に保存されます。<br>
  　　保存されるディレクトリは、store_dirで設定されます。<br>
<br>
### 　3. アップロード画像のカラムを追加
　　次に、アップロード画像の情報を保存するavatarカラムを追加する為に、マイグレーションファイルを作成します。<br>
　　　Ubuntu<br>
  　　　　``$ docker compose exec web rails g migration add_avatar_image_to_users avatar_image:string``<br>
　　上記コマンドで生成されたマイグレーションファイルを以下のように編集してください。<br>

  ```
  　　　　class AddAvatarImageToUsers < ActiveRecord::Migration[7.0]
  　　　　  def change
  　　　　    add_column :users, :avatar_image, :string
  　　　　  end
  　　　　end
```
　　マイグレーションを実行して、usersテーブルにavatarカラムを追加します。<br>
　　　Ubuntu<br>
   ```
  　　　　$ docker compose exec web rails db:migrate
```
　　usersテーブルを確認すると、avatarカラムが追加されています。<br>
　　後ほどアバター画像の登録・更新機能を追加しますが、このavatarカラムには、<br>
　　「画像のデータ」ではなく「画像のファイル名」を保存します。<br>

### 　❇️DBにファイル名を保存する理由とは？
　　ファイル名ではなく画像データを保存すると、データベースサーバーの要領が圧迫するからです。<br>
　　avatarカラムには、どの画像ファイルか確認出来るようにファイル名だけを保存し、画像を表示するときに<br>
　　「画像が格納されてるパス」と「DBに保存されているファイル名」を使います。<br>

　　最後に、データベースに保存出来る様にusers_controllerのuser_paramsにavatarカラムを追加します。
　　app/controllers/users_controller.rb
```
  　　　　def user_params
  　　　　  params.require(:user).permit(:nickname, :age) # 変更前
  　　　　end

  　　　　def user_params
  　　　　  params.require(:user).permit(:nickname, :age, :avatar_image, :avatar_image_cache) # 変更後
  　　　　end
```
　　avatar_imageはユーザーがアップロードした画像ファイルを、avatar_image_cacheはそのキャッシュ情報を含むため、<br>
　　これらを許可リストに追加することで、適切にデータを処理し、保存することができます。<br>

### 　4. アップローダークラスとカラムの紐づけ
　　次に、「アップロード画像用のカラム」と「アップローダークラス」を紐づける為に、以下のコードをモデルに記述します。<br>
　　　app/models/モデル名.rb<br>
```
  　　　　class モデル名 < ActiveRecord::Base
  　　　　  mount_uploader [:カラム名], [アップローダークラス]
  　　　　end
```
　　今回は、ユーザーのアバター画像のアップロード機能を追加するので、以下のように<br>
　　Userモデルに先ほど作成したアバター画像用の「avatar_imageカラム」と「AvatarUploaderクラス」を紐づけます。<br>
　　　app/models/user.rb<br>
```
  　　　　class User < ActiveRecord::Base
  　　　　  mount_uploader :avatar_image, AvatarImageUploader
  　　　　end
```
　　mount_uploader :avatar_image, AvatarImageUploader の記述を加えることで、Userモデルに対して<br>
　　CarrierWave の アップローダークラス（AvatarImageUploader）をマウントします。<br>
　　これにより、Userモデルのインスタンスで avatar_image という属性を持つことができ、<br>
　　画像のアップロードや取得が簡単に行えるようになります。<br>
<br>
　　また、上記を記述することにより、CarrierWave が提供するアップロードされた画像のURLを取得することが<br>
　　できるようになります。(今回の場合 avatar_image_url メソッドを使用して、画像のURLを取得できるようになります。)<br>
<br>
　　これは、CarrierWaveがモデルに対して動的にメソッドを追加し、ファイルの保存場所やURLを管理する機能を<br>
　　提供するためです。この設定により、Userモデルは画像のアップロード機能を持ち、ビューで画像を表示するための<br>
　　URLを生成することが可能となります。<br>

#### 　❇️補足説明
　　今回はアバター画像用のアップロード機能ですが、CSVのアップロード機能を追加する場合は<br>
　　CSV用のカラムとアップローダーを生成してモデルで紐づける必要があります。<br>

### 　5. アバター画像の登録・更新
　　ユーザー登録・編集フォームでアバター画像を追加出来る様に、app/views/users/の既存のフォームに<br>
　　file_filedでファイル選択ボックスを作成します。<br>
```
  　　　　app/views/users/_form.html.erb
  　　　　<div class="field">
  　　　　  <%= form.label :nickname %>
  　　　　  <%= form.text_field :nickname %>
  　　　　</div>

  　　　　<div class="field">
  　　　　  <%= form.label :age %>
  　　　　  <%= form.number_field :age %>
  　　　　</div>

  　　　　<!-- ここから追加するコード -->
  　　　　<div class="field">
  　　　　<%= f.label :avatar_image %>
  　　　　<%= f.file_field :avatar_image, class: "form-control", accept: 'image/*' %>
  　　　　<%= f.hidden_field :avatar_image_cache %>
  　　　　</div>
```
　　上記の11行目からのコードを追加してlocalhost:3000/users/newを確認すると、<br>
　　ファイル選択ボックスが作成されてアバター画像の登録・追加が行える様になっています。<br>
<br>
　　file_field は、HTMLのinput要素のtype属性をfile に設定するためのform_withのヘルパーメソッドです。<br>
　　file_field を使用することにより、ユーザーがファイルを選択してアップロードできるフォームフィールドを生成します。<br>
　　また、acceptオプションを使用して、許可するファイルタイプを指定することも可能です。<br>
　　例えば、file_field, accept: 'image/*' と記述することで、画像ファイルのみを選択できる<br>
　　ファイルアップロードフィールドを作成します。<br>
　　また、アップロードした画像は「public/uploads/モデル名/画像のカラム名/id」配下に保存されます。<br>
<br>
　　hidden_fieldは、ファイルアップロードのキャッシュを保持するための隠しフィールドを生成しています。<br>
　　CarrierWaveでは、ファイルの一時保存を行い、フォームの再送信時にも同じファイルを使用できるように<br>
　　する機能があります。avatar_image_cache は、このキャッシュ機能をサポートするために使用されます。<br>
　　例えば、フォームのバリデーションエラーが発生した場合でも、<br>
　　ユーザーが再度ファイルを選択する必要がないように、このキャッシュ機能を利用します。<br>

　　※隠しフィールドとは、ユーザーには見えない形でフォームにデータを保存するためのものです。<br>
　　　これにより、フォームを送信したときに必要なデータを隠して送ることができます。<br>
<br>
#### 　✅アップロードできるファイルを jpg, jpeg, png, gif のみにする<br>
　　app/uploaders/avatar_image_uploader.rbの extension_allowlist 部分を以下のように編集する。<br>
```
  　　　　class AvatarImageUploader < CarrierWave::Uploader::Base
  　　　　  ... 省略 ...

  　　　　  def extension_allowlist
  　　　　    %w[jpg jpeg gif png]
  　　　　  end

  　　　　  ... 省略 ...
  　　　　end
```
　　extension_allowlist メソッドは、CarrierWave で許可されるファイル拡張子のリストを指定するために使用されます。<br>
<br>
　　このメソッドを定義することで、アップロードできるファイルの種類を制限し、セキュリティとデータの整合性を確保します。<br>
　　上記コードでいえば、extension_allowlistメソッド に %w[jpg jpeg gif png] を指定することで、<br>
　　アップロード可能なファイル形式を jpg、jpeg、gif、png のみに制限しています。<br>
　　これにより、想定外のファイル形式がアップロードされるのを防ぐことができます。<br>
<br>
#### 　✅avatar_imageカラムに値が無い場合は、指定した画像（avatar_placeholder.png）を表示するように設定する<br>
　　app/uploaders/avatar_image_uploader.rbの default_url 部分を以下のように編集する。<br>
```
  　　　　class AvatarImageUploader < CarrierWave::Uploader::Base
  　　　　  ... 省略 ...
  　　　　  def default_url
  　　　　    'avatar_placeholder'
  　　　　  end
  　　　　  ... 省略 ...
  　　　　end
```
　　default_urlメソッド は、ファイルがアップロードされていない場合にデフォルトで表示する画像のURLを指定します。<br>
<br>
　　CarrierWaveでは、アップロードされたファイルが存在しない場合に、このメソッドで定義されたURLを使用して<br>
　　デフォルトの画像を表示します。例えば、default_url 'avatar_placeholder' と設定することで、<br>
　　avatar_placeholder という名前の画像が表示されるようになります。<br>
　　この設定により、画像がアップロードされていない投稿に対しても、デザインを統一することができます。<br>
