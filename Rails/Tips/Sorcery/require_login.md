### ログインをしていないユーザーをアクション単位で弾く。 

アクセスしようとしたURLをセッションに格納し、
`not_authenticated`を実行するメソッド。  
以下のように`before_action`で指定する。

``` ruby
# hoges_controller
before_action :require_login
```

