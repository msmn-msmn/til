# ★ActiveRecordのメソッドチェーンについて
## 🧩概要
ActiveRecordのメソッドチェーンのやり方を忘れたときの備忘録


## ✅テーブルの結合
```
+---------+       1       *        +----------+      1       *         +-----------+
| Users   |------------------------| Orders   |------------------------| Payments  |
|---------|                        |----------|                        |-----------|
| id PK   |                        | id PK    |                        | id PK     |
| name    |                        | user_id FK |                      | order_id FK |
+---------+                        +----------+    1                   +-----------+
                                         |
+----------+       1       *        +------------+ *
| Products |------------------------| OrderItems |
|----------|                        |------------|
| id PK    |                        | id PK      |
| name     |                        | order_id FK|
| price    |                        | product_id FK |
+----------+                        +------------+


Users —< Orders：ユーザー（1）に対して注文（多）が紐づく 1対多。
Orders —< Payments：注文（1）に対して複数の支払い（多）が可能な 1対多。
Orders —< OrderItems >— Products：注文と商品は 多対多 の関係で、OrderItems テーブルを通じてつながる
```
というテーブル構造があるとして<br>

### ・基本的な結合
```
Users.joins(:orders).where(order: id = 2)
```
とすることで注文IDが2に対応するユーザーを表示することができる。


### ・ネストした結合
```
Users.joins(orders: {orderitems: :products}).where(products: name = toy)
```
