# ★gem 'config' を使った定数管理の方法
## ❇️gem 'config' とは？
　　gem 'config'とは、Railsアプリケーションで設定管理を簡単に行うためのgemです。<br>
　　環境ごとに異なる設定を一元管理することができ、開発環境、テスト環境、本番環境で異なる設定値を使いたい場合に<br>
　　非常に便利です。config/settings.ymlファイル に共通の設定を書き、config/settings/development.yml、<br>
　　config/settings/production.yml、config/settings/test.yml に環境ごとの設定を書き分けます。<br>
<br>
## ❇️基本的な使い方
### 1.gemのインストール

