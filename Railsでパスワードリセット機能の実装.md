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
### 　　***✅パスワードリセット用のMailerを作成***
　　　最初にActionMailerを使用してパスワードリセット用のMailerを作成します。<br>
　　　Ubuntu<br>
  ```
　　　docker compose exec web rails g mailer UserMailer reset_password_email
```

```
　　　※reset_password_emailと付けることでジェネレータで生成するときに直接アクションを指定することも可能です。
　　　　以下のように、メーラーをジェネレータで生成するときに直接アクションを指定することも可能です。
　　　　　$ docker compose exec web bin/rails generate mailer User welcome_email

　　　　上のコマンドは、空のwelcome_emailメソッドを持つUserMailerクラスを生成します。
```    
　　　作成されるファイル<br>
　　　app/mailers/user_mailer.rb<br>
　　　app/views/user_mailer<br>
　　　app/views/user_mailer/reset_password_email.text.erb<br>
　　　app/views/user_mailer/reset_password_email.html.erb<br>
　　　test/mailers/user_mailer_test.rb<br>
　　　test/mailers/previews/user_mailer_preview.rb<br>
<br>　

### 　　***✅メールを送信するためのメソッドを作成***
　　　***app/mailers/user_mailer.rb***
   ```
　　　class UserMailer < ApplicationMailer
　　　  default from: 'from@example.com'

　　　  def reset_password_email(user)
　　　    @user = User.find(user.id)
　　　    @url  = edit_password_reset_url(@user.reset_password_token)
　　　    mail(to: user.email, subject: t('defaults.password_reset'))
　　　  end
　　　end
```

　　　***default from***: 'from@example.com'メールの送信元のアドレスを指定できます。<br>
　　　***mail(to: user.email,subject***: 'パスワードリセット')メールの宛先、件名を指定します。<br>
　　　@userと@urlはmailのviewで使用します。<br>
　
### 　　***✅reset_password サブモジュールを追加し、使用するメーラーを定義する***<br>
　　　***config/initializers/sorcery.rb***<br>
   ```
　　　Rails.application.config.sorcery.submodules = [:reset_password, blabla, blablu, ...]

　　　Rails.application.config.sorcery.configure do |config|
　　　  config.user_config do |user|
　　　    user.reset_password_mailer = UserMailer
　　　  end
　　　end
```

　　　`user.reset_password_mailer =`　　　ファイル内のこの記述のコメントアウトを外し、UserMailerを追加します。<br>

　
### 　　***✅メールの設定***

　　　***app/views/user_mailer/reset_password_email.html.erb***<br>
   ```
　　　<p><%= @user.name %>様</p>
　　　<p>パスワード再発行のご依頼を受け付けました。</p>
　　　<p>こちらのリンクからパスワードの再発行を行ってください。</p>
　　　<p><a href="<%= @url %>"><%= @url %></a></p>
   ```

　　　***app/views/user_mailer/reset_password_email.text.erb***<br>
   ```
　　　<%= @user.name %>様
　　　パスワード再発行のご依頼を受け付けました。
　　　こちらのリンクからパスワードの再発行を行ってください。
　　　<%= @url %>
```
### 　　***❓なんで２種類のviewを用意する❓***<br>
```
　　　　テキストバージョンも用意しておくと、HTMLレンダリングで問題が発生した場合に信頼できるフォールバックとして
　　　　機能するため、HTML形式とテキスト形式の両方を送信することがベストプラクティスと見なされます。

　　　　なお、インスタンス変数（@userと@url）は、HTMLテンプレートとテキストテンプレートの両方で利用可能になります。
　　　　これで、mailメソッドを呼び出せば、Action Mailerは2種類のテンプレート（テキストおよびHTML）を探索して、
　　　　multipart/alternative形式のメールを自動生成します。
```

## 　3.controller作成
### 　***✅コントローラーの生成***<br>
　　　すべてのパスワードアクションを処理するコントローラーを生成します。<br>
　　　Ubuntu<br>
```
        docker compose exec web rails g controller PasswordResets create edit update
```

　　　***app/controllers/password_resets_controller.rb***<br>
```
　　　class PasswordResetsController < ApplicationController
　　　# Rails 5 以降では、ApplicationController に
　　　# before_action :require_login
　　　# が宣言されていないとエラーになります。
　　　  skip_before_action :require_login
    
　　　# パスワードリセットをリクエストします。
　　　# ユーザーがリセットパスワードフォームにメールアドレスを入力して送信したときにここに来ます。
　　　  def create 
　　　    @user = User.find_by_email(params[:email])
        
　　　# この行は、ランダムなトークン付きURLの手順を含むメールをユーザーに送信します。
　　　    @user.deliver_reset_password_instructions! if @user
        
　　　# メールアドレスが見つかったかどうかにかかわらず、ユーザーに案内を送信したことを通知します。
　　　# これは、システムに存在するメールアドレス情報を攻撃者に漏らさないためです。
　　　    redirect_to(root_path, :notice => 'Instructions have been sent to your email.')
　　　  end
    
　　　# パスワードリセットフォームです。
　　　  def edit
　　　    @token = params[:id]
　　　    @user = User.load_from_reset_password_token(params[:id])

　　　    if @user.blank?
　　　      not_authenticated
　　　      return
　　　    end
　　　  end
      
  # ユーザーがリセットパスワードフォームを送信したときに発火します。
　　　  def update
         @token = params[:id]
         @user = User.load_from_reset_password_token(params[:id])

         if @user.blank?
           not_authenticated
             return
         end

    # 次の行は、パスワード確認バリデーションを機能させます。
         @user.password_confirmation = params[:user][:password_confirmation]
    # 次の行は、一時トークンをクリアし、パスワードを更新します。
         if @user.change_password(params[:user][:password])
           redirect_to(root_path, :notice => 'Password was successfully updated.')
         else
           render :action => "edit"
         end
　　　  end
　　　end
```
　
### 　***✅ルーティングの設定***
　　　すべてのパスワードアクションのルーティングを設定します。<br>
　　　***config/routes.rb***<br>
   ```
        resources :password_resets, only: [:create, :edit, :update]
```
　
　　　Userモデルにバリデーションを追記<br>
　　　***models/user.rb***<br>
```
        validates :reset_password_token, presence: true, uniqueness: true, allow_nil: true
```

　　　`uniqueness: true`　　　tokenは一意なものでなければならないのでユニーク成約を付けます。<br>
 　　　　　　　　　　　　　　これがないと別の人のパスワードを変更できてしまう可能性があります。<br>
<br>
　　　`allow_nil: true`　　　 対象の値がnilの場合にバリデーションをスキップするもの。<br>
 　　　　　　　　　　　　　　トークンの初期値はnilのため、このままではユニーク制約に引っかかりユーザーを<br>
 　　　　　　　　　　　　　　複数登録できなくなる問題が発生してしまいます。<br>
 　　　　　　　　　　　　　　この問題を回避するために付与します。<br>
