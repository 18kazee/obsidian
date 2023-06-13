
## wheneverとは

`○時になったら○○のコマンドを実行`してくれる`cron`というものがある。
この`cron`の設定を、`ruby`の簡単な文法で扱えるようにしたライブラリが`whenever`。

## Mailerとは

`Rails`には、デフォルトで`ActionMailer`と呼ばれるメール送信機能が備わっている。  
通常のテキストメール送信に加え、HTMLメールが送信できたり、添付ファイルを付けるなど様々なオプションがある。

## 実装内容

毎日am9:00に下記の内容を管理者にメールで送信。

・公開済の記事の総記事数  
・昨日公開された記事の件数とタイトル  
・メールの件名には「公開済記事の集計結果」と設定  
・管理者のメールアドレスには`admin@example.com`を設定  
・`users`テーブルに`email`カラムは追加しなくてOK

## 実装の流れ

1.モデルにスコープを追加  
2.メールを作成  
3.毎日am9:00にメールを送信できるよう設定

## 2.メールを作成

`Mailer`は導入済みで進める。  

```
bin/rails g mailer ArticleMailer
```

上記のコマンドで  

・`ActiveMailer::Base`を継承した`ApplicationMailer`と`ArticleMailer`(自分で命名したもの)が生成される。
・`Mailer`を作成すると`view`のディレクトリとテストも同時に作成される。

app/mailers/application_mailer.rb
```ruby
class ApplicationMailer < ActionMailer::Base
  default from: 'from@example.com'
  layout 'mailer'
end
```

app/mailers/article_mailer.rb
```ruby
class ArticleMailer < ApplicationMailer
    def report_summary
        @published_article_count = Article.published.count
        @articles_published_at_yesterday = Article.published_at_yesterday
        　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　　#↑先程モデルで定義したスコープ
        mail(to: "admin@example.com",subject: "公開済記事の集計結果")
    end
end
```

次に、メール本文を記述する。

views/article_mailer/report_summary.text.erb
```ruby
公開済の記事数: <%= @published_article_count %>件

<% if @articles_published_at_yesterday.present? %>
    昨日公開された記事数: <%= @articles_published_at_yesterday.count %>件
    <% @articles_published_at_yesterday.each do |article| %>
        タイトル: <%= article.title %>
    <% end %>
<% else %>
    昨日公開された記事はありません
<% end %>
```

`config`にてメールが直接確認できるよう設定。

config/enviroments/development.rb
```ruby
config.action_mailer.delivery_method = :letter_opener_web
#↑配信方法を指定している

config.action_mailer.default_url_options = { host: 'localhost:3000' }
#↑アプリケーションのホスト情報をメイラー内で使うためのオプション
```

## 2.毎日am9:00にメールを送信できるよう設定

冒頭で`whenever`とは○時になったら○○のコマンドを実行してくれる〜と記載したが、まずは実行する **中身** を`rakeタスク`にて作成していく。
今回はタスク名を`article_summary`として命名。

```
$ bin/rails g task article_summary
```

##### article_summary.rake
`lib/tasks/article_summary.rake`が作成されるので、この中に定義する。

lib/tasks/article_summary.rake
```ruby
namespace :article_summary do
  desc "am9時に公開済記事の集計結果を管理者にメールで送る"
    task mail_article_summary: :environment do
      ArticleMailer.report_summary.deliver_now
    end
end
```

**`ArticleMailer.report_summary`**  
先程メール作成で定義したもの

・**`deliver_now`**  
非同期ではなく、すぐにメールを送信する。つまりメールを送信し終えるまで、次の行は実行されない。  
用途としては`railsコンソール`や`rakeタスク`など、すぐに処理を実行して終了まで待ちたいときに使う。

似たようなもので`deliver_later`がある。こちらは`Active Job`を使用して非同期でメールを送信する。つまりメールの送信処理を待たなくても、次の行を実行できる。  
これによりユーザーはメール送信という時間のかかる処理を待たずに、次の画面を見ることができる。


##### schedule.rb
実行する中身を定義できたので、後は「毎日am9:00にメールを送信」を設定すればOK

config/schedule.rb
```ruby
require File.expand_path(File.dirname(__FILE__) + '/environment')
# Rails.rootを使用するために必要

rails_env = ENV['RAILS_ENV'] || :development
# cronを実行する環境変数

set :environment, rails_env
# cronを実行する環境変数をセット

set :output, "#{Rails.root}/log/cron.log"
# cronのログの吐き出し場所

every 1.day, at: '9am' do
    rake 'article_summary:mail_article_summary'
end
```

https://qiita.com/mmaumtjgj/items/8eacc597a34091f53262