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
  　　　　``$ docker compose  bundle exec rails g uploader アップローダー名``<br>
