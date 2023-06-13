
### 実行するテストケースを限定したいとき

`it`の代わりに`fit`を使います。  
`spec_helper.rb`で `config.filter_run_when_matching :focus`と設定しているため、各テストの頭文字に`f`をつけることで、`focus: true`を設定できます。

### 別タブで開いたページをテストしたいとき

リンクをクリックし、遷移したページ先の項目についてテストしたいときは、`within_window(windows.last)`で最後に開いたタブを指定します。

```ruby
it 'Project詳細からTask一覧ページにアクセスした場合、Taskが表示されること' do
        visit project_path(project)
        click_link 'View Todos' #これでページ遷移する。
        within_window(windows.last)do #以下、遷移先の内容
          expect(page).to have_content task.title
          expect(Task.count).to eq 1
          expect(current_path).to eq project_tasks_path(project)
        end
    end
end
```

##### withinメソッド
Capybaraの`within`メソッドは、ページ内の特定のエリアやアクションを指定できます。  
`within_window`はウィンドウ画面の指定。

### 確認画面のページ操作したいとき

何かの項目を削除するとき、「本当に削除しますか？」とアラートメッセージがでますが、それの操作は以下のようにします。

```ruby
 #許可するとき
page.driver.browser.switch_to.alert.accept 

 #拒否するとき
page.driver.browser.switch_to.alert.dismiss
```

https://blog.regonn.tokyo/rails/2017-06-13-rspec-capybara/


### ApplicationHelperで定義したメソッドを[RSpec](http://d.hatena.ne.jp/keyword/RSpec)で使いたいとき

spec/rails_helper
```
config.include ApplicationHelper #追加
```
これで、ApplicationHelperで定義したメソッド`short_time`が[Rspec](http://d.hatena.ne.jp/keyword/Rspec)でも使用できます。
(今回の場合、viewでshort_timeという時刻表示を使用していてる。short_timeをviewに合わせてRSpecでも使用しようとするとApplicationHelperを読み込んでいないためエラーとなるのでspec/rails_helper.rbに記載する。そうすると使用できる。)

```ruby
it 'Taskを編集した場合、一覧画面で編集後の内容が表示されること' do
        fill_in 'Deadline', with: Time.current #今の時刻にdeadlineを更新する。
        click_button 'Update Task'
        click_link 'Back'
        expect(find('.task_list')).to have_content(short_time(task.Time.current)) #ここ
        expect(current_path).to eq project_tasks_path(project)
end
```

https://kunilog.hatenablog.com/entry/2020/08/11/145929

### findメソッド

Capybaraの`find`メソッドは特定の要素を指定できます。  
さきのコード例では、`find('.task_list')`で、viewで定義している`class = "task_list"`を指定しています。

### strftimeメソッド

[Ruby](http://d.hatena.ne.jp/keyword/Ruby)のメソッドです。
時刻を format 文字列に従って文字列に変換した結果を返します。  
[Time#strftime (Ruby 2.7.0 リファレンスマニュアル)](https://docs.ruby-lang.org/ja/latest/method/Time/i/strftime.html)

以下のように使います。
```ruby
expect(page).to have_content(Time.current.strftime('%Y-%m-%d'))
```

### FactoryBotのtraitを使う

属性をいろいろ指定して定義したいとき、`trait`を使います。  
テスト内で何度も同じ属性を指定して呼び出していると、コードの重複が生じてしまいます。  
traitを使えばスッキリとした見た目になります。  
また、添付ファイルを予めセットすることもできます。(やり方は『everyday [Rails](http://d.hatena.ne.jp/keyword/Rails)』P174 )

```ruby
#trait定義前
let!(:task) {create(:task, project_id: project.id, status: :done, completion_date: Time.current.yesterday)}

#trait定義後
let!(:task) { create(:task, :done) }
```

```ruby
FactoryBot.define do
  factory :task do
    sequence(:title, "title_1")
    status { rand(2) }
    from = Date.parse("2019/08/01")
    to   = Date.parse("2019/12/31")
    deadline { Random.rand(from..to) }
    association :project

    trait :done do　#ここ
      status { :done }
      completion_date { Time.current.yesterday }
    end
  end
end
```

### FactoryBotにおけるassociation

関連するデータを一緒に作成してくれるメソッド。  
例として`task`を作成する際に、`project`も作成必須の場合で`association`無しのコードを見てみると、

```ruby
#associationを使わない方法
let(:project) { create(:project) }
let(:task) { create(:task,project: project) }
```

このように`task`と`project`の両方のテストコードを記述しなければならない。  
本来なら、さらに関連するテーブルが増えていくので、後々しんどくなる。  
これを解決するのが`association`メソッド。

`association`の記述の仕方は簡単で、　`Factory`にアソシエーションするものを記述するだけである。

factories/tasks.rb
```ruby
FactoryBot.define do
  factory :task do
    title { 'Task' }
    content { 'content' }
    association :project
  end
end
```

次に`spec`ファイルにて記入する。`association`の有無は以下の通り。
```ruby
#associationを使わない方法
  let(:project) { create(:project) }
  let(:task) { create(:task,project: project) }

#associationを使った方法
  let(:project) { create(:project) }
  let(:task) { create(:task) }
```

アソシエーションしているので、**`let(:task) { create(:task) }`**と記載することで、`task`データ作成と同時に`project`データも作成され、`project`の`project_id`も自動で`id`が入るようになる。

##### associationをした事による引数に注意

例えば下記のようなテストコードがあったとする。この記述は`association`を使用していない記述である。

```ruby
RSpec.describe 'Task', type: :system do
  let(:project) { create(:project) }
  let(:task) { create(:task,project: project) }
  describe 'Task詳細' do
    context '正常系' do
      it 'Taskが表示されること' do
        visit project_task_path(project, task)
        expect(page).to have_content(task.title)
        expect(page).to have_content(task.status)
        expect(page).to have_content(task.deadline.strftime('%Y-%m-%d %H:%M'))
        expect(current_path).to eq project_task_path(task.project, task)
      end
    end
  end
end
```

このコードを`association`を使って記述するとき、`let`の部分は`let(:task) { create(:task)}`と書けば済むが、

`visit project_task_path(project, task)`の引数をこのままでは、  
第１引数にtaskと結びついていないproject_idを呼び出す事になってしまう。

第２引数の`task`に紐づくよう、第1引数は`task.project`にしなければならない。

```ruby
# 上記のコードをassociationで記述した場合
RSpec.describe 'Task', type: :system do
  let(:project) { create(:project) }
  let(:task) { create(:task) } #ここと
  describe 'Task詳細' do
    context '正常系' do
      it 'Taskが表示されること' do
        visit project_task_path(task.project, task) #ここ
        expect(page).to have_content(task.title)
        expect(page).to have_content(task.status)
        expect(page).to have_content(task.deadline.strftime('%Y-%m-%d %H:%M'))
        expect(current_path).to eq project_task_path(task.project, task)
      end
    end
  end
end
```

https://nqiita.com/mmaumtjgj/items/a0eab1618c1f742517b9
https://qiita.com/Ryoga_aoym/items/741c57e266a9d811a2d4