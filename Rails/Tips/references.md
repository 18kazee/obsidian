
### アソシエーション(関連づけ)するために使用する

新しく作成するテーブルのカラムに、
作成済みのテーブルを指定する場合に使う。
既存のテーブルを参照するので、reference。（参照という意味）

###  references使用法

``` ruby
rails g model post name:string body:string user:references
```

reference型を指定したカラム名は テーブル名_id となる。  
(user:reference であれば、 user_id というカラムが生成される)

### マイグレーションファイルにも特別な記述が入る。

```ruby
t.references :テーブル名, null: false, foreign_key: true
```

```ruby
class CreatePosts < ActiveRecord::Migration[6.1] 
	def change 
		create_table :posts do |t| 
			t.references :user, null: false, foreign_key: true 
			t.string :name 
			t.string :body 
		
			t.timestamps 
		end 
	end
end

```

`t.references`  
カラムが他のテーブルを参照していることを示す。

`null: false`  
NULL値、つまり空欄を許可しないという指定。

`foreign_key: true`  
外部キーであることを示している。
[[foreign_key（外部キー）]]

referencesを指定したことで、
生成されたモデルファイルに[[belongs_to]] :userが追加される。
親となるモデルには[[has_many]]を書く必要がある。


### 参考
https://qiita.com/krile136/items/0e824a6fb1d657bd3f97
https://prograshi.com/framework/rails/references_and_foreign-key/
https://railsguides.jp/association_basics.html