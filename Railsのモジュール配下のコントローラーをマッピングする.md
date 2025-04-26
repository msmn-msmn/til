# ★モジュール配下のコントローラーをマッピングする方法
## ***❇️目的***
　簡単に言うと、<br>
　***モジュール（グループ）に属しているコントローラーに、「それ専用のルート（道）を作ってあげる」*** <br>
　ということ。<br>
　具体例としては ***管理者(admin)*** と ***一般ユーザー(user)*** でコントローラーを分けたいとする。<br>
　それぞれ、 ***違う道順（ルーティング）*** で動いてほしいから専用のルートを<br>
　設定しますということ。<br>

| 用語 | 意味 |
|:----|:-----|
| モジュール配下 |	adminとかuserとか、グループ分けしたフォルダの中 |
| コントローラー	| たとえば Admin::UsersController みたいに、グループ専用のコントローラー |
| マッピング	| そのコントローラーにアクセスするための「道」を設定すること |

## ***❇️やり方***
　管理者と一般ユーザーとでルートページを分けるという想定で作業を行っていきます。<br>
 <br>
## ***💠コントローラーを生成する***
　　Ubuntu
  ```
　　docker compose exec web rails generate contoroller Admin::Books
```
　　これでAdmin::BooksContorollerが、/contoroller/adminフォルダーの配下に<br>
　　books_contoroller.rbという名前で生成されます。<br>

## ***💠ルートを設定する***
　　このようなモジュール対応のコントローラークラスに対して、RESTfulインターフェースを<br>
　　定義するには、namespaceブロックを利用します。<br>
  　routes.rb
   ```
　　namespace :admin do                                     #/admin/ っていうパスを自動的に付けて、コントローラーも Admin:: 配下にする
　　  root to: "books#index"                                #/admin にアクセスしたら Admin::BooksController の index アクションに行く。つまり、管理者用トップページ！
　　  resources :books, only: %i[index]                        #Admin::BooksContorollerのアクションを指定する
　　  get 'login', to: 'user_sessions#new', :as => :login     #/admin/login にGETアクセスすると、Admin::UserSessionsController#new を呼び出す（ログインフォーム表示）。admin_login_path って名前も付く。
　　  post 'login', to: 'user_sessions#create'                #/admin/login にPOSTすると、ログイン処理（セッション作成）
　　  delete 'logout', to: 'user_sessions#destroy', :as => :logout                   #/admin/logout にDELETEすると、ログアウト処理。admin_logout_path って名前も付く。
　　end
```
### 　　***:as => :login***とは？<br>
　　`このルーティングに「名前（名前付きルート）」をつけてる`ということ。<br>
　　ここでひとつポイントがあって<br>
<br>
　　namespace :admin の中にあるから<br>
　　単純に login_path じゃなくて、<br>
　　admin_login_path<br>
　　っていう名前が自動でつく。<br>
　　つまり、<br>
```
　　namespace :admin なしなら → login_path

　　namespace :admin ありなら → admin_login_path
```
　　になる.<br>
　　まとめると<br>

| 書き方	| 生成されるヘルパー名 |
|:------|:-------------------|
| as: :login （namespaceなし）|	login_path, login_url |
|namespace :admin 内で as: :login | admin_login_path, admin_login_url |

　　このように`as: :(任意のワード)`を使う事でパス名を自由にカスタマイズできる。<br>
  <br>
### 　　***ちょっとした注意⚡***<br>
　　・as: :hoge でつけた名前は絶対**ユニーク（かぶらない）**にしないとダメ！<br>
　　・同じ as: が複数あるとエラーになる（Railsがどっち使えばいいか分からなくなる）<br>
　　・パスヘルパーを使いやすく読みやすくするために、わかりやすい名前をつけた方がいい！<br>


　　
　
