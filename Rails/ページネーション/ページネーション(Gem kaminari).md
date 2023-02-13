
kaminariとはページネーションを簡単に実装するためのGemです。
このGemをインストールすることによって、簡単にページネーションを実装することができます。

ちなみに、ページネーションとは、長くなってしまったWebページを分割するものです。


## コード手順 

```gemfile
gem 'kaminari', '~> 0.17.0'
```

```shell
$ bundle install
```

```shell
$ bin/rails g kaminari:config
```

### kaminari.config

1ページあたりに表示するレコード数を20にしたい
```ruby
# kaminari_config.rb
Kaminari.configure do |config| 
	config.default_per_page = 20 
end
```

### boards_controller

 indexアクション内の掲示板一覧の取得処理で、pageスコープを使用します。
```ruby
def index @boards = Board.all.includes(:user).order(created_at: :desc).page(params[:page]) end
```

### index.html.erbにページネーションを表示させる

paginateヘルパーを使用
```ruby
<%= paginate @boards %>
```


##### 注意
kaminariではページ数が1の場合はページネーションが表示されません。
`config.default_per_page = 20` と設定されているので、21個以上作成するか、 `default_per_page` を少ない値に変更してみて、もう１度確認。  


## ページネーションにbootstrapを適用

### bootstrap4-kaminari-views

```Gemfile
gem 'bootstrap4-kaminari-views'
```

```shell
$ bundle install
```

###  paginateヘルパーに以下の追加記述

```ruby
<%= paginate @boards, theme: 'twitter-bootstrap-4' %>
```


----
>参照
https://zenn.dev/yukihaga/articles/3d49208638e397