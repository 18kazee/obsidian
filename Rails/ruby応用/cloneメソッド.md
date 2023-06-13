
レシーバーであるオブジェクトのインスタンス変数をコピーして別のオブジェクトとしてインスタンス変数を生成するメソッド。
同じくコピーして生成するメソッドにdupメソッドがありますが、dupメソッドだと状態によりコピーできない場合があります。

具体的には、オブジェクトが凍結状態、(freezeメソッドで変更不可となっている)をdupメソッドでコピーできません。
また得意メソッド(classメソッド)がある場合でもdupメソッドはコピーできません。
これらはcloneメソッドであればコピー可能です。（freeze状態であればその状態のままコピー、特異メソッドもそのままコピー）

```ruby
class Train
	attr_accessor :name

	def initialize(name)
		@name = name
	end
end

EXE = Train.new('hoge')
exe = EXE.clone

p EXE.name
p exe.name

#=> 'hoge'
#=> 'hoge'
```

コピーできていることが確認できました。
開発現場では元のメソッドを変更するわけにいかないので、こうしてコピーを作成していろいろと変更を試しているとのこと。

https://magazine.techacademy.jp/magazine/19904