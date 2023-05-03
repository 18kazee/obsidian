
rakeタスクの自動アップデートをテストするために、
spec/rake_helper.rbとlib/tasks/update_article_state_spec.rbをそれぞれ作成する

spec/rake_helper.rb
```ruby
require 'rake' 

RSpec.configure do |config| 
	config.before(:suite) do 
		Rails.application.load_tasks # Load all the tasks just as Rails does (`load 'Rakefile'` is another simple way)
	end 
	
	config.before(:each) do 
		Rake.application.tasks.each(&:reenable) # Remove persistency between examples
	end
end
```

これはRSpecのテストで、Rakeタスクを使用する場合に必要な設定です。
Rakeタスクとは、コマンドラインから実行できるRubyのタスクで、ファイルの生成やデータベースの初期化などのタスクを自動化するために使用されます。

このコードでは、RSpecの設定を変更して、各テストの前にRakeタスクをリセットします。具体的には、最初の`before(:suite)`ブロックで、`Rails.application.load_tasks`を実行して、Rakeタスクをロードしています。2番目の`before(:each)`ブロックでは、各テストの前にRakeタスクをリセットするために、`Rake.application.tasks.each(&:reenable)`を実行しています。

このように設定することで、各テストが互いに影響を与えないようにし、テストの信頼性を確保できます。また、Rakeタスクをテストの前にリセットすることで、以前に実行したテストの影響を受けずにテストを再実行できるようになります。

`Rails.application.load_tasks`は、Railsアプリケーションに関連するRakeタスクをすべてロードするためのメソッドです。Rakeタスクとは、コマンドラインから実行できるRubyのタスクで、ファイルの生成やデータベースの初期化などのタスクを自動化するために使用されます。

`Rake.application.tasks.each(&:reenable)`は、現在ロードされているすべてのRakeタスクをリセットするためのコードです。Rakeタスクは通常、タスクの結果をキャッシュすることで、再度実行されたときにキャッシュを使うことができます。しかし、テストでは、各テストが互いに独立して実行される必要があります。そのため、各テストの前にすべてのRakeタスクをリセットする必要があります。

`Rake.application.tasks.each(&:reenable)`は、Rakeアプリケーションにロードされているすべてのタスクをリセットするために使われます。`each`メソッドは、配列の要素を1つずつ処理するためのメソッドです。`&:reenable`は、Rubyの短縮記法の1つで、ブロックの中で引数に対して呼び出すメソッドを指定する方法です。つまり、`each(&:reenable)`は、`tasks`配列の各要素に対して、`reenable`メソッドを呼び出すことを意味します。これにより、すべてのRakeタスクがリセットされます。


lib/tasks/update_article_state_spec.rb
```ruby
require 'rake_helper'

describe 'article_state:update_article_state' do
  subject(:task) { Rake.application['article_state:update_article_state'] }
  before do
    create(:article, state: :publish_wait, publish_at: Time.current - 1.day)
    create(:article, state: :publish_wait, publish_at: Time.current + 1.day)
    create(:article, state: :draft)
  end

  it 'update_article_state' do
    expect { task.invoke }.to change { Article.published.size }.from(0).to(1)
  end
end
```

コードの意味
```ruby
require 'rake_helper'
```
`rake_helper`というライブラリを読み込んでいます。このライブラリは、Rakeタスクをテストするために必要なメソッドなどを提供します。

```ruby
describe 'article_state:update_article_state' do
```
`describe`メソッドを使用して、`article_state:update_article_state`という文字列を渡しています。これは、テスト対象のタスクの名前を表しています。このテストは、このタスクが期待通りに動作するかどうかを検証するものです。

```ruby
subject(:task) { Rake.application['article_state:update_article_state'] }
```
`subject`メソッドを使用して、テスト対象のオブジェクトを定義しています。ここでは、`Rake.application['article_state:update_article_state']`というRakeタスクを取得して、`task`という名前で定義しています。

```ruby
before do
	create(:article, state: :publish_wait, publish_at: Time.current - 1.day)
	create(:article, state: :publish_wait, publish_at: Time.current + 1.day)
	create(:article, state: :draft)
end
```
`before`メソッドを使用して、各テストが実行される前に実行するコードを定義しています。ここでは、3つの記事を生成しています。1つは過去の時刻に公開予定、1つは未来の時刻に公開予定、1つは下書き状態となっています。

```ruby
it 'update_article_state' do
	expect { task.invoke }.to change { Article.published.size }.from(0).to(1)
end
```
`it`メソッドを使用して、テストケースを定義しています。`update_article_state`という文字列は、このテストが何を検証しているかを表しています。`expect`ブロック内で、タスクを実行する前後で`Article.published.size`が変化することを期待しています。`from(0)`は、期待値の開始点を表しており、この場合は0を表します。`to(1)`は、期待値の終了点を表しており、この場合は1を表します。つまり、このテストは、公開状態の記事の数が0から1に変化することを期待しているということです。

`task.invoke`は、Rakeタスクを実行するためのメソッドです。このメソッドを呼び出すことで、Rakeタスクの実行をシミュレートできます。
つまり、`article_state:update_article_state`という名前のRakeタスクを実行し、その結果をテストしています。具体的には、公開待ち記事が1つだけ公開されることを期待しています。

### 参考
https://qiita.com/aeroastro/items/c97bd26ce8b8818b6bed