
## ルーティングの設定

```ruby
resources :users, only: %i[new create] 
resources :boards do 
	resources :comments, only: %i[create update destroy], shallow: true 
	collection do 
		get :bookmarks
	end
end 
resources :bookmarks, only: %i[create destroy]
```

## コントローラーの設定

### bookmarksコントローラーの定義

```shell
$ bin/rails g controller bookamrks
```

### userモデルにbookmarksコントローラーの内容を設定

```ruby
# user.rb
def bookmark(board)
	bookmark_boards << board 
end

def unbookmark(board)
	bookmark_boards.destroy(board)
end

def bookmark?(board)
	bookmark_boards.include?(board)
end
```

コントローラーに記載すると、アクション内部の記述が複雑になってしまい、可読性が落ちる。
また[[なぜBookmarkモデルではなくUserモデルにbookmarkを定義するのか。]]を参照すること。

** `has_many :bookmark_boards, through: :bookmarks, source: :board`で`has many through` オプションをつけたので、Userクラスのインスタンスメソッド`bookmark_boards`が定義されています。**

[[bookmark_board  << board]]
[[unbookmark(board)]]
[[bookmark?(board)]]

### bookmarksコントローラの実装

```ruby
# bookmarks_controller
class BookmarksController < ApplicationController
	def create
	 board = Board.find(params[:board_id])
	  current_user.bookmark(board) redirect_back fallback_location: root_path, success: 'ブックマークしました' 
	end
	
	def destroy 
		board = current_user.bookmarks.find(params[:id]).board 
		current_user.unbookmark(board) redirect_back fallback_location: root_path, success: 'ブックマークを外しました' 
	end
end
```

`redirect_back fallback_location:`を使うと、直前のページにリダイレクトをしてくれる。
[[redirect_back]]






