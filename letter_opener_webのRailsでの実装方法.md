# ★letter_opener_webのRailsでの実装方法
　　​letter_opener_webは、Rails開発環境で送信されたメールをブラウザ上で確認できる便利なGemです。<br>
​　　これにより、実際のメール送信を行わずに、メールの内容やレイアウトを簡単に確認できます。<br>
　　ただし、メールを送信するにはAction Mailerの設定がされている必要があります。<br>
<br>
## 　✅ 基本的な使い方（開発環境）
### 　　1. Gemのインストール
　　　　Gemfileの開発環境グループに以下を追加し、bundle installを実行します。​<br>
　　　　ruby<br>
```
       group :development do
         gem 'letter_opener_web', '~> 3.0'
       end
```
　　　　Ubuntu<br>
```
       docker compose exec web bundle install
       docker compose restart 　　　　　//Dockerの再起動でgemを適用する
```
<br>

### 　　2. ルーティングの設定
　　　　`config/routes.rb`に以下を追加します。​<br>
　　　　ruby<br>
```
       Rails.application.routes.draw do
         mount LetterOpenerWeb::Engine, at: "/letter_opener" if Rails.env.development?
       end
```
　　　　これにより、http://localhost:3000/letter_opener にアクセスすることで<br>
　　　　送信されたメールをブラウザで確認できます。​<br>
　　　　※3. メール配信方法の設定がされている必要があります。<br>
<br>
### 　　3. メール配信方法の設定
　　　　`config/environments/development.rb`に以下を追加します。​<br>
　　　　ruby
```
       config.action_mailer.delivery_method = :letter_opener_web
```
　　　　これにより、メールは実際に送信されず、ブラウザ上で確認できるようになります。<br>
