
`モデルクラス.enumカラム名の複数形`で､定義した定数リストをハッシュ形式で取得できます｡

```ruby
[16] pry(main)> User.roles
=> {"general"=>0, "admin"=>1}
```

また､keyだけ欲しい場合・valueだけ欲しい場合は下記の様に書くと取得できます。

```ruby
[7] pry(main)> User.roles
=> {"general"=>0, "admin"=>1}
[8] pry(main)> User.roles.keys
=> ["general", "admin"]
[11] pry(main)> User.roles.values
=> [0, 1]
```