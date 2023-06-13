
## まとめ

-   これらは[オブジェクト指向](http://d.hatena.ne.jp/keyword/%A5%AA%A5%D6%A5%B8%A5%A7%A5%AF%A5%C8%BB%D8%B8%FE)の[カプセル化](http://d.hatena.ne.jp/keyword/%A5%AB%A5%D7%A5%BB%A5%EB%B2%BD)を壊す不適切なメソッドになりやすい
-   オブジェクト内部状態を使用した処理を実現したい場合は、オブジェクト内で処理し、その結果だけを返すようにする。（処理結果を返すメソッドをクラス内に定義すると言うこと）
-   これにより、オブジェクトが管理する内容が変わっても、他のクラスやメソッドに影響することなく、クラスを変更することができる！


## ゲッターとは

インスタンス変数をクラス内から参照するのがゲッター
本来Rubyではクラス内からしかインスタンス変数の値は取得できない
そこで外部からも可能にするためにクラス内にインスタンス変数を参照する専用のメソッドゲッターを定義しています

```ruby
class Movie
  def initialize(name)
    @name = name
  end
end

obj1 = Movie.new('Forrest Gump')

p obj1.name #=>undefined method `name' for #<Movie:0x00007fd4558526f0 @name="Forrest Gump">
```

これを参照するためにはクラス内にインスタンス変数を参照するメソッド「ゲッター」を定義する必要があります。
以下の例ではメソッド`Movie#getName`が「ゲッター」です。  
これでインスタンス変数をクラス外で参照することができます。

```ruby
class Movie
  def initialize(name)
    @name = name
  end

  def getName #「ゲッター」
    @name
  end
end

obj1 = Movie.new('Forrest Gump')

p obj1.getName #=> 'Forrest Gump'
```

## セッターとは

インスタンス変数をクラス内から更新するメソッドのこと
参照同様、更新もクラス内部しかできません
こちらも外部から可能とするためにインスタンス変数の更新を可能とする専用メソッド「セッター」を定義します

_例えば、通常では以下の`Movie`クラスからインスタンス変数の直接的な変更はできません。_

```ruby
class Movie
  def initialize(name)
    @name = name
  end
end

obj1 = Movie.new('Forrest Gump')

p obj1.name = 'Fight Club' #=>undefined method `name=' for #<Movie:0x00007ffd4a83b400 @name="Forrest Gump"> (NoMethodError)
```

このような更新を実行するためにはクラス内にインスタンス変数を更新するメソッド**「セッター」**を定義する必要があります。以下の例ではメソッド`Movie#changeName`が**「セッター」**です。  
これでインスタンス変数をクラス外で更新することができます。

```ruby
class Movie
  def initialize(name)
    @name = name
  end

  def getName #「ゲッター」
    @name
  end

  def changeName=(name) #「セッター」
    @name = name
  end
end

obj1 = Movie.new('Forrest Gump')
obj1.changeName = 'Fight Club'
p obj1.getName #=> 'Fight Club'
```

## アクセスメソッドとは

上記の「ゲッター」**・**「セッター」を定義せずともインスタンス変数の参照・更新を可能にする方法です。  
３種類の「アクセスメソッド」が存在します。

attr_reader: 変数名　参照を可能にする。（「ゲッター」と同じ役割）
attr_writer: 変数名　更新を可能にする。（「セッター」と同じ役割）
attr_accessor: 変数名　参照・更新の両方を可能にする。（「セッター」**・**「ゲッター」と同じ役割）

クラス内に「attr_reader」を追加することで「ゲッター」と同じ役割を果たします。（インスタンス変数のクラス外からの参照を可能にする。）
またクラス内に「attr_writer」を追加することで「セッター」と同じ役割を果たします。（インスタンス変数のクラス外からの更新を可能にする。）
また、「attr_accessor 」を追加することで「セッター」**・**「ゲッター」両方の役割を果たします（インスタンス変数のクラス外からの参照・更新を可能にする。）。


https://qiita.com/k-penguin-sato/items/5b75be386be4c55e3abf
https://okmt-aya-26.hatenablog.com/entry/2022/05/16/133544