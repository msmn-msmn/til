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
　モデル無しの設定の場合、モデル間の依存関係が設定できないのでdependent: :destroyが使えず<br>
　同様の機能を実装するためにはカスタムメソッドを作成する必要がある。<br>
　また、この状態から中間テーブルにモデルを後付けすることも可能です。<br>

　以上で中間テーブルの基本的な構築手順は完了です。あとは実装機能に合わせてコントローラー、モデル、ビューの編集をします。<br>


### 🥉中間テーブルにモデルを後付けする方法（既存のテーブルをそのまま使う）<br>
　モデル無しの設定の時、後で関係に追加の情報（属性）を持たせたくなった等のモデル有のメリットを採用したくなった場合に<br>
　後付けでモデルを追加する方法を解説する。<br>

　


### 　***作成手順*** 
　例えば、すでに posts_tags テーブル（create_join_table :posts, :tags により生成）が存在している場合、次のようにします。<br>
　まずは中間テーブルに対応したモデル名をモデルをマイグレーション無しのオプションで作成します。<br>
 <br>
```
docker compose exec web rails generate model PostsTag --skip-migration

PostsTag：既存テーブル posts_tags に対応するモデル名（Railsの規約：スネークケース→キャメルケース）
--skip-migration：マイグレーションファイルを生成せず、app/models/posts_tag.rb だけを作成
```

　次にモデル無しからモデル有になったのでモデル同士のアソシエーション設定を忘れずに変更します。<br>
　※ このタイミングでアソシエーションを`has_many :through`に変更することを忘れずに。br>　
 
```
#モデル無しの場合のアソシエーション
# post.rb
has_and_belongs_to_many :tags

# tag.rb
has_and_belongs_to_many :posts



#モデル有の場合のアソシエーション     #<=こちらの設定に変更する
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
　この方法なら：<br>
```
既存の中間テーブルやそのデータはそのまま維持
モデルにバリデーションやメソッドを追加可能
アソシエーションも has_many :through に切り替え可能
```
　とすることが出来ます。<br>
 <br>
　`create_join_table`で生成された中間テーブルには`id`カラムがないため、後付けモデルでは主キーのないモデルとして扱われます。<br>
　そのため`update`や`destroy`に制限が出る場合があります。必要であれば別途`add_column :id`などで`ID`を付与することも可能です。<br>
 <br>
　Railsの ActiveRecord は、基本的に1つの主キー（通常は id）があることを前提に動いているので<br>
　主キーがないと…<br>
 ```
・PostsTag.find(1) みたいな「idで探す」操作ができない
・update, destroy, dependent: :destroy など一部のメソッドがうまく動かないことがある
・バリデーションやフォームの扱いで、id による参照がないと混乱することがある
```
　という問題が発生する可能性があります。<br>
 <br>
　また、中間テーブルに属性(カラム)を追加するには<br>
```
docker compose exec web rails generate migration AddRoleToPostsTags role:string

Role:追加したいカラム名
PostsTags:カラムを追加したいテーブル名
role:データ型
```
　とカラム名を追加するマイグレーションを発行します。<br>
　発行されたマイグレーションファイルの中身は<br>
 ```
# db/migrate/XXXXXXXXXXXXXX_add_role_to_posts_tags.rb
class AddRoleToPostsTags < ActiveRecord::Migration[6.1]
  def change
    add_column :posts_tags, :role, :string
  end
end
```
　となります。これを<br>
 ```
　　　　docker compose exec web rails db:migrate
```
　でマイグレートを実行して属性(カラム)追加完了です。<br>

 
### 　***補足として*** 
　後付けではないが、開発段階でデータは消えてもいいから設計自体をやり直したい時の方法を紹介する。<br>
　具体的には既にあるテーブルを削除して新たに任意のテーブルを作成します。<br>
 <br>
 ◎マイグレーションファイル削除の手順<br>
 　・まず現状のマイグレーションを確認します。<br>
```
docker compose exec web rails db:migrate:status

#結果
 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20210707021252  Create users
   up     20210707021454  Create profiles
   up     20210707041601  Create posts
   up     20210709035845  Create comments
   up     20250610045201  Create tags
   up     20250610051747  Create join table　　　<=これを削除したい
```

　・次に削除したいファイルのステータスをdownにする<br>
```
docker compose exec web rails db:migrate:down VERSION=20250610051747

VERSION=20250610051747はdownさせたいファイルのMigration IDを指定する。
```

　・downしているか確認<br>
```
docker compose exec web rails db:migrate:status

#結果
 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20210707021252  Create users
   up     20210707021454  Create profiles
   up     20210707041601  Create posts
   up     20210709035845  Create comments
   up     20250610045201  Create tags
  down    20250610051747  Create join table
```

　・downしたマイグレーションファイルの削除<br>
```
rm db/migrate/20250610051747_create_join_table.rb

#結果
 Status   Migration ID    Migration Name
--------------------------------------------------
   up     20210707021252  Create users
   up     20210707021454  Create profiles
   up     20210707041601  Create posts
   up     20210709035845  Create comments
   up     20250610045201  Create tags
```
　‼️注意<br>
 　・必ずdownさせてから削除する<br>
 　・Migration IDの指定を間違えないように注意する<br>
