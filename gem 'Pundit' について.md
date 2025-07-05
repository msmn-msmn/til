# ★gem'Pundit' について
## gem'Pundit' とは
Pundit を使うと、ざっくり以下のことができるようになります。
```
・モデルごとに Policy クラスを定義 し、「誰が何をできるか」をシンプルなメソッド（create? や update?）で管理
・コントローラで authorize を呼ぶだけで、指定したアクションの権限チェックを自動実行
・policy_scope で「そのユーザーに見せてよいレコードだけ」を簡単に絞り込める
・rescue_from Pundit::NotAuthorizedError を使って、権限エラー時に 403 ページを表示したりリダイレクトが可能
・strong parameters と連携し、ユーザーごとに更新を許可する属性を動的に制御
```
こんな感じで、認可ロジックをスッキリ分離＆自動化できるのが Pundit の強みです。


## ❇️使い方
