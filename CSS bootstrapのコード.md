# bootstrapのコード
## 概要
bootstrapのコードの備忘録<br>
<br>
<br>
## コード
### 削除ボタン
```
= link_to '削除', article_path(@article),
          method: :delete,
          data: { confirm: '本当に削除しますか？' },
          class: 'btn btn-danger'
```
