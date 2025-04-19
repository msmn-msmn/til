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
