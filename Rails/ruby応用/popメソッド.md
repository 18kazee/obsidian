配列の末尾の要素を削除するメソッド
popメソッドに引数を渡せばその個数文削除することができる。

削除された要素はpopの戻り値として渡されます。

```ruby
p array = [0, 1, 2] #=> [0, 1, 2]
p array.pop #=> 2
p array #=> [0, 1]
```

```ruby
p array = [0, 1, 2] #=> [0, 1, 2]
p array.pop(2) #=> [1, 2]
p array #=> [0]
```

https://www.sejuku.net/blog/75457