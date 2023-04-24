
以下の内容に従ってプログラムを作ってください。

-   最初に産まれたての子猫(kitten)が1匹いる
-   子猫(kitten)は産まれてから2年後に猫(cat)になる
-   猫(cat)は1年ごとに子猫(kitten)を1匹産む
-   猫(cat)になった年から子猫(kitten)を産む

以上のルールに従い、経過年数から子猫、猫の数を出力する。

```
# 例
## 初年度
kitten-A

合計：kitten1匹

## 1年後
kitten-A

合計：kitten1匹

## 2年後
kitten-A => cat-A

kitten-B < cat-A

合計：cat1匹、kitten1匹

## 3年後
cat-A

kitten-B
kitten-C < cat-A

合計：cat1匹、kitten2匹

## 4年後
cat-A
kitten-B => cat-B

kitten-C
kitten-D < cat-A
kitten-E < cat-B

合計：cat2匹、kitten3匹

## 5年後
cat-A
cat-B
kitten-C => cat-C

kitten-D
kitten-E
kitten-F < cat-A
kitten-G < cat-B
kitten-H < cat-C

合計：cat3匹、kitten5匹
```

### 自分の回答

```ruby
class Cat
  attr_reader :age
  @@cat_count = 0
  @@kitten_count = 0

  def initialize
    @age = 2
  end

  def self.cat_count
    @@cat_count
  end

  def grow_up
    @age += 1
    if @age == 2
      @@kitten_count -= 1
      @@cat_count += 1
      Cat.new
    else
      self
    end
  end

  def give_birth
    kittens = []
    if @@cat_count > 0
      @@cat_count.times do
        kitten = Kitten.new
        kittens << kitten
      end
    end
    kittens
  end
end

class Kitten < Cat
  def initialize
    @@kitten_count += 1
    @age = 0
  end

  def self.kitten_count
    @@kitten_count
  end
end

years = ARGV[0].to_i
kitten = Kitten.new
cat = []
years.times do |year|
  kitten = kitten.grow_up
  cat = cat.map do |c|
    c.grow_up
  end + kitten.give_birth
end

p Kitten.kitten_count
p Cat.cat_count
```

-   Catクラスは、年齢(@age)とクラス変数である猫の総数(@@cat_count)と子猫の総数(@@kitten_count)を持ちます。
-   Kittenクラスは、Catクラスを継承しています。また、年齢(@age)の初期値を0とし、クラス変数である子猫の総数(@@kitten_count)を持ちます。
-   Catクラスのインスタンスメソッドであるgrow_upメソッドは、猫の年齢を1つ増やします。もし年齢が2になったら、子猫の総数(@@kitten_count)から1引き、猫の総数(@@cat_count)に1を加え、新しい猫のインスタンスを作成します。年齢が2になっていない場合は、自身を返します。
-   Catクラスのインスタンスメソッドであるgive_birthメソッドは、猫の総数(@@cat_count)が0より大きい場合、@@cat_countの数だけKittenクラスのインスタンスを作成して、配列に格納します。その配列を返します。
-   Kittenクラスのインスタンスメソッドであるgrow_upメソッドは、年齢を1つ増やします。もし年齢が2になったら、子猫の総数(@@kitten_count)から1引き、猫の総数(@@cat_count)に1を加え、新しい猫のインスタンスを作成します。年齢が2になっていない場合は、自身を返します。
-   Kittenクラスのクラスメソッドであるkitten_countメソッドは、子猫の総数(@@kitten_count)を返します。
-   years変数には、引数で渡された年数が設定されています。kitten変数には、Kittenクラスの新しいインスタンスが代入されています。
-   cat変数は空の配列で初期化されており、ループでcat変数を更新しています。各年について、kitten変数の年齢を1つ増やし、cat変数の各要素をgrow_upメソッドで更新しています。そして、give_birthメソッドで生成された配列とkitten変数を結合して、cat変数に代入します。

##### 補足
以下のコードでかなり手こずった。
kittenがgive_birthで生まれてくるのにgrow_upさせることができなかったため。
```ruby
years.times do |year|
  kitten = kitten.grow_up
  cat = cat.map do |c|
    c.grow_up
  end + kitten.give_birth
end
```
結果はcatという空配列に、ループを書き、処理の最後でgive_birthで産んだkittensを連結させ、catに代入している。(産んだkittenはgive_birthのkittensという配列で配列で返って来ている)。これによりgrow_upすることができるようになった。
