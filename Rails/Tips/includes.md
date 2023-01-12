
### 引数に指定された関連モデルを1度のアクセスでまとめて取得するメソッド

```ruby
モデル名.includes(:アソシエーションで紐付くモデル名)
```

 includesメソッドを使わない場合、
 board(投稿)の数だけuserテーブルにアクセスすることになり、
 データ取得に時間がかかる。
 その結果、アプリケーションのパフォーマンスが下がる。
 これを「[[N+1問題]]」という。
 
***「[[N+1問題]]」を解決するためにincludesメソッドを使用する。***


### 掲示板一覧での使用例

```ruby
# boards_controller.rb
class BoardsController < ApplicationController
  def index
    @boards = Board.all.includes(:user).order(created_at: :desc)
  end
end
```

```ruby
# index.html.erb
<%= render @boards %> 

```

***userテーブルの情報をboardテーブルの情報と共に、
先にまとめて取得させている。***
また、パーシャルで呼び出す、部分テンプレートも省略形で書き換える。
