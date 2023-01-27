
## 実装の流れ

- モデルの設定
- ルーティングの設定
- コントローラーの設定
- ビュー側の設定

## モデルの設定

userモデルとboardモデルの「中間テーブル(中間モデル)」(bookmark)を作成。
同じ人が複数回お気に入りできないようにする。

[[中間テーブル(中間モデル)]]

### モデルの作成

```shell
$ bin/rails g model bookmark user:references board:references
```

### マイグレーションファイル設定

```ruby
# マイグレーションファイル
class CreateBookmarks < ActiveRecord::Migration[5.2] 
	def change 
		create_table :bookmarks do |t| 
		
			t.references :user, foreign_key: true 
			t.references :board, foreign_key: true 

			t.timestamps 

			t.index [:user_id, :board_id], unique: true　#追記する
		
		end
	end
end
```

### バリデーション設定

```ruby
#bookmark.rb
class Bookmark < ApplicationRecord
	belongs_to :user
	belongs_to :board
	
	validates :user_id, uniqueness: { scope: :board_id } #追記 
end
```

同じユーザーが、同じ掲示板に複数回「お気に入り」をすることを防ぐため、
`t.index [:user_id, :board_id], unique: true`　
を追記します。
`user_id`と`board_id`の組み合わせが`unique`って意味。

OKだったら、`$ rails db:migrate`

>参照
[[uniqueness]]

### 各モデルのアソシエーション設定

```ruby
# user.rb
has_many :boards, dependent: :destroy
has_many :comments, dependent: :destroy
has_many :bookmarks, dependent: :destroy　#追記 
has_many :bookmark_boards, through: :bookmarks, source: :board　#追記
```

```ruby
# board.rb
has_many :bookmarks, dependent: :destroy　#追記
```

----

`has_many :bookmark_boards,`
Userモデルは、多数のbookmark_boardsを持っている

`bookmark_boards`とは？？
本当は、`boards`が使いたかったけど、既に一行目で使ってしまっているので、それらしい名前を付けている

`through: :bookmarks,` 
bookmarksモデルを通して、

`source: :board`
boardモデルからデータを参照する

##### has many :throughオプションで関連付けすると何がいいのか
ユーザーがお気に入りした掲示板を直接取得することができるようになる！

----

>参照
[[sourceオプション]]
[[throughオプション has_many]]




