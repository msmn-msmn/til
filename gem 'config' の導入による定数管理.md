# ★gem 'config' を使った定数管理の方法
## ❇️gem 'config' とは？
　　gem 'config'とは、Railsアプリケーションで設定管理を簡単に行うためのgemです。<br>
　　環境ごとに異なる設定を一元管理することができ、開発環境、テスト環境、本番環境で異なる設定値を使いたい場合に<br>
　　非常に便利です。config/settings.ymlファイル に共通の設定を書き、config/settings/development.yml、<br>
　　config/settings/production.yml、config/settings/test.yml に環境ごとの設定を書き分けます。<br>
<br>
## ❇️基本的な使い方
## 　1.gemのインストール
　　　Gemfileに追記してから、bundle installを行います。<br>
　　　Gemfile<br>
   ```
　　　　gem 'config'
```
　　　Ubuntu<br>
```
　　　　$ docker compose exec web bundle install
　　　　$ docker compose restart 　　　　　//Dockerの再起動でgemを適用する
```
　　　その後にｇコマンドで設定用のファイルを生成します。<br>
　　　Ubuntu<br>
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

　　　4.config/settings.local.yml<br>
　　　5.config/settings/#{environment}.local.yml<br>
　　　6.config/environments/#{environment}.local.yml<br>
   <br>
　　　これにより、一般的な設定を共通ファイルに記述し、環境や開発者ごとの特定の設定をローカルファイルで上書きすることが可能です。<br>
