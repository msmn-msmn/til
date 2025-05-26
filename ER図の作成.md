# ★ER図の作成方法(PlantUML版)
## 目的
　アプリのテーブル設計の為のER図が作成したい

## 経緯
　テーブル設計の為のER図を描くために最初にDrowIOを使っていたけど、使いずらかったのでPlantUMLに変えた。<br>
　その時に行ったPlantUMLの環境構築方法の備忘録。<br>
 <br>
 <br>

## PlantUMLの環境構築
### 1.Java(JDK)のインストール
　以下ページからJavaのインストーラーをダウンロードする。<br>
　　　https://java.com/en/download/

### 2.Graphvizのインストール
　　以下からインストーラーをダウンロード<br>
　　　https://www.graphviz.org/download/<br>
　　途中の選択は`Add Graphviz to the system PATH for all users`<br>

### 3.PlantUMLプラグインのインストール
　　VSCodeの拡張機能からPlantUML(デベロッパーはjebbs)をインストールする<br>
<br>
<br>
<br>
## PlantUMLの使い方
　ファイルの拡張子を.puまたは.pumlや.plantumlにして作図用のファイルを新規作成する。<br>
　VSCodeで作図用ファイルをPlantUMLの構文を使用して、コードで記述する。<br>
　コードを書いてAlt + d で図のプレビュー表示<br>

## PlantUMLのコードの書き方
## 1.サンプルコード<br>
  ```
    @startuml sample_site
    entity "Users" {
        *user_id : integer <<PK>>
        --
        *gender_id : integer <<FK>>
        *age_range_id : integer <<FK>>
        *username : string, null: false
        *email : string, null: false
        *selfintro : text
        *age : integer
        *crypted_password : string, null: false
        *salt : string
        *created_at : datetime
        *updated_at : datetime
    }

    entity "Genders" {
        *gender_id : integer <<PK>>
        --
        *gender_name : string
    }

    entity "Age_ranges" {
        *age_range_id : integer <<PK>>
        --
        *range_name : string
    }

    entity "Surveys" {
        *survey_id : integer <<PK>>
        --
        *user_id : integer <<FK>>
        *title : string, null: false
        *body : text, null: false
        *is_multiple_choice : boolean
        *start_at : datetime, null: false
        *end_at   : datetime, null: false
        *created_at : datetime
        *updated_at : datetime
    }

    entity "Age_targets" {
        *survey_id : integer <<FK>>
        *age_range_id : integer <<FK>>
    }

    entity "Gender_targets" {
        *survey_id : integer <<FK>>
        *gender_id : integer <<FK>>
    }

    entity "Reminders" {
        *survey_id : integer <<FK>>
        *user_id : integer <<FK>>
        *remind_at : datetime
        *created_at : datetime
    }

    entity "Lists" {
        *list_id : integer <<PK>>
        --
        *user_id : integer <<FK>>
        *list_name : string
    }

    entity "List_items" {
        *list_id : integer <<FK>>
        *survey_id : integer <<FK>>
    }

    entity "Categories" {
        *category_id : integer <<PK>>
        --
        *category_name : string
    }

    entity "Choices" {
        *choice_id : integer <<PK>>
        --
        *survey_id : integer <<FK>>
        *choice_text : string
    }

    entity "Answers" {
        *answer_id : integer <<PK>>
        --
        *choice_id : integer <<FK>>
        *user_id : integer <<FK>>
    }

    entity "Survey_categories" {
        *survey_id : integer <<FK>>
        *category_id : integer <<FK>>
    }

    Users ||--o{ Surveys
    Users ||--o{ Lists
    Users ||--o{ Reminders
    Lists ||--o{ List_items
    Surveys ||--o{ List_items
    Surveys ||--o{ Reminders
    Users }o--|| Genders
    Users }o--|| Age_ranges
    Surveys ||--o{ Choices
    Genders ||--o{ Gender_targets
    Surveys ||--o{ Age_targets
    Surveys ||--o{ Gender_targets
    Age_ranges ||--o{ Age_targets
    Answers }o--|| Choices
    Users ||--o{ Answers
    Surveys ||--o{ Survey_categories
    Categories ||--o{ Survey_categories
    @enduml

```
## 2.コードの解説
　1. @startuml sample_site と @enduml<br>
　　@startuml と @enduml は、PlantUMLの図の開始と終了を示すディレクティブです。<br>
　　sample_site は図のタイトルや識別子として使用されます。<br>
<br>
　2. entity "EntityName" { ... }<br>
　　entity キーワードは、エンティティ（データベースのテーブルやクラス）を定義します。<br>
　　"EntityName" はエンティティの名前で、ダブルクォーテーションで囲まれています。<br>

　3. 属性の定義<br>
　　エンティティ内の属性は、以下の形式で定義されます：<br>
```
　　　　*attribute_name : type
```
　　* は主キーを示します。<br>
　　attribute_name は属性名です。<br>
　　type はデータ型を示します（例：integer、string、datetime）。<br>
　　-- は主キーとその他の属性を区切るために使用されます。<br>
<br>
　4. 関係の定義<br>
　　エンティティ間の関係は、以下の形式で定義されます：<br>
```
　　　　EntityA ||--o{ EntityB
```
　　|| は EntityA 側が「1」を意味します。<br>
　　o{ は EntityB 側が「0以上」を意味します。<br>
<br>
　　他の記号の意味：<br>
<br>
　　|o-- ：「0または1」<br>
　　}|-- ：「1以上」<br>
　　}o-- ：「0以上」<br>

　5. データ型と制約<br>
　　属性のデータ型は、一般的なデータベースの型に準じています。<br>
　　null: false は、その属性がNULLを許容しないことを示します。<br>

  6. 例：Users エンティティ
```
    entity "Users" {
        *user_id : integer <<PK>>　　　　　*user_id は主キーを示します。また、<<PK>> は主キーのステレオタイプを示します。
        --
        *gender_id : integer <<FK>>　　　　　　<<FK>> は外部キーのステレオタイプを示します。
        *age_range_id : integer <<FK>>
        *username : string, null: false
        *email : string, null: false
        *selfintro : text
        *age : integer
        *crypted_password : string, null: false　　　　null: false はNULLを許容しない制約を示します。
        *salt : string
        *created_at : datetime
        *updated_at : datetime
    }
```
　







