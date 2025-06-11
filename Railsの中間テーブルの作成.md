# ★Railsの中間テーブルの作成
## ❇️概要
　多対多の関係を表現したい場合や、その関係に追加の情報（属性）を持たせたい場合に使用する<br>
　中間テーブルの作成方法のメモ。<br>
<br>
<br>
## 中間テーブルの作成方法
　中間テーブルの実装には主に2通りの方法がある。<br>
<br>
　　🥇中間テーブルとモデルをセットで使う方法<br>
　　🥈中間テーブルをモデルなしで使う方法<br>
<br>
　それぞれの運用方法のメリット・デメリット
 | 項目                    | モデルなし（`has_and_belongs_to_many`） | モデルあり（`has_many :through`）   |
| --------------------- | -------------------------------- | ---------------------------- |
| **中間テーブルにモデル定義**      | なし                               | あり                           |
| **中間テーブルの操作**         | Railsで直接操作しづらい                   | 通常のモデルと同様に操作可能               |
| **バリデーション設定**         | 不可※                               | 可能（例：`validates` など）         |
| **追加の属性の付与**          | 不可※                               | 可能（例：`created_at`, `note` 等） |
| **メソッド追加**            | 不可※                             | 可（独自ロジックを追加できる）              |
| **可読性・拡張性**           | 限定的                              | 高い                           |
| **実装の手軽さ**            | シンプルで早い                          | やや手間だが柔軟                     |
| **削除時の制御（dependent）** | 難しい(モデルがないので依存関係が作れない)                              | `dependent: :destroy` が使える   |
| **おすすめされる場面**         | 一時的・単純な多対多関係の場合                  | 拡張性やビジネスロジックが絡む場合            |

※モデルが存在しないため、これらの設定は行えない<br>
<br>
<br>
　次にそれぞれの作成手順を解説していく。<br>
<br>
### 🥇中間テーブルとモデルをセットで使う方法<br>
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
　追記がなければそのままマイグレートを実行(実行するまではテーブルは存在しない)<br>
 ```
docker compose exec web rails db:migrate
```
　作成したテーブルと既存のテーブルにアソシエーションを設定します。<br>
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
　以上で中間テーブルの基本的な構築手順は完了です。あとは実装機能に合わせてコントローラー、モデル、ビューの編集をします。<br>


### 🥈中間テーブルをモデルなしで使う方法<br>
　モデル無しの中間テーブルを作成する。<br>
 ```
docker compose exec web rails generate migration create_join_table :posts, :tags, table_name: :post_tags

create_join_table を使うと、デフォルトで id: false が付与され、主キーが作られません（複合主キー的に扱われる）
table_name:カスタム設定項目。任意のテーブル名(今回は:post_tagsの部分)を指定できる。
```
　コマンドを実行して出来たもの<br>
```
      invoke  active_record
      create    db/migrate/20250609172551_create_join_table.rb
```
　マイグレーションファイルを編集する<br>
```
class CreateJoinTable < ActiveRecord::Migration[6.1]
  def change
    create_join_table :posts, :tags do |t|
      t.index :post_id   #<=post,tag双方向から検索ができるように両方にインデックスをつける
      t.index :tag_id   #<=post,tag双方向から検索ができるように両方にインデックスをつける
    end
  end
end
```
　マイグレーションを実行して中間テーブルを作成(実行するまではテーブルは存在しない)<br>
```
　　　　docker compose exec web rails db:migrate
```
　できたスキーマのテーブル<br>
```
create_table "posts_tags", id: false, force: :cascade do |t|
    t.bigint "post_id", null: false
    t.bigint "tag_id", null: false
    t.index ["post_id"], name: "index_posts_tags_on_post_id"
    t.index ["tag_id"], name: "index_posts_tags_on_tag_id"
  end
```
　モデルファイルのアソシエーションの設定をする。(モデル有の場合とは違うので注意！)<br>
```
# post.rb
has_and_belongs_to_many :tags

# tag.rb
has_and_belongs_to_many :posts
```
　モデル無しの場合の設定の場合、モデル間の依存関係が設定できないのでdependent: :destroyが使えず<br>
　同様の機能を実装するためにはカスタムメソッドを作成する必要がある。<br>
　また、この状態から中間テーブルにモデルを後付けすることも可能です。<br>

　以上で中間テーブルの基本的な構築手順は完了です。あとは実装機能に合わせてコントローラー、モデル、ビューの編集をします。<br>
