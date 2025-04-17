★Turbo StreamsをRailsのアプリで実装する
◎前提条件
　下記条件はRails7系でrails newを行うと勝手に設定されていると思います

  turbo-railsgemがインストールされていること(viewでturbo_streamを使いやすくなります。
  (入っていなければインストールしてください)
  app/javascript/application.jsでturboを使う設定が出来ていること

◎実装方法
　ViewからTURBO_STREAMリクエストを送る
　getリクエストを送るリンクの場合下記のようにdata: { turbo_stream: true }をつけると
　コントローラーでturbo_streamのリクエストとして受け取ることが出来るようになります。

　　<%= link_to 'Edit', edit_post_path(post), data: { turbo_stream: true } %>
　get以外のリクエストの場合下記のようにdata-turbo_methodを指定することでturbo_streamのリクエストを送れます。

 　 <%= link_to 'Destroy', post, data: { turbo_confirm: 'Are you sure?', turbo_method: :delete } %>
　サーバーログに下記のようなログが出ていればOKです。

　　Processing by PostsController#edit as TURBO_STREAM
　　Controllerの設定
　　Controllerの設定は特に必要ありません。

　ViewからTURBO_STREAMのリクエストを送れていればアクション名と対応するaction名.turbo_stream.erbファイルを探してくれます。

　　def edit
　　  @post = Post.find(params[:id])
 　　 # render :editは書かなくてもedit.turbo_stream.erbファイルを探してくれる
　　end
　または、下記のようにrender turbo_stream: turbo_streamの構文でコントローラー側で実装することも出来ます

　　def edit
　　  @post = Post.find(params[:id])
 　　 render turbo_stream: turbo_stream.replace("post_#{@post.id}", partial: "form", locals: { post: @post })
　　end

◎Turbo Streams用のViewを用意する
　turbo-railsではturbo_stream.アクション 対象要素のID, partial: 表示したい部分テンプレート, locals: 部分テンプレートで使いたい変数のような形でTurbo Streams用のViewファイルを記述することが出来ます。

　　<%= turbo_stream.update "post_#{@post.id}", partial: "form", locals: { post: @post } %>
　またはパーシャルを指定せずに下記のように直接Erbを記述することが出来ます。

　　<%= turbo_stream.update "post_#{@post.id}" do %>
　　  <%= form_with model: @post do |f| %>
  　　  <%= f.text_field :body %>
  　　  <%= f.submit %>
　　  <% end %>
　　<% end %>
　これは下記のように解釈されブラウザに送られます。
　下記の要素がブラウザでキャッチされるとapplication.jsで設定したHotwireが動いてDOMを更新したり、
　削除したりする操作をしてくれます。

　　<turbo-stream action="アクション" target="対象要素のID">
　　  <template>
  　　  パーシャルで指定した内容
　　  </template>
　　</turbo-stream>
　ブラウザのネットワークタブを確認すると上記の形式でレスポンスが返ってきているのを確認できます。
　※サンプルコードを簡略化するためHTMLの属性を削っているのでstyleとかが入っています。

◎複数のturbo_stream
　turbo_streamで複数の処理を行いたいときは下記のように一つのViewに複数のturbo_streamの処理を書くことができます。

　　<%= turbo_stream.replace "post_form_#{@post.id}", partial: "form", locals: { post: @post } %>

　　<%= turbo_stream.append "error_messages_#{@post.id}" do %>
　　  <% @post.errors.full_messages.each do |message| %>
　　    <%= message %>
　　  <% end %>
　　<% end %>

◎Turbo Streamsの7つのアクション+CSSの指定方法
❇️Append
　Appendアクションは、指定した要素の内部に新しいコンテンツを追加します。
　具体的には、以下のように使用します。
　　　<%= turbo_stream.append "comments" do %>
 　　　 <%= render @comment %>
　　　<% end %>
　このコードは、comments という ID を持つ要素の内部に、@comment の部分テンプレートを追加します。
　これにより、新しいコメントがリストの最後に追加されます。
　Appendアクションは、新しい要素を既存のコンテンツの後に追加する際に使用します。
　
　対象のidを持つ要素の内側の一番お尻に要素を追加します。↓

　　<%= turbo_stream.append "posts", partial: "post", locals: { post: @post } %>
　
　パーシャルを使わない場合こんな感じで書けます。↓
　　<%= turbo_stream.append "posts" do %>
　　  <div id="<%= dom_id @post %>">
 　　   <%= @post.body %>
　　    <%= link_to 'Edit', edit_post_path(@post), data: { turbo_stream: true } %>
 　　   <%= link_to 'Destroy', @post, data: { turbo_confirm: 'Are you sure?', turbo_method: :delete } %>
　　  </div>
　　<% end %>

❇️Prepend
  Prepend アクションは、指定した要素の内部の最初に新しいコンテンツを追加します。
  具体的には、以下のように使用します。
　　　<%= turbo_stream.prepend "comments" do %>
 　　　 <%= render @comment %>
　　　<% end %>
　このコードは、comments という ID を持つ要素の内部の最初に、@comment の部分テンプレートを追加します。
　これにより、新しいコメントがリストの先頭に追加されます。
　Prepend アクションは、新しい要素を既存のコンテンツの前に追加する際に使用します。
　

❇️Replace
　Replace アクションは、指定した要素を完全に置き換えます。具体的には、以下のように使用します。
　　　<%= turbo_stream.replace "comment_#{@comment.id}" do %>
 　　　 <%= render @comment %>
　　　<% end %>
　このコードは、comment_#{@comment.id} という ID を持つ要素を、@comment の部分テンプレートで置き換えます。
　これにより、編集後のコメントが元のコメントと入れ替わります。
　Replaceアクションは、既存の要素を新しい内容で置き換える際に使用します。

❇️Update
　Update アクションは、指定した要素の内容を更新します。
　具体的には、以下のように使用します。
　　　<%= turbo_stream.update "comment_likes_#{@comment.id}" do %>
　　　  <%= @comment.likes_count %>
　　　<% end %>
　このコードは、comment_likes_#{@comment.id} という ID を持つ要素の内容を、@comment.likes_count で更新します。
　これにより、いいね数がリアルタイムで更新されます。
　Updateアクションは、既存の要素の内容を部分的に更新する際に使用します。

❇️Remove
　Remove アクションは、指定した要素を削除します。
　具体的には、以下のように使用します。
　　　<%= turbo_stream.remove "comment_#{@comment.id}" %>
　このコードは、comment_#{@comment.id} という ID を持つ要素を削除します。
　これにより、指定したコメントが DOM から削除されます。
　Remove アクションは、不要になった要素を DOM から取り除く際に使用します 。

❇️Before
　Before アクションは、指定した要素の直前に新しいコンテンツを追加します。
　具体的には、以下のように使用します。
　　　<%= turbo_stream.before "comment_#{@comment.id}" do %>
  　　　<%= render @new_comment %>
　　　<% end %>
　このコードは、comment_#{@comment.id} という ID を持つ要素の直前に、
　@new_comment の部分テンプレートを追加します。これにより、新しいコメントが指定した位置の前に追加されます。
　Before アクションは、特定の要素の前に新しい要素を追加する際に使用します。

❇️After
　After アクションは、指定した要素の直後に新しいコンテンツを追加します。
　具体的には、以下のように使用します。
　　　<%= turbo_stream.after "comment_#{@comment.id}" do %>
 　　　 <%= render @new_comment %>
　　　<% end %>
　このコードは、comment_#{@comment.id} というIDを持つ要素の直後に、
　@new_comment の部分テンプレートを追加します。これにより、新しいコメントが指定した位置の後に追加されます。
　After アクションは、特定の要素の後に新しい要素を追加する際に使用します。

◎class属性を指定する方法
　アクション名+_allを使うとclass指定することが出来ます。
　対象のclassを持つ要素全てを操作します。
　classを指定するときはCSSセレクタを使用して.クラス名のように書きます。

　　<%= turbo_stream.append_all ".content" do %>
  　　<div>おいちい</div>
　　<% end %>

◎参照
　　https://turbo.hotwired.dev/reference/streams
　　https://github.com/hotwired/turbo-rails/blob/main/app/models/turbo/streams/tag_builder.rb
