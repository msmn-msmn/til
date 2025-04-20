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

Mailerの作成
