# ★Railsのモデルの作り方
## 　大まかな流れ
　　モデルクラスとマイグレーションファイルを生成<br>
  　　　　　　　　　　　↓<br>
　　マイグレーションファイルを実行して、設定内容をdbのテーブル構造に反映させる<br>
    <br>
## 　modelの作り方
　　ターミナルで以下のコマンドを実行し、モデルクラスとマイグレーションファイルを生成 <br>
  
```
　　rails generate model name field:type options
```
| 用語 | 意味 |
|----------------|---------------------|
| name        | モデル名(頭文字は大文字かつ単数形)            |
| field       | カラム名          |
| type        | データ型            |
| options         | オプション要素              |

### 　オプション要素の種類
| オプション          | 説明                                                    | 例                                   |
| -------------- | ----------------------------------------------------- | ----------------------------------- |
| `null: false`  | このカラムは `NULL` を許容しない                                  | `title:string:null:false`           |
| `default:`     | このカラムの**初期値**を設定                                      | `status:string:default:"draft"`     |
| `index: true`  | このカラムに**インデックス**を追加                                   | `email:string:index`                |
| `unique: true` | このカラムに**ユニークインデックス**を追加（`index: { unique: true }`と同義） | `email:string:uniq`                 |
| `limit:`       | カラムのサイズ制限（主に `string` や `integer`）                    | `username:string:limit:50`          |
| `precision:`   | 数値カラムの**全体の桁数**（小数を含む）                                | `price:decimal:precision:8`         |
| `scale:`       | 数値カラムの**小数点以下の桁数**                                    | `price:decimal:precision:8,scale:2` |
| `array: true`  | PostgreSQLで使用。**配列型**のカラム                             | `tags:string:array:true`            |
| `comment:`     | マイグレーションファイルに**カラムのコメント**を付加（Rails 5.2+）              | `age:integer:comment:"年齢"`          |
| `foreign_key:` | 外部キー制約（別途 `references` 型と一緒に使う）                       | `user:references:foreign_key:true`  |

　　でmodelクラスとdb用のマイグレーションファイルを生成される<br>
　　この時点では、生成したモデルのテーブルはまだできていない<br>
  
　　マイグレーションファイルのチェンジメソッドに追記が無い場合<br>
```
　　rails db:migrate
```
　　でマイグレーションファイルを実行してテーブルを生成する<br>
  
　　テーブルにおけるカラムは<br>
　　・rails gであらかじめ入力してマイグレーションファイルに反映させるやり方<br>
　　・モデル名だけでrails gをしてあとでマイグレーションファイルに任意のカラム設定を追記するやり方<br>
　　がある<br>
<br>
　　モデルのクラス名が2語以上の複合語である場合、Rubyの規約であるキャメルケース<br>
　　（語頭を大文字にしてスペースなしでつなぐ記法。例：SampleName）に従ってください。<br>
　　一方、テーブル名はスネークケース（小文字とアンダースコアで構成する記法。例：sample_name）<br>
　　にしなければなりません。以下の例を参照ください。<br>
<br>
　　モデルのクラス名: 単数形、語頭を大文字にする（例: BookClub）<br>
　　データベースのテーブル名: 複数形、語はアンダースコアで区切る（例: book_clubs）<br>
<br>
| モデル / クラス | テーブル / スキーマ |
|----------------|---------------------|
| Article        | articles            |
| LineItem       | line_items          |
| Product        | products            |
| Person         | people              |

