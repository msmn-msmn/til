✅ errors.full_messagesの基本
　　errors.full_messagesは、バリデーションエラーのメッセージを配列で返します。
　　各メッセージは「属性名 メッセージ」の形式で構成されます。
　　例えば、Commentモデルのbody属性が空の場合、次のように表示されます。​

　　ruby
　　　comment = Comment.new(body: "")
　　　comment.valid?
　　　comment.errors.full_messages
　　　　# => ["Bodyを入力してください"]
　　このメッセージは、ビューで以下のように表示できます。​

　　erb
　　　<% if @comment.errors.any? %>
　　　  <ul>
    　　　<% @comment.errors.full_messages.each do |message| %>
      　　　<li><%= message %></li>
   　　　 <% end %>
 　　　 </ul>
　　　<% end %>

🛠 エラーメッセージのカスタマイズ方法
　1. モデル内で直接メッセージを指定
　　モデルのバリデーションでmessageオプションを使用して、カスタムメッセージを設定できます。​
　　ruby
　　　class Comment < ApplicationRecord
 　　　 validates :body, presence: { message: "コメントを入力しろ！" }
　　　end
　　この設定により、bodyが空の場合、エラーメッセージは「コメントを入力しろ！」と表示されます。​

　2. ロケールファイルでのカスタマイズ
　　config/locales/ja.ymlなどのロケールファイルで、属性名やエラーメッセージを定義できます。​
　　yaml
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
　　この設定により、bodyが空の場合、「コメントを入力しろ！」と表示されます。​
　　Railsはこのロケールファイルを参照して、エラーメッセージを自動的に翻訳します。​

⚠️ 注意点
　errors.full_messagesは、各属性に対するエラーメッセージを「属性名 メッセージ」の形式で返します。​
　属性名を含めずにメッセージを表示したい場合は、errors.messages[:attribute]を使用して、
　属性ごとのメッセージ配列を取得できます。​
　　erb
　　　<%= @comment.errors.messages[:body].first %>
　Rails 7以降では、form_withがデフォルトで非同期リクエスト（remote: true）として動作します。​
　このため、バリデーションエラー時にフォームが正しく再描画されない場合があります。​
　その場合は、renderメソッドでステータスコードをunprocessable_entity（422）に設定することで解決できます。​
　　ruby
　　　render :new, status: :unprocessable_entity

📝 まとめ
　errors.full_messagesは、バリデーションエラーのメッセージを配列で取得するためのメソッドです。
　エラーメッセージは、モデル内で直接指定するか、ロケールファイルでカスタマイズできます。
　非同期リクエスト時のエラーメッセージ表示には、ステータスコードの設定が必要な場合があります。
