# ***★Rubyのenumとは何か***
　Rubyのenumは、特定の属性に対して一連の固定値を定義するための便利な機能です。<br>
　これは、特にモデルが特定の状態を持つ必要がある場合に役立ちます。<br>
　例えば、注文のステータス（pending、confirmed、shippedなど）や<br>
　ユーザーのロール（admin、member、guestなど）を表現するのに使われます。<br>
<br>
　enumは、これらの値をシンボルとして定義し、それぞれに対応する整数値をデータベースに保存します。<br>
　これにより、データベースの効率を保ちつつ、コード内で意味のある名前を使用することができます。<br>
<br>

## ***✅Ruby on Railsのモデルでenumを定義する基本的な使い方***
　　この例では、status属性はpending、confirmed、shippedの3つの状態を持つことができ、<br>
　　それぞれが0、1、2という整数値にマッピングされます。<br>
　　これにより、Articleモデルのインスタンスはorder.pending?、order.confirmed!などの<br>
　　メソッドを使用して状態を照会したり変更したりすることができます。<br>

```
　　サンプルコード
      class Article < ApplicationRecord
        # たった1行でこれまでの実装をカバー
        enum status: { pending: 0, confirmed: 1, shipped: 2 }
      end

      # 生成されるメソッド例
      article.pending!       # ステータスを変更
      article.pending?       # 状態をチェック
      article.status         # 現在の状態を取得
      Article.pending        # scopeとして使用可能
```



## ***✅enumを使うメリット***
　　Rubyのenumを使用することには、いくつかのメリットがあります。<br>

| メリット | 効果 |
|:--------|:-----|
| コードの可読性 | enumを使用すると、コード内で意味のある名前を使用できます。これにより、コードの可読性が向上し、他の開発者がコードを理解しやすくなります。 |
| データの整合性 | enumは、特定の属性が取りうる値を制限します。これにより、データの整合性が保たれ、予期しない値が設定されるのを防ぐことができます。 |
| 便利なメソッド | enumを定義すると、自動的にいくつかの便利なメソッドが提供されます。これにより、状態の確認や変更を行うことが容易になります。 |

　　以下に、これらのメリットを具体的に示すコードの例を示します。<br>

#### ***💠Orderモデルのインスタンスを作成***
```
      order = Order.new
```

#### ***💠状態を確認***
```
      order.pending?  # => true
```
#### ***💠状態を変更***
```
      order.confirmed!
      order.status  # => "confirmed"
```

#### ***💠状態に応じた処理***
```
      case order.status
      when "pending"
        puts "Order is pending."
      when "confirmed"
        puts "Order is confirmed."
      when "shipped"
        puts "Order is shipped."
      end
```
　　このように、Rubyのenumはコードの可読性、データの整合性、便利なメソッドの提供というメリットを提供します。<br>

## ***✅enumと文字列の関係***
　　Rubyのenumは、基本的には整数値をデータベースに保存しますが、<br>
　　これらの整数値はコード内でシンボルとして表現されます。<br>
　　これにより、文字列としての表現と整数値としての効率性を両立することができます。<br>
<br>
　　しかし、enumと文字列の間にはいくつかの注意点があります。<br>

  | 注意点 | 内容 |
  |:-------|:----|
  |文字列としてのenum | enumの値は、コード内ではシンボルとして表現されますが、これらのシンボルは必要に応じて文字列に変換することができます。例えば、order.statusは"pending"のような文字列を返します。|
  |文字列からのenumの設定 | enumの値は文字列から設定することも可能です。例えば、order.status = "confirmed"のように設定することができます。ただし、存在しない値を設定しようとするとエラーが発生します。|
  |文字列としての保存 | デフォルトでは、enumは整数値としてデータベースに保存されます。しかし、文字列として保存したい場合は、enumの定義で明示的に文字列をマッピングすることができます。|

　　以下に、これらの関係を示すコードの例を示します。<br>

#### ***💠Orderモデルのインスタンスを作成***
```
      order = Order.new
```
#### ***💠enumの値を文字列として取得***
```
      status = order.status  # => "pending"
```
#### ***💠文字列からenumの値を設定***
```
      order.status = "confirmed"
      order.status  # => "confirmed"
```
#### ***💠存在しない値を設定しようとするとエラーが発生***
```
      order.status = "unknown"  # => ArgumentError: 'unknown' is not a valid status
```
　　以上が、Rubyのenumと文字列の関係についての説明となります。<br>


## ***✅enumを使う具体的な方法***
　　Rubyのenumを使う具体的な方法について説明します。<br>
　　以下に、Ruby on Railsのモデルでenumを定義し、それを操作する基本的なコードを示します。<br>

#### ***💠Orderモデルの定義***
```
      class Order < ApplicationRecord
        enum status: { pending: 0, confirmed: 1, shipped: 2 }
      end
```
      
#### ***💠新しいOrderを作成し、statusを設定***
```
      order = Order.new(status: :pending)
```
      
#### ***💠statusを確認***
```
      puts order.status  # => "pending"
```
      
#### ***💠statusを変更***
```
      order.status = :confirmed
      puts order.status  # => "confirmed"
```
      
#### ***💠statusを確認するメソッドを使用***
```
      puts order.pending?  # => false
      puts order.confirmed?  # => true
```
      
#### ***💠statusを変更するメソッドを使用***
```
      order.shipped!
      puts order.status  # => "shipped"
```
      
　　このように、enumを使うことで、モデルの特定の属性に対して一連の固定値を定義し、<br>
　　それらの値を効率的に操作することができます。<br>
　　ただし、enumは整数値をデータベースに保存するため、データベースから直接値を取得した場合、<br>
　　その値は整数になります。そのため、データベースの値を直接操作する場合や、<br>
　　データベースの値を他のシステムと共有する場合は注意が必要です。<br>
<br>
　　以上が、Rubyのenumを使う具体的な方法についての説明となります。<br>
<br>
## ***✅enumを活用した実例とコード***
　　Rubyのenumを活用した実例として、オンラインショッピングサイトの注文管理システムを考えます。<br>
　　注文は、pending（保留中）、confirmed（確認済み）、shipped（発送済み）、delivered（配達済み）<br>
　　の4つの状態を持つことができます。<br>
<br>
　　以下に、このシステムを実装するための基本的なコードを示します。<br>
<br>
#### ***💠Orderモデルの定義***
```
      class Order < ApplicationRecord
        enum status: { pending: 0, confirmed: 1, shipped: 2, delivered: 3 }
      end
```
#### ***💠新しいOrderを作成し、statusを設定***
```
      order = Order.new(status: :pending)
```
#### ***💠statusを確認***
```
      puts order.status  # => "pending"
```
#### ***💠statusを変更***
```
      order.status = :confirmed
      puts order.status  # => "confirmed"
```
#### ***💠statusを確認するメソッドを使用***
```
      puts order.pending?  # => false
      puts order.confirmed?  # => true
```
#### ***💠statusを変更するメソッドを使用***
```
      order.shipped!
      puts order.status  # => "shipped"
```
#### ***💠statusを変更するメソッドを使用***
```
      order.delivered!
      puts order.status  # => "delivered"
```
　　このように、Rubyのenumを活用することで、注文の状態を効率的に管理することができます。<br>
　　また、enumは他の多くの場面でも活用することができます。<br>
　　例えば、ユーザーのロール管理、タスクの進行状況管理、商品の在庫状況管理など、<br>
　　さまざまなシチュエーションでenumを活用することが可能です。<br>
<br>
　　以上が、Rubyのenumを活用した実例とコードについての説明となります。<br>
<br>

## ***✅enumの注意点とトラブルシューティング***
　　Rubyのenumは非常に便利な機能ですが、使用する際にはいくつかの注意点があります。<br>

| 注意点 | 内容 |
|:------|:-----|
| 整数値のマッピング | enumはデフォルトで整数値をデータベースに保存します。これは効率的ですが、データベースの値が直接見られる場合や、データベースの値を他のシステムと共有する場合には注意が必要です。そのような場合には、enumの値を文字列として保存することを検討すると良いでしょう。|
| 存在しない値の設定 | enumの値は固定されているため、存在しない値を設定しようとするとエラーが発生します。このようなエラーを防ぐためには、入力値の検証を行うことが重要です。|
| enumの値の変更 | 一度enumを定義すると、その後で値を追加したり削除したりすることは容易ではありません。特に、既存の値を削除したり、値の順序を変更したりすると、データベースの整合性が失われる可能性があります。enumの値を変更する必要がある場合には、マイグレーションを慎重に計画することが重要です。|

## ***✅応用例資料***
　　【保守性抜群】Ruby on Rails enumの完全ガイド：基礎から実践的な活用まで解説<br>
　　https://dexall.co.jp/articles/?p=1402&utm_source=chatgpt.com
