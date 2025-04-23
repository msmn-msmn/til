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
<br>
　　　作成されたファイルを以下のように編集します。<br>
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
　　　docker compose exec web rails generate mailer UserMailer reset_password_email
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
        
　　　# DBからデータを受け取れていれば、パスワードリセットの方法を記載したメールをユーザーに送信する。
　　　#（ランダムトークン付きのURL/有効期限付き）
　　　    @user.deliver_reset_password_instructions! if @user
        
　　　# メールアドレスが見つかったかどうかにかかわらず、ユーザーに案内を送信したことを通知します。
　　　# これは、システムに存在するメールアドレス情報を攻撃者に漏らさないためです。
　　　    redirect_to(root_path, :notice => 'Instructions have been sent to your email.')
　　　  end
    
　　　# パスワードリセットフォームです。
　　　  def edit
　　　# postされてきた値を取得
　　　    @token = params[:id]
　　　# リクエストで送信されてきたトークンを使って、ユーザーの検索を行い, 有効期限のチェックも行う。
　　　# トークンが見つかり、有効であればそのユーザーオブジェクトを@userに格納する
　　　    @user = User.load_from_reset_password_token(params[:id])

　　　# @userがnilまたは空の場合、not_authenticatedメソッドを実行する
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

　　　# password_confirmation属性の有効性を確認
         @user.password_confirmation = params[:user][:password_confirmation]
　　　# change_passwordメソッドで、パスワードリセットに使用したトークンを削除し、パスワードを更新する
         if @user.change_password(params[:user][:password])
           redirect_to(root_path, :notice => 'Password was successfully updated.')
         else
           render action: 'edit'
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
　
### 　***✅Userモデルにバリデーションを追記<br>
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

## 　4.viewファイル作成
　　　ログインページに下記のリンクを追加します。<br>
　　　***app/views/user_sessions/new.html.erb***<br>
   ```
        <%= link_to 'passwordをお忘れの方はコチラ', new_password_reset_path %>
```

　　　パスワード申請フォームと再設定フォームを作成します。<br>
　　　***password_resets/new.html.erb***<br>
   ```
        <h3>パスワードリセット申請</h3>
        <%= form_with url: password_resets_path do |f| %>
            <%= f.label :email %>
            <%= f.email_field :email %>
            <%= f.submit '申請' %>
        <% end %>
```

　　　***password_resets/edit.html.erb***<br>
   ```
        <h3>パスワード再設定</h3>
        <%= form_with model: @user, url: password_reset_path(@token) do |f| %>
            <%= render 'shared/error_message', object: f.object %>
        	  <%= f.label :email %>
            <%= @user.email %>

            <%= f.label :password %>
            <%= f.password_field :password %>

            <%= f.label :password_confirmation %>
            <%= f.password_field :password_confirmation %>

            <%= f.submit 'リセット' %>
        <% end %>
```

## 　5.letter_opener_webの導入
　　　letter_opener_webを使えば、開発環境では実際にメールを送信することなくメールが送信されているか確認することができます。<br>
　　<br>
### 　***✅letter_opener_webのインストール***
　　　letter_opener_webは開発環境でしか使用しない為、development配下に記述します。<br>
　　　***Gemfile***<br>
   ```
        group :development do
          gem 'letter_opener_web', '~> 2.0'
        end
```
　　　ターミナルでbundle installします。Dockerを使用している場合はコンテナ内で実行します。<br>
　　　***Ubuntu***<br>
```
        docker compose exec web bundle install
        docker compose restart 　　　　　//Dockerの再起動でgemを適用する
```   


### 　***✅ルーティングの設定***
　　　***routes.rb***<br>
   ```
        Rails.application.routes.draw do
          mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?

        end
```
　　　***config/environments/development.rb***<br>
   ```
        config.action_mailer.delivery_method = :letter_opener_web # 送信方法を指定
        config.action_mailer.perform_deliveries = true # メールを実際に送信するかどうかを指定
```

　　　こちらのアドレスでletter_opener_webを開けます。<br>
   ```
　　　http://localhost:3000/letter_opener
```

## 　6.gem configを導入する
　　　configは定数を管理するgemです。configを導入することで、ローカル環境でのメールをletter_opener_web、<br>
　　　本番環境のメールを指定したプロトコルでなど、簡単に分けることが可能です。<br>
   
### 　***✅gemのインストール***
　　　Gemfileに追記してから、bundle installを行います。<br>
　　　***Gemfile***<br>
   ```
　　　　gem 'config'
```
　　　***Ubuntu***<br>
```
　　　　$ docker compose exec web bundle install
　　　　$ docker compose restart 　　　　　//Dockerの再起動でgemを適用する
```
　　　その後にｇコマンドで設定用のファイルを生成します。<br>
　　　***Ubuntu***<br>
```
　　　　$ docker compose exec web rails g config:install
```
　　　これにより、以下の結果が得られます。<br>
   ```
　　　 create  config/initializers/config.rb
　　　 create  config/settings.yml
　　　 create  config/settings.local.yml
　　　 create  config/settings
　　　 create  config/settings/development.yml
　　　 create  config/settings/production.yml
　　　 create  config/settings/test.yml
　　　 append  .gitignore
```
## 　　生成されたファイル
　　　生成されたそれぞれの設定ファイルの役割は以下の通りです。<br>

| ファイル名 | 役割 |
|:----------|:-----|
| config/initializers/config.rb | configの設定ファイル|
| config/settings.yml | すべての環境で利用する定数を定義|
| config/settings.local.yml | ローカル環境のみで利用する定数を定義 |
| config/settings/development.yml | 開発環境のみで利用する定数を定義 |
| config/settings/production.yml	| 本番環境のみで利用する定数を定義 |
| config/settings/test.yml	| テスト環境のみで利用する定数を定義 |

　　　なお、インストール時に.gitignoreに追記されていた内容は以下の通り。<br>
```
　　　config/settings.local.yml
　　　config/settings/*.local.yml
　　　config/environments/*.local.yml
```

　　　なので、上記の設定ファイル＋これから作成するファイルのうち、localで使うものはgithubに上げないようになってます。<br>

## 　　🔄 環境ごとの設定の優先順位
　　　config gemは、以下の順序で設定ファイルを読み込みます。後に読み込まれたファイルの設定が優先されます。<br>
<br>
　　　1. config/settings.yml<br>
　　　2. config/settings/#{environment}.yml<br>
　　　3. config/environments/#{environment}.yml<br>

　　　4. config/settings.local.yml<br>
　　　5. config/settings/#{environment}.local.yml<br>
　　　6. config/environments/#{environment}.local.yml<br>
   <br>
　　　これにより、一般的な設定を共通ファイルに記述し、環境や開発者ごとの特定の設定をローカルファイルで上書きすることが可能です。<br>

　　
### 　***✅開発環境での定数設定***

　　　***config/environments/development.rb***
   ```
　　　config.action_mailer.delivery_method = :letter_opener_web 
　　　config.action_mailer.default_url_options = Settings.default_url_options.to_h
```
　　　同ファイル内にaction_mailerのコードがあるので、その下辺りに記述すると分かりやすいです。<br>
<br>
　　　`config.action_mailer.delivery_method = :letter_opener_web`<br>
　　　Railsアプリケーションでメールを送信する際に使用する配送方法（delivery method）を設定しています。<br>
<br>
　　　`config.action_mailer.default_url_options =　Settings.default_url_options.to_h`<br>
　　　ActionMailerがメールを生成する際に使用するデフォルトのURLオプションを設定しています。<br>
　　　以下のdevelopment.ymlで記述された定数default_url_optionsを読み込んでいます。<br>
　　

　　　***config/settings/development.yml***
   ```
　　　default_url_options:
　　　host: 'localhost:3000'
```
　　　開発環境でのhost設定です。<br>

　　
### 　***✅本番環境での定数設定***

　　　***config/environments/production.rb***
   ```
　　　config.action_mailer.default_url_options = Settings.default_url_options.to_h
```

　　　***config/settings/production.yml***
```
　　　default_url_options:
　　　  protcol: 'https'
　　　  host: 'example.com'
```
