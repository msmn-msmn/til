# ✅ errors.full_messagesの基本
　　errors.full_messagesは、バリデーションエラーのメッセージを配列で返します。<br>
　　各メッセージは「属性名 メッセージ」の形式で構成されます。<br>
　　例えば、Commentモデルのbody属性が空の場合、次のように表示されます。​<br>

　　ruby<br>
  ```
　　　comment = Comment.new(body: "")
　　　comment.valid?
　　　comment.errors.full_messages
　　　　# => ["Bodyを入力してください"]
```
　　このメッセージは、ビューで以下のように表示できます。​<br>

　　erb<br>
  ```
　　　<% if @comment.errors.any? %>
　　　  <ul>
    　　　<% @comment.errors.full_messages.each do |message| %>
      　　　<li><%= message %></li>
   　　　 <% end %>
 　　　 </ul>
　　　<% end %>
```

## 🛠 エラーメッセージのカスタマイズ方法
　1. モデル内で直接メッセージを指定<br>
　　モデルのバリデーションでmessageオプションを使用して、カスタムメッセージを設定できます。​<br>
　　ruby<br>
  ```
　　　class Comment < ApplicationRecord
 　　　 validates :body, presence: { message: "コメントを入力しろ！" }
　　　end
```
　　この設定により、bodyが空の場合、エラーメッセージは「コメントを入力しろ！」と表示されます。​<br>

　2. ロケールファイルでのカスタマイズ<br>
　　config/locales/ja.ymlなどのロケールファイルで、属性名やエラーメッセージを定義できます。​<br>
　　yaml
  ```
　　　ja:
 　　　 activerecord:
   　　　 models:
     　　　 comment: コメント
   　　　 attributes:
    　　　  comment:
      　　　  body: コメント
 　　　 errors:
    　　　messages:
    　　　  blank: "を入力しろ！"
```
　　この設定により、bodyが空の場合、「コメントを入力しろ！」と表示されます。​<br>
　　Railsはこのロケールファイルを参照して、エラーメッセージを自動的に翻訳します。​<br>

⚠️ 注意点<br>
　errors.full_messagesは、各属性に対するエラーメッセージを「属性名 メッセージ」の形式で返します。​<br>
　属性名を含めずにメッセージを表示したい場合は、errors.messages[:attribute]を使用して、<br>
　属性ごとのメッセージ配列を取得できます。​<br>
　　erb<br>
  ```
　　　<%= @comment.errors.messages[:body].first %>
```
　Rails 7以降では、form_withがデフォルトで非同期リクエスト（remote: true）として動作します。​<br>
　このため、バリデーションエラー時にフォームが正しく再描画されない場合があります。​<br>
　その場合は、renderメソッドでステータスコードをunprocessable_entity（422）に設定することで解決できます。​<br>
　　ruby<br>
  ```
　　　render :new, status: :unprocessable_entity
```

📝 まとめ<br>
　errors.full_messagesは、バリデーションエラーのメッセージを配列で取得するためのメソッドです。<br>
　エラーメッセージは、モデル内で直接指定するか、ロケールファイルでカスタマイズできます。<br>
　非同期リクエスト時のエラーメッセージ表示には、ステータスコードの設定が必要な場合があります。<br>
