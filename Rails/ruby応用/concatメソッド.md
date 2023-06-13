
concatメソッドはレシーバーに引数の配列を破壊的に連結させるメソッドです。

```
array = [100, 200]
a = [400, 500]
array.concat(a)

p array #=> [100, 200, 300, 400, 500]
p a #> [400, 500]
```

https://docs.ruby-lang.org/ja/latest/method/Array/i/concat.html