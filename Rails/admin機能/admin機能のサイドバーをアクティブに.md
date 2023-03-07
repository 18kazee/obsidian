
掲示板関連のページを使用しているときは、サイドバーの「掲示板一覧」をアクティブの状態にしたい。  
ユーザーのときも同様。

### 実装

Bootstrapを使用しているので、例えばこのように`class`に`active`を含めると、アクティブにしてくれます。

```ruby
<a class="nav-link active" href="#">Active</a>
```

これを使って、アクティブにしたいときには`class`に`active`を含めるようにして実装していきます。

```ruby
def active_if(path)
    path == controller_path ? 'active' : ''
end
```

[三項演算子](http://d.hatena.ne.jp/keyword/%BB%B0%B9%E0%B1%E9%BB%BB%BB%D2)を使い、真のときは`active`、偽のときは何も返しません。  
`controller_path`でコントローラー名を取得できるって知りませんでした。

```ruby
<%= link_to admin_boards_path, class: "nav-link #{active_if('admin/boards')}" do %>

<%= link_to admin_users_path, class: "nav-link #{active_if('admin/users')}" do %>
```

さっきのヘルパーメソッドを使って、現在のコントローラーが`admin/boards`pathと一致しているときは、`active`を返す。  
すると`class: "nav-link active"`となるので、アクティブになる。