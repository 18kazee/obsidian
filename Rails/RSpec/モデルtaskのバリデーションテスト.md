### テストの結果を見やすく
.rspecに「–format documentation」を追加することで、テスト結果を読みやすいドキュメント形式に変更できます。  
※.rspecが見当たらない場合は、「open -e .rspec」で開ける

.rspec
```ruby
--require spec_helper
--format documentation
```

上記設定をしなかった場合は、以下のように記述しなければいけない

```ruby
user = FactoryBot.create(:user)

上記設定をしたら下記のように省略して記述できる
user = create(:user)
```

### Factory_botにテストデータを書く

テストするための記載をする。
バリデーションuserカラムとtaskカラムにあるため両方書く

```ruby
FactoryBot.define do
  factory :task do
    sequence(:title, "title_1")   #  ユニーク制約に抵触しないようにシーケンスを使う
    content {"content"}   #  カラム名{"カラムに投入するデータ"}の形式になっている
    status { :todo }
    deadline { 1.week.from_now }
    association :user
  end
end
```

```ruby
FactoryBot.define do
  factory :user do
    sequence(:email) {|n| "test#{n}@example.com"}
    password { "password" }
    password_confirmation {"password" }
  end
end
```


#### 補足
##### task
・factory :task doの部分は、生成する[インスタンス](http://d.hatena.ne.jp/keyword/%A5%A4%A5%F3%A5%B9%A5%BF%A5%F3%A5%B9)の名前をモデル名と一緒の名前にするが、モデル名と違う名前の[インスタンス](http://d.hatena.ne.jp/keyword/%A5%A4%A5%F3%A5%B9%A5%BF%A5%F3%A5%B9)名にしたい時は、[factory :other_task, class: Task do]のようにする

・sequenceを使うとtaskインスタンスを生成する度に、ユニーク制約に抵触しないように違う値が作られる  
sequence(:title, "title_1")の場合(sequence(:カラム名, "カラムに投入するデータ")は、sequenceにブロックを渡さずに第二引数を渡している。 

sequenceにブロックを渡さずに第二引数を渡している場合は、.nextメソッドが呼ばれ末尾の数値を増やしてくれる。(形が、英字＋記号＋数字の場合)  
sequenceに第二引数を渡して値が変わるのは、末尾の数字だけなので末尾の数字を変えたい場合は、ブロックが省略できるが、末尾以外の部分の数字を増やしたい場合は、ブロックが必要

##### user
・sequence(:email) { |n| "test#{n}@example.com" }は、下記と一緒  

```ruby
sequence(:email) do |n|
  "test#{n}@example.com"
end
```

上記は、メールアドレスの[n]の部分がuserが作成される度に数値が増えていくので、ユニーク制約に抵触しない

### バリデーションのテストを書く

spec/models/task_spec.rb
```ruby
require 'rails_helper'

RSpec.describe Task, type: :model do
  describe 'validation' do
    it 'is valid with all attributes' do   #   全ての属性が存在する場合
      task = build(:task)      #   taskインスタンスを作成(DBには、保存していない)
      expect(task).to be_valid   #   be_validは、valid?をマッチャにしている
      expect(task.errors).to be_empty
    end

    it 'is invalid without title' do   #   タイトルが無い場合
      task_without_title = build(:task, title: "")   #   インスタンスを作成する時に、カラムを渡すことでFactoryBotの内容をオーバーライドできる
      expect(task_without_title).to be_invalid
      expect(task_without_title.errors[:title]).to eq ["can't be blank"]
    end

    it 'is invalid without status' do   #  ステータスが無い場合
      task_without_status = build(:task, status: nil)
      expect(task_without_status).to be_invalid
      expect(task_without_status.errors[:status]).to eq ["can't be blank"]
    end

    it 'is invalid with a duplicate title' do   #  タイトルが重複している場合
      task = create(:task)
      task_with_duplicated_title = build(:task, title: task.title)   #   上記のtaskと同じタイトルのインスタンスを生成
      expect(task_with_duplicated_title).to be_invalid
      expect(task_with_duplicated_title.errors[:title]).to eq ["has already been taken"]
    end

    it 'is valid with another title' do   #   タイトルが異なる場合
      task = create(:task)
      task_with_another_title = build(:task, title: 'another_title')
      expect(task_with_another_title).to be_valid
      expect(task_with_another_title.errors).to be_empty
    end
  end
end
```

#### 補足
・[build]メソッド: [build]は、[インスタンス]を生成しただけでDBには保存していない  

・[create]メソッド: [create]は、[インスタンス]を生成してDBに保存する  

・[be_○○]マッチャ: [valid?]メソッドや[empty?]メソッドなどのように、最後が[?]になっていて、[true]もしくは[false]を返すメソッドは、[be_○○]のようにして使える(このマッチャがtrueになったらテストが通過する)  

・[task_without_title = build(:task, title: "") ]のように記述することで、FactoryBotのカラム内容をオーバーライドして[インスタンス]を生成できる

・.rspecファイルに[--format documentation]を下記のように記述するとテスト実行時にメッセージが表示される
	上記記述しない場合
	![[Pasted image 20230320113555.png]]
	上記記述した場合
	![[Pasted image 20230320113722.png]]
