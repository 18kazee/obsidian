
pushメソッドは配列の末尾に引数を追加します。
このメソッドはレシーバーであるメソッドを変更してしまう破壊的メソッドです。
!がついていないですが破壊的メソッドになります。

```ruby
object = ["aa", "bb", "cc"]
object.push('hoge')

p object

#=> ["aa", "bb", "cc", "hoge"]
```

https://magazine.techacademy.jp/magazine/19868