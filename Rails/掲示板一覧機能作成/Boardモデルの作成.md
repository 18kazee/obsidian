
### モデルの作成

```ruby
# ターミナル
$ bin/rails generation model board title:string body:text user:references

```

モデルとマイグレーションファイルが作られる

「user:[[references]]」をつけることでUserモデルを参照してくれる。
(Boardモデルには「[[belongs_to]]: user」が追加される)
マイグレーションファイルには、「t.references:user, foreign_key:true」が追加される。



### BoardモデルとUserモデルの関連付け

```ruby
# app/models/user.rb
has_many :boards, dependent: :destroy

```

***[[belongs_to]]と[[has_many]]はテーブル同士を関連づけるもの***

そのクラス（ここではUser）のidを外部キーとして抱える、他のクラス（ここではBoard）があり、UserはBoardを複数登録可能になる。
has_many(user)は、belongs_to(board)を複数投稿できるイメージ。

[[dependent]]: :destroy オプションをつけることによって「boardに紐づいたuserが消されたら該当boardも削除」という作業を行なってくれる。****



### バリデーションの設定

```ruby
# app/models/board.rb
class Board < ApplicationRecord 
	belongs_to :user 
	
	validates :title, presence: true, length: { maximum: 255 }
	validates :body, presence: true, length: { maximum: 65_535 }
end
```

>参照
>[[validates]]


### マイグレーションファイル

```ruby
# db/migrate/XXXXXXXXXXXXXX_create_boards.rb
class CreateBoards < ActiveRecord::Migration[5.2] 
	def change 
		create_table :boards do |t| 
			t.string :title, null: false 
			t.text :body, null: false 
			t.references :user, foreign_key: true 
	
			t.timestamps 
		end 
	end 
end
```

titleとbodyにnull: falseを追加
t.references :user, foreign_key: trueは、boardテーブルに「user_id」という外部キーカラムを追加するもの。

***掲示板に投稿した際に、user_idカラムがあることで、削除や、編集を自分以外が出来ないようにする為。***

### マイグレート

```ruby
# ターミナル
$ bin/rails db:migrate
```
