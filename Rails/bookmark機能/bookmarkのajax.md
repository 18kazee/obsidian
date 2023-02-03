
## Bookmarksコントローラーの修正

```ruby
class BookmarksController < ApplicationController
  def create
    @board = Board.find(params[:board_id])
    current_user.bookmark(@board)
  end

  def destroy
    @board = current_user.bookmarks.find(params[:id]).board
    current_user.unbookmark(@board)
  end
end
```

インスタンス変数に変更して、redirect_backを削除する。

## ajax化する

link_to に remote: true を指定して、非同期処理にします。

```ruby
#app/views/boards/_bookmark.html.erb
<%= link_to bookmarks_path(board_id: board.id),
            id: "js-bookmark-button-for-board-#{board.id}",
            class: "float-right",
            method: :post,
            remote: true do %>
  <%= icon 'far', 'star' %>
<% end %>
```

```ruby
#app/views/boards/_unbookmark.html.erb
<%= link_to bookmark_path(current_user.bookmarks.find_by(board_id: board.id)),
            id: "js-bookmark-button-for-board-#{board.id}",
            class:"float-right",
            method: :delete,
            remote: true do %>
  <%= icon 'fas', 'star' %>
<% end %>
```

## Bookmarkボタンの切り替え処理を追加

ブックマーク時と解除時に、ブックマークのアイコンを切り替える処理をjavascriptで実装します。

```js
//app/views/bookmarks/create.js.erb
$("#js-bookmark-button-for-board-<%= @board.id %>").replaceWith("<%= j(render('boards/unbookmark', board: @board)) %>");
```

```js
//app/views/bookmarks/destroy.js.erb
$("#js-bookmark-button-for-board-<%= @board.id %>").replaceWith("<%= j(render('boards/bookmark', board: @board)) %>");
```

----
>参照
[[replaceWith]]
https://qiita.com/kidokoro-syusei/items/2a2b57026798cc8a2b9d

