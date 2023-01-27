
## render @bookmark_boards
というインスタンス変数で複数形を使ったrenderは

[[部分テンプレートの省略]]
の複数形の省略


## パーシャルの_boardがrenderされる

railsはコレクション内のモデル名に基いてどのパーシャルを呼び出すかを決めている。  
変数ではなく、変数の中身（実体）を元によしなに適切なファイルを自動的に指定してくれている。

という前提をもとに、定義した

```ruby
@bookmark_boards = current_user.bookmark_boards.includes(:user).order(created_at: :desc)
```

の中身は`Board`モデルの中身を示しているものであり、よって`_board.html.erb`のパーシャルを呼び出してくれている。

実際にターミナルで表示される以下も今回は該当の掲示板が3つのため、

```
↳ app/views/boards/_unbookmark.html.erb:1
  Rendered boards/_unbookmark.html.erb (1.3ms)
  Rendered boards/_bookmark_button.html.erb (3.4ms)
  Rendered collection of boards/_board.html.erb [3 times] (15.8ms)
```

①Rendered collection of boards/_board.html.erb [3 times] (15.8ms) `1回目`  
②Rendered boards/_unbookmark.html.erb (1.3ms)  
③Rendered boards/_bookmark_button.html.erb (3.4ms)

④Rendered collection of boards/_board.html.erb [3 times] (15.8ms) `2回目`  
⑤Rendered boards/_unbookmark.html.erb (1.3ms)  
⑥Rendered boards/_bookmark_button.html.erb (3.4ms)

⑦Rendered collection of boards/_board.html.erb [3 times] (15.8ms) `3回目`  
⑧Rendered boards/_unbookmark.html.erb (1.3ms)  
⑨Rendered boards/_bookmark_button.html.erb (3.4ms)

という順で実行されており、まずは、`_board.html.erb`を見に行っていることが分かる。

そして、どこのパーシャルを表すかは`@bookmark_boards.first.to_partial_path`で確認することができる。

```ruby
@bookmark_boards.first.to_partial_path
# => "boards/board"
```
