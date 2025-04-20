# ★ActionMailerを使用したメール送信機能の実装方法（開発環境）
　　ActionMailerとは、Railsでメールを送信するためのフレームワークです。<br>
　　メールのテンプレートや送信設定を簡単に行うことができ、HTML形式やテキスト形式の<br>
　　メールを送信することが可能です。<br>
　　ActionMailerを使用することで、ユーザー認証 や パスワードリセット、通知メールなど、<br>
　　様々なメール機能をアプリケーションに実装することができます。<br>
　　ActionMailerは、Railsの特別なコンポーネントとして動作し、Mailオブジェクトを生成して送信します。<br>
　　メールのビューは app/views/xxxx_mailer.rb に保存され、HTMLとテキストのテンプレートを<br>
　　作成することができます。<br>
### 　　***※開発環境の場合、gem 'letter_opener_web'が導入されていること。*** <br>
  <br>
  
## 　✅メーラーとビューを作成する
## 　　1.メーラーを生成する
　　　　最初に、以下の「メーラー」ジェネレータコマンドを実行して、メーラー関連のクラスを作成します。<br>
　　　　Ubuntu<br>
```
　　　　　$ docker compose exec web bin/rails generate mailer User
```
　　　　生成されるすべてのメーラークラスは、以下のUserMailerと同様にApplicationMailerを継承します。<br>
<br>

### 　　　❇️app/mailers/user_mailer.rb
```
　　　　　class UserMailer < ApplicationMailer
　　　　　end
```
　　　　上のApplicationMailerクラスはActionMailer::Baseを継承しており、すべてのメーラーに共通する属性を以下のように定義できます。<br>

### 　　　❇️app/mailers/application_mailer.rb
```
　　　　　class ApplicationMailer < ActionMailer::Base
　　　　　  default from: "from@example.com"
　　　　　  layout "mailer"
　　　　　end
```
　　　　ジェネレータを使いたくない場合は、app/mailersディレクトリ以下にファイルを手動で作成します。<br>
　　　　このクラスは、必ずActionMailer::Baseを継承するようにしてください。<br>

### 　　　❇️app/mailers/custom_mailer.rb
```
　　　　　class CustomMailer < ApplicationMailer
　　　　　end
```
<br>

## 　　2.メーラーを編集する
　　　　app/mailers/user_mailer.rbに最初に作成されるUserMailerクラスには、メソッドがありません。<br>
　　　　そこで、次に特定のメールを送信するメソッド（アクション）をメーラーに追加します。<br>
　　　　メーラーには「アクション」と呼ばれるメソッドがあり、コントローラーのアクションと同様に、<br>
　　　　ビューを用いてコンテンツを構造化します。コントローラーはクライアントに送り返すHTMLコンテンツを生成しますが、<br>
　　　　メーラーはメールで配信されるメッセージを作成します。<br>
    <br>
### 　　　***Ex.*** UserMailerクラスに以下のようにwelcome_emailというメソッドを追加して、ユーザーの登録済みメールアドレスにメールを送信する。<br>
```
　　　　　class UserMailer < ApplicationMailer
　　　　　  default from: "notifications@example.com"

　　　　　  def welcome_email
　　　　　    @user = params[:user]
　　　　　    @url  = "http://example.com/login"
　　　　　    mail(to: @user.email, subject: "私の素敵なサイトへようこそ")
　　　　　  end
　　　　　end
```
　　　　※メーラー内のメソッド名は、末尾を_emailにする必要はありません。<br>
<br>
　　　　上のメソッドで使われているメーラー関連のメソッドについて簡単に説明します。<br>
<br>
　　　　default: メーラーから送信するあらゆるメールで使われるデフォルト値のハッシュです。<br>
　　　　　　　　　上の例の場合、:fromヘッダーにこのクラスのすべてのメッセージで使う値を1つ設定しています。<br>
　　　　　　　　　この値はメールごとに上書きすることもできます。<br>
　　　　mail: 実際のメールメッセージです。ここでは:toヘッダーと:subjectヘッダーを渡しています。<br>
　　　　　　 また、headersメソッドは、メールのヘッダーをハッシュで渡すか、<br>
　　　　　　 headers[:field_name] = 'value'を呼び出すことで指定できます（上のサンプルで使っていません）。<br>
<br>
　　　　以下のように、メーラーをジェネレータで生成するときに直接アクションを指定することも可能です。<br>
```
　　　　　$ docker compose exec web bin/rails generate mailer User welcome_email
```
　　　　上のコマンドは、空のwelcome_emailメソッドを持つUserMailerクラスを生成します。<br>
<br>
　　　　1個のメーラークラスから複数のメールを送信することも可能です。<br>
　　　　これは、関連するメールをグループ化するのに便利です。たとえば、、UserMailerクラスには<br>
　　　　welcome_emailメソッドの他に、password_reset_emailメソッドも追加できます。<br>

## 　　3.メーラービューを作成する
　　　　次に、welcome_emailアクションに対応するwelcome_email.html.erbというファイル名のビューを<br>
　　　　app/views/user_mailer/ディレクトリの下に作成する必要があります。<br>
　　　　以下は、ウェルカムメールに使えるHTMLテンプレートのサンプルです。<br>
```
　　　　<h1><%= @user.name %>様、example.comへようこそ。</h1>
　　　　<p>
　　　　  example.comへのサインアップが成功しました。
　　　　  ユーザー名は「<%= @user.login %>」です。<br>
　　　　</p>
　　　　<p>
　　　　  このサイトにログインするには、次のリンクをクリックしてください: <%= link_to 'login`, login_url %>
　　　　</p>
　　　　<p>本サイトにユーザー登録いただきありがとうございます。</p>
```
　　　　上のサンプルは<body>タグの内容です。これは、<html>タグを含むデフォルトのメーラーレイアウトに埋め込まれます。<br>
　　　　詳しくは、メーラーのレイアウトを参照してください。<br>
<br>
　　　　また、上のメールのテキストバージョンをapp/views/user_mailer/ディレクトリのwelcome_email.text.erbファイルに<br>
　　　　保存することも可能です（拡張子がhtml.erbではなく.text.erbである点にご注意ください）。<br>
　　　　テキストバージョンも用意しておくと、HTMLレンダリングで問題が発生した場合に信頼できるフォールバックとして機能するため、<br>
　　　　HTML形式とテキスト形式の両方を送信することがベストプラクティスと見なされます。テキストメールの例を次に示します。<br>
```
　　　　<%= @user.name %>様、example.comへようこそ。
　　　　===============================================

　　　　example.comへのサインアップが成功しました。ユーザー名は「<%= @user.login %>」です。

　　　　このサイトにログインするには、次のリンクをクリックしてください: <%= @url %>

　　　　本サイトにユーザー登録いただきありがとうございます。
```
　　　　なお、インスタンス変数（@userと@url）は、HTMLテンプレートとテキストテンプレートの両方で利用可能になります。<br>
　　　　これで、mailメソッドを呼び出せば、Action Mailerは2種類のテンプレート（テキストおよびHTML）を探索して、<br>
　　　　multipart/alternative形式のメールを自動生成します。<br>

## 　　4.動作確認
　　　　ここまで来たら、rails console からメール送信のメソッドを実行してみます。<br>
　　　　以下のようなエラーが出る場合は<br>
```
　　　　> SampleMailer.welcome.deliver_now!
　　　　...
　　　　'initialize': Cannot assign requested address - connect(2) for "localhost" port 25 (Errno::EADDRNOTAVAIL)
```
　　　　これは ActionMailer の設定で、デフォルトが :smtp になっているのにもかかわらず、<br>
　　　　接続先のメールサーバーが存在しないために起きるようです。<br>
　　　　参考:<br>

　　　　https://railsguides.jp/action_mailer_basics.html#delivery-method<br>
　　　　https://qiita.com/yyy_muu/items/76b745abc0d11280f219<br>
　　　　config/environments/development.rb に以下を追記して、<br>
　　　　開発環境では letter_opener_web を使うようにしましょう。<br>
```
　　　　config.action_mailer.delivery_method = :letter_opener_web
```

　　　　これで rails を再起動し、再度メール送信のメソッドを実行します。<br>
　　　　送信したメールのメーラーに対応するテンプレートは以下のとおりです。<br>
　　　　app/views/sample_mailer/welcome.html.erb <br>
```
　　　　<h1>
　　　　  メールです
　　　　</h1>
　　　　<div>
　　　　  <%= @user_name %> さんへ
　　　　</div>
　　　　<div>
　　　　  テストメールです。
　　　　</div>
```
　　　　実行結果は以下のとおりです。<br>
```
　　　　> SampleMailer.welcome.deliver_now!
　　　　...
　　　　=> #<Mail::Message:174580, Multipart: false, Headers: <Date: Mon, 01 Jan 2024 06:34:48 +0000>, <From: hogehoge@example.com>, <To: fugafuga@example.com>, <Message-ID: <65925d085b235_30e9c45238@83e7cccb0712.mail>>, <Subject: subject です>, <Mime-Version: 1.0>, <Content-Type: text/html>, <Content-Transfer-Encoding: base64>>
```
　　　　localhost:3000/letter_opener に再度アクセスし、「Reflesh」を押して、<br>
　　　　送信されたメール内容が確認できれば成功です。<br>
