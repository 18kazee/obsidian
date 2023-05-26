
[RSpec](http://d.hatena.ne.jp/keyword/RSpec)のテスト方法にはCapybaraを使ったテスト方法が主流かなと思っています。
テスト時に実際にブラウザを起動させて行うテストで、エラーがあったときにはエラー時の[スクリーンショット](http://d.hatena.ne.jp/keyword/%A5%B9%A5%AF%A5%EA%A1%BC%A5%F3%A5%B7%A5%E7%A5%C3%A5%C8)を取ってくれる便利なやつです。
設定ファイルは以下のようになります。

spec/support/capybara.rb
```ruby
RSpec.configure do |config|
  config.before(:each, type: :system) do
    driven_by :selenium, using: :chrome, screen_size: [1920, 800]
  end
end
```

`spec/support`の[ディレクト](http://d.hatena.ne.jp/keyword/%A5%C7%A5%A3%A5%EC%A5%AF%A5%C8)リを生成し、`spec_helper.rb`とは違うファイルでテストの設定を行えるようにしています。
一見完璧に見えるCapybaraなのですが、意外と欠点があるんですよね。
それが「ページの[ステータスコード](http://d.hatena.ne.jp/keyword/%A5%B9%A5%C6%A1%BC%A5%BF%A5%B9%A5%B3%A1%BC%A5%C9)を取得できない」です。
例えば、権限がないページにユーザーがアクセスしたときに、403エラー(`:forbidden`)が発生します。そして、エラーが発生した時の[ステータスコード](http://d.hatena.ne.jp/keyword/%A5%B9%A5%C6%A1%BC%A5%BF%A5%B9%A5%B3%A1%BC%A5%C9)は403となるはずです。ですので、[ステータスコード](http://d.hatena.ne.jp/keyword/%A5%B9%A5%C6%A1%BC%A5%BF%A5%B9%A5%B3%A1%BC%A5%C9)を検証したいのですが、それがCapybaraだとできないんです。

### driven_by(:rack_test)

[ステータスコード](http://d.hatena.ne.jp/keyword/%A5%B9%A5%C6%A1%BC%A5%BF%A5%B9%A5%B3%A1%BC%A5%C9)を取得するテストはどうすれば良いのかというと、実は簡単でCapybaraではなく、テストの設定を`driven_by(:rack_test)`に変えるだけです。
```ruby
RSpec.describe "ファイル名", type: :system do
  before do
    driven_by(:rack_test) # ここを追加！！！！
  end
```


この設定は実はシステムスペックファイルを作成するコマンド実行時にデフォルトで生成されたシステムスペックファイルに記述されています。
```ruby
$ bin/rails g rspec:system sample
      create  spec/system/samples_spec.rb
```

作成されたシステムスペックファイル
```ruby
# spec/system/samples_spec.rb
require 'rails_helper'

RSpec.describe "Samples", type: :system do
  before do
    driven_by(:rack_test) # ここを追加！！！！
  end

  pending "add some scenarios (or delete) #{__FILE__}"
end
```

これで[ステータスコード](http://d.hatena.ne.jp/keyword/%A5%B9%A5%C6%A1%BC%A5%BF%A5%B9%A5%B3%A1%BC%A5%C9)を取得できる。

