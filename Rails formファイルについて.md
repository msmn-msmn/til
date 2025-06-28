# ★Rails formファイルについて
## 1. Form オブジェクトとは？
目的：単一のモデルに収まらないフォーム入力を扱ったり、ビジネスロジックをコントローラから分離したりするための「仮想的なモデル」<br>
メリット：<br>
・複数モデルの同時更新を簡潔にまとめられる<br>
・テストしやすい<br>
・コントローラがすっきりする<br>
<br>
<br>
## 2. ファイルを置く場所
Formオブジェクトは以下の場所に配置します。<br>
```
app/forms/  # フォルダがなければ作成
└─ contact_form.rb
```


## 3. 基本的なクラス定義
```
# app/forms/contact_form.rb
class ContactForm
  include ActiveModel::Model        # バリデーションや errors オブジェクトを提供
  include ActiveModel::Attributes   # attribute API を利用する場合

  # ① 仮想属性の定義（フォームで扱うフィールド）
  attribute :name,    :string
  attribute :email,   :string
  attribute :message, :string

  # ② バリデーション
  validates :name,    presence: true
  validates :email,   presence: true, format: { with: URI::MailTo::EMAIL_REGEXP }
  validates :message, presence: true, length: { minimum: 10 }

  # ③ 保存処理などのビジネスロジック
  def save
    return false unless valid?

    # 例：複数のモデルを更新したい場合など
    ContactMailer.with(
      name:    name,
      email:   email,
      message: message
    ).deliver_now

    true
  end
end
```


## 4. コントローラとの連携
```
# app/controllers/contacts_controller.rb
class ContactsController < ApplicationController
  def new
    @form = ContactForm.new
  end

  def create
    @form = ContactForm.new(contact_form_params)
    if @form.save
      redirect_to root_path, notice: "お問い合わせを送信しました😊"
    else
      render :new
    end
  end

  private

  def contact_form_params
    # attribute で定義した属性名を permit に追加
    params.require(:contact_form).permit(:name, :email, :message)
  end
end
```

## 5. ビューでのフォーム定義
```
<!-- app/views/contacts/new.html.erb -->
<%= form_with model: @form, url: contacts_path do |f| %>
  <% if @form.errors.any? %>
    <div class="errors">
      <h2><%= pluralize(@form.errors.count, "件") %> のエラーがあります：</h2>
      <ul>
        <% @form.errors.full_messages.each do |msg| %>
          <li><%= msg %></li>
        <% end %>
      </ul>
    </div>
  <% end %>

  <div class="field">
    <%= f.label :name, "お名前" %>
    <%= f.text_field :name %>
  </div>

  <div class="field">
    <%= f.label :email, "メールアドレス" %>
    <%= f.email_field :email %>
  </div>

  <div class="field">
    <%= f.label :message, "お問い合わせ内容" %>
    <%= f.text_area :message %>
  </div>

  <div class="actions">
    <%= f.submit "送信" %>
  </div>
<% end %>
```
model: @form となるのはコントローラに<br>
```
def new
    @form = ContactForm.new
end
```
と定義されているからです。なので任意の名称に変更も簡単にできます。<br>
