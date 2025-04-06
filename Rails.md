★アラート　削除前の確認ダイアログを表示させる方法
link_to メソッドにdata: {confirm:""} を追加する。

html.erb
<%= link_to post_path(@post.id), method: :delete, data: {confirm: "削除しますか？"} %>

上のようにlink_to にdata: {confirm: "表示したい文章"} を追加すると、削除ボタンを押した時に確認ダイアログが出てきます。
