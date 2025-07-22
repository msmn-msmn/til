# ★rakeタスクの作成方法
## ❓rakeタスクとは
Rakeとは：Rubyコードをタスク化できる。Railsでは rake db:migrate などよく使う 。<br>

定義場所：Rakefile + lib/tasks/*.rake<br>

namespace と依存関係：整理に有用で環境読み込み依存も可能。<br>

定期実行：Whenever＋Cron（schedule.rb を管理）<br>

便利コマンド：rake --tasks（一覧）、rake --trace（詳細ログ）<br>

複数ファイル管理：Rakefile から .rake ファイルを読み込む仕組みで一括管理できます。<br>


## 🗂️ Rakeタスクの基本構成
定義場所<br>
Rails標準：Rakefile または lib/tasks/*.rake<br>

Rakefile は全ファイルを読み込み、独自タスクは lib/tasks/ に配置するのが一般的です 。<br>
<br>
## タスクファイルの作成
まずは処理を記述するタスクファイルを作成します。<br>
```
$ rails g task ファイル名
```
作成したファイルの中に処理を記述することで定義が出来ます。<br>
<br>
## 基本構文
```
#lib/tasks/*.rake

desc "タスクの説明"
task タスク名:  :environment do
  # 処理内容
end
```
namespace を使えば整理しやすくなります。<br>
```
namespace :db do
  desc "バックアップ実行"
  task backup: :environment do
    # ...
  end
end
```
この書き方により rake db:backup など直感的に呼べます 。<br>

## タスクの依存関係
他のタスクと組み合わせて実行可能です。<br>
```
task deploy: ["db:backup", "db:cleanup_logs"]
```
:environment を依存に含めることで、Rails のモデルや DB にアクセスできます 。<br>

## 引数付きタスク（オプション）
```
task :greet, [:name] => :environment do |t, args|
  puts "Hello, #{args[:name] || 'World'}!"
end
```
実行時に rake greet[アリス] のように引数を渡せます 。<br>


## 🌟 主な活用場面
・外部サービスとのデータ連携（例えば API 呼び出しやCSV同期）<br>
・データベースの定期バックアップ<br>
・古いデータの削除やキャッシュのクリアなど定期的なメンテナンス<br>
<br>
→ Rails に限らず Ruby を使うプロジェクトでは自動化や繰り返し処理の定番手段です。<br>


## 🗓️ 定期実行の実装例：Whenever gem
### Whenever gem とは？
Cron ジョブを Ruby 記法で管理できる gem。<br>

schedule.rb を DSL で書いて、whenever --update-crontab で crontab に反映します <br>

config/schedule.rb の例<br>
```
set :output, { error: "log/cron_error.log", standard: "log/cron.log" }
set :environment, "production"

# 毎日0時にDBバックアップ
every 1.day, at: 'midnight' do
  rake "db:backup"
end

# 1時間ごとにログ削除
every :hour do
  rake "db:cleanup_logs"
end
```
変更後は whenever --update-crontab を実行。crontab -l で登録内容を確認できます。<br>


## 📌 便利コマンド
```
rake --tasks
```
機能：「現在使える全タスクの一覧と説明を表示」<br>
rake -T でも同様の出力が可能です。これにより、どんなタスクがあるのか一目で把握できます。<br>
例：`rake db:migrate」「rake tmp:clear」`など <br>

```
rake --trace
```
機能：「実行時の詳細な処理ログ（スタックトレース）を表示」<br>
エラーや動きが想定と違うときに、どこで何が失敗したのか深く追いやすくなるのでデバッグにとても役立ちます 。<br>
