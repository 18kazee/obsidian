
### 他のテーブルとの関係性を示すもので、下に複数のテーブルが紐づくことを示す

親となるモデルに記載。

```ruby
has_many :モデル名(単数系)
```

[[references]]を使えば、[[belongs_to]]は自動で記述されるが、
***has_manyはつかないので、自分で記述する必要がある。***

オプションをつけることもできる。
***[[dependent]]***: :destroy

### 参考
https://railsguides.jp/association_basics.html