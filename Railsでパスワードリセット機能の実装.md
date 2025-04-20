# ★パスワードリセット機能の実装
## ❇️おおまかな実装内容
  　　・***gem 'sorcery'*** の***Reset password***モジュールをインストールする<br>
  　　・Mailerを作成しパスワードリセット用のメールを送信できるようにする(***ActionMailer***を使用する)<br>
  　　・コントローラとビューの作成<br>
  　　・***gem 'letter_opener_web`*** を導入しローカル環境でメールが送信されているか確認する<br>
  　　・***gem 'config'*** を導入し、開発環境と本番環境の設定を分ける<br>

## ❇️実装手順
## 　1.モジュールをインストールする
　　　まずはパスワードリセット用のモジュールをインストールします<br>
　　　Ubuntu<br>
```
        docker compose exec web rails g sorcery:install reset_password --only-submodules
```
　　　作成されたファイルがこちら。<br>
　　　config/initializers/sorcery.rb<br>
　　　app/models/user.rb<br>
　　　db/migrate/20231105121233_sorcery_reset_password.rb<br>

　　　***config/initializers/sorcery.rb***<br>
   ```
　　　Rails.application.config.sorcery.submodules = [:reset_password]
```
　　　***migrate/(タイムスタンプ)_sorcery_reset_password.rb***<br>
   ```
　　　class SorceryResetPassword < ActiveRecord::Migration[7.0]
　　　  def change
　　　    add_column :users, :reset_password_token, :string, default: nil
　　　    add_column :users, :reset_password_token_expires_at, :datetime, default: nil
　　　    add_column :users, :reset_password_email_sent_at, :datetime, default: nil
　　　    add_column :users, :access_count_to_reset_password_page, :integer, default: 0

　　　    add_index :users, :reset_password_token
　　　  end
　　　end
```
　　　上記で追加されたカラムの内容は以下のとおりです。<br>
| カラム |	内容 |
|:-------|:----|
| reset_password_email_sent_at | パスワードリセット用のメールを送信した時刻を保持する |
| reset_password_token_expires_at	| トークンの有効期限が切れる時刻を記録する |
| reset_password_token | トークン文字列を保持する |
| access_count_to_reset_password_page	|パスワードリセット画面にアクセスした回数を記録する |

　　　***※access_count_to_reset_password_page***<br>
　　　　N回以上はパスワードリセット画面に移動できないようにするなどの機能を実装する時に用いられるカラム<br>
    <br>
　　　作成されたファイルを確認したらコンソールでdb:migrateします。<br>
Ubuntu<br>
```
　　　docker compose exec web rails db:migrate
```

## 　2.Mailerの作成
### 　　　***✅パスワードリセット用のMailerを作成***
　　　最初にActionMailerを使用してパスワードリセット用のMailerを作成します。
  ```
rails g mailer UserMailer reset_password_email
```
　　　※reset_password_emailを付けることで
　　　作成されるファイル
　　　app/mailers/user_mailer.rb
　　　app/views/user_mailer
　　　app/views/user_mailer/reset_password_email.text.erb
　　　app/views/user_mailer/reset_password_email.html.erb
　　　test/mailers/user_mailer_test.rb
　　　test/mailers/previews/user_mailer_preview.rb

　
　
### 　　　***✅メールを送信するためのメソッドを作成***

app/mailers/user_mailer.rb
class UserMailer < ApplicationMailer
  default from: 'from@example.com'

  def reset_password_email(user)
    @user = User.find(user.id)
    @url  = edit_password_reset_url(@user.reset_password_token)
    mail(to: user.email,
         subject: t('defaults.password_reset'))
  end
end
default from: 'from@example.com'メールの送信元のアドレスを指定できます。
mail(to: user.email,subject: 'パスワードリセット')メールの宛先、件名を指定します。
@userと@urlはmailのviewで使用します。

　
　
次に、reset_password サブモジュールを追加し、使用するメーラーを定義します。

config/initializers/sorcery.rb
Rails.application.config.sorcery.submodules = [:reset_password, blabla, blablu, ...]

Rails.application.config.sorcery.configure do |config|
  config.user_config do |user|
    user.reset_password_mailer = UserMailer
  end
end
user.reset_password_mailer =
この記述が405行目にあります（現在）コメントアウトを外し、UserMailerを追加しましょう。

　
### 　　　***✅メールの設定***

app/views/user_mailer/reset_password_email.html.erb
<p><%= @user.name %>様</p>

<p>パスワード再発行のご依頼を受け付けました。</p>

<p>こちらのリンクからパスワードの再発行を行ってください。</p>

<p><a href="<%= @url %>"><%= @url %></a></p>
app/views/user_mailer/reset_password_email.text.erb
<%= @user.name %>様

パスワード再発行のご依頼を受け付けました。

こちらのリンクからパスワードの再発行を行ってください。

<%= @url %>
