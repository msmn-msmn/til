# ★Railsの中間テーブルの作成
## ❇️概要
　アソシエーションが多対多になった時やデータに属性を持たせたいときに使う<br>
　中間テーブルの作成方法のメモ。<br>
<br>
<br>
## 中間テーブルの作成方法
　中間テーブルの運用方法は主に２つある。<br>
<br>
　　🥇中間テーブルとモデルをセットで使う方法<br>
　　🥈中間テーブルをモデルなしで使う方法<br>
<br>
　それぞれの作成手順を解説していく。<br>
<br>
<br>
### 🥇中間テーブルとモデルをセットで使う方法
　最初にモデルファイルを作成する。<br>
 ```
docker compose exec web rails generate model PostsTag post:references tag:references
```

　このコマンドを実行すると以下のファイルが作成される。<br>
 
```
invoke  active_record
      create    db/migrate/20250611065137_create_posts_tags.rb
      create    app/models/posts_tag.rb
```

　マイグレーションファイルの中身は<br>

 ```
#db/migrate/20250611065137_create_posts_tags.rb
class CreatePostsTags < ActiveRecord::Migration[6.1]
  def change
    create_table :posts_tags do |t|
      t.references :post, null: false, foreign_key: true    #<=referencesと宣言したことで自動で入力される
      t.references :tag, null: false, foreign_key: true    #<=referencesと宣言したことで自動で入力される

      t.timestamps
    end
  end
end
```
　また、references のカラム型は自動で`bigint`になります。<br>
　何故かというと、Rails 5以降、主キー（id）のデフォルトのデータ型が `bigint` になったからです。<br>
　だから t.references :post って書くと、Railsは自動的に post_id を `bigint` 型として生成するようになっています。<br>
<br>
　追記がなければそのままマイグレートを実行
 ```
docker compose exec web rails db:migrate
```
作成したテーブルと既存のテーブルにアソシエーションを設定します。
```
# post.rb
has_many :posts_tags
has_many :tags, through: :posts_tags   #<=through: :posts_tagsで中間テーブルを通してやり取りするとRailsに伝えている

# tag.rb
has_many :posts_tags
has_many :posts, through: :posts_tags   #<=through: :posts_tagsで中間テーブルを通してやり取りするとRailsに伝えている

# posts_tag.rb ←中間モデル
belongs_to :post
belongs_to :tag
validates :tag_id, uniqueness: { scope: :post_id }   #<=postとtagの組み合わせの一意性を守るためにバリデーションを追加

```
