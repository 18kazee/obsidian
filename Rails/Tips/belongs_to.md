
### 他のテーブルとの関係性を示すもの

テーブル同士を関連づけるもので、
自分のテーブルが、指定したテーブルの下に紐づいている（属している）ことを示す。
モデルに指定される。

```ruby
belongs_to :モデル名(単数形)
```

親となるテーブルには、
[[has_many]] :モデル名(複数形)を記述。

[[references]]を使えば、belongs_toは自動で記述されるが、
***[[has_many]]はつかないので、自分で記述する必要がある。***


### 参考
https://railsguides.jp/association_basics.html