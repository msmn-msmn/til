# ★rakeタスクの作成方法
## ❓rakeタスクとは
Rakeとは：Rubyコードをタスク化できる。Railsでは rake db:migrate などよく使う 。

定義場所：Rakefile + lib/tasks/*.rake

namespace と依存関係：整理に有用で環境読み込み依存も可能

定期実行：Whenever＋Cron（schedule.rb を管理）

便利コマンド：rake --tasks（一覧）、rake --trace（詳細ログ）

複数ファイル管理：Rakefile から .rake ファイルを読み込む仕組みで一括管理できます


## 🗂️ Rakeタスクの基本構成
定義場所
Rails標準：Rakefile または lib/tasks/*.rake

Rakefile は全ファイルを読み込み、独自タスクは lib/tasks/ に配置するのが一般的です 

## 基本構文
```
desc "タスクの説明"
task :タスク名 => :environment do
  # 処理内容
end
```
namespace を使えば整理しやすくなります：
```
namespace :db do
  desc "バックアップ実行"
  task backup: :environment do
    # ...
  end
end
```
この書き方により rake db:backup など直感的に呼べます 

## タスクの依存関係
他のタスクと組み合わせて実行可能です：
```
task deploy: ["db:backup", "db:cleanup_logs"]
```
:environment を依存に含めることで、Rails のモデルや DB にアクセスできます 

## 引数付きタスク（オプション）
```
task :greet, [:name] => :environment do |t, args|
  puts "Hello, #{args[:name] || 'World'}!"
end
```
実行時に rake greet[アリス] のように引数を渡せます 
